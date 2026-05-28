# ADR-0002 — Autenticación con Middleware de Next.js y `@supabase/ssr`

- **Estado**: Aprobado
- **Fecha**: 28 mayo 2026
- **Decidido por**: Bryan/Dev18

---

## Contexto

La spec requiere que un visitante sin sesión que intenta acceder a cualquier ruta protegida sea redirigido a `/login` sin ver contenido privado (criterio 6). También requiere que el logout invalide la sesión en el backend (criterio 5) y que una sesión expirada muestre un modal específico (criterio 19).

Next.js 15 App Router ofrece tres capas donde se puede interceptar una sesión: Middleware (Edge Runtime), Server Components y Client Components. La elección de capa afecta cuándo ocurre el redirect, qué tan expuesto queda el contenido HTML antes de que el guard actúe, y cómo se gestiona el ciclo de vida del token JWT de Supabase (expiración y refresh).

Drivers relevantes:
- Stack fijo: Next.js 15 App Router + Supabase Auth.
- No se permite OAuth ni dependencias de auth adicionales.
- El redirect debe ocurrir antes de que el contenido llegue al navegador.

---

## Decisión

**Usar `middleware.ts` de Next.js con `@supabase/ssr`** (`createServerClient`) para leer y refrescar el token desde cookies HttpOnly en cada request. El Middleware redirige a `/login` si no hay sesión válida. El cliente usa `createBrowserClient` del mismo paquete para operaciones autenticadas desde el browser.

---

## Alternativas consideradas

### Alternativa B — Guards client-side en layout/componentes

No se usa Middleware. Cada layout o página protegida verifica `supabase.auth.getSession()` al montar el Client Component y redirige con `router.push('/login')` si no hay sesión. El token se gestiona en `localStorage` (comportamiento default de `@supabase/js` sin SSR).

**Trade-offs:**

| Campo | Detalle |
|---|---|
| **Requiere** | Solo `@supabase/js` estándar; un hook `useAuth()` o un layout con guard |
| **Impacto en rendimiento** | Flash of Unauthenticated Content (FOUC): el servidor envía el HTML de la página protegida; el guard redirige ~200–400ms después del hydration. El contenido HTML ya llegó al navegador |
| **Complejidad de desarrollo** | Baja inicialmente. Alta en mantenimiento: cada ruta nueva necesita su guard; fácil de omitir en rutas anidadas |
| **Seguridad** | Token en `localStorage` es accesible desde JavaScript — vulnerable a XSS. El criterio 6 ("sin ver contenido privado") no se cumple de forma estricta: el HTML de la página protegida se envía antes del redirect |

**Por qué no se eligió**: Viola el criterio 6 de la spec de forma técnica (el HTML llega al cliente antes del redirect). El token en `localStorage` introduce una superficie de ataque evitable. El mantenimiento de guards por componente es propenso a omisiones en rutas nuevas.

### Alternativa C — Middleware Edge con verificación JWT manual (`jose`)

Middleware que valida el JWT de Supabase directamente usando la librería `jose` y el JWT secret, sin usar `@supabase/ssr`. RLS en Supabase actúa como única barrera de datos.

**Por qué no se eligió**: Sin el helper oficial, el Middleware no puede refrescar tokens expirados. Usuarios con sesiones próximas a expirar ven errores 401 inesperados. Manejar el ciclo de vida del JWT manualmente añade complejidad sin beneficio neto sobre la Alternativa A, que usa el mismo mecanismo pero con soporte oficial de Supabase.

---

## Consecuencias

### Positivas

- El redirect a `/login` ocurre en Edge Runtime (~1–3ms overhead) antes de que cualquier contenido del Server Component se renderice — criterio 6 garantizado.
- Las cookies HttpOnly son inaccesibles desde JavaScript del cliente, eliminando la superficie de XSS para robo de tokens.
- El refresh automático del token ocurre en el Middleware antes de que expire, sin interrupciones visibles para el usuario.
- El logout llama a `supabase.auth.signOut()`, que invalida el token en el backend de Supabase — criterio 5 cubierto.

### Negativas (trade-offs aceptados)

- **Complejidad de setup no trivial**: la integración de `@supabase/ssr` con Next.js 15 App Router requiere configurar correctamente `updateSession` en el Middleware Y el cliente de Supabase en el Server Component del root layout. Un error común (doble refresh del token) genera bucles de redirect. Estimado de setup: 3–5h con la documentación oficial.
- **Dependencia del paquete `@supabase/ssr`**: si Supabase depreca o cambia este paquete, la integración requiere migración. Es el paquete oficial recomendado, pero introduce acoplamiento al ciclo de releases de Supabase.
- **Middleware se ejecuta en Edge Runtime**: no tiene acceso a APIs de Node.js. Si en el futuro se requiere lógica de auth más compleja (ej. verificar roles en una tabla), debe hacerse en Server Components, no en el Middleware.