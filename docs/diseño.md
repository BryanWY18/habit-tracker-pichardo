# Sistema Visual — Habit Tracker

---

## 1. Paleta de colores

Todos los valores corresponden a tokens de la paleta estándar de Tailwind CSS.
No se requiere extender `tailwind.config.ts` para la paleta base.

| Rol | Token Tailwind | Hex | Justificación funcional |
|---|---|---|---|
| **Primario** | `indigo-600` | `#4F46E5` | Acción principal: botones CTA, toggle activo, links de navegación. Suficiente peso visual para jerarquía sin ser agresivo en pantalla desktop. |
| **Fondo** | `slate-50` | `#F8FAFC` | Superficie base. Blanco levemente frío que reduce la fatiga visual en sesiones largas de escritorio. |
| **Texto** | `slate-900` | `#0F172A` | Cuerpo, labels, títulos. Contraste AA sobre `slate-50`: ratio ~16:1. |
| **Error** | `red-600` | `#DC2626` | Validaciones negativas, mensajes de fallo, rollback visible. |
| **Éxito** | `emerald-600` | `#059669` | Check-in completado, racha activa, confirmación de acción exitosa. |

**Variantes hover/disabled (Tailwind nativo):**
- Primario hover: `indigo-700` (`#4338CA`)
- Primario disabled: `indigo-600/50` + `cursor-not-allowed`
- Error hover: `red-700`
- Éxito hover: `emerald-700`
- Texto secundario (metadatos, placeholders): `slate-500`
- Bordes y divisores: `slate-200`
- Fondo de cards/superficies elevadas: `white` (`#FFFFFF`)

---

## 2. Tipografía

Una sola familia. El system font stack cubre Inter, SF Pro y Segoe UI según
el SO del usuario sin carga de red adicional. No se necesita `next/font` ni
configuración extra.

**Familia:** `font-sans` → `ui-sans-serif, system-ui, -apple-system, sans-serif`

No se usa una segunda familia. Para jerarquía se usa peso y tamaño, no familia.

### Escala

| Paso | Clase Tailwind | px | Peso(s) | Casos de uso |
|---|---|---|---|---|
| XS | `text-xs` | 12px | `font-medium` | Labels de categoría, metadatos de racha, timestamps |
| SM | `text-sm` | 14px | `font-normal`, `font-medium` | Descripciones de hábito, texto de formulario secundario, toasts |
| Base | `text-base` | 16px | `font-normal`, `font-medium` | Cuerpo principal, nombre de hábito en listado, inputs |
| LG | `text-lg` | 18px | `font-semibold` | Títulos de sección, nombre de categoría en grupo |
| XL | `text-xl` | 20px | `font-semibold` | Título de página (`/estadisticas`, `/archivados`) |
| 2XL | `text-2xl` | 24px | `font-semibold` | Título principal del dashboard, títulos de auth |

**Pesos en uso:** `font-normal` (400), `font-medium` (500), `font-semibold` (600).
`font-bold` (700) queda reservado para valores numéricos de racha destacados.

---

## 3. Escala de espaciado

Subset de la escala estándar de Tailwind. Sin valores arbitrarios `[]`.

| Token | px | Uso típico |
|---|---|---|
| `2` | 8px | Gap interno entre icono y texto dentro de un botón o badge |
| `4` | 16px | Padding interno de inputs, botones y badges; gap entre elementos en fila |
| `6` | 24px | Padding interno de cards (HabitCard, CategoryGroup header) |
| `8` | 32px | Separación vertical entre grupos de categoría en el dashboard |
| `12` | 48px | Padding horizontal del contenedor principal de página |
| `16` | 64px | Separación entre secciones mayores (`/estadisticas`, `/archivados`) |

**Contenedor de página:** `max-w-4xl mx-auto px-12` — ancho máximo 896px,
centrado, con padding lateral de 48px. Adecuado para desktop sin que el
contenido se estire en pantallas anchas.

**Radios:** `rounded-md` (6px) para inputs y botones; `rounded-lg` (8px) para
cards y modales.

**Sombras:** `shadow-sm` para cards en reposo; `shadow-md` para modales y
dropdowns; sin sombra en elementos inline.

---

## 4. Inventario de componentes

Trazabilidad: cada componente referencia los criterios de aceptación de
`spec.md` que lo justifican.

| # | Componente | Estados requeridos | Origen | Nota |
|---|---|---|---|---|
| C-01 | `Button` | default, hover, disabled, loading | `shadcn/ui` | Variantes: `primary`, `secondary`, `destructive`. Loading muestra spinner inline. |
| C-02 | `Input` | default, focus, error, disabled | `shadcn/ui` | Incluye label flotante o label superior + mensaje de error debajo. Criterios 1–4, 8. |
| C-03 | `HabitCard` | default, completado, archivado | `custom` | Contiene nombre, descripción truncada, badge de racha, `CheckInToggle`. Criterio 12. |
| C-04 | `CheckInToggle` | idle, completado, loading, error | `custom` | Botón único de toggle con estado optimista. Rollback visual en error. Criterios 13–19. |
| C-05 | `CategoryGroup` | default, vacío | `custom` | Agrupa `HabitCard`s bajo un header con nombre de categoría y contador. Criterio 12. |
| C-06 | `HabitForm` | default, submitting, error por campo | `custom` | Formulario de crear/editar hábito. Usa `Input`, `Select`, `Textarea` de shadcn/ui. Criterios 7–9. |
| C-07 | `ConfirmDialog` | idle, confirmando | `shadcn/ui` (`AlertDialog`) | Confirmación de desmarcar check-in ("¿Desmarcar este check-in?"). Criterio 17. |
| C-08 | `Toast` | info, error, éxito | `shadcn/ui` (`Sonner`) | Mensajes detallados de fallo de toggle, errores de red. Criterio 19. |
| C-09 | `SessionExpiredModal` | visible, cerrando | `shadcn/ui` (`Dialog`) | Modal específico para 401: "Sesión expirada — inicia sesión nuevamente". Criterio 19. |
| C-10 | `StreakBadge` | activa (>0), inactiva (0) | `custom` | Badge inline con ícono y valor numérico. Color `emerald-600` si activa, `slate-400` si 0. Criterios 20–24. |
| C-11 | `CategoryProgressCard` | default, vacío | `custom` | Muestra porcentaje semanal y racha promedio de la categoría. Criterios 28–29. |
| C-12 | `Skeleton` | loading | `shadcn/ui` | Placeholder durante fetch inicial de hábitos y estadísticas. |
| C-13 | `EmptyState` | visible | `custom` | Mensaje + CTA cuando no hay hábitos, no hay archivados, o categoría vacía. |
| C-14 | `NavBar` | autenticado | `custom` | Logo/nombre app + enlace a `/estadisticas` + botón logout. Fija en top. Criterios 3, 5. |
| C-15 | `Select` | default, open, disabled, error | `shadcn/ui` | Selector de frecuencia y categoría en `HabitForm`. Criterios 7, 25–27. |

---

## 5. Estructura de páginas

---

### `/login` — Inicio de sesión

**Zonas:** Página centrada verticalmente. Un único bloque central de ancho
fijo (`max-w-sm`) con: título "Iniciar sesión" (`text-2xl font-semibold`),
dos campos `Input` (email, contraseña), botón primario "Iniciar sesión" a
ancho completo, y enlace de texto hacia `/signup`. Sin navbar. Sin footer.

**Componentes:** `Input` (×2), `Button` (primary), `Toast` (error de
credenciales).

**Estado vacío / error:** Si las credenciales son incorrectas, `Toast` de
error con mensaje "Email o contraseña inválidos" (criterio 4). El formulario
no se limpia.

---

### `/signup` — Registro

**Zonas:** Idéntica estructura a `/login`. Título "Crear cuenta". Campos:
email, contraseña. Botón primario "Registrarse". Enlace hacia `/login`.
Sin navbar.

**Componentes:** `Input` (×2), `Button` (primary), `Toast` (email ya
registrado).

**Estado vacío / error:** Si el email ya existe, `Toast` con "El email ya
está registrado" (criterio 2). Contraseña mínimo 8 caracteres validada
en cliente antes de submit.

---

### `/` — Dashboard

**Zonas:** `NavBar` fija en la parte superior (altura 56px). Contenedor
central `max-w-4xl`. Encabezado de página con título "Mis hábitos" y botón
secundario "Nuevo hábito" alineado a la derecha. Debajo, lista vertical de
`CategoryGroup`, cada uno colapsable, ordenados alfabéticamente. Dentro de
cada grupo, lista vertical de `HabitCard`. Botón flotante o inline al final
de la lista para agregar hábito (alternativo al encabezado si la lista es larga).

**Componentes:** `NavBar`, `CategoryGroup`, `HabitCard`, `CheckInToggle`,
`StreakBadge`, `ConfirmDialog`, `Toast`, `SessionExpiredModal`, `Skeleton`,
`EmptyState`.

**Estado vacío:** Si el usuario no tiene hábitos, `EmptyState` con texto
"Aún no tienes hábitos" y botón "Crear primer hábito" que abre `HabitForm`.

---

### `/habit/[id]` — Detalle y edición de hábito

**Zonas:** `NavBar`. Contenedor central `max-w-2xl`. Encabezado con nombre
del hábito (`text-2xl`) y botón destructivo "Eliminar hábito" alineado a la
derecha. Debajo, `HabitForm` prellenado con los datos actuales. Botón primario
"Guardar cambios" y botón secundario "Cancelar" (vuelve al dashboard).
Sección separada por divisor: historial de check-ins de los últimos 30 días
en formato lista compacta (fecha + estado).

**Componentes:** `NavBar`, `HabitForm`, `Button` (×3), `StreakBadge`,
`ConfirmDialog` (confirmación de eliminación), `Toast`, `Skeleton`.

**Estado vacío:** Si el hábito no tiene check-ins, sección de historial
muestra `EmptyState` compacto "Sin check-ins registrados".

---

### `/estadisticas` — Progreso por categoría

**Zonas:** `NavBar`. Contenedor central `max-w-4xl`. Título de página
"Estadísticas". Grid de 2 columnas (una por categoría activa con hábitos).
Cada celda es un `CategoryProgressCard` con: nombre de categoría, porcentaje
de cumplimiento de la semana en curso (valor numérico grande, `text-2xl
font-bold`), barra de progreso visual (div con ancho dinámico en `%`, color
`indigo-600`), y racha promedio de los últimos 14 días.

**Componentes:** `NavBar`, `CategoryProgressCard`, `Skeleton`, `EmptyState`.

**Estado vacío:** Si no hay categorías con hábitos activos, `EmptyState`
con "No hay datos esta semana" y enlace al dashboard.

---

### `/archivados` — Hábitos archivados

**Zonas:** `NavBar`. Contenedor central `max-w-4xl`. Título "Históricos".
Lista vertical de `HabitCard` en variante `archivado` (sin `CheckInToggle`,
con indicador visual "Archivado" en `slate-400`). Cada card es solo lectura:
muestra nombre, categoría, fecha de archivado y racha máxima alcanzada.
Botón secundario "Restaurar" por card (reactiva el hábito).

**Componentes:** `NavBar`, `HabitCard` (variante archivado), `Button`
(secondary, por card), `EmptyState`, `Skeleton`.

**Estado vacío:** `EmptyState` con "No hay hábitos archivados".

---

## 6. Reglas de aplicación

1. **El color primario (`indigo-600`) solo se usa en acciones que mutan
   estado o inician una navegación principal.** Para acciones secundarias
   (cancelar, volver, ver detalle) usar `Button` variante `secondary`
   con texto `slate-700` y fondo `slate-100`.

2. **Cada página tiene un único botón primario visible a la vez.** Si hay
   más de una acción principal en pantalla, una es `primary` y el resto
   `secondary`. Nunca dos botones `indigo-600` en el mismo bloque visual.

3. **El estado de carga siempre es `Skeleton`, nunca un spinner de página
   completa.** Los skeletons replican la forma del contenido real
   (mismo alto y ancho aproximado) para evitar layout shift al hidratar.

4. **Los mensajes de error van en `Toast`, no inline en la página**, salvo
   errores de validación de formulario que van debajo del campo
   correspondiente en `text-sm text-red-600`. El `Toast` de error usa
   `red-600`; el de éxito usa `emerald-600`.

5. **`StreakBadge` con valor 0 usa `slate-400`, no `red-600`.** Racha cero
   es un estado neutro (hábito recién creado o sin actividad reciente),
   no un estado de error. `red-600` se reserva para fallos del sistema,
   no para el progreso del usuario.

---
