# ADR-0004 — Acceso a Supabase: cliente directo con `anon key` y RLS

- **Estado**: Aprobado
- **Fecha**: 28 mayo 2026
- **Decidido por**: Bryan/Dev18

---

## Contexto

La spec lista rutas `/api/habits`, `/api/checkins`, `/api/categories` sin especificar si son proxies reales hacia Supabase o si el frontend llama a Supabase directamente. Esta decisión determina qué claves de Supabase se exponen al navegador, dónde vive la lógica de validación de negocio (ventana de gracia de 12h, deduplicación de check-ins) y la superficie de seguridad del sistema.

Decisiones relacionadas que afectan el contexto:
- **ADR-0001**: el cálculo de racha ocurre en el cliente con datos de `check_ins` leídos desde Supabase.
- **ADR-0003**: client-first con React Query; las queries se ejecutan desde el browser.
- **H1 resuelto**: el servidor confía en el `TIMESTAMPTZ` enviado por el cliente; no hay revalidación de timezone en servidor.

Drivers relevantes:
- No introducir dependencias de backend adicionales fuera de Supabase.
- RLS configurada como capa de aislamiento por usuario (requerido por spec).
- La lógica de validación de la ventana de gracia de 12h debe estar implementada en algún lugar.

---

## Decisión

**El frontend llama a Supabase directamente desde el browser usando `@supabase/js` con la `anon key`**. La `anon key` es pública y vive en `NEXT_PUBLIC_SUPABASE_ANON_KEY`. Row-Level Security (RLS) en Supabase es la barrera primaria de aislamiento de datos entre usuarios. La validación de la ventana de gracia de 12h y la deduplicación de check-ins se implementan en el cliente (TypeScript) antes de ejecutar la query.

Esta decisión supersede la Ronda 3 de la spec ('Rutas: Minimal + endpoints REST'). Las rutas /api/* listadas en spec.md no se implementan en el scope inicial; el acceso directo a Supabase desde el cliente las hace innecesarias. Si en el futuro se requieren (ej. webhooks, integración externa), se crean en ese momento

---

## Alternativas consideradas

### Alternativa B — API Routes de Next.js como proxy con `service_role key`

El cliente llama a `/api/checkins`, `/api/habits`, etc. Next.js valida la sesión y ejecuta la query en el servidor usando `SUPABASE_SERVICE_ROLE_KEY` (nunca expuesta al cliente). La lógica de negocio (validación de 12h, deduplicación) vive en el servidor.

**Trade-offs:**

| Campo | Detalle |
|---|---|
| **Requiere** | `SUPABASE_SERVICE_ROLE_KEY` en env servidor; un Route Handler por recurso; validación de sesión en cada handler; `supabase-js` con `service_role` en servidor |
| **Impacto en rendimiento** | Latencia adicional de ~30–50ms por el hop Next.js → Supabase. El optimistic update de React Query oculta esta latencia en la UI, pero afecta el tiempo real de confirmación |
| **Complejidad de desarrollo** | Alta relativa al scope: cada operación requiere su Route Handler con auth check explícito, manejo de errores estandarizado y tipado. Estimado: +8–12h sobre la Alternativa A |
| **Seguridad** | `service_role key` nunca en el cliente. La lógica de validación de 12h es aplicada en servidor y no es bypasseable desde devtools del navegador |

**Por qué no se eligió**: El scope del proyecto es desktop personal con usuarios registrados bajo email/contraseña. La lógica de validación de 12h en el cliente es suficiente para el uso declarado. La complejidad adicional de Route Handlers no aporta valor proporcional al riesgo real. RLS garantiza el aislamiento de datos que es el riesgo de seguridad más crítico.

### Alternativa C — Híbrido: reads directo al cliente, writes vía API Routes

Queries de lectura van directo a Supabase desde el cliente. Las mutaciones pasan por API Routes donde se aplica la lógica de negocio.

**Por qué no se elegió**: Introduce dos clientes de Supabase con diferentes niveles de permisos. La lógica de validación queda repartida entre cliente (reads, precondiciones) y servidor (writes, aplicación de reglas), creando dos lugares donde puede haber inconsistencias. La complejidad de mantener sincronía entre ambos supera el beneficio de seguridad para este scope.

---

## Consecuencias

### Positivas

- Arquitectura más simple: un solo cliente de Supabase (`createBrowserClient`) para todas las operaciones.
- Sin overhead de latencia por capa intermedia — las queries van directamente de React Query a Supabase.
- Desarrollo más rápido: sin Route Handlers que mantener, tipar y testear.
- Patrón estándar y documentado de Supabase para proyectos con RLS.

### Negativas (trade-offs aceptados)

- **La `anon key` es inspeccionable en el navegador**: es una clave pública por diseño en el modelo de Supabase, pero expone el Project URL y la clave a cualquier usuario con devtools. La seguridad real depende enteramente de que RLS esté correctamente configurada en todas las tablas. Un gap de RLS expone datos de otros usuarios sin ninguna capa adicional de defensa.
- **La validación de la ventana de gracia de 12h es bypasseable desde el cliente**: un usuario técnico puede registrar un check-in para fechas fuera de la ventana usando la API de Supabase directamente con su token. Aceptado por el criterio de uso del proyecto (herramienta personal, no aplicación con usuarios adversariales).
- **La deduplicación de check-ins ocurre en lectura/cliente**: si por algún motivo llegan dos check-ins para la misma unidad temporal, el sistema los consolida en lectura pero ambos registros existen en la base de datos. Requiere un `UNIQUE` constraint o lógica de limpieza periódica si se quiere evitar acumulación.