# ADR-0003 — Frontera cliente/servidor: Client-first con React Query

- **Estado**: Aprobado
- **Fecha**: 28 mayo 2026
- **Decidido por**: Bryan/Dev18

---

## Contexto

Next.js 15 App Router permite un espectro de estrategias de rendering: desde Server Components puros (datos en el HTML inicial, sin JS de fetching en el cliente) hasta Client Components puros (datos fetched post-hydration). La elección afecta el tiempo de primera carga, la experiencia del toggle de check-in (la acción más frecuente del producto), y la complejidad de implementar optimistic updates con rollback (criterio 19).

La spec requiere:
- Toggle de check-in que se siente instantáneo con rollback en error y toast detallado (criterio 19).
- Sincronía de ≤5 segundos entre dispositivos (criterio 31, resuelta en ADR-0005).
- Interfaz desktop-first; la latencia de primera carga no es un KPI crítico.

Drivers relevantes:
- No introducir dependencias de backend adicionales.
- TypeScript estricto.
- El toggle de check-in es la acción central del producto — su UX tiene prioridad sobre la velocidad de primera carga.

---

## Decisión

**Client-first con TanStack Query v5 (React Query)**. Los datos del dashboard y las vistas se obtienen desde Client Components usando hooks de React Query. Las mutaciones (toggle check-in, CRUD de hábitos) usan `useMutation` con `onMutate` para optimistic updates y `onError` para rollback. El HTML inicial se sirve con skeletons; los datos se hidratan en el cliente post-mount.

---

## Alternativas consideradas

### Alternativa B — Server Components para reads + Server Actions para mutations

Los datos del dashboard se cargan en Server Components (sin JS de fetching en el cliente). Las mutaciones usan Server Actions de Next.js 15 con `revalidatePath` para invalidar el cache. Los optimistic updates usan `useOptimistic` de React 19.

**Trade-offs:**

| Campo | Detalle |
|---|---|
| **Requiere** | `use server` en funciones de mutación; `revalidatePath` después de cada mutación; `useOptimistic` de React 19 para feedback visual; `@supabase/ssr` para leer sesión en Server Components |
| **Impacto en rendimiento** | First Contentful Paint más rápido: datos en el HTML inicial. Pero `revalidatePath` invalida el cache de Next.js y dispara un re-render completo de la ruta (~300–500ms adicionales por toggle) — latencia perceptible con 200 hábitos |
| **Complejidad de desarrollo** | Media-alta para el UX del toggle. `useOptimistic` cubre el caso básico, pero el rollback en error es más manual que en React Query. Debugging de Server Actions menos ergonómico (sin DevTools equivalentes) |
| **Seguridad** | El `service_role` key de Supabase nunca llega al cliente — mayor superficie de seguridad que la Alternativa A |

**Por qué no se eligió**: El `revalidatePath` post-toggle genera un re-render completo de la ruta que degrada la experiencia en el dashboard con muchos hábitos. El toggle de check-in es la acción más frecuente del producto; su latencia perceptible es el trade-off más costoso para el usuario. `useOptimistic` de React 19 no tiene paridad de ergonomía con React Query para el caso de rollback con toast detallado.

### Alternativa C — Híbrido: Server Components para carga inicial + React Query para reactividad

Server Components cargan el estado inicial; React Query toma el control post-hydration usando `dehydrate`/`HydrationBoundary`.

**Por qué no se eligió**: El patrón de hidratación compartida entre Server Components y React Query es el más complejo de los tres y el menos documentado para Next.js 15 específicamente. Introduce dos superficies de fetching que deben mantenerse en sincronía. El beneficio de rendimiento en primera carga no justifica la complejidad adicional para un producto desktop-first donde la velocidad de primera carga no es el KPI principal.

---

## Consecuencias

### Positivas

- Los optimistic updates del toggle de check-in son nativos en React Query (`onMutate`/`onError`/`onSettled`): la UI responde de forma instantánea y revierte automáticamente si el servidor falla.
- React Query DevTools acelera el debugging en desarrollo: el estado del cache, las queries en vuelo y los errores son inspeccionables visualmente.
- El patrón client-first es el más documentado para la combinación Next.js + Supabase; reduce el tiempo de onboarding de developers nuevos.
- Cache de React Query deduplica requests: si el usuario navega entre vistas, los datos ya cargados no se re-fetchen innecesariamente.

### Negativas (trade-offs aceptados)

- **First Contentful Paint más lento**: el HTML inicial contiene skeletons, no datos. Los datos se cargan post-hydration (~200–400ms adicionales en conexiones lentas). Aceptado porque el target es desktop con conexión estable.
- **`anon key` de Supabase en el bundle del cliente**: es el modelo estándar de Supabase con RLS, pero implica que la clave pública es inspeccionable en el navegador. La seguridad real depende de que RLS esté correctamente configurada — cualquier gap en RLS expone datos directamente, sin capa servidor como intermediario.
- **Tamaño del bundle aumenta**: TanStack Query v5 añade ~13KB gzip al bundle del cliente. Aceptable para desktop-first.