# Plan de implementación — Habit Tracker

Cada bloque es una unidad de trabajo entregable de forma independiente.
Cada tarea dentro de un bloque es atómica: un commit, una cosa.

---

## Bloque 1 — Scaffolding inicial
- [ ] T1.1: Inicializar proyecto Next.js 15 con TypeScript estricto, Tailwind CSS y App Router
- [ ] T1.2: Configurar cliente Supabase (`@supabase/ssr`) y variables de entorno
- [ ] T1.3: Configurar ESLint y tsconfig con `strict: true`

## Bloque 2 — Base de datos
- [ ] T2.1: Migración inicial: tablas `habits`, `checkins`, `categories` con constraints
- [ ] T2.2: Activar RLS: políticas de aislamiento por `user_id` en las tres tablas
- [ ] T2.3: Seed de categorías del sistema (`is_system = true`)

## Bloque 3 — Autenticación: registro y login
- [ ] T3.1: Página `/signup` — formulario con validación (email válido, password ≥ 8 chars)
- [ ] T3.2: Página `/login` — formulario con mensajes de error por credenciales incorrectas
- [ ] T3.3: Server Action de registro con Supabase Auth; redirección a dashboard

## Bloque 4 — Autenticación: sesión y rutas protegidas
- [ ] T4.1: Middleware Next.js que protege rutas privadas y redirige a `/login`
- [ ] T4.2: Logout — invalidación de sesión en backend y redirección a `/login`
- [ ] T4.3: Hook `useSession` para acceder al usuario autenticado en Client Components

## Bloque 5 — CRUD hábitos: creación y listado
- [ ] T5.1: API route `POST /api/habits` — validación y persistencia
- [ ] T5.2: API route `GET /api/habits` — listado filtrado por usuario, excluyendo archivados
- [ ] T5.3: Componente `HabitForm` — modal de creación con campos name, description, frequency, category
- [ ] T5.4: Página `/` (dashboard) — listado de hábitos agrupados por categoría

## Bloque 6 — CRUD hábitos: edición y eliminación
- [ ] T6.1: API route `PUT /api/habits/[id]` — actualización parcial de campos
- [ ] T6.2: API route `DELETE /api/habits/[id]` — eliminación lógica o física según racha
- [ ] T6.3: Página `/habit/[id]` — formulario de edición precargado

## Bloque 7 — Archivado de hábitos
- [ ] T7.1: Columna `deleted_at` en `habits`; lógica de archivado en `DELETE /api/habits/[id]`
- [ ] T7.2: Página `/archivados` — listado de hábitos archivados con filtro UI

## Bloque 8 — Categorías
- [ ] T8.1: API route `GET /api/categories` — sistema + personalizadas del usuario
- [ ] T8.2: API route `POST /api/categories` — creación de categoría personalizada con validación
- [ ] T8.3: Integrar selector de categorías en `HabitForm`

## Bloque 9 — Check-ins: registro y ventana de gracia
- [ ] T9.1: API route `POST /api/checkins` — persistir `TIMESTAMPTZ` UTC; validar ventana de gracia 12h
- [ ] T9.2: Lógica de derivación de `date` local según timezone del navegador (enviado en header o body)
- [ ] T9.3: Componente toggle de check-in en dashboard con optimistic update

## Bloque 10 — Check-ins: desmarcado y deduplicación
- [ ] T10.1: API route `DELETE /api/checkins/[id]` — validar ventana de 12h desde registro; confirmación explícita en UI
- [ ] T10.2: Deduplicación en lectura: `GET /api/checkins` consolida múltiples eventos por unidad temporal

## Bloque 11 — Racha
- [ ] T11.1: Función `calcularRacha(checkins, frequency)` — calcula racha on-the-fly para hábito diario
- [ ] T11.2: Extender `calcularRacha` para hábito semanal (períodos de 7 días)
- [ ] T11.3: Mostrar racha en tarjeta de cada hábito en dashboard

## Bloque 12 — Dashboard completo
- [ ] T12.1: React Query setup (`QueryClientProvider`) y hooks `useHabits`, `useCheckins`
- [ ] T12.2: Renderizado de tarjetas por categoría con estado "completado hoy / pendiente"
- [ ] T12.3: Toggle de check-in con rollback optimista en caso de error

## Bloque 13 — Manejo de errores de UI
- [ ] T13.1: Componente `Toast` — muestra mensajes de error con descripción detallada; auto-dismiss 5s
- [ ] T13.2: Modal `SessionExpiredModal` — se activa en respuesta 401; botón "Iniciar sesión"
- [ ] T13.3: Integrar `Toast` y `SessionExpiredModal` en el layout raíz; conectar con errores de React Query

## Bloque 14 — Vista de progreso por categoría
- [ ] T14.1: API route `GET /api/estadisticas` — porcentaje semanal con denominador prorrateo <30d
- [ ] T14.2: Racha promedio de últimos 14 días por categoría (solo hábitos activos con ≥1 racha completa)
- [ ] T14.3: Página `/estadisticas` — renderizado de métricas por categoría

## Bloque 15 — Sincronía
- [ ] T15.1: Polling de `useHabits` y `useCheckins` cada 5s con `refetchInterval`
- [ ] T15.2: (Opcional) Supabase Realtime como alternativa; feature flag o env var para elegir

## Bloque 16 — Validaciones y límites
- [ ] T16.1: Límite de 200 hábitos por usuario — validar en `POST /api/habits`
- [ ] T16.2: Unicidad `(user_id, name)` — manejar error de constraint en API y mostrar mensaje en UI

## Bloque 17 — Testing técnico
- [ ] T17.1: Test de aislamiento RLS — usuario A no puede leer datos de usuario B
- [ ] T17.2: Test de invalidación de sesión en logout
- [ ] T17.3: Test de precisión de racha diaria y semanal con gaps
- [ ] T17.4: Test de deduplicación de check-ins
- [ ] T17.5: Test de persistencia TIMESTAMPTZ y conversión de timezone
- [ ] T17.6: Test de conservación de históricos al archivar hábito con racha
