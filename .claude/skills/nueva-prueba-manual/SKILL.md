---
name: nueva-prueba-manual
description: Usar cuando se quiere agregar una prueba manual al plan de pruebas del proyecto. Garantiza ID correlativo, referencia a exactamente un criterio de la spec, pasos sin ambigüedad y resultado esperado observable y verificable.
---

Eres un procedimiento de registro, no un agente que razona. Tu único trabajo es garantizar formato y completitud de la prueba manual. No ejecutas pruebas, no decides si una prueba pasó o falló, no editas pruebas existentes.

---

## Paso 1 — Recolectar datos

Antes de escribir cualquier cosa, reúne los siguientes campos. Si alguno falta, pregunta explícitamente. No inferir ni inventar.

| Campo | Descripción |
|---|---|
| `criterio` | ID exacto del criterio de `spec.md` que esta prueba cubre (ej. `AC-04`) |
| `precondicion` | Estado del sistema y datos necesarios antes de ejecutar la prueba |
| `pasos` | Lista numerada de acciones del usuario, una por línea |
| `resultado_esperado` | Qué debe ocurrir exactamente al finalizar los pasos |

El campo `estado` siempre se inicializa en `pendiente`. No preguntar al usuario.

---

## Paso 2 — Validar antes de escribir

### Una prueba cubre exactamente 1 criterio

Si los pasos o el resultado cubren claramente más de un criterio de la spec, detente y explica cuántas pruebas separadas se deben crear. No colapses dos criterios en una sola prueba.

### Rechaza el resultado esperado si:
- Es vago y no puede responderse con sí/no (ej. "la app funciona bien", "todo se ve correcto").
- No menciona ningún elemento concreto: mensaje visible, dato en pantalla, cambio de estado, navegación a una ruta específica.
- Describe un proceso en lugar de un estado final.

### Rechaza un paso si:
- Es ambiguo sobre qué acción exacta realizar (ej. "el usuario navega un poco", "completar el formulario").
- Combina más de una acción sin distinguirlas.
- Asume conocimiento implícito que otro ejecutor no tendría.

Un paso válido nombra: qué elemento tocar/escribir/seleccionar y con qué valor o interacción exacta.

### Rechaza la precondición si:
- Es "ninguna" cuando la prueba necesita datos o un estado previo concreto.
- Es tan genérica que no permite reproducir la prueba (ej. "usuario logueado" sin indicar qué usuario ni qué datos debe tener).

---

## Paso 3 — Auto-numerar la prueba

1. Leer el archivo `docs/pruebas-manuales.md`.
2. Buscar todos los encabezados con patrón `### TC-{número}` dentro del archivo.
3. Calcular `siguiente = max(números encontrados) + 1`, con zero-padding a 4 dígitos.
4. Si el archivo no existe o no contiene ningún `TC-`, comenzar en `0001`.

No preguntar al usuario el número.

---

## Paso 4 — Agregar la prueba al archivo

Ubicación: `docs/pruebas-manuales.md`.

Si el archivo no existe, crearlo con este encabezado antes de agregar la primera prueba:

```markdown
# Plan de pruebas manuales

<!-- Generado con /nueva-prueba-manual -->
```

Agregar el siguiente bloque al **final** del archivo:

```markdown
### TC-{siguiente}: {Descripción corta en 3–5 palabras}

**Criterio cubierto:** {ID} — {Texto literal del criterio en spec.md}
**Estado:** pendiente
**Fecha de creación:** {YYYY-MM-DD}

**Precondición:**
{Estado del sistema y datos necesarios antes de iniciar la prueba.}

**Pasos:**

1. {Acción exacta del usuario sobre un elemento concreto}
2. {Siguiente acción}

**Resultado esperado:**
{Descripción observable y verificable. Debe poder responderse con sí/no.}

---
```

El separador `---` al final permite distinguir pruebas visualmente.

---

## No-goals

- No ejecuta pruebas.
- No decide si una prueba pasó o falló.
- No actualiza el campo `estado` de pruebas existentes.
- No edita ni elimina pruebas existentes.
- No valida si el criterio referenciado existe realmente en `spec.md`.

---

## Criterio de éxito

El skill termina exitosamente cuando:
1. El bloque de prueba está al final de `docs/pruebas-manuales.md` con el ID correlativo correcto.
2. La prueba referencia exactamente un criterio de la spec.
3. Cada paso describe una acción concreta y sin ambigüedad.
4. El resultado esperado es observable y verificable con sí/no.
5. La precondición describe el estado inicial necesario para reproducir la prueba.
