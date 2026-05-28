# ADR-0005 — Sincronía entre dispositivos: Supabase Realtime

- **Estado**: Aprobado
- **Fecha**: 28 mayo 2026
- **Decidido por**: Bryan/Dev18

---

## Contexto

La spec exige que un cambio de estado (toggle de check-in, nuevo hábito) sea visible en otros dispositivos en ≤5 segundos (criterio 31). React Query por sí solo no hace push de datos: una vez que los datos están en cache, no se actualizan hasta que hay una invalidación explícita o un refetch programado. Sin una estrategia activa, un usuario con dos tabs abiertas (o dos dispositivos) verá estados desincronizados hasta la siguiente acción manual.

Supabase ofrece Realtime como mecanismo de push nativo: suscripciones WebSocket a cambios en tablas de Postgres. El plan Free de Supabase tiene un límite de 200 conexiones concurrentes simultáneas.

Drivers relevantes:
- Criterio 31: ≤5 segundos de latencia de sincronía entre dispositivos.
- No introducir dependencias de backend adicionales fuera de Supabase.
- ADR-0003 establece React Query como capa de estado; la solución de sync debe integrarse con `queryClient.invalidateQueries()`.

---

## Decisión

**Usar Supabase Realtime con suscripciones WebSocket** a los canales de `habits` y `check_ins` del usuario autenticado. Cuando Supabase emite un evento (`INSERT`, `UPDATE`, `DELETE`) en cualquiera de las tablas, el handler llama a `queryClient.invalidateQueries()` para que React Query re-fetche los datos afectados. Se implementa un hook `useRealtimeSync()` que gestiona el ciclo de vida de la suscripción (mount, cleanup, reconexión).

---

## Alternativas consideradas

### Alternativa B — Short-polling con `refetchInterval` de React Query

React Query re-fetcha los datos cada N segundos de forma automática. Sin WebSocket, sin infraestructura adicional.

**Trade-offs:**

| Campo | Detalle |
|---|---|
| **Requiere** | `refetchInterval: 5000` (5s) en los queries del dashboard. Zero infraestructura adicional — una línea por query |
| **Impacto en rendimiento** | Latencia de sync: 0–5s (depende del ciclo). Cumple el criterio justo en el peor caso. Genera tráfico constante a Supabase aunque no haya cambios: con 50 usuarios activos × 3 queries × cada 5s = ~30 req/s a Supabase (dentro del límite del Free tier, cualitativa) |
| **Complejidad de desarrollo** | Mínima — ~30 minutos. Sin manejo de conexiones ni cleanup |
| **Observabilidad** | Cada poll aparece en los logs de Supabase. Fácil de monitorear, pero genera ruido constante |

**Por qué no se eligió**: El polling genera tráfico innecesario cuando no hay cambios — en uso personal con baja frecuencia de cambios, la mayoría de los polls serán vacíos. Supabase Realtime es el mecanismo de push nativo del stack y está incluido sin costo adicional. La diferencia de latencia (push inmediato vs poll cada 5s) es relevante en la experiencia de uso multi-dispositivo.

### Alternativa C — Polling largo (30 segundos) + invalidación manual post-toggle

`refetchInterval: 30000` para sincronía pasiva. Los cambios del mismo dispositivo se reflejan inmediatamente por la invalidación de React Query post-mutación.

**Por qué no se eligió**: No cumple el criterio 31 de la spec (≤5 segundos). La latencia de hasta 30s en otros dispositivos es inaceptable como comportamiento declarado. Solo sería viable si el criterio 31 se relajara explícitamente, lo cual no fue aprobado.

---

## Consecuencias

### Positivas

- Latencia de sync entre dispositivos de ~500ms–2s — muy por debajo del límite de 5s del criterio 31.
- Sin tráfico de red innecesario: Supabase solo envía eventos cuando hay cambios reales en la base de datos.
- La integración con React Query es limpia: el evento de Realtime llama a `invalidateQueries()` y React Query se encarga del re-fetch y actualización de la UI.
- RLS aplica también a Realtime: un usuario solo recibe eventos de sus propios registros si las políticas están configuradas correctamente.

### Negativas (trade-offs aceptados)

- **Complejidad de gestión del ciclo de vida**: las suscripciones de Supabase Realtime deben limpiarse explícitamente al desmontar el componente o al hacer logout. Una limpieza incorrecta genera suscripciones duplicadas (especialmente en React StrictMode que monta/desmonta componentes dos veces en desarrollo) y memory leaks. Requiere un hook `useRealtimeSync()` bien implementado con cleanup en el `useEffect` return. Estimado: 4–6h para implementar y validar correctamente.
- **Conexión WebSocket persistente**: cada tab abierta mantiene una conexión WebSocket activa (~50KB de overhead de memoria por tab). Irrelevante para uso personal, pero es una diferencia respecto al modelo stateless de polling.
- **Límite de 200 conexiones en Supabase Free**: si el proyecto escala más allá de ~150–180 usuarios concurrentes, se requiere upgrade a Supabase Pro ($25/mes). Para el scope actual (uso personal / pequeño equipo), es un riesgo teórico sin impacto práctico inmediato.
- **Debugging de reconexiones**: cuando la conexión WebSocket se pierde (cambio de red, suspensión del dispositivo), la reconexión automática de Supabase Realtime puede tardar varios segundos. Durante ese intervalo, cambios en otros dispositivos no llegan. Se recomienda implementar un indicador de estado de conexión visible en la UI para que el usuario sepa cuándo la sincronía está activa.