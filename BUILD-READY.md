# BUILD-READY — Habit Tracker

> Fecha de revisión: 2026-06-01  
> Documentos revisados: `spec.md` v1.0, `ADR-0001…0005`, `docs/diseño.md`,
> `docs/pruebas-manuales.md`, `plan.md`, `.env.example`, `SETUP.md`, `AGENTS.md`  
> **Estado actual: CASI LISTO — pendiente smoke-tests de agentes/skills (sección 8)**

---

## Cómo usar este documento

Marcar cada casilla como **[x]** (sí) o deja el espacio vacío **[ ]** (no).  
Si todas las casillas quedan en **[x]**, arranca el build sin dudar.  
Si alguna queda en **[ ]**, ese ítem es lo que hay que cerrar primero.

---

## 1. Spec sin huecos bloqueadores

- [x] `spec.md` tiene objetivo, scope in y scope out definidos.
- [x] Existen 31 criterios de aceptación numerados con resultado observable (URL que cambia, toast con texto exacto, valor de racha visible).
- [x] Los 7 criterios técnicos (aislamiento RLS, invalidación de token, precisión de racha…) están marcados explícitamente como no verificables desde la UI, con justificación escrita.
- [x] Todos los elementos de scope out están listados de forma explícita (mobile nativo, PWA, OAuth, gamificación, notificaciones, funciones sociales, CSV, integraciones externas).
- [x] La inconsistencia entre las rutas `/api/*` declaradas en `spec.md` y su supresión en ADR-0004 está documentada en `CONTEXT.md` y no bloquea el build: ADR-0004 registra la decisión de no implementar esas rutas en el scope inicial.

---

## 2. ADRs cerrados y consistentes con la spec

- [x] Existen 5 ADRs (`docs/adr/0001…0005`), todos con campo `Estado: Aprobado`.
- [x] ADR-0001 (racha on-the-fly) cubre los criterios 20–24 y justifica el índice `(habit_id, timestamp DESC)` necesario en T-03.
- [x] ADR-0002 (middleware `@supabase/ssr`) cubre los criterios 5 (logout invalida backend), 6 (redirect sin ver contenido) y 19 (modal sesión expirada).
- [x] ADR-0003 (client-first React Query) cubre el criterio 19 (optimistic update + rollback) y es coherente con ADR-0005 para el criterio 31.
- [x] ADR-0004 (anon key + RLS) cubre el modelo de seguridad, registra que la validación de la ventana de gracia de 12 h vive en el cliente y documenta el riesgo aceptado (bypasseable desde devtools).
- [x] ADR-0005 (Supabase Realtime) cubre el criterio 31 (≤5 segundos entre dispositivos) y es la única decisión que satisface ese criterio sin polling de 30 s.
- [x] Ningún ADR contradice a otro sin documentación explícita. La cadena de dependencias (ADR-0001 ← ADR-0003 ← ADR-0005) es consistente.

---

## 3. Diseño coherente con los ADRs y la spec

- [x] `docs/diseño.md` existe y tiene las cinco secciones requeridas: paleta de colores, tipografía, escala de espaciado, inventario de componentes e inventario de páginas.
- [x] Los 15 componentes (C-01…C-15) referencian al menos un criterio de aceptación de `spec.md`.
- [x] Las 6 páginas documentadas (`/login`, `/signup`, `/`, `/habit/[id]`, `/estadisticas`, `/archivados`) coinciden exactamente con las rutas definidas en `plan.md` y con las rutas del stack en `spec.md` (después de la supresión de `/api/*` por ADR-0004).
- [x] La paleta usa únicamente tokens nativos de Tailwind (sin valores `[]` arbitrarios ni extensiones de `tailwind.config.ts` requeridas para la paleta base).
- [x] El origen de cada componente es coherente con las restricciones de AGENTS.md: shadcn/ui para primitivas (Button, Input, Dialog, Select, Sonner, Skeleton) y `custom` para lógica de negocio (HabitCard, CheckInToggle, CategoryGroup, StreakBadge, CategoryProgressCard, EmptyState, NavBar).
- [x] Las cinco reglas de aplicación del diseño (1 botón primario por pantalla, Skeleton en lugar de spinner de página, errores en Toast salvo validación de campo, StreakBadge con 0 usa slate-400 no red-600, barra de progreso con indigo-600) no contradicen ningún criterio de la spec.

---

## 4. Cada criterio de aceptación con su prueba

- [x] Los 31 criterios manuales (spec criterios 1–31) tienen exactamente un caso de prueba asignado (TC-01…TC-31) en `docs/pruebas-manuales.md`.
- [x] Cada TC referencia exactamente un criterio de la spec; ningún TC referencia un criterio inexistente o duplicado.
- [x] Cada TC no rechazado tiene las cuatro secciones obligatorias: **Precondiciones**, **Datos de entrada**, **Pasos** numerados y **Resultado esperado** con texto observable (sin "funciona correctamente" ni expresiones ambiguas).
- [x] Los 7 criterios técnicos de la spec (§ "Pruebas técnicas fuera de QA manual") tienen un TC correspondiente (TC-32…TC-38) marcado como **RECHAZADO** con motivo escrito que explica por qué no es verificable desde la UI.
- [x] Los TCs que requieren datos precargados (TC-11, TC-20, TC-21, TC-23, TC-24, TC-28, TC-29) declaran explícitamente esa precondición.

---

## 5. plan.md con criterio de hecho por tarea

- [x] `plan.md` existe en la raíz y cubre las 14 tareas (T-01…T-14) en orden de dependencia documentado con un grafo de dependencias al final.
- [x] Las 14 tareas tienen sección "**Criterio de hecho**" redactada de forma verificable: comando a ejecutar (`npm run dev`, `npm run build`), TC que pasa manualmente, o estado observable en Supabase Studio.
- [x] Ninguna tarea tiene más de 2 dependencias de código directas (cumple la regla de `AGENTS.md`).
- [x] Las tareas referencian los ADRs y TCs que aplican (T-02 → ADR-0002/0003/0004; T-10 → TC-13…19; T-14 → ADR-0005, TC-31).
- [x] T-03 (esquema SQL) es independiente del código y su criterio de hecho es verificable en Supabase Studio (tablas, RLS y Realtime activados).
- [x] El orden recomendado de ejecución al pie de `plan.md` es coherente con el grafo de dependencias.

---

## 6. .env.example presente y documentado

- [x] El archivo `.env.example` existe en la raíz del repositorio.
- [x] Documenta `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY` con instrucciones de dónde obtener cada valor (Supabase Dashboard → Settings → API).
- [x] Distingue entre variables de desarrollo (`.env.local`, ignorada por git) y producción (Vercel Environment Variables, sin archivo local).
- [x] No contiene credenciales reales: usa únicamente placeholders documentados (`xxxxxxxxxxxxxxxxxxxx`, `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.ejemplo...`).
- [x] Incluye una sección de extensión futura (comentada) para variables no definidas en el scope inicial.

---

## 7. SETUP.md presente

- [x] El archivo `SETUP.md` existe en la raíz del repositorio.
- [x] Documenta prerrequisitos de herramientas locales (Node.js ≥18.18, npm 9.x, Git) con comando de verificación para cada uno.
- [x] Explica cómo crear los dos proyectos Supabase (dev y prod) y dónde obtener `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY` (ruta exacta en el Dashboard).
- [x] Explica cómo crear `.env.local` a partir de `.env.example` con el comando para Windows (`Copy-Item`).
- [x] Explica cómo configurar las variables de producción en Vercel.
- [x] Termina con un checklist de nueve ítems verificables antes de lanzar T-01.

---

## 8. Agentes y skills con smoke-test pasado

Los archivos existen; confirmar que responden correctamente antes de usarlos en el build.

**Agentes** (`.claude/agents/`):

- [x] `arquitecto.md` existe.
- [x] `diseñador.md` existe.
- [x] `implementer.md` existe.
- [x] `qa.md` existe.
- [x] `reviewer.md` existe.

**Skills** (`.claude/skills/`):

- [x] `nueva-prueba-manual/SKILL.md` existe.
- [x] `nuevo-adr/SKILL.md` existe.

**Smoke-test de comportamiento** (marcar tras ejecutar en una sesión de Claude Code):

- [ ] El agente `implementer` invocado con "implementar tarea T-01" lee `spec.md` y `AGENTS.md` antes de proponer acciones, y pide aprobación previa antes de tocar código.
- [ ] El skill `nueva-prueba-manual` produce una prueba con ID correlativo al último TC existente, referencia exacta a un criterio de `spec.md`, pasos sin ambigüedad y resultado esperado observable.
- [ ] El skill `nuevo-adr` produce un ADR con campo `Estado`, al menos una alternativa real con trade-offs y al menos una consecuencia negativa explícita.

**Nota**: No hay log persistido de ejecuciones anteriores de smoke-test. Estos tres ítems deben marcarse manualmente tras una ejecución en sesión.

---

## 9. Repositorio en gitflow con develop al día

- [x] Existen las ramas `main` y `develop` en el repositorio (locales y remotas).
- [x] Las ramas de trabajo creadas siguen la convención de `AGENTS.md`: `feat/`, `fix/`, `docs/`, `chore/`.
- [x] `develop` contiene todos los entregables de la fase de documentación: `spec.md`, los 5 ADRs, `docs/diseño.md`, `docs/pruebas-manuales.md`, `plan.md`, los 5 agentes y los 2 skills.
- [x] No hay ramas de trabajo abiertas con trabajo pendiente de mergear a `develop`, excepto `docs/build-ready` (la rama actual).
- [x] `develop` no tiene divergencia no resuelta con `main` más allá de los commits de la fase de documentación (18 commits adelante de `main`; divergencia esperada y aceptada hasta el primer release).
- [ ] Esta rama (`docs/build-ready`) ha sido mergeada a `develop` tras aprobar el checklist completo.

---

## Resumen ejecutivo

| # | Ítem | Estado |
|---|---|---|
| 1 | Spec sin huecos bloqueadores | **SÍ** |
| 2 | ADRs cerrados y consistentes con la spec | **SÍ** |
| 3 | Diseño coherente con los ADRs | **SÍ** |
| 4 | Cada criterio con su prueba manual | **SÍ** |
| 5 | plan.md con criterio de hecho por tarea | **SÍ** |
| 6 | .env.example presente y documentado | **SÍ** |
| 7 | SETUP.md presente | **SÍ** |
| 8 | Agentes y skills con smoke-test pasado | **PENDIENTE — ejecutar en sesión** |
| 9 | Repo en gitflow con develop al día | **PARCIAL — mergear esta rama** |

**Único bloqueador real**: sección 8 (smoke-tests de agentes/skills, sin log persistido).  
La sección 9 se cierra en <5 min mergeando `docs/build-ready` → `develop` una vez aprobado el checklist.
