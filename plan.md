# Plan de implementación — Habit Tracker (núcleo)

**Versión**: 1.0  
**Generado**: 2026-05-31  
**Agente ejecutor**: `implementer`

Cada tarea es ejecutable en ≤1 hora. Las dependencias se listan explícitamente; ninguna tarea depende de más de 2 tareas previas. Los criterios de aceptación referenciados corresponden a `spec.md`.

---

## T-01 — Inicializar proyecto Next.js 15

**Descripción**  
Crear el proyecto base con `create-next-app` usando las opciones correctas para este stack.

```bash
npx create-next-app@latest . \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --no-src-dir \
  --import-alias "@/*"
```

Ajustar `tsconfig.json` para strict mode si no viene activado (`"strict": true`). Borrar contenido de ejemplo en `app/page.tsx` (dejar solo `export default function Home() { return <main /> }`).

**Depende de**: —

**Criterio de hecho**  
`npm run dev` levanta sin errores en `http://localhost:3000`. `npx tsc --noEmit` termina con exit code 0.

---

## T-02 — Instalar y configurar cliente Supabase

**Descripción**  
Instalar las librerías necesarias y crear los helpers de cliente/servidor.

```bash
npm install @supabase/supabase-js @supabase/ssr
```

Crear `.env.local` con:
```
NEXT_PUBLIC_SUPABASE_URL=<url del proyecto Supabase>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon key>
```

Crear `lib/supabase/client.ts` (para Client Components, usa `createBrowserClient`) y `lib/supabase/server.ts` (para Server Components/Route Handlers, usa `createServerClient` con cookies de `next/headers`).

**Depende de**: T-01

**Criterio de hecho**  
`npx tsc --noEmit` pasa limpio con los dos archivos creados. `lib/supabase/client.ts` exporta una función `createClient()` que no produce error de tipos.

---

## T-03 — Crear schema de base de datos en Supabase

**Descripción**  
Crear las 3 tablas del modelo de datos ejecutando el siguiente SQL en el SQL Editor de Supabase Dashboard (o via Supabase CLI con `supabase migration new`):

```sql
-- Categorías
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  is_system BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT categories_user_name_unique UNIQUE (user_id, name)
);

-- Hábitos
CREATE TABLE habits (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL CHECK (char_length(name) <= 100),
  description TEXT CHECK (char_length(description) <= 1000),
  frequency TEXT NOT NULL CHECK (frequency IN ('daily', 'weekly')),
  target_per_week INT NOT NULL DEFAULT 1 CHECK (target_per_week BETWEEN 1 AND 7),
  category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT habits_user_name_unique UNIQUE (user_id, name)
);

-- Check-ins
CREATE TABLE check_ins (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  habit_id UUID NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
  timestamp TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- RLS: activar en las 3 tablas
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;
ALTER TABLE habits ENABLE ROW LEVEL SECURITY;
ALTER TABLE check_ins ENABLE ROW LEVEL SECURITY;

-- Políticas básicas (acceso solo a filas propias)
CREATE POLICY "categories: own rows" ON categories
  USING (user_id = auth.uid() OR is_system = true);

CREATE POLICY "habits: own rows" ON habits
  USING (user_id = auth.uid());

CREATE POLICY "check_ins: own rows" ON check_ins
  USING (habit_id IN (SELECT id FROM habits WHERE user_id = auth.uid()));
```

**Depende de**: T-02

**Referencia spec**: modelo de datos (`spec.md` §Arquitectura > Modelo de datos)

**Criterio de hecho**  
En Supabase Table Editor aparecen las tablas `categories`, `habits`, `check_ins`. Query `SELECT constraint_name FROM information_schema.table_constraints WHERE table_name = 'habits' AND constraint_type = 'UNIQUE'` devuelve `habits_user_name_unique`. RLS aparece como "enabled" en cada tabla.

---

## T-04 — Seed de categorías del sistema

**Descripción**  
Insertar las categorías predefinidas con `user_id = NULL` e `is_system = true`. Ejecutar en SQL Editor:

```sql
INSERT INTO categories (user_id, name, is_system) VALUES
  (NULL, 'Salud',       true),
  (NULL, 'Ejercicio',   true),
  (NULL, 'Alimentación',true),
  (NULL, 'Lectura',     true),
  (NULL, 'Productividad',true),
  (NULL, 'Bienestar',   true),
  (NULL, 'Aprendizaje', true);
```

Actualizar la política RLS de `categories` para permitir `user_id IS NULL` en filas de sistema (ya incluida en T-03 con `is_system = true`).

**Depende de**: T-03

**Referencia spec**: CA-25 (ver categorías predefinidas al crear hábito)

**Criterio de hecho**  
`SELECT * FROM categories WHERE is_system = true` devuelve exactamente 7 filas. La política RLS permite leer estas filas a cualquier usuario autenticado (verificar con un usuario de prueba desde el client de Supabase).

---

## T-05 — Auth: páginas /login y /signup

**Descripción**  
Crear las dos páginas de autenticación usando Supabase Auth.

- `app/login/page.tsx`: formulario con campos email y contraseña. Al submit llama `supabase.auth.signInWithPassword()`. En error `invalid_credentials` muestra "Email o contraseña inválidos". En éxito redirige a `/`.
- `app/signup/page.tsx`: formulario con campos email y contraseña (mín. 8 caracteres, validación de email válido en cliente). Al submit llama `supabase.auth.signUp()`. En error de email duplicado muestra "El email ya está registrado". En éxito redirige a `/`.

Usar `useRouter()` de `next/navigation` para redirects. Los mensajes de error van en un `<p>` visible bajo el formulario.

**Depende de**: T-02, T-03

**Referencia spec**: CA-1, CA-2, CA-3, CA-4

**Criterio de hecho**  
Registrarse con un email nuevo crea el usuario en Supabase Dashboard > Authentication > Users. Iniciar sesión con esas credenciales no genera error. Registrarse con el mismo email muestra "El email ya está registrado". Iniciar con contraseña incorrecta muestra "Email o contraseña inválidos".

---

## T-06 — Middleware de protección de rutas

**Descripción**  
Crear `middleware.ts` en la raíz del proyecto. Usar `createServerClient` de `@supabase/ssr` para leer la sesión desde cookies.

Lógica:
- Si la ruta es `/login` o `/signup` y el usuario tiene sesión → redirect a `/`.
- Si la ruta es cualquier otra (excepto `/api/*`, `/_next/*`, archivos estáticos) y el usuario no tiene sesión → redirect a `/login`.

Configurar `matcher` en `middleware.ts` para excluir `_next`, `favicon`, y archivos estáticos.

**Depende de**: T-05

**Referencia spec**: CA-5, CA-6

**Criterio de hecho**  
Abrir `/` en navegador sin sesión activa redirige a `/login` sin mostrar contenido privado. Con sesión activa, navegar a `/login` redirige a `/`. Hacer logout desde Supabase y refrescar `/` redirige a `/login`.

---

## T-07 — API /api/categories (GET, POST)

**Descripción**  
Crear `app/api/categories/route.ts` con dos handlers:

- `GET`: devuelve `[...categorías sistema, ...categorías del usuario autenticado]` ordenadas por nombre. Usar `createServerClient` para leer la sesión; retornar 401 si sin sesión.
- `POST`: recibe `{ name: string }`. Valida que `name` no esté vacío y tenga ≤50 caracteres. Inserta con `user_id = session.user.id` e `is_system = false`. Retorna 201 con la categoría creada. Retorna 409 si ya existe una categoría con ese nombre para el usuario.

**Depende de**: T-04, T-06

**Referencia spec**: CA-25, CA-26

**Criterio de hecho**  
`GET /api/categories` con sesión válida devuelve JSON array incluyendo las 7 categorías sistema. `POST /api/categories` con `{ "name": "Meditación" }` crea la categoría y la devuelve en el próximo GET. `GET /api/categories` sin header de cookie de sesión retorna 401.

---

## T-08 — API /api/habits (GET, POST)

**Descripción**  
Crear `app/api/habits/route.ts`:

- `GET`: devuelve hábitos activos del usuario (`deleted_at IS NULL`), incluyendo el nombre de la categoría (join con `categories`). Ordenados por `categories.name ASC`, luego `habits.name ASC`. Retorna 401 sin sesión.
- `POST`: recibe `{ name, description?, frequency, target_per_week, category_id? }`. Validaciones:
  - `name` requerido → 400 "El nombre es requerido"
  - `name` ≤100 chars → 400 "El nombre no puede superar 100 caracteres"
  - `description` ≤1000 chars → 400
  - `frequency` debe ser `'daily'` o `'weekly'` → 400
  - `target_per_week` entre 1 y 7 → 400
  - Duplicate name del mismo user → 409 "Ya existe un hábito con ese nombre"
  - Límite de 200 hábitos activos por usuario → 429

**Depende de**: T-07

**Referencia spec**: CA-7, CA-8; modelo de datos `spec.md`

**Criterio de hecho**  
`POST /api/habits` con body vacío retorna 400 con mensaje "El nombre es requerido". `POST` válido crea el hábito y aparece en el `GET` siguiente. Crear dos hábitos con el mismo nombre retorna 409. `GET` sin sesión retorna 401.

---

## T-09 — API /api/habits/[id] (PUT, DELETE)

**Descripción**  
Crear `app/api/habits/[id]/route.ts`:

- `PUT`: actualiza `name`, `description`, `frequency`, `target_per_week`, `category_id`. Mismas validaciones que POST. Retorna 404 si el hábito no pertenece al usuario. Actualiza `updated_at = now()`.
- `DELETE`: lógica de archivado selectivo:
  - Si el hábito nunca tuvo racha (sin check-ins, o check-ins insuficientes para completar un período): hard delete de hábito y sus check-ins.
  - Si tuvo al menos una racha completa (diario: ≥7 días consecutivos alguna vez; semanal: ≥2 períodos): soft delete (`deleted_at = now()`), conserva check-ins.

**Depende de**: T-08

**Referencia spec**: CA-9, CA-10, CA-11

**Criterio de hecho**  
`PUT /api/habits/:id` con nuevo nombre lo actualiza en el GET siguiente. `DELETE` sobre hábito sin historial: desaparece de la tabla `habits`. `DELETE` sobre hábito con historial: `deleted_at` queda seteado, check-ins permanecen en `check_ins`. Intentar PUT/DELETE de hábito de otro usuario retorna 404.

---

## T-10 — Dashboard: listar hábitos agrupados por categoría

**Descripción**  
Implementar `app/page.tsx` como Client Component usando React Query.

- `useQuery` llama a `GET /api/habits`.
- Renderiza los hábitos agrupados por `category.name`. Cada hábito muestra: nombre, racha (mostrar `0` mientras no haya cálculo real), botón de toggle (sin funcionalidad aún, solo visual).
- Manejar estado loading con skeleton y estado vacío con mensaje "No tienes hábitos todavía — crea uno".
- Instalar React Query: `npm install @tanstack/react-query` y crear `app/providers.tsx` que envuelve el layout con `QueryClientProvider`.

**Depende de**: T-08, T-06

**Referencia spec**: CA-12

**Criterio de hecho**  
Dashboard con al menos un hábito creado muestra ese hábito bajo su categoría con nombre visible. Sin hábitos muestra el mensaje de estado vacío. El loading state muestra al menos un skeleton visible (no pantalla en blanco). `npx tsc --noEmit` pasa.

---

## T-11 — Formulario crear/editar hábito

**Descripción**  
Crear `app/habit/new/page.tsx` (formulario de creación) y `app/habit/[id]/page.tsx` (formulario de edición).

Campos del formulario:
- Nombre (input text, requerido, max 100)
- Descripción (textarea, opcional, max 1000)
- Frecuencia (select: Diaria / Semanal)
- Veces por semana (input number 1-7, visible solo si frecuencia = Semanal)
- Categoría (select con opciones de `GET /api/categories`)

En creación: `POST /api/habits`. En edición: precarga datos vía `GET /api/habits/:id` y usa `PUT /api/habits/:id`. Tras éxito, redirect a `/` con `router.push('/')`. Mostrar errores de validación bajo el campo correspondiente.

**Depende de**: T-07, T-08

**Referencia spec**: CA-7, CA-8, CA-9

**Criterio de hecho**  
Crear un hábito desde el formulario y verlo en el dashboard inmediatamente. Editar el nombre de un hábito existente y ver el cambio reflejado en el dashboard. Intentar crear sin nombre muestra "El nombre es requerido" bajo el campo. `npx tsc --noEmit` pasa.

---

## T-12 — API /api/checkins (POST, DELETE)

**Descripción**  
Crear `app/api/checkins/route.ts`:

- `POST`: recibe `{ habit_id, client_timestamp }` (ISO 8601 con timezone del navegador). Valida:
  - Hábito pertenece al usuario y no está archivado.
  - La fecha local derivada de `client_timestamp` es hoy o D-1.
  - Si es D-1: verificar que `now() - local_midnight_of_D < 12h` (ventana de gracia). Si no cumple → 422 "Fuera de la ventana de gracia".
  - Si ya existe check-in para esa unidad temporal → 409 "Check-in ya registrado".
  - Inserta con `timestamp = now()` (UTC).
- `DELETE`: recibe `{ checkin_id }`. Valida:
  - Check-in pertenece al usuario (via habit).
  - `now() - check_in.created_at < 12h`. Si no → 422 "No se puede desmarcar después de 12 horas".
  - Elimina el registro.

**Depende de**: T-08

**Referencia spec**: CA-13, CA-14, CA-15, CA-17, CA-18

**Criterio de hecho**  
`POST` válido crea un registro en `check_ins`. Un segundo `POST` para la misma unidad temporal retorna 409. `DELETE` dentro de 12h elimina el check-in. `DELETE` con un `check_in.created_at` artificialmente antiguo (>12h, modificado en DB para test) retorna 422. `POST` para D-1 fuera de ventana de gracia retorna 422.

---

## T-13 — Toggle check-in en dashboard con optimismo

**Descripción**  
Conectar el botón de toggle en el dashboard (creado en T-10) con la API.

- `useMutation` de React Query para POST (marcar) y DELETE (desmarcar).
- Actualización optimista: al hacer click, actualizar el cache de React Query inmediatamente con el nuevo estado visual, sin esperar la respuesta.
- En error: rollback al estado anterior con `onError`.
- En error 401: mostrar modal "Sesión expirada — inicia sesión nuevamente".
- En cualquier otro error: mostrar toast con mensaje del error de la API. Instalar `react-hot-toast` (o similar) para toasts.
- Lógica de desmarcar: si el check-in tiene ≤12h, mostrar diálogo de confirmación ("¿Desmarcar este check-in?") antes de llamar DELETE.

**Depende de**: T-12, T-10

**Referencia spec**: CA-13, CA-17, CA-19

**Criterio de hecho**  
Click en toggle cambia el estado visual del botón inmediatamente (antes de recibir respuesta). Si la API responde con error, el botón vuelve al estado anterior y aparece un toast. Marcar un hábito ya marcado (con ≤12h) muestra el diálogo de confirmación. `npx tsc --noEmit` pasa.

---

## T-14 — Cálculo de racha on-the-fly

**Descripción**  
Crear `lib/streak.ts` con la función pura `calculateStreak(checkins: CheckIn[], frequency: 'daily' | 'weekly', clientTimezone: string): number`.

Lógica:
- **Diaria**: convertir cada check-in a fecha local (usando `clientTimezone`). Ordenar descendente. Contar días consecutivos desde hoy hacia atrás sin gaps. Retornar 0 si no hay check-in hoy ni ayer (sin ventana de gracia; la racha se pierde si hoy no hay check-in Y ayer tampoco).
- **Semanal**: agrupar check-ins por semana ISO. Contar períodos consecutivos de 7 días (desde hoy hacia atrás) que tengan al menos un check-in. Retornar 0 si no hay check-in en el período actual ni en el anterior.

Integrar en `GET /api/habits`: incluir check-ins recientes (últimos 60 días) en la query y calcular racha antes de devolver la respuesta. Añadir campo `streak: number` a cada hábito en la respuesta.

**Depende de**: T-12

**Referencia spec**: CA-20, CA-21, CA-22, CA-23, CA-24

**Criterio de hecho**  
Hábito diario con 7 check-ins en días consecutivos (incluyendo hoy) muestra `streak: 7` en `GET /api/habits`. Hábito sin check-ins muestra `streak: 0`. Hábito diario con check-ins hasta anteayer (gap ayer) muestra `streak: 0`. El dashboard muestra el número de racha correcto junto a cada hábito.

---

## T-15 — Vista /estadisticas: progreso por categoría

**Descripción**  
Crear `app/estadisticas/page.tsx` como Client Component con React Query.

Crear `GET /api/stats/categories` que devuelve por cada categoría con hábitos activos:
- `completionRate`: check-ins actuales en la semana calendario (lun-dom) ÷ denominador. Denominador = suma de `target_per_week` de hábitos activos; prorratear por `(días activos en la semana / 7)` para hábitos creados hace <30 días.
- `avgStreak`: promedio de rachas de los últimos 14 días (incluyendo hoy) de hábitos activos con al menos una racha completa (diario: ≥1 día; semanal: ≥1 período). Si ningún hábito califica, omitir el campo.

La vista muestra una tarjeta por categoría con: nombre, barra de progreso visual, porcentaje de `completionRate`, racha promedio si existe.

**Depende de**: T-14, T-07

**Referencia spec**: CA-28, CA-29

**Criterio de hecho**  
Categoría con 2 hábitos activos (target_per_week: 7 y 3) muestra denominador 10 en la semana completa. Si se hicieron 5 check-ins esa semana, muestra 50%. Categoría con un hábito creado hace 3 días y target_per_week: 7 muestra denominador ~3 (prorrateo de 3/7 días). `npx tsc --noEmit` pasa.

---

## Resumen de dependencias

| Tarea | Depende de |
|-------|-----------|
| T-01  | —          |
| T-02  | T-01       |
| T-03  | T-02       |
| T-04  | T-03       |
| T-05  | T-02, T-03 |
| T-06  | T-05       |
| T-07  | T-04, T-06 |
| T-08  | T-07       |
| T-09  | T-08       |
| T-10  | T-08, T-06 |
| T-11  | T-07, T-08 |
| T-12  | T-08       |
| T-13  | T-12, T-10 |
| T-14  | T-12       |
| T-15  | T-14, T-07 |

## Notas para el implementer

- Cada tarea asume que las anteriores de las que depende ya pasaron su criterio de hecho.
- No incluir lógica de extensiones (exportar CSV, estadísticas avanzadas, responsive móvil).
- Si una tarea falla su criterio de hecho, no avanzar a la siguiente que dependa de ella.
- El orden sugerido de ejecución lineal es: T-01 → T-02 → T-03 → T-04 → T-05 → T-06 → T-07 → T-08 → T-09 → T-10 → T-11 → T-12 → T-13 → T-14 → T-15.
