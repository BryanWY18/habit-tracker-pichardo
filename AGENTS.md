# AGENTS.md — Contrato del proyecto Habit Tracker

Este archivo es leido por todos los agentes y por cualquier developer que se incorpore.
Define las decisiones cerradas por contrato y el espacio de libertad restante.

---

## Stack

- **Framework**: Next.js 15 con App Router (no Pages Router).
- **Base de datos y auth**: Supabase (Postgres + Auth + Storage).
- **Deploy**: Vercel.
- **Lenguaje**: TypeScript en modo estricto (`strict: true`).
- **Estilos**: Tailwind CSS. Sin librerías de componentes externas pesadas.
- **Data fetching**: React Query (o SWR). Cliente-first con Client Components.

---

## Convenciones de TypeScript

- `strict: true` siempre activo; no se desactivan flags de strictness.
- Prohibido el uso de `any` sin comentario que explique por que es inevitable.
- Tipos globales del dominio en `src/types/`; no se repiten tipos inline entre archivos.
- Nombres de tipos e interfaces en PascalCase. Variables y funciones en camelCase.
- Funciones de utilidad puras en `src/lib/`; sin efectos secundarios ocultos.

---

## Estructura de carpetas esperada

```
src/
  app/
    (auth)/
      login/
      signup/
    (protected)/
      page.tsx              # dashboard — lista de habitos por categoria
      habit/[id]/           # detalle y edicion del habito
      estadisticas/         # progreso por categoria
      archivados/           # habitos archivados / historicos
    api/
      habits/
      checkins/
      categories/
    layout.tsx
  components/               # componentes reutilizables
  lib/
    supabase/               # cliente Supabase y helpers
  types/                    # tipos del dominio
  hooks/                    # custom hooks (React Query, toggles, etc.)
```

No se crean carpetas fuera de esta estructura sin decision documentada en CONTEXT.md.

---

## Politica de commits

- Un commit por unidad funcional. Prohibido el commit "implement everything".
- Formato: `tipo(scope): descripcion en imperativo, minusculas, sin punto final`.
- Tipos validos: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`.
- El cuerpo del commit es opcional; usarlo solo cuando el "por que" no es obvio.

---

## Flujo git (Gitflow)

- `main`: rama estable. Solo recibe merges desde `develop` en releases.
- `develop`: rama de integracion. Todo trabajo converge aqui.
- Ramas de trabajo tipadas: `feat/`, `docs/`, `chore/`, `fix/` — se crean desde `develop`.
- Toda rama tipada se mergea a `develop` al completarse. Nunca directamente a `main`.
- Nada se queda sin commitear al final de una sesion de trabajo.

---

## Regla de CONTEXT.md

Cualquier edicion manual de codigo — es decir, no generada por un agente siguiendo
un plan aprobado — debe registrarse en `CONTEXT.md` con:

1. Fecha.
2. Archivo(s) modificados.
3. Justificacion de por que no paso por el flujo normal de agente/plan.

---

## Reglas operativas

- No se escribe codigo sin plan aprobado por el usuario.
- Los agentes no toman decisiones de arquitectura no contempladas en `spec.md` sin
  consultar primero. Si la spec no cubre un caso, el agente lo escala; no asume.
- El scope del proyecto esta cerrado en `spec.md`. Los agentes no implementan
  funcionalidad fuera de ese scope (movil premium, PWA, gamificacion, notificaciones,
  funciones sociales, integraciones externas).

---

## Prohibiciones explicitas

- `any` sin comentario justificativo.
- Librerias de componentes pesadas: Material UI, Chakra UI, Mantine, shadcn en escala,
  u otras que impongan sistema de diseno propio sobre Tailwind.
- Tests automatizados (unitarios, integracion, e2e): fuera del alcance de este proyecto.
  La verificacion de calidad se hace mediante pruebas manuales documentadas en el plan de QA.
- Commits que agrupen mas de una unidad funcional.
- Ramas que mergeen directo a `main` sin pasar por `develop`.
