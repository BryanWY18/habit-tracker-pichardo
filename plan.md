# Plan de implementación — Habit Tracker

> Generado desde spec.md v1.0, ADR-0001…0005, diseño.md y pruebas-manuales.md.  
> Cada tarea la ejecuta el agente `implementer`.  
> Orden: setup → modelo de datos → auth → primer CRUD → check-in → vistas secundarias → sync.

---

## Reglas

- Máx. 1 hora por tarea.
- Criterio de hecho: enunciado verificable en primera persona ("puedo ver X").
- ADR y prueba manual referenciados cuando aplica.
- Ninguna tarea tiene más de 2 dependencias de código directas.
- Las tareas de extensión (webhooks, CSV, PWA) quedan fuera de este plan.

---

## T-01 — Inicializar proyecto Next.js 15 con TypeScript estricto, Tailwind y shadcn/ui

**Descripción**: Crear la base del proyecto ejecutable dentro del repositorio existente. Instalar y configurar shadcn/ui con los componentes que usarán las tareas siguientes.

**Depende de**: —

**Acciones**:
1. Ejecutar en la raíz del repo:
   ```
   npx create-next-app@latest . --typescript --tailwind --app --no-src-dir --import-alias "@/*" --yes
   ```
2. Verificar que `tsconfig.json` tiene `"strict": true` (create-next-app lo incluye por defecto).
3. Inicializar shadcn/ui: `npx shadcn@latest init` (style: `default`, baseColor: `slate`, CSS variables: sí).
4. Instalar los componentes que se usarán:
   ```
   npx shadcn@latest add button input dialog alert-dialog sonner skeleton select textarea
   ```
5. Vaciar `app/page.tsx` dejando solo `export default function Home() { return <main /> }`.
6. Vaciar `app/globals.css` dejando solo los tres imports de Tailwind (`@tailwind base/components/utilities`).
7. Ejecutar `npm run dev`.

**Criterio de hecho**: `npm run dev` arranca sin errores. `localhost:3000` carga sin errores de consola. `npm run build` completa sin errores de TypeScript.

---

## T-02 — Instalar Supabase y React Query; crear helpers de cliente y providers

**Descripción**: Instalar `@supabase/ssr`, `@supabase/js` y `@tanstack/react-query`. Crear los helpers para cliente de browser y servidor. Envolver el layout raíz con `QueryClientProvider`.

**Depende de**: T-01

**Acciones**:
1. `npm install @supabase/ssr @supabase/js @tanstack/react-query @tanstack/react-query-devtools`
2. Crear `lib/supabase/client.ts`:
   ```ts
   import { createBrowserClient } from '@supabase/ssr'
   export const createClient = () =>
     createBrowserClient(
       process.env.NEXT_PUBLIC_SUPABASE_URL!,
       process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
     )
   ```
3. Crear `lib/supabase/server.ts` con `createServerClient` y los cookie handlers **exclusivamente para el Middleware** (`middleware.ts`). No usar este helper en Server Components para data fetching — toda lectura de datos ocurre en Client Components vía React Query (ADR-0003). Los cookie handlers leen de `request.cookies` y escriben en `response.cookies`.
4. Crear `.env.local` con `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY` (valores reales del proyecto Supabase).
5. Crear `app/providers.tsx` (Client Component) con `QueryClientProvider` wrapeando `{children}` y `ReactQueryDevtools` en desarrollo.
6. Envolver el `<body>` de `app/layout.tsx` con `<Providers>`.

**Criterio de hecho**: `npm run build` completa sin errores de TypeScript. Importar `createClient` desde `@/lib/supabase/client` en cualquier componente no genera error de tipos.

**ADR referenciados**: ADR-0002, ADR-0003, ADR-0004

---

## T-03 — Crear esquema de base de datos en Supabase

**Descripción**: Ejecutar las migraciones SQL que crean las tablas, constraints, índices, políticas RLS, Realtime habilitado y el seed de categorías del sistema. Se ejecuta manualmente en el editor SQL de Supabase Studio.

**Depende de**: — (independiente de T-01/T-02 en código)

**SQL a ejecutar en Supabase Studio** (en este orden):

```sql
-- 1. Tabla categories
CREATE TABLE categories (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  name        text NOT NULL,
  is_system   boolean NOT NULL DEFAULT false,
  created_at  timestamptz NOT NULL DEFAULT now()
);

-- 2. Tabla habits
CREATE TABLE habits (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name            text NOT NULL CHECK (char_length(name) <= 100),
  description     text CHECK (char_length(description) <= 1000),
  frequency       text NOT NULL CHECK (frequency IN ('daily','weekly')),
  target_per_week int NOT NULL DEFAULT 7,
  category_id     uuid NOT NULL REFERENCES categories(id),
  deleted_at      timestamptz,
  created_at      timestamptz NOT NULL DEFAULT now(),
  updated_at      timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT habits_user_name_unique UNIQUE (user_id, name)
);

-- 3. Tabla check_ins
CREATE TABLE check_ins (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  habit_id    uuid NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
  timestamp   timestamptz NOT NULL,
  created_at  timestamptz NOT NULL DEFAULT now()
);

-- 4. Índice de rendimiento (ADR-0001)
CREATE INDEX idx_checkins_habit_timestamp
  ON check_ins (habit_id, timestamp DESC);

-- 5. RLS
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;
ALTER TABLE habits     ENABLE ROW LEVEL SECURITY;
ALTER TABLE check_ins  ENABLE ROW LEVEL SECURITY;

-- Políticas categories: sistema visible para todos; personalizadas solo al dueño
CREATE POLICY "categories_select" ON categories FOR SELECT
  USING (is_system = true OR user_id = auth.uid());
CREATE POLICY "categories_insert" ON categories FOR INSERT
  WITH CHECK (user_id = auth.uid());
CREATE POLICY "categories_delete" ON categories FOR DELETE
  USING (user_id = auth.uid());

-- Políticas habits: solo el dueño
CREATE POLICY "habits_all" ON habits FOR ALL
  USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());

-- Políticas check_ins: el dueño del hábito padre
CREATE POLICY "checkins_all" ON check_ins FOR ALL
  USING (habit_id IN (SELECT id FROM habits WHERE user_id = auth.uid()))
  WITH CHECK (habit_id IN (SELECT id FROM habits WHERE user_id = auth.uid()));

-- 6. Seed categorías del sistema
INSERT INTO categories (name, is_system) VALUES
  ('Salud',       true),
  ('Deporte',     true),
  ('Aprendizaje', true),
  ('Trabajo',     true),
  ('Personal',    true);
```

Después del SQL: en Supabase Studio → Database → Replication, habilitar Realtime para las tablas `habits` y `check_ins`.

**Criterio de hecho**: En Supabase Studio las tres tablas aparecen con la estructura correcta. Las 5 categorías del sistema están en la tabla `categories`. Las políticas de RLS están activas en las tres tablas. Realtime está habilitado en `habits` y `check_ins`.

**ADR referenciados**: ADR-0001, ADR-0004

---

## T-04 — Middleware de autenticación con @supabase/ssr

**Descripción**: Crear `middleware.ts` en la raíz del proyecto. Protege todas las rutas excepto `/login`, `/signup` y assets estáticos. Refresca el token automáticamente.

**Depende de**: T-02

**Acciones**:
1. Crear `middleware.ts` en la raíz con `createServerClient` de `@supabase/ssr`.
2. Configurar los cookie handlers usando `request.cookies` y `response.cookies`.
3. Llamar `supabase.auth.getUser()` para verificar la sesión.
4. Si no hay usuario y la ruta no es `/login` ni `/signup`: `return NextResponse.redirect(new URL('/login', request.url))`.
5. Configurar el `matcher`:
   ```ts
   export const config = {
     matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
   }
   ```

**Criterio de hecho**: Navegando a `localhost:3000/` en el navegador sin sesión activa, la URL cambia inmediatamente a `localhost:3000/login`. TC-06 pasa manualmente.

**ADR referenciados**: ADR-0002  
**Pruebas manuales**: TC-06

---

## T-05 — Páginas /login y /signup con validación y toasts

**Descripción**: Formularios de autenticación con validación en cliente, mensajes de error exactos de la spec y redirección al dashboard tras autenticarse.

**Depende de**: T-02, T-04

**Acciones**:
1. Crear `app/login/page.tsx` (Client Component):
   - Campos: email (type=email), contraseña (type=password).
   - Botón "Iniciar sesión" (primario).
   - Enlace `href="/signup"` al pie.
   - Al submit: `supabase.auth.signInWithPassword()` → si error: `toast.error("Email o contraseña inválidos")` → si éxito: `router.push('/')`.
2. Crear `app/signup/page.tsx` (Client Component, misma estructura):
   - Botón "Registrarse".
   - Validaciones cliente: formato de email, contraseña ≥8 caracteres (antes de llamar a Supabase).
   - `supabase.auth.signUp()` → si error de email duplicado: `toast.error("El email ya está registrado")` → si éxito: `router.push('/')`.
3. Añadir `<Toaster />` de Sonner en `app/layout.tsx`.
4. Estilar ambas páginas: `min-h-screen flex items-center justify-center`, bloque central `max-w-sm w-full space-y-6`, título `text-2xl font-semibold text-slate-900`.

**Criterio de hecho**: TC-01, TC-02, TC-03, TC-04 pasan manualmente.

**ADR referenciados**: ADR-0002  
**Pruebas manuales**: TC-01, TC-02, TC-03, TC-04

---

## T-06 — NavBar, route group protegido y logout

**Descripción**: Crear el layout para rutas autenticadas con `NavBar` fija. Implementar logout con invalidación de sesión en backend y redirección a `/login`.

**Depende de**: T-05

**Acciones**:
1. Crear `app/(protected)/layout.tsx` que renderiza `<NavBar />` sobre `{children}`.
2. Crear `components/NavBar.tsx` (Client Component):
   - Logo/nombre "Habit Tracker" a la izquierda (`font-semibold text-slate-900`).
   - Enlace a `/estadisticas` en el centro/derecha.
   - Botón "Logout" a la derecha.
   - Estilo: `h-14 border-b border-slate-200 bg-white px-12 flex items-center justify-between sticky top-0 z-10`.
3. Botón logout: `supabase.auth.signOut()` → `router.push('/login')` → `router.refresh()`.
4. Mover `app/page.tsx` a `app/(protected)/page.tsx`.

**Criterio de hecho**: TC-05 pasa manualmente: hacer logout redirige a `/login`; intentar navegar a `/` nuevamente redirige de vuelta a `/login`.

**ADR referenciados**: ADR-0002  
**Pruebas manuales**: TC-05, TC-06

---

## T-07 — Función pura de cálculo de racha (TypeScript)

**Descripción**: Implementar `calculateStreak` en TypeScript puro, sin dependencias de React ni Supabase. Cubre frecuencia diaria y semanal.

**Depende de**: T-01

**Acciones**:
1. Crear `lib/streak.ts` con:
   ```ts
   export type StreakResult = { current: number; unit: 'días' | 'semanas' }

   export function calculateStreak(
     checkIns: { timestamp: string }[],
     frequency: 'daily' | 'weekly',
     targetPerWeek: number
   ): StreakResult
   ```
2. Lógica diaria: derivar la fecha local de cada timestamp (usando `Intl.DateTimeFormat` con la timezone del navegador), deduplicar por día, ordenar descendente, contar días consecutivos hacia atrás desde hoy. Si el último check-in no es de hoy ni de ayer (dentro de la misma medianoche local), racha = 0.
3. Lógica semanal: agrupar check-ins en ventanas de 7 días (empezando desde el lunes de la semana actual hacia atrás), verificar si cada ventana tiene al menos 1 check-in, contar semanas consecutivas hacia atrás hasta el primer gap.
4. Retornar `{ current: 0, unit: 'días' }` para array vacío.

**Criterio de hecho**: Añadir al final de `lib/streak.ts` un bloque temporal:
```ts
// bloque temporal — eliminar antes del commit
if (require.main === module) {
  console.log(calculateStreak([], 'daily', 7))
  const sevenDays = Array.from({ length: 7 }, (_, i) => ({
    timestamp: new Date(Date.now() - i * 86400000).toISOString()
  }))
  console.log(calculateStreak(sevenDays, 'daily', 7))
}
```
Ejecutar `npx tsx lib/streak.ts`. El primer log devuelve `{ current: 0, unit: 'días' }` y el segundo `{ current: 7, unit: 'días' }`. Eliminar el bloque antes del commit. Mínimo garantizado sin tsx: `npm run build` completa sin errores de TypeScript.

**ADR referenciados**: ADR-0001

---

## T-08 — Dashboard: listar hábitos con CategoryGroup, HabitCard y StreakBadge

**Descripción**: Implementar el dashboard principal. Fetch de hábitos y check-ins desde Supabase con React Query. Renderizar los componentes de listado con racha calculada.

**Depende de**: T-06, T-07  
**Nota**: Requiere T-03 ejecutado (tablas existentes en Supabase).

**Acciones**:
1. Crear `hooks/useHabits.ts` con `useQuery`:
   - Consultar `habits` donde `deleted_at IS NULL` para el usuario autenticado.
   - Join con `categories`.
   - Consultar `check_ins` de los últimos 30 días para todos los hábitos del usuario.
   - Calcular `calculateStreak` para cada hábito.
2. Crear `components/StreakBadge.tsx`: badge con valor numérico; `text-emerald-600` si >0, `text-slate-400` si 0; `font-bold text-sm`.
3. Crear `components/HabitCard.tsx`: nombre (`text-base font-medium`), descripción truncada 2 líneas (`text-sm text-slate-500`), `<StreakBadge />`, slot para `CheckInToggle` (placeholder `<div />` por ahora).
4. Crear `components/CategoryGroup.tsx`: header `text-lg font-semibold slate-900` con nombre de categoría y contador de hábitos. Lista vertical de `HabitCard`s.
5. Crear `components/EmptyState.tsx`: mensaje centrado + botón CTA opcional.
6. Actualizar `app/(protected)/page.tsx`: título "Mis hábitos", botón "Nuevo hábito" (placeholder), lista de `CategoryGroup` agrupados por `category_id`, `Skeleton` durante carga.

**Criterio de hecho**: Con un usuario autenticado y al menos un hábito en Supabase, el dashboard muestra ese hábito dentro de su grupo de categoría con la racha correcta. Si no hay hábitos, muestra `EmptyState`. TC-12 y TC-22 pasan manualmente.

**ADR referenciados**: ADR-0001, ADR-0003  
**Pruebas manuales**: TC-12, TC-20, TC-21, TC-22, TC-23, TC-24

---

## T-09 — Formulario de creación y edición de hábito (HabitForm)

**Descripción**: Implementar `HabitForm` reutilizable para crear y editar hábitos. El botón "Nuevo hábito" del dashboard abre el formulario en un `Dialog`. Las categorías del sistema y las del usuario se cargan desde Supabase.

**Depende de**: T-08

**Acciones**:
1. Crear `hooks/useCategories.ts` con `useQuery` que fetch categorías del sistema (`is_system = true`) + las del usuario autenticado.
2. Crear `hooks/useCreateHabit.ts` con `useMutation`:
   - Insertar en `habits`.
   - `onSuccess`: `queryClient.invalidateQueries({ queryKey: ['habits'] })`.
3. Crear `components/HabitForm.tsx`:
   - `Input` nombre (requerido, maxlength 100). Error inline "El nombre es requerido" si vacío.
   - `Textarea` descripción (maxlength 1000).
   - `Select` frecuencia (`Diaria` / `Semanal`).
   - `Input` numérico `target_per_week` (visible solo si frecuencia = Semanal, default 1).
   - `Select` categoría (cargado desde `useCategories`, requerido, no puede quedar vacío).
   - Botón "Guardar" (primario) + botón "Cancelar" (secundario).
4. En `app/(protected)/page.tsx`: botón "Nuevo hábito" abre `Dialog` con `HabitForm`. Al guardar: cerrar dialog + toast opcional de éxito.
5. En `HabitForm`, añadir opción "Crear categoría..." al final del `Select` de categoría. Al seleccionarla: mostrar un `Input` inline con botón "Añadir". Al confirmar: `supabase.from('categories').insert({ name, user_id })` → `queryClient.invalidateQueries(['categories'])` → la nueva categoría queda seleccionada automáticamente (TC-26).

**Criterio de hecho**: TC-07, TC-08, TC-25, TC-26, TC-27 pasan manualmente. Un hábito creado aparece en el dashboard sin recargar la página.

**Pruebas manuales**: TC-07, TC-08, TC-25, TC-26, TC-27

---

## T-10 — Check-in toggle con optimistic update, ventana de gracia y desmarcar

**Descripción**: Implementar `CheckInToggle` con actualización optimista y rollback en error. Incluye la validación de la ventana de gracia de 12 horas para marcar D-1 y para desmarcar.

**Depende de**: T-08

**Acciones**:
1. Crear `hooks/useToggleCheckin.ts` con `useMutation`:
   - `onMutate`: snapshot del cache → actualizar optimistamente el estado del hábito en el cache de React Query.
   - `onError`: rollback al snapshot anterior + `toast.error("No se pudo registrar el check-in")`.
   - `onSettled`: `queryClient.invalidateQueries({ queryKey: ['habits'] })`.
2. Crear `components/CheckInToggle.tsx` (estados: idle, completado, loading, disabled):
   - Al hacer clic en estado `idle`: llamar a `useToggleCheckin` → insertar en `check_ins` con `timestamp = new Date().toISOString()`.
   - Al hacer clic en estado `completado` dentro de 12h del check-in: abrir `AlertDialog` con texto "¿Desmarcar este check-in?" → confirmar → DELETE del check-in.
   - Si check-in tiene más de 12h: botón visualmente deshabilitado (`opacity-50 cursor-not-allowed`), sin diálogo.
3. Validación de gracia para D-1 en cliente: si `Date.now() < medianoche_local_hoy + 12h`, permitir registrar D-1.
4. Montar `CheckInToggle` dentro de `HabitCard` (reemplazar el placeholder de T-08).
5. Manejar el caso del modal 401: si el error de la mutación es 401, mostrar `Dialog` con "Sesión expirada — inicia sesión nuevamente".

**Criterio de hecho**: TC-13, TC-14, TC-15, TC-16, TC-17, TC-18, TC-19 pasan manualmente. El toggle cambia de estado de forma instantánea (antes de recibir respuesta del servidor).

**ADR referenciados**: ADR-0003, ADR-0004  
**Pruebas manuales**: TC-13, TC-14, TC-15, TC-16, TC-17, TC-18, TC-19

---

## T-11 — Página de detalle y edición de hábito (/habit/[id])

**Descripción**: Implementar la página con `HabitForm` prellenado, historial de check-ins y acción de eliminación (física o lógica según si hay racha).

**Depende de**: T-09, T-10

**Acciones**:
1. Crear `app/(protected)/habit/[id]/page.tsx` (Client Component).
2. `useQuery` para cargar el hábito por ID con sus check-ins.
3. Crear `hooks/useUpdateHabit.ts` con `useMutation` que hace PATCH en `habits` + `router.back()` al éxito.
4. Crear `hooks/useDeleteHabit.ts` con `useMutation`:
   - Si el hábito no tiene ningún check-in registrado (`count = 0`): DELETE físico (CASCADE elimina `check_ins`).
   - Si tiene al menos un check-in: soft-delete (`deleted_at = now()`), preserva el historial en `check_ins`.
   - `onSuccess`: `queryClient.invalidateQueries(['habits'])` + `router.push('/')`.
5. Reutilizar `HabitForm` en modo edición: props `defaultValues` con los datos del hábito.
6. Sección de historial: lista de los últimos 30 días de check-ins (fecha local + "✓" o vacío), formato compacto.
7. Botón "Eliminar hábito" → `AlertDialog` de confirmación → ejecutar delete.

**Criterio de hecho**: TC-09, TC-10, TC-11 pasan manualmente. Los cambios al editar se reflejan en el dashboard al volver a `/`.

**ADR referenciados**: ADR-0001  
**Pruebas manuales**: TC-09, TC-10, TC-11

---

## T-12 — Vista de estadísticas (/estadisticas) con CategoryProgressCard

**Descripción**: Implementar la página de progreso por categoría. Calcular porcentaje semanal y racha promedio de los últimos 14 días para cada categoría con hábitos activos.

**Depende de**: T-07, T-08

**Acciones**:
1. Crear `hooks/useCategoryStats.ts` con `useQuery` que:
   - Fetch hábitos activos + check-ins de la semana calendario actual (lunes–domingo).
   - Por categoría: calcular `completionPct = check_ins_semana / target_total_semana × 100` (prorrateo para hábitos con `created_at` < 30 días).
   - Por categoría: calcular racha promedio sobre los últimos 14 días de hábitos con al menos una racha completa (usando `calculateStreak`).
2. Crear `components/CategoryProgressCard.tsx`:
   - Nombre de categoría (`text-lg font-semibold`).
   - Porcentaje en `text-2xl font-bold text-indigo-600`.
   - Barra de progreso: `<div>` con `width: ${pct}%` y fondo `bg-indigo-600 h-2 rounded-full`.
   - Racha promedio en `text-sm text-slate-500`.
3. Crear `app/(protected)/estadisticas/page.tsx`: título "Estadísticas" (`text-xl font-semibold`), grid 2 columnas de `CategoryProgressCard`, `EmptyState` si no hay datos, `Skeleton` durante carga.

**Criterio de hecho**: TC-28 y TC-29 pasan manualmente con datos precargados en Supabase (al menos 2 hábitos activos con check-ins esta semana).

**ADR referenciados**: ADR-0001  
**Pruebas manuales**: TC-28, TC-29

---

## T-13 — Vista de hábitos archivados (/archivados)

**Descripción**: Implementar la página de históricos. Listar hábitos con `deleted_at IS NOT NULL` en variante solo-lectura de `HabitCard`.

**Depende de**: T-11

**Acciones**:
1. Crear `hooks/useArchivedHabits.ts` con `useQuery` que fetch `habits` donde `deleted_at IS NOT NULL` para el usuario autenticado, con su categoría y racha máxima calculada desde check-ins históricos.
2. Añadir variante `archived` a `HabitCard.tsx`: sin `CheckInToggle`, badge "Archivado" en `text-slate-400`, mostrar `deleted_at` formateado como "Archivado el DD/MM/YYYY".
3. Crear `app/(protected)/archivados/page.tsx`: título "Históricos", lista de `HabitCard` variante `archivado`, `EmptyState` si no hay archivados.
4. Añadir enlace a `/archivados` en la `NavBar` (junto a `/estadisticas`).

**Criterio de hecho**: TC-30 pasa manualmente: un hábito eliminado con racha aparece en `/archivados` y no aparece en `/`. El hábito en `/archivados` no tiene botón de toggle activo.

**Pruebas manuales**: TC-30

---

## T-14 — Hook useRealtimeSync para sincronía entre dispositivos

**Descripción**: Implementar el hook `useRealtimeSync()` que suscribe a cambios en `habits` y `check_ins` del usuario autenticado vía Supabase Realtime e invalida las queries de React Query al recibir eventos.

**Depende de**: T-08

**Acciones**:
1. Crear `hooks/useRealtimeSync.ts`:
   ```ts
   export function useRealtimeSync(userId: string) { ... }
   ```
2. En `useEffect`: crear un canal de Supabase Realtime con filtro `user_id=eq.${userId}` para la tabla `habits` y un canal para `check_ins` (filtrado por `habit_id` perteneciente al usuario via join, o tabla completa con RLS aplicando).
3. Handler de cada evento (`INSERT`, `UPDATE`, `DELETE`): `queryClient.invalidateQueries({ queryKey: ['habits'] })`.
4. Cleanup en el return del `useEffect`: `supabase.removeChannel(channel)`.
5. Usar el hook en `app/(protected)/layout.tsx` para que aplique a todas las páginas protegidas.

**Criterio de hecho**: TC-31 pasa manualmente: marcar un hábito en la Pestaña A lo muestra como completado en la Pestaña B en menos de 5 segundos, sin recargar.

**ADR referenciados**: ADR-0005  
**Pruebas manuales**: TC-31

---

## Resumen de dependencias

```
T-01 ──► T-02 ──► T-04 ──► T-05 ──► T-06 ──► T-08 ──► T-09 ──► T-11 ──► T-13
          │                                     │  │              │
          │                                     │  └────────────► T-10
          │                                     │
          │                                  T-07 ──► T-12
          │
T-03 (Supabase Studio, independiente de código)
T-07 (también depende de T-01)
T-14 depende de T-08
```

**Orden recomendado de ejecución**: T-01 → T-02 → T-03 (paralelo con T-02) → T-04 → T-05 → T-06 → T-07 → T-08 → T-09 → T-10 → T-11 → T-12 → T-13 → T-14
