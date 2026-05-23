# Contrato de agentes y colaboradores

## Stack
- Next.js 15 con App Router.
- Supabase: Postgres, Auth y Storage.
- Vercel para despliegue.
- TypeScript en modo estricto.
- Tailwind CSS para estilos.

## Convenciones de TypeScript
- Estricto habilitado (`strict: true`).
- Evitar `any`; solo usarlo con justificación documentada y aprobada.
- Usar tipos explícitos en APIs públicas, props y retorno de funciones.
- Preferir tipos de unión, `unknown` y genéricos bien acotados sobre castings inseguros.

## Estructura de carpetas esperada
- `app/` para rutas y layouts de Next.js.
- `src/` para lógica compartida si se usa.
- `components/` o `app/components/` para componentes reutilizables ligeros.
- `lib/`, `services/` o `utils/` para abstracciones de negocio.
- `styles/` o `app/styles/` para estilos globales y configuración de Tailwind.

## Política de commits
- Commits atómicos y verificables.
- Un solo propósito funcional por commit.
- Mensajes claros: tipo breve de cambio y contexto.
- No se permite "implement everything" ni cambios agrupados sin propósito.

## Flujo git
- `main`: rama estable.
- `develop`: rama de integración.
- Ramas tipadas por unidad de trabajo:
  - `feat/` para nuevas funcionalidades.
  - `fix/` para correcciones.
  - `docs/` para documentación.
  - `chore/` para tareas de mantenimiento.
- Todas las ramas tipadas se mergean a `develop`.
- `main` solo recibe merges desde `develop` cuando está listo para release.

## Regla de CONTEXT.md
- Cualquier edición manual de código no generada por un agente debe documentarse en `CONTEXT.md`.
- La documentación debe incluir la justificación del cambio.

## Prohibiciones explícitas
- No usar `any` sin justificación documentada.
- No incorporar librerías de componentes pesadas como Material UI o Chakra UI.
- No escribir tests automatizados dentro de este alcance contractual.
- No modificar `main` directamente para trabajo en progreso.
- No emplear hacks de tipado o bypasses de lint sin una nota clara en `CONTEXT.md`.
