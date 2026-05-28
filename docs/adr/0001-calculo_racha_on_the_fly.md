# ADR-0001 — Cálculo de racha on-the-fly desde historial de check-ins

- **Estado**: Aprobado
- **Fecha**: 28 mayo 2026
- **Decidido por**: Bryan/Dev18

---

## Contexto

La spec requiere mostrar la racha actual de cada hábito (días o períodos consecutivos con check-in) en el dashboard y en la vista de progreso por categoría. El cálculo involucra detectar gaps en el historial de fechas, respetando la frecuencia del hábito (`daily` con `target_per_week = 7`, o `weekly` con `target_per_week` configurable).

La racha es el dato más consultado: se muestra en el listado principal (un valor por hábito activo) y como promedio agregado en la vista de categoría (últimos 14 días). Con el límite de 200 hábitos por usuario, el costo de leer el historial afecta directamente la latencia percibida del dashboard.

Drivers relevantes:
- Sin dependencias de backend adicionales fuera de Supabase.
- TypeScript estricto; la lógica debe ser testeable de forma aislada.
- Cambios en un check-in deben reflejarse en la racha en ≤5 segundos.

---

## Decisión

**Calcular la racha on-the-fly en cada lectura**, a partir del historial de `check_ins` consultado desde Supabase. No se persiste el valor de racha en ninguna columna de `habits` ni en una tabla separada. El cálculo ocurre en el cliente (TypeScript) después de que React Query obtiene los check-ins del período relevante.

---

## Alternativas consideradas

### Alternativa B — Columna materializada `current_streak` en `habits`

Añadir `current_streak INT` a la tabla `habits`, actualizado mediante un trigger Postgres o una función RPC llamada en cada mutación de check-in.

**Trade-offs:**

| Campo | Detalle |
|---|---|
| **Requiere** | Trigger Postgres o función RPC con lógica de racha; migración de esquema; recálculo en operaciones de desmarcado dentro de la ventana de 12h |
| **Impacto en rendimiento** | Dashboard: 1 query plano a `habits` con `current_streak` incluido — latencia mínima. Toggle: +1 operación síncrona por cada check-in (~5–10ms adicionales) |
| **Complejidad de desarrollo** | Alta. El trigger debe manejar correctamente: ventana de gracia de 12h, desmarcado, frecuencia semanal con `target_per_week` variable. Un bug en el trigger corrompe silenciosamente la racha de todos los usuarios sin señal visual de error |
| **Seguridad** | Funciones con `SECURITY DEFINER` requieren revisión explícita de permisos |
| **Observabilidad** | El valor actual es consultable directamente. Pero inconsistencias entre `current_streak` y el historial real son difíciles de detectar sin un script de reconciliación |

**Por qué no se eligió**: La lógica de racha semanal con `target_per_week` variable hace que el trigger sea complejo y frágil. Un bug introduce inconsistencias persistentes en la base de datos difíciles de revertir. El beneficio de rendimiento no justifica ese riesgo para el volumen de datos esperado (<100 usuarios activos, ≤200 hábitos por usuario).

### Alternativa C — Tabla separada `streak_snapshots`

Una tabla `streak_snapshots(habit_id, period_start, streak_value)` actualizada por un cron o por evento en cada toggle.

**Por qué no se eligió**: Requiere Supabase Pro ($25/mes) para cron jobs. Introduce una segunda fuente de verdad que puede desincronizarse. Overkill para el scope declarado.

---

## Consecuencias

### Positivas

- La lógica de racha vive en TypeScript puro, testeable con fixtures sin necesidad de una base de datos real.
- No hay estado persistido que pueda corromperse; el historial de `check_ins` es la única fuente de verdad.
- Cambios en la definición de racha (ej. futura regla de hábitos semanales) solo requieren actualizar la función de cálculo, sin migraciones de datos.
- Sin triggers ni procedimientos almacenados: la lógica de negocio no está repartida entre la base de datos y el frontend.

### Negativas (trade-offs aceptados)

- **Latencia en dashboard con muchos hábitos**: con 200 hábitos activos, el cliente necesita leer hasta 200 × 30 días de check-ins para calcular todas las rachas. Mitigación: una función RPC en Supabase que devuelva las rachas calculadas en una sola query agregada, y un índice en `(habit_id, timestamp DESC)`.
- **Racha no consultable directamente desde SQL**: no es posible hacer `SELECT name, current_streak FROM habits` sin ejecutar la lógica de cálculo. Dificulta consultas administrativas o de soporte.
- **Carga en el cliente**: en conexiones lentas o dispositivos lentos, el cálculo on-the-fly suma tiempo de procesamiento al tiempo de red. Aceptable para el target desktop-first del proyecto.