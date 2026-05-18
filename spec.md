# Habit Tracker — Spec

## Objetivo

Crear una aplicación web de escritorio que permita a usuarios registrados gestionar hábitos personales (con frecuencia diaria o semanal) y registrar check-ins diarios, proporcionando vistas de progreso por hábito y por categoría, sin distracciones ni opciones de gamificación, notificaciones o integración con servicios externos.

---

## Scope

### Qué SÍ entra en este proyecto

- **Autenticación**: Registro e inicio de sesión con email y contraseña usando Supabase Auth; logout y redirección a login desde rutas protegidas.
- **CRUD de hábitos**: Crear, leer, actualizar, eliminar hábitos con campos: nombre (máx. 100 caracteres, único por usuario), descripción (máx. 1000 caracteres), frecuencia (diaria o semanal con `target_per_week` configurable), categoría (una por hábito).
- **Check-ins**: Registrar check-ins por hábito con fecha/timestamp persistido en `TIMESTAMPTZ` (UTC), derivando la `date` local según timezone del navegador. Ventana de gracia de 12 horas para registrar el día anterior. Posibilidad de desmarcar un check-in dentro de 12 horas posteriores con confirmación explícita. Un solo check-in por unidad temporal (día/semana según frecuencia).
- **Racha**: Cálculo de racha actual (días o períodos consecutivos según frecuencia) realizado on-the-fly desde el historial de check-ins. Para hábitos sin check-ins, mostrar racha `0`.
- **Categorías**: Sistema de categorías predefinidas del sistema más categorías personalizadas creadas por el usuario. Asignación de exactamente una categoría por hábito.
- **Vistas de progreso**:
  - Por hábito: muestra racha actual expresada en días consecutivos (diario) o períodos de 7 días (semanal).
  - Por categoría: porcentaje de check-ins completados sobre check-ins esperados en la semana calendario en curso (denominador = suma de `target_per_week` de hábitos activos, prorrateado para hábitos creados hace <30 días). Racha promedio calculada sobre últimos 14 días (incluyendo hoy) de hábitos activos que lleven al menos una racha completa.
- **Archivado de hábitos**: Eliminación lógica con columna `deleted_at`/`archived` en `habits`; check-ins se conservan como históricos. No se permite crear nuevos check-ins en hábitos archivados. Hábitos archivados filtrables en UI (por defecto ocultos).
- **Interfaz de escritorio**: Funcional y simple, desarrollada con Next.js 15 (App Router), TypeScript estricto, Tailwind CSS.

### Qué NO entra en este proyecto

- Aplicación móvil nativa; diseño móvil premium o responsive avanzado (desktop-first).
- OAuth / social login.
- PWA (Progressive Web App).
- Exportar / importar CSV ni datos.
- Sincronización con calendarios externos, wearables ni APIs de terceros.
- Monetización, planes premium, suscripciones.
- Gamificación (badges, niveles, retos, puntos).
- Notificaciones push, emails ni recordatorios de ningún tipo.
- Compartir hábitos, progreso, ni funciones sociales.
- Análisis avanzados ni estadísticas más allá de racha y porcentaje semanal.
- Integración profunda con servicios externos; cualquier integración futura será diseñada como extensión opcional fuera del scope inicial.

---

## Criterios de aceptación

### Autenticación

1. Un visitante sin cuenta puede completar el formulario de registro con email y contraseña (mín. 8 caracteres, email válido) y queda autenticado; redirección a dashboard.
2. Si intenta registrarse con email ya existente, mostrar mensaje "El email ya está registrado".
3. Un visitante con cuenta existente puede iniciar sesión con email y contraseña correctos; accede al listado de hábitos tras autenticación.
4. Si inicia sesión con credenciales incorrectas, mostrar mensaje "Email o contraseña inválidos".
5. Un usuario autenticado puede cerrar sesión (logout) y es redirigido a la pantalla de login; la sesión se invalida en el backend.
6. Un visitante sin sesión que intenta acceder a cualquier ruta protegida es redirigido a `/login` sin ver contenido privado.

### CRUD de hábitos

7. Un usuario autenticado puede crear un hábito con nombre, descripción, frecuencia (diaria/semanal con `target_per_week`, ej. 3x/semana) y categoría; el hábito aparece en el listado.
8. Si intenta crear un hábito sin nombre, mostrar error "El nombre es requerido".
9. Un usuario autenticado puede editar un hábito existente (nombre, descripción, frecuencia, categoría); cambios se reflejan inmediatamente y aplican solo hacia adelante; el historial previo no se recalcula.
10. Un usuario autenticado puede eliminar un hábito que nunca alcanzó una racha; el hábito y todos sus check-ins desaparecen del sistema.
11. Un usuario autenticado puede eliminar un hábito que sí alcanzó al menos una racha (diario: ≥7 días, semanal: ≥2 check-ins en 2 semanas); el hábito desaparece del listado pero sus check-ins se conservan como históricos.
12. El listado muestra hábitos agrupados por categoría con nombre, estado (completado hoy / no completado), racha actual y botón de toggle check-in.

### Check-in

13. Un usuario autenticado puede registrar un check-in para el día actual (hoy) marcando un hábito como completado; el sistema guarda `TIMESTAMPTZ` UTC y actualiza la racha.
14. Un usuario autenticado en el día D puede registrar un check-in para el día D-1 si han transcurrido menos de 12 horas desde la medianoche local (según timezone del navegador) que inició el día D; el sistema acepta el check-in datándolo con D-1.
15. Un usuario autenticado en el día D no puede registrar un check-in para el día D-1 si ya transcurrieron más de 12 horas desde la medianoche local de D; rechaza el intento.
16. Si un usuario intenta registrar un segundo check-in para la misma unidad temporal (día para diarios, semana para semanales) del mismo hábito, el sistema deduplica en lectura / normaliza la acción (no genera error, pero consolida visualmente).
17. Un usuario autenticado que marcó un hábito como completado puede desmarcarlo (deshacer check-in) dentro de 12 horas posteriores al registro; el sistema muestra una confirmación explícita ("¿Desmarcar este check-in?") y, si confirma, elimina el check-in y recalcula la racha.
18. Un usuario autenticado no puede desmarcar un check-in después de 12 horas desde su registro; el sistema rechaza la acción.
19. Si falla una acción de toggle (error de red, 401, fallo del servidor), mostrar toast detallado; en 401 mostrar modal "Sesión expirada — inicia sesión nuevamente"; usar rollback optimista en toggles.

### Racha y progreso por hábito

20. Un hábito diario muestra racha actual en días consecutivos con check-in (ej. "7 días").
21. Un hábito semanal muestra racha actual en períodos de 7 días consecutivos con al menos un check-in (ej. "3 semanas").
22. Un hábito recién creado sin check-ins muestra racha `0`.
23. Un hábito diario pierde su racha al día siguiente si no hay check-in registrado.
24. Un hábito semanal pierde su racha si pasan más de 7 días sin un check-in.

### Categorías

25. Un usuario autenticado puede ver categorías predefinidas del sistema al crear/editar un hábito.
26. Un usuario autenticado puede crear una categoría personalizada con nombre; queda disponible para asignar a hábitos.
27. Un usuario autenticado puede asignar exactamente una categoría a cada hábito (constraint de unicidad por hábito).

### Vista de progreso por categoría

28. Un usuario autenticado puede acceder a una vista de progreso por categoría que muestra: porcentaje de check-ins completados sobre check-ins esperados en la semana calendario en curso. El denominador se calcula como suma de `target_per_week` de todos los hábitos activos en esa categoría; para hábitos creados hace <30 días, el denominador se prorratea por días activos.
29. La misma vista muestra racha promedio de todos los hábitos activos en la categoría que lleven al menos una racha completa, calculada sobre los últimos 14 días (incluyendo hoy).
30. Los hábitos archivados no se muestran en el listado principal pero pueden filtrase en una sección "Históricos" para consultar períodos pasados.

### Sincronía y rendimiento

31. Un cambio de estado en un dispositivo debe ser visible en otros dispositivos en ≤5 segundos (short-polling o Realtime).

---

## Pruebas técnicas fuera de QA manual

Las siguientes garantías deben validarse mediante pruebas técnicas (unitarias, integración, e2e) y no son verificables desde la UI manual:

1. **Aislamiento de datos por usuario**: Un usuario autenticado no puede ver, modificar ni eliminar hábitos, check-ins ni categorías de otro usuario.
2. **Invalidación de sesión en logout**: Después de hacer logout, el token/sesión en el backend se invalida y cualquier intento de acceder a rutas protegidas con ese token retorna 401.
3. **Precisión de cálculo de racha**: Racha diaria se calcula correctamente como días consecutivos de check-ins sin gaps. Racha semanal se calcula como períodos de 7 días (desde creación o último break) con al menos un check-in en el período.
4. **Deduplicación de check-ins**: Si en una misma unidad temporal se intenta registrar múltiples check-ins (múltiples eventos), el sistema consolida/deduplicará correctamente en lectura o normalización.
5. **Persistencia en TIMESTAMPTZ y conversión de timezone**: Check-ins se guardan en UTC; conversión a `date` local del usuario ocurre correctamente según timezone del navegador.
6. **Conservación de históricos**: Al eliminar un hábito que alcanzó racha, sus check-ins se conservan en BD y no influyen en cálculos de racha de otros hábitos.
7. **Constraint de unicidad (user_id, habit_name)**: Dos hábitos del mismo usuario no pueden tener el mismo nombre.

---

## Arquitectura y decisiones técnicas

### Modelo de datos

- **Hábitos**: `id (PK), user_id (FK), name, description, frequency (daily|weekly), target_per_week, category_id, deleted_at (nullable), created_at, updated_at`.
- **Check-ins**: `id (PK), habit_id (FK), timestamp (TIMESTAMPTZ), created_at`. La `date` se deriva en lectura según timezone del cliente.
- **Categorías**: `id (PK), user_id (FK), name, is_system (boolean, false para personalizadas), created_at`.
- **Constraint**: `UNIQUE(user_id, name)` en `habits`.
- **Cálculo de racha**: On-the-fly desde check-ins; no persistido.

### Frontend

- **Rendering**: Cliente-first con Client Components y React Query para data fetching.
- **Rutas**:
  - `/` (dashboard, lista de hábitos por categoría)
  - `/login` (login)
  - `/signup` (registro)
  - `/habit/[id]` (detalle / edición del hábito)
  - `/estadisticas` (vista de progreso por categoría)
  - `/archivados` (hábitos archivados/históricos)
  - `/api/habits` (GET, POST, PUT, DELETE)
  - `/api/checkins` (GET, POST, DELETE)
  - `/api/categories` (GET, POST)
- **Estado y fetching**: React Query (o SWR) para cache, deduplicación y revalidación. Toggles usan optimismo con rollback en error.
- **Manejo de errores**: Toasts con mensajes detallados; modal de sesión expirada en 401; rollback optimista en toggles fallidos; reintentos automáticos para fallos de red.
- **Validaciones**: Email válido, contraseña ≥8 caracteres, `name` ≤100 chars, `description` ≤1000 chars, límite de 200 hábitos por usuario.

### Autenticación y seguridad

- Supabase Auth para email/password.
- Sesiones basadas en JWT o cookies según configuración de Supabase.
- Logout invalida sesión en backend.
- Row-level security (RLS) en Supabase para garantizar aislamiento por usuario.

---

## Decisiones tomadas en entrevista

### Ronda 1 — Arquitectura de datos

- **Frecuencia semanal** (Opción B): Modelada como `target_per_week` configurable (ej. 1, 2, 3 veces/semana).
- **Modelo de check-in** (Opción A): Fila por día/semana; deduplicación en lectura.
- **Archivado** (Opción A): Columna `deleted_at`/`archived`; check-ins conservados.
- **Unicidad nombre** (Opción A): `UNIQUE(user_id, name)` constraint.
- **Deduplicación check-in** (Opción B): Múltiples eventos permitidos; normalización en lectura.
- **Racha histórica** (Opción A): Calculada on-the-fly desde check-ins.
- **Formato fecha** (Opción 1): `TIMESTAMPTZ` UTC; derivar `date` local en cliente.

### Ronda 2 — QA y criterios verificables

- **Sincronía** (Opción B): Cambios visibles en ≤5 segundos.
- **Hábito sin check-ins** (Opción A): Mostrar racha `0`.
- **Últimos 14 días** (Opción A): Incluye día actual.
- **Hábitos archivados** (Opción C): Filtrables (defecto ocultos).
- **Denominador** (Opción A+1): Suma `target_per_week`, prorrateo <30d.
- **Criterios no verificables** (Opción A): Pruebas técnicas aparte.
- **Signup/logout** (Opción A): Validaciones y mensajes detallados.

### Ronda 3 — Developer entrante

- **Rendering** (Opción B): Cliente-first con React Query.
- **Rutas** (Opción B): Minimal + endpoints REST.
- **Errores** (Opción B): Toasts detallados, modal 401, rollback optimista.
- **Validaciones** (Opción A): Reglas estrictas.
- **Fetching** (Opción A): Client + React Query para reactividad e optimismo.

### Ronda 4 — Consistencia de scope vs brief

- Mobile premium: **Excluido** como no-goal.
- PWA: **Excluido** como no-goal.
- Monetización: **Excluida** como no-goal.
- Gamificación: **Excluida** como no-goal.
- Notificaciones: **Excluidas** como no-goal.
- Funciones sociales: **Excluidas** como no-goal.
- Integración profunda con servicios externos: **Excluida** como no-goal.

---

**Versión**: 1.0 (final post-entrevista)  
**Fecha**: 17 de mayo de 2026
