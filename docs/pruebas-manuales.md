# Plan de pruebas manuales — Habit Tracker

> Generado desde spec.md v1.0 (17 mayo 2026).  
> Datos de prueba base: `tester01@ejemplo.com` / `Test1234!`  
> Ejecutar en orden. Cada caso asume que el anterior fue exitoso
> salvo que sus precondiciones indiquen lo contrario.

---

## Autenticación

### TC-01 — Registrar cuenta nueva con email y contraseña válidos

**Criterio**: spec.md — Autenticación #1  
**Precondiciones**: La app está en `/signup`. El email `tester01@ejemplo.com`
no existe en el sistema.  
**Datos de entrada**: Email `tester01@ejemplo.com`, contraseña `Test1234!`  
**Pasos**:
1. Navegar a `/signup`.
2. Escribir `tester01@ejemplo.com` en el campo Email.
3. Escribir `Test1234!` en el campo Contraseña.
4. Hacer clic en "Registrarse".

**Resultado esperado**: La URL cambia a `/`. El dashboard muestra el contenido
del usuario autenticado (navbar con botón logout y estado vacío de hábitos).

---

### TC-02 — Registrar con email ya existente muestra mensaje de error

**Criterio**: spec.md — Autenticación #2  
**Precondiciones**: TC-01 ejecutado. El email `tester01@ejemplo.com` ya existe.  
**Datos de entrada**: Email `tester01@ejemplo.com`, contraseña `OtraClave99!`  
**Pasos**:
1. Navegar a `/signup`.
2. Escribir `tester01@ejemplo.com` en el campo Email.
3. Escribir `OtraClave99!` en el campo Contraseña.
4. Hacer clic en "Registrarse".

**Resultado esperado**: Aparece un toast o mensaje con el texto exacto
"El email ya está registrado". La URL permanece en `/signup`. El formulario
sigue visible sin limpiarse.

---

### TC-03 — Iniciar sesión con credenciales correctas

**Criterio**: spec.md — Autenticación #3  
**Precondiciones**: TC-01 ejecutado. Sesión cerrada. La app está en `/login`.  
**Datos de entrada**: Email `tester01@ejemplo.com`, contraseña `Test1234!`  
**Pasos**:
1. Navegar a `/login`.
2. Escribir `tester01@ejemplo.com` en el campo Email.
3. Escribir `Test1234!` en el campo Contraseña.
4. Hacer clic en "Iniciar sesión".

**Resultado esperado**: La URL cambia a `/`. La navbar muestra el botón de
logout. El contenido del dashboard es visible.

---

### TC-04 — Iniciar sesión con contraseña incorrecta muestra mensaje de error

**Criterio**: spec.md — Autenticación #4  
**Precondiciones**: Sesión cerrada. La app está en `/login`.  
**Datos de entrada**: Email `tester01@ejemplo.com`, contraseña `ClaveWrong1!`  
**Pasos**:
1. Navegar a `/login`.
2. Escribir `tester01@ejemplo.com` en el campo Email.
3. Escribir `ClaveWrong1!` en el campo Contraseña.
4. Hacer clic en "Iniciar sesión".

**Resultado esperado**: Aparece un toast o mensaje con el texto exacto
"Email o contraseña inválidos". La URL permanece en `/login`. El formulario
sigue visible.

---

### TC-05 — Cerrar sesión redirige al login e impide re-acceso inmediato

**Criterio**: spec.md — Autenticación #5  
**Precondiciones**: TC-03 ejecutado. Usuario autenticado en `/`.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Hacer clic en el botón "Logout" en la navbar.
2. Tras la redirección, escribir manualmente `/` en la barra del navegador
   y presionar Enter.

**Resultado esperado**: Paso 1 — la URL cambia a `/login`; la navbar del
dashboard desaparece. Paso 2 — la app redirige de nuevo a `/login` sin mostrar
contenido del dashboard.

---

### TC-06 — Acceder a ruta protegida sin sesión redirige al login sin mostrar contenido privado

**Criterio**: spec.md — Autenticación #6  
**Precondiciones**: Sesión cerrada (TC-05 ejecutado o sesión limpia en
navegador privado).  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Con sesión cerrada, escribir directamente `<URL-base>/` en la barra
   del navegador y presionar Enter.
2. Intentar también con `<URL-base>/estadisticas`.

**Resultado esperado**: En ambos casos la URL cambia a `/login`. No se
muestra ningún dato de hábitos ni interfaz del dashboard en ningún momento
previo a la redirección.

---

## CRUD de hábitos

### TC-07 — Crear un hábito con todos los campos válidos

**Criterio**: spec.md — CRUD de hábitos #7  
**Precondiciones**: Usuario `tester01@ejemplo.com` autenticado. Dashboard vacío
o con hábitos previos.  
**Datos de entrada**: Nombre `Leer`, descripción `Leer 20 minutos cada día`,
frecuencia `Diaria`, categoría `Salud`.  
**Pasos**:
1. Navegar a `/`.
2. Hacer clic en "Nuevo hábito".
3. Escribir `Leer` en el campo Nombre.
4. Escribir `Leer 20 minutos cada día` en el campo Descripción.
5. Seleccionar frecuencia `Diaria`.
6. Seleccionar categoría `Salud`.
7. Hacer clic en "Guardar" o "Crear".

**Resultado esperado**: El formulario se cierra. En `/` aparece el grupo
`Salud` con el hábito `Leer` mostrando racha `0` y estado "no completado hoy".

---

### TC-08 — Crear hábito sin nombre muestra error de validación

**Criterio**: spec.md — CRUD de hábitos #8  
**Precondiciones**: Usuario autenticado. Formulario de creación abierto.  
**Datos de entrada**: Nombre vacío, descripción `Sin nombre`, frecuencia
`Diaria`, categoría `Salud`.  
**Pasos**:
1. Abrir el formulario de nuevo hábito.
2. Dejar el campo Nombre vacío.
3. Rellenar descripción, frecuencia y categoría con los datos de entrada.
4. Hacer clic en "Guardar" o "Crear".

**Resultado esperado**: Aparece un mensaje de error debajo del campo Nombre con
el texto exacto "El nombre es requerido". El hábito no se crea. El formulario
permanece abierto.

---

### TC-09 — Editar un hábito refleja los cambios en el listado inmediatamente

**Criterio**: spec.md — CRUD de hábitos #9  
**Precondiciones**: TC-07 ejecutado. El hábito `Leer` existe en el listado.  
**Datos de entrada**: Nuevo nombre `Leer libros`, nueva descripción
`Leer 30 minutos`.  
**Pasos**:
1. Desde `/`, hacer clic en el hábito `Leer` o su botón de edición para
   navegar a `/habit/[id]`.
2. Cambiar el nombre a `Leer libros`.
3. Cambiar la descripción a `Leer 30 minutos`.
4. Hacer clic en "Guardar cambios".

**Resultado esperado**: La app vuelve a `/` o muestra confirmación. El hábito
aparece ahora como `Leer libros` en el listado. Los check-ins previos (si los
hay) siguen presentes con sus fechas originales en el historial del hábito.

---

### TC-10 — Eliminar hábito sin racha borra el hábito y sus check-ins

**Criterio**: spec.md — CRUD de hábitos #10  
**Precondiciones**: Existe el hábito `Meditar` (creado para este caso) sin
ningún check-in registrado ni racha alcanzada.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Crear el hábito `Meditar`, frecuencia `Diaria`, categoría `Salud`,
   sin registrar check-ins.
2. Navegar a `/habit/[id]` del hábito `Meditar`.
3. Hacer clic en "Eliminar hábito".
4. Confirmar la eliminación en el diálogo.

**Resultado esperado**: La URL vuelve a `/`. El hábito `Meditar` no aparece
en el dashboard ni en `/archivados`.

---

### TC-11 — Eliminar hábito con racha lo oculta del listado y conserva histórico

**Criterio**: spec.md — CRUD de hábitos #11  
**Precondiciones**: Existe el hábito `Ejercicio` con al menos 7 check-ins
diarios consecutivos (racha ≥7 días). Requiere datos precargados en entorno
de prueba o ejecución tras 7 días de uso real.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/habit/[id]` del hábito `Ejercicio`.
2. Hacer clic en "Eliminar hábito".
3. Confirmar la eliminación en el diálogo.
4. Navegar a `/`.
5. Navegar a `/archivados`.

**Resultado esperado**: `Ejercicio` no aparece en `/`. En `/archivados`
aparece `Ejercicio` con indicador "Archivado" y su historial de check-ins
o racha máxima visible.

> ⚠️ La verificación de que las filas de `check_ins` persisten en BD
> (sin rastro en UI) cae fuera del alcance manual — ver TC-37.

---

### TC-12 — El listado agrupa hábitos por categoría con nombre, estado, racha y toggle

**Criterio**: spec.md — CRUD de hábitos #12  
**Precondiciones**: TC-07 ejecutado (o equivalente). El hábito `Leer libros`
existe en categoría `Salud`.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Localizar la sección encabezada con `Salud`.

**Resultado esperado**: La sección `Salud` muestra el hábito `Leer libros`
con: su nombre, un indicador de estado ("completado hoy" o "no completado"),
el valor de racha actual (número), y un botón de toggle de check-in accionable.

---

## Check-in

### TC-13 — Registrar check-in para hoy actualiza estado y racha

**Criterio**: spec.md — Check-in #13  
**Precondiciones**: TC-07 ejecutado. El hábito `Leer libros` no tiene check-in
para hoy.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Localizar el hábito `Leer libros` en el grupo `Salud`.
3. Hacer clic en el botón de toggle (marcar check-in).

**Resultado esperado**: El estado cambia visualmente a "completado hoy"
(ícono o color de éxito). La racha aumenta en 1. El cambio es inmediato
(actualización optimista; no requiere recargar la página).

---

### TC-14 — Registrar check-in para el día anterior dentro de la ventana de gracia

**Criterio**: spec.md — Check-in #14  
**Precondiciones**: La hora local del navegador es **antes de las 12:00 del
mediodía** del día actual. El hábito `Leer libros` no tiene check-in para
ayer ni para hoy.  
**Datos de entrada**: Fecha D-1 (ayer) seleccionada en la UI.  
**Pasos**:
1. Verificar que la hora del sistema es antes de las 12:00.
2. Navegar a `/`.
3. En el hábito `Leer libros`, usar la opción de registrar check-in para
   el día anterior (selector de fecha o botón específico si existe en la UI).
4. Confirmar la acción.

**Resultado esperado**: El check-in queda registrado. La racha se recalcula
incluyendo ayer. No aparece mensaje de error.

> ⚠️ ADR-0004: la validación de la ventana de 12h ocurre en cliente
> (TypeScript), no en servidor. El resultado en UI es el mismo, pero la
> restricción es bypasseable desde la consola del navegador con el token
> de sesión activo.

---

### TC-15 — Rechazar check-in para el día anterior fuera de la ventana de gracia

**Criterio**: spec.md — Check-in #15  
**Precondiciones**: La hora local del navegador es **después de las 12:00 del
mediodía** del día actual. El hábito `Leer libros` no tiene check-in para ayer.  
**Datos de entrada**: Intento de seleccionar fecha D-1 (ayer).  
**Pasos**:
1. Verificar que la hora del sistema es después de las 12:00.
2. Navegar a `/`.
3. En el hábito `Leer libros`, intentar usar la opción de registrar
   check-in para ayer.

**Resultado esperado**: La opción de seleccionar ayer está deshabilitada o
no visible, o la app muestra un mensaje indicando que la ventana de gracia
expiró. No se registra ningún check-in para ayer.

> ⚠️ ADR-0004: misma advertencia que TC-14.

---

### TC-16 — Segundo intento de check-in en la misma unidad temporal no genera error

**Criterio**: spec.md — Check-in #16  
**Precondiciones**: TC-13 ejecutado. El hábito `Leer libros` ya tiene
check-in para hoy y muestra estado "completado hoy".  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Hacer clic de nuevo en el toggle del hábito `Leer libros` (que ya
   está en estado "completado").
3. Observar el comportamiento de la UI durante 3 segundos.

**Resultado esperado**: No aparece toast de error. El hábito permanece en
estado "completado hoy". No se generan dos entradas duplicadas visibles en
el historial del hábito en `/habit/[id]`.

---

### TC-17 — Desmarcar check-in dentro de las 12 horas con confirmación explícita

**Criterio**: spec.md — Check-in #17  
**Precondiciones**: TC-13 ejecutado hace menos de 12 horas. El hábito
`Leer libros` tiene check-in de hoy en estado "completado".  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Hacer clic en el toggle del hábito `Leer libros` (estado: completado).
3. Leer el texto del diálogo de confirmación que aparece.
4. Hacer clic en "Confirmar" o "Sí" en el diálogo.

**Resultado esperado**: El diálogo muestra el texto exacto
"¿Desmarcar este check-in?". Tras confirmar, el estado cambia a
"no completado hoy". La racha se recalcula (puede decrecer o quedar en 0
según el historial previo).

---

### TC-18 — No es posible desmarcar un check-in después de 12 horas de su registro

**Criterio**: spec.md — Check-in #18  
**Precondiciones**: Existe un check-in registrado hace **más de 12 horas**
(ej. check-in de ayer o check-in de hoy registrado antes de las 00:00
del día actual). Requiere esperar 12 horas desde TC-13 o usar datos
precargados con fecha anterior.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/` o a `/habit/[id]` del hábito con el check-in antiguo.
2. Intentar hacer clic en el toggle o en el check-in para desmarcarlo.

**Resultado esperado**: El toggle está visualmente deshabilitado (sin
cursor de acción, estilo opaco) o aparece un mensaje indicando que no es
posible desmarcar. No se abre el diálogo de confirmación.

---

### TC-19 — Fallo de toggle muestra toast de error y revierte el estado (rollback)

**Criterio**: spec.md — Check-in #19  
**Precondiciones**: Usuario autenticado. El hábito `Leer libros` no tiene
check-in para hoy. DevTools del navegador disponibles.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Abrir DevTools → pestaña Network → activar modo **Offline**.
3. Hacer clic en el toggle del hábito `Leer libros`.
4. Observar la UI durante 5 segundos sin desactivar Offline.
5. Reactivar la conexión (desactivar Offline en DevTools).

**Resultado esperado**: Inmediatamente en el paso 3, la UI muestra
`Leer libros` como "completado" (optimistic update). Pasados 1–3 segundos
aparece un toast con mensaje de error detallado (ej. "No se pudo registrar
el check-in — error de red" o similar). El estado del hábito revierte a
"no completado hoy" (rollback visible).

---

## Racha y progreso por hábito

### TC-20 — Hábito diario muestra racha expresada en días consecutivos

**Criterio**: spec.md — Racha y progreso #20  
**Precondiciones**: El hábito `Leer libros` tiene check-ins registrados para
los últimos 7 días consecutivos incluido hoy. Requiere datos precargados o
7 días de uso real.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Localizar el hábito `Leer libros` en el grupo `Salud`.
3. Leer el valor de racha mostrado.

**Resultado esperado**: La racha muestra `7` o el texto "7 días". El valor
es un entero positivo; no aparece "semanas" ni unidades ambiguas.

---

### TC-21 — Hábito semanal muestra racha expresada en períodos de 7 días

**Criterio**: spec.md — Racha y progreso #21  
**Precondiciones**: Existe el hábito `Caminar` de frecuencia semanal
(target 3/semana) con check-ins en 3 semanas consecutivas. Requiere
datos precargados.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Localizar el hábito `Caminar`.
3. Leer el valor de racha mostrado.

**Resultado esperado**: La racha muestra `3` o el texto "3 semanas". El
formato indica períodos semanales, no días individuales.

---

### TC-22 — Hábito recién creado sin check-ins muestra racha 0

**Criterio**: spec.md — Racha y progreso #22  
**Precondiciones**: Usuario autenticado.  
**Datos de entrada**: Nombre `Correr`, frecuencia `Diaria`, categoría `Salud`.  
**Pasos**:
1. Crear el hábito `Correr` sin registrar ningún check-in.
2. Navegar a `/`.
3. Localizar el hábito `Correr` y leer el valor de racha.

**Resultado esperado**: La racha muestra `0`. El badge de racha aparece
en estado inactivo (color neutro, no rojo de error). No se muestra un número
negativo ni un mensaje de error.

---

### TC-23 — Hábito diario pierde su racha si no hay check-in al día siguiente

**Criterio**: spec.md — Racha y progreso #23  
**Precondiciones**: El hábito `Leer libros` tenía racha activa (valor ≥1)
con el último check-in registrado **ayer**. Hoy no se ha registrado check-in.
Ejecutar este caso después de la medianoche sin check-in de hoy.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Verificar que `Leer libros` tenía racha ≥1 ayer.
2. No registrar check-in para hoy.
3. Al día siguiente (o tras medianoche), navegar a `/`.
4. Localizar `Leer libros` y leer su racha.

**Resultado esperado**: La racha muestra `0`. El badge de racha aparece
en estado inactivo.

---

### TC-24 — Hábito semanal pierde su racha si pasan más de 7 días sin check-in

**Criterio**: spec.md — Racha y progreso #24  
**Precondiciones**: El hábito semanal `Caminar` tenía racha activa. El
último check-in tiene fecha de hace **más de 7 días**. Requiere datos
precargados con fecha anterior.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Verificar en `/habit/[id]` de `Caminar` que el último check-in es de
   hace más de 7 días.
2. Navegar a `/`.
3. Localizar `Caminar` y leer su racha.

**Resultado esperado**: La racha muestra `0`. El badge aparece en estado
inactivo.

---

## Categorías

### TC-25 — Las categorías del sistema están disponibles al crear un hábito

**Criterio**: spec.md — Categorías #25  
**Precondiciones**: Usuario autenticado. Formulario de creación de hábito
abierto.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Hacer clic en "Nuevo hábito".
2. Hacer clic en el selector de Categoría.
3. Observar las opciones disponibles sin haber creado ninguna categoría
   personalizada.

**Resultado esperado**: El selector muestra al menos las categorías
predefinidas del sistema (ej. `Salud` u otras definidas en la app) sin
necesidad de crearlas previamente. Son seleccionables.

---

### TC-26 — Crear categoría personalizada y verla disponible en el selector

**Criterio**: spec.md — Categorías #26  
**Precondiciones**: Usuario autenticado. La categoría `Aprendizaje` no existe.  
**Datos de entrada**: Nombre `Aprendizaje`.  
**Pasos**:
1. Navegar al flujo de creación de categoría personalizada (desde
   "Nuevo hábito" u opción de gestión de categorías en la UI).
2. Escribir `Aprendizaje` en el campo de nombre.
3. Guardar la categoría.
4. Abrir el formulario de nuevo hábito.
5. Hacer clic en el selector de Categoría.

**Resultado esperado**: `Aprendizaje` aparece en el selector junto a las
categorías predefinidas del sistema. Puede seleccionarse para asignarla
a un hábito.

---

### TC-27 — Cada hábito tiene exactamente una categoría; no es posible dejar el campo vacío

**Criterio**: spec.md — Categorías #27  
**Precondiciones**: TC-09 ejecutado. El hábito `Leer libros` tiene categoría
`Salud`.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/habit/[id]` del hábito `Leer libros`.
2. Observar el campo Categoría en el formulario de edición.
3. Intentar deseleccionar la categoría actual o seleccionar ninguna.

**Resultado esperado**: El campo Categoría muestra exactamente `Salud`
seleccionada. No es posible guardar el hábito con el campo de categoría
vacío (botón deshabilitado o mensaje de validación). No es posible
seleccionar más de una categoría simultáneamente.

---

## Vista de progreso por categoría

### TC-28 — Vista de estadísticas muestra porcentaje de cumplimiento semanal por categoría

**Criterio**: spec.md — Vista de progreso #28  
**Precondiciones**: Existen al menos 2 hábitos activos en `Salud` con
`target_per_week` conocido (ej. `Leer libros` diario = 7/semana, `Correr`
diario = 7/semana; total esperado = 14 check-ins/semana). Se han registrado
exactamente 7 check-ins esta semana entre ambos hábitos. Todos los hábitos
llevan más de 30 días activos (sin prorrateo).  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/estadisticas`.
2. Localizar la tarjeta de la categoría `Salud`.
3. Leer el valor de porcentaje mostrado.

**Resultado esperado**: El porcentaje mostrado es `50%` (7 check-ins
completados / 14 esperados). El valor cambia si se registra un check-in
adicional y se recarga o actualiza la vista.

---

### TC-29 — Vista de estadísticas muestra racha promedio de hábitos activos en la categoría

**Criterio**: spec.md — Vista de progreso #29  
**Precondiciones**: Existen hábitos activos en `Salud` con al menos una
racha completa registrada en los últimos 14 días.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/estadisticas`.
2. Localizar la tarjeta de la categoría `Salud`.
3. Leer el valor de racha promedio mostrado.

**Resultado esperado**: Se muestra un valor numérico positivo representando
la racha promedio de los hábitos activos en `Salud` que tienen al menos
una racha completa en los últimos 14 días. Si ningún hábito cumple esa
condición, muestra `0` o "Sin datos".

---

### TC-30 — Hábitos archivados están ocultos en el dashboard y visibles en Históricos

**Criterio**: spec.md — Vista de progreso #30  
**Precondiciones**: TC-11 ejecutado. El hábito `Ejercicio` está archivado.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Navegar a `/`.
2. Buscar `Ejercicio` en todos los grupos de categoría del dashboard.
3. Navegar a `/archivados`.
4. Buscar `Ejercicio` en el listado.

**Resultado esperado**: En `/` el hábito `Ejercicio` no aparece en ningún
grupo. En `/archivados` aparece con un indicador visual de estado archivado
y su historial de check-ins o racha máxima visible. El hábito en
`/archivados` no tiene botón de toggle de check-in activo.

---

## Sincronía entre dispositivos

### TC-31 — Cambio de estado es visible en otra pestaña en menos de 5 segundos

**Criterio**: spec.md — Sincronía y rendimiento #31  
**Precondiciones**: Usuario `tester01@ejemplo.com` autenticado. Dos pestañas
del navegador abiertas en `/`. El hábito `Leer libros` no tiene check-in
para hoy en ninguna de las dos pestañas.  
**Datos de entrada**: Ninguno.  
**Pasos**:
1. Abrir la app en Pestaña A y en Pestaña B (mismo navegador, misma sesión).
2. Verificar que ambas muestran `Leer libros` como "no completado".
3. En Pestaña A, hacer clic en el toggle de `Leer libros`.
4. Sin recargar, observar Pestaña B y medir el tiempo hasta que refleja el
   cambio.

**Resultado esperado**: La Pestaña B muestra `Leer libros` como "completado
hoy" en menos de 5 segundos desde el paso 3, sin necesidad de recargar
manualmente.

---

## Criterios inverificables manualmente

### TC-32 — Aislamiento de datos entre usuarios [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #1  
**Motivo**: Verificar que el usuario A no puede leer ni modificar datos del
usuario B requiere intentar una operación autenticada con el token de A contra
registros de B, directamente en la API de Supabase (ej. con curl o Postman).
No es observable en la UI estándar, donde cada usuario solo ve sus propios
datos por diseño de pantalla. La garantía real es que RLS esté configurada
correctamente en BD.

---

### TC-33 — Invalidación del token en backend tras logout [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #2  
**Motivo**: Confirmar que el token JWT está efectivamente invalidado en
Supabase tras el logout (y no solo eliminado del cliente) requiere reutilizar
el token capturado antes del logout para intentar una petición autenticada
directa a la API. No es observable desde la UI: el redirect a `/login` solo
confirma que el cliente borró el token, no que el backend lo invalidó.

---

### TC-34 — Precisión del algoritmo de cálculo de racha [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #3  
**Motivo**: Verificar que el algoritmo no produce gaps falsos ni cuenta
períodos incorrectamente (incluyendo cambios de horario, años bisiestos,
semanas con `target_per_week` variable) requiere insertar fixtures de
check-ins con fechas arbitrarias en la tabla `check_ins` de Supabase y
comparar el resultado del cálculo con el valor esperado. No es ejecutable
desde la UI sin control directo sobre las fechas de inserción.

---

### TC-35 — Deduplicación de check-ins a nivel de base de datos [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #4  
**Motivo**: Verificar que dos inserciones concurrentes para la misma unidad
temporal se consolidan en una sola fila (o que la lectura devuelve siempre
una sola) requiere inspeccionar las filas de `check_ins` en Supabase. La UI
solo muestra el estado consolidado; no expone el número de filas reales en BD.

---

### TC-36 — Persistencia en TIMESTAMPTZ y conversión correcta de timezone [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #5
**Motivo**: Verificar que el timestamp se persiste en UTC y que la conversión
a fecha local del usuario es correcta requiere comparar la columna
`timestamp` en BD (valor UTC) con la fecha mostrada en UI para un usuario
en timezone distinto a UTC. No es observable solo desde la UI sin acceso a
los datos crudos de Supabase o sin cambiar manualmente el timezone del SO
y verificar la coherencia.

---

### TC-37 — Las filas de check-ins persisten en BD al eliminar un hábito con racha [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #6  
**Motivo**: Confirmar que las filas de `check_ins` del hábito eliminado
permanecen en BD y no afectan cálculos de otros hábitos requiere consultar
directamente la tabla `check_ins` en Supabase. La presencia del hábito
en `/archivados` con historial visible (TC-11 y TC-30) es verificable en UI,
pero la integridad a nivel de filas en BD no lo es.

---

### TC-38 — El constraint UNIQUE(user_id, habit_name) existe a nivel de base de datos [RECHAZADO]

**Criterio**: spec.md — Pruebas técnicas #7  
**Motivo**: El efecto observable de no poder crear dos hábitos con el mismo
nombre sí es verificable en UI (intentar crear `Leer libros` dos veces
y observar el error). Sin embargo, la garantía de que el constraint existe
en BD (y no solo en validación del cliente) requiere intentar una inserción
directa en Supabase bypasseando la UI para confirmar que la BD rechaza
la fila. La prueba de UI cubre el comportamiento pero no el constraint en BD.

---