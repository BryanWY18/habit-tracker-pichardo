CONTEXTO:
Tengo una spec terminada para un Habit Tracker. Antes de commitearla quiero
verificar que cumple los requisitos del Diplomado Vibe Coding Profesional.

MI SPEC:
CONTEXTO:
Soy alumno del Diplomado Vibe Coding Profesional. Tengo un borrador de spec
para un Habit Tracker. El curso evalúa specs sobre estos puntos:

- Claridad del objetivo
- Scope realista para 3 horas de build asistido por IA
- Criterios de aceptación verificables (no aspiracionales)
- No-goals explícitos
- Decisiones tomadas (no "ya veremos")

Banderas rojas que descalifican automáticamente el proyecto:
- Spec genérico sin criterios verificables
- Decisiones implícitas que se van a improvisar en el build
- Criterios subjetivos tipo "intuitivo", "bonito", "rápido"

MI SPEC:
# Habit Tracker — Spec

## Objetivo

Aplicación web de escritorio para que individuos registren, organicen y visualicen el cumplimiento de sus hábitos diarios y semanales, separados por ámbitos de vida, con categorías como extensión elegida.

---

## Scope

### Qué SÍ entra en este proyecto

- Autenticación de usuarios con email y contraseña (registro, login, logout).
- CRUD completo de hábitos con nombre, descripción y frecuencia (diaria o semanal).
- Check-in diario por hábito, con ventana de gracia de 12 horas para registrar el día anterior y posibilidad de desmarcar dentro de las 12 horas siguientes al registro con confirmación explícita.
- Categorías predefinidas por el sistema (ej. Personal, Trabajo, Salud) y categorías personalizadas creadas por el usuario.
- Asignación de un hábito a exactamente una categoría.
- Etiquetas decorativas opcionales por hábito (color y filtro visual, sin impacto en métricas).
- Conservación del historial de check-ins como registros huérfanos al eliminar un hábito, únicamente si ese hábito alcanzó al menos una racha antes de ser eliminado.

### Qué NO entra en este proyecto

- Aplicación móvil o diseño responsive prioritario; el target es desktop.
- Notificaciones push, emails recordatorios o alertas de cualquier tipo.
- Integración con servicios externos (calendarios, wearables, APIs de terceros).
- Función de pausa o archivado de hábitos; la única salida es la eliminación.
- Gamificación (puntos, insignias, niveles, rankings).
- Compartir hábitos o progreso con otros usuarios.
- Recálculo retroactivo de racha al cambiar la frecuencia de un hábito.

---

## Criterios de aceptación

### Autenticación

1. Dado un visitante sin cuenta, cuando completa el formulario de registro con email y contraseña válidos, entonces se crea su cuenta y queda autenticado en la aplicación.
2. Dado un visitante con cuenta existente, cuando ingresa su email y contraseña correctos, entonces accede a su sesión y ve su listado de hábitos.
3. Dado un usuario autenticado, cuando ejecuta la acción de logout, entonces su sesión termina y es redirigido a la pantalla de login.
4. Dado un visitante, cuando intenta acceder a cualquier ruta protegida sin sesión activa, entonces es redirigido al login sin ver contenido privado.

### CRUD de hábitos

5. Dado un usuario autenticado, cuando crea un hábito con nombre, descripción, frecuencia y categoría, entonces el hábito aparece en su listado asociado a esa categoría.
6. Dado un usuario autenticado, cuando edita el nombre, descripción o frecuencia de un hábito existente, entonces los cambios se reflejan de inmediato en el listado y aplican solo hacia adelante; el historial previo no se altera.
7. Dado un usuario autenticado, cuando elimina un hábito que nunca alcanzó una racha, entonces ese hábito y todos sus check-ins desaparecen por completo del sistema.
8. Dado un usuario autenticado, cuando elimina un hábito diario, el sistema detecta que el hábito tiene al menos una racha de 7 días consecutivos completados en su historial, entonces los registros de check-ins se preservan. Si el hábito es semanal, cuando el sistema detecta que el hábito cuenta con al menos dos check-ins realizados en semanas distintas desde su creación, entonces el sistema debe preservar sus datos.
9. Dado un usuario autenticado, cuando intenta guardar un hábito sin nombre, entonces el sistema impide el guardado y muestra un mensaje de error.

### Categorías

10. Dado un usuario autenticado al crear o editar un hábito, cuando selecciona una categoría, entonces solo puede asignar exactamente una categoría por hábito.
11. Dado un usuario autenticado, cuando accede al selector de categorías, entonces ve las categorías predefinidas del sistema junto con las categorías personalizadas que haya creado.
12. Dado un usuario autenticado, cuando crea una categoría personalizada con nombre, entonces esa categoría queda disponible para asignarla a hábitos, no es obligatorio vincular ni crear etiquetas. Cuando la elimina, previa confirmación, todo lo vinculado a ella se elimina también.

### Etiquetas

13. Dado un usuario autenticado crea o edita un hábito, cuando selecciona una o más etiquetas (predeterminadas o personalizadas), entonces el sistema debe asociar dichas etiquetas al hábito sin que esto sea un requisito obligatorio para guardar. Dado un hábito que tiene etiquetas asociadas, cuando el usuario visualiza el hábito en su listado o panel de detalles, entonces cada etiqueta debe mostrarse como un elemento visual independiente que incluya el símbolo # seguido del nombre de la etiqueta.
14. Dado un usuario autenticado, cuando filtra su listado por una etiqueta, entonces solo se muestran los hábitos que tienen esa etiqueta asignada, independientemente de su categoría.

### Check-in

15. Dado un usuario autenticado, cuando marca un hábito como completado en el día actual, entonces el sistema registra el check-in con la fecha de hoy, negando un segundo check in si es dentro del mismo día. Cuando es un hábito semanal, se puede completar cualquier día dentro de la semana lógica que corre a partir del alta del hábito, negando un segundo check in si es dentro de la misma semana establecida.
16. Dado un usuario autenticado en el día D, cuando intenta registrar un check-in para el día D-1 y han transcurrido menos de 12 horas desde la medianoche que inició el día D, entonces el sistema acepta el check-in con la fecha D-1. El tiempo se toma de acuerdo con el horario local del navegador.
17. Dado un usuario autenticado en el día D, cuando intenta registrar un check-in para el día D-1 y ya transcurrieron más de 12 horas desde la medianoche que inició el día D, entonces el sistema rechaza el intento y no registra el check-in.
18. Dado un usuario autenticado que marcó un hábito como completado, cuando solicita desmarcarlo dentro de las 12 horas siguientes al registro, entonces el sistema muestra una confirmación explícita y, si el usuario confirma, elimina el check-in.
19. Dado un usuario autenticado que marcó un hábito como completado, cuando intenta desmarcarlo después de 12 horas desde el registro, entonces el sistema rechaza la acción y el check-in permanece marcado.

---

## No-goals

1. **Notificaciones de cualquier tipo.** La app no envía emails, push notifications ni recordatorios in-app para completar hábitos.
2. **Pausa o archivado de hábitos.** No existe un estado intermedio entre activo y eliminado; un hábito o está en el listado o fue borrado.
4. **Métricas ni estadísticas.** Se prioriza la creación / gestión de categorías y los chek in
5. **Gamificación.** No hay puntos, insignias, niveles ni ningún mecanismo de recompensa visual más allá de la racha numérica.
6. **Funciones sociales.** No existe la posibilidad de compartir hábitos, comparar progreso con otros usuarios ni hacer el perfil público.
7. **Integración con servicios externos.** No hay conexión con calendarios, aplicaciones de salud, wearables ni APIs de terceros.
8. **Servicio offline.** Al ser solo en versión App web en Desktop, si no hay conección, no carga la app.

OBJETIVO:
Critica este spec desde 3 perspectivas distintas. No me lo reescribas. Solo
señala problemas. Quiero salir con una lista de cosas a arreglar.

PERSPECTIVA 1 — Senior en code review:
¿Qué decisiones técnicas implícitas hay que el spec no toma?
¿Qué casos extremos no se contemplan? (concurrencia, datos vacíos, errores)
¿Qué supuestos del autor pueden romperse?

PERSPECTIVA 2 — QA que va a probar la app:
¿Qué criterios de aceptación NO son verificables?
¿Cuáles dependen de juicio subjetivo?
¿Para qué criterios no podrías escribir un caso de prueba?

PERSPECTIVA 3 — Otro alumno que va a implementarlo sin hablar conmigo:
¿Qué partes del spec quedan ambiguas?
¿Qué decisiones le tocaría inventar a quien implemente?
¿Qué información necesitaría preguntarle al autor?

RESTRICCIONES:
- No propongas redacciones alternativas. Solo señala el problema.
- No me digas "está bien" si está bien. Asume que hay problemas. Búscalos.
- Sé específico: cita el criterio o sección exacta, no generalidades.
- No inventes problemas que no existen. Si una sección está sólida, no la
  toques.

CRITERIO DE ÉXITO:
Termino con una lista de mínimo 5 problemas concretos, cada uno citando el
texto exacto del spec que tiene el problema y describiendo en qué consiste el
problema.

OBJETIVO:
Evalúa la spec contra este checklist. Para cada ítem responde: ✓ cumple,
✗ no cumple, o ⚠ cumple parcialmente. Si no cumple o cumple parcial,
cita la sección exacta y explica qué falta. No me digas cómo arreglarlo,
solo señala.

CHECKLIST:

A. Los 4 elementos obligatorios
A.1 Existe sección "Objetivo" con una sola línea que describe producto y audiencia
A.2 Existe sección "Scope" con lista de qué SÍ entra
A.3 Existe sección "Criterios de aceptación" numerada
A.4 Existe sección "No-goals" con mínimo 4 ítems explícitos

B. Cobertura del núcleo del proyecto
B.1 Hay criterios para autenticación (registro, login, logout)
B.2 Hay criterios para creación, edición y eliminación de hábitos
B.3 Hay criterios para check-in con comportamiento descrito (incluye qué pasa
    al deshacer si aplica, y qué pasa con frecuencia semanal vs diaria)
B.4 Hay criterios para vista de progreso (qué se muestra)
B.5 Hay criterios que describen cómo se calcula la racha (qué la rompe,
    qué la mantiene)
B.6 Hay criterios para la extensión elegida

C. Verificabilidad
C.1 Cada criterio de aceptación se puede responder sí/no después de probar
    la app
C.2 Ningún criterio usa palabras subjetivas: "bonito", "intuitivo", "rápido",
    "fácil", "amigable"
C.3 Los criterios describen comportamiento observable, no intención interna

D. Decisiones tomadas
D.1 No hay frases tipo "se decidirá después", "ya veremos", "según convenga"
D.2 No hay etiquetas [DECIDIR: ...] sin resolver
D.3 Casos extremos están cubiertos: ¿qué pasa con datos vacíos? ¿qué pasa al
    eliminar un hábito con historial? ¿qué pasa si no hay conexión?

E. Scope realista
E.1 La extensión es exactamente UNA, no varias
E.2 No hay features fuera del núcleo + la extensión declarada
E.3 Los no-goals incluyen explícitamente features que el alumno podría tener
    la tentación de agregar (tipo "no se hacen estadísticas históricas si la
    extensión no es esa")

RESTRICCIONES:
- Sé estricto. Si dudas entre ✓ y ⚠, marca ⚠.
- No inventes problemas. Si algo cumple, di ✓ y sigue.
- No propongas correcciones. Solo señala qué falta.

CRITERIO DE ÉXITO:
Termino con un reporte donde cada uno de los 15 ítems del checklist tiene
una marca clara y, para los que no cumplen, una cita textual del spec que
muestra el problema.