# SETUP — Pre-requisitos antes de ejecutar T-01 y T-02

Esta guía cubre todo lo que un desarrollador nuevo necesita tener listo antes de
tocar código. Sigue los pasos en orden; al final hay un checklist que confirma
que T-01 puede ejecutarse sin interrupciones.

---

## 1. Herramientas locales

| Herramienta | Versión mínima | Verificación |
|---|---|---|
| Node.js | 18.18.0 | `node -v` |
| npm | 9.x | `npm -v` |
| Git | cualquier reciente | `git --version` |

Next.js 15 requiere Node 18.18+. Si usas `nvm`, ejecuta `nvm use 20` para
asegurarte de estar en una versión LTS reciente.

---

## 2. Cuenta Supabase

1. Ve a [supabase.com](https://supabase.com) y crea una cuenta (o inicia sesión).
2. Necesitarás **dos proyectos**: uno para desarrollo local y otro para producción.
   Puedes crearlos en el mismo `organization` de tu cuenta gratuita.

---

## 3. Crear el proyecto Supabase de desarrollo

1. En el Dashboard de Supabase → **New project**.
2. Nombre sugerido: `habit-tracker-dev`.
3. Elige la región más cercana a tu ubicación (ej. `South America (São Paulo)`).
4. Genera y guarda la **Database Password** — la necesitarás si alguna vez
   conectas un cliente SQL directo; no la uses en `.env.local`.
5. Espera a que el proyecto termine de provisionar (~1 min).

---

## 4. Crear el proyecto Supabase de producción

Repite el paso 3 con nombre `habit-tracker-prod`.

> Mantener dos proyectos separados evita que migraciones de esquema o seeds de
> prueba afecten datos reales. Las tablas del esquema (T-03) se ejecutan en
> ambos proyectos de forma independiente.

---

## 5. Dónde obtener cada variable de entorno

### `NEXT_PUBLIC_SUPABASE_URL`

Ruta en el Dashboard de Supabase:
```
Settings → API → Project URL
```
Ejemplo de valor: `https://abcdefghijklmnop.supabase.co`

### `NEXT_PUBLIC_SUPABASE_ANON_KEY`

Ruta en el Dashboard de Supabase:
```
Settings → API → Project API Keys → anon / public
```
Es la clave larga que empieza con `eyJ...`. **Es una clave pública** — el
navegador la verá. La seguridad depende de que RLS esté habilitado en todas
las tablas (se activa en T-03).

---

## 6. Crear `.env.local` para desarrollo

En la raíz del repositorio, copia `.env.example` a `.env.local`:

```powershell
Copy-Item .env.example .env.local
```

Luego edita `.env.local` y reemplaza los valores de ejemplo con los del proyecto
`habit-tracker-dev`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://<tu-project-id>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ<resto-de-tu-anon-key>
```

Reglas importantes:
- `.env.local` está en `.gitignore` — nunca lo commitees.
- El prefijo `NEXT_PUBLIC_` hace que la variable sea accesible en el navegador.
  No añadas secretos reales con ese prefijo.

---

## 7. Variables de producción en Vercel

Cuando llegue el momento del deploy (fuera de T-01/T-02):

1. En Vercel: Dashboard del proyecto → **Settings → Environment Variables**.
2. Añadir `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY` con los
   valores del proyecto `habit-tracker-prod`.
3. Alcance: marcar **Production** (y opcionalmente **Preview**).

No hay que tocar Vercel para ejecutar T-01 o T-02 localmente.

---

## 8. Extensión (alcance futuro)

El plan actual no incluye integraciones con servicios externos (spec.md §Scope).
Cuando se diseñe la extensión, este documento se actualizará con:

- Qué cuentas externas crear.
- Cómo obtener cada clave.
- Qué variables añadir a `.env.local` y a Vercel.

Por ahora no se necesita ninguna cuenta externa adicional.

---

## Checklist — "T-01 puede ejecutarse"

Marca cada ítem antes de lanzar `npx create-next-app@latest`:

### Entorno local
- [ ] `node -v` devuelve 18.18.0 o superior
- [ ] `npm -v` devuelve 9.x o superior
- [ ] El repositorio está clonado y el directorio de trabajo es la raíz del repo

### Supabase dev
- [ ] El proyecto `habit-tracker-dev` existe en Supabase y está en estado **Active**
- [ ] Tengo la **Project URL** del proyecto dev a mano
- [ ] Tengo la **anon key** del proyecto dev a mano

### Archivo de configuración local
- [ ] `.env.local` existe en la raíz del repo (copiado de `.env.example`)
- [ ] `NEXT_PUBLIC_SUPABASE_URL` en `.env.local` contiene la URL real del proyecto dev
- [ ] `NEXT_PUBLIC_SUPABASE_ANON_KEY` en `.env.local` contiene la anon key real del proyecto dev
- [ ] `.env.local` **no** está commiteado (`git status` no lo lista como tracked)

### Supabase prod (puede dejarse para después de T-02, pero conviene tenerlo listo)
- [ ] El proyecto `habit-tracker-prod` existe en Supabase y está en estado **Active**
- [ ] Tengo la URL y anon key de prod guardadas en un lugar seguro (no en el repo)

Con todos los ítems marcados, ejecutar T-01 no debería bloquearse por falta de
credenciales o configuración.
