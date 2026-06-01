---
name: nuevo-adr
description: Usar cuando se quiere registrar una nueva decisión arquitectónica del proyecto. Garantiza que el ADR resultante tenga formato estandarizado, alternativa real con trade-offs y al menos una consecuencia negativa explícita.
---

Eres un procedimiento de registro, no un agente que razona. Tu único trabajo es garantizar formato y completitud del ADR. No propones decisiones, no juzgas si son correctas, no editas ADRs existentes.

---

## Paso 1 — Recolectar datos

Antes de escribir cualquier archivo, reúne los siguientes campos. Si alguno falta, pregunta explícitamente. No inferir ni inventar.

| Campo | Descripción |
|---|---|
| `titulo` | Frase imperativa corta (ej. "Usar PostgreSQL como base de datos principal") |
| `estado` | Uno de: `propuesto`, `aceptado`, `deprecado` |
| `contexto` | Por qué se necesita esta decisión. 2–6 oraciones concretas. |
| `decision` | Qué se decidió. 1–3 oraciones. |
| `alternativas` | Al menos una alternativa real con sus trade-offs (ver reglas de rechazo) |
| `consecuencias` | Al menos una consecuencia negativa o trade-off aceptado (ver reglas de rechazo) |

---

## Paso 2 — Validar antes de escribir

### Rechaza la alternativa si:
- Es "no hacer nada" / "mantener el status quo" sin describir qué trade-off implica eso.
- Es una reformulación de la decisión elegida.
- Es un placeholder: "ninguna", "N/A", "no aplica".

Cuando detectes esto, detente y pide una alternativa real: una tecnología diferente, un enfoque distinto, una solución conocida que se descartó y por qué.

### Rechaza las consecuencias si:
- Solo listan beneficios, ganancias o mejoras.
- Son una reformulación de la decisión.
- Son un placeholder: "ninguna", "N/A".

Una consecuencia negativa válida nombra algo que se vuelve más difícil, más lento, más caro, o que crea una obligación futura como resultado directo de esta decisión.

### Rechaza el contexto si:
- Es genérico y podría aplicar a cualquier proyecto (ej. "necesitamos una base de datos").
- No menciona ninguna restricción, problema o evento concreto del proyecto actual.

---

## Paso 3 — Auto-numerar el archivo

1. Listar todos los archivos que coincidan con el patrón `docs/adr/[0-9]*.md`.
2. Extraer el prefijo numérico de cada nombre (ej. `0003-usar-postgres.md` → `3`).
3. Calcular `siguiente = max(prefijos encontrados) + 1`, con zero-padding a 4 dígitos.
4. Si no existe ningún ADR todavía, comenzar en `0001`.

No preguntar al usuario el número. Calcularlo siempre desde el estado actual de `docs/adr/`.

---

## Paso 4 — Escribir el archivo

Ruta del archivo nuevo:

```
docs/adr/{siguiente}-{titulo-en-kebab-case}.md
```

Contenido exacto (respetar secciones, orden y nombres):

```markdown
# ADR-{siguiente}: {Titulo}

**Estado:** {estado}
**Fecha:** {YYYY-MM-DD}

## Contexto

{contexto — por qué existe esta decisión, qué problema o restricción la origina}

## Decisión

{decision — qué se va a hacer}

## Alternativas consideradas

### {Nombre de la alternativa}

{Descripción de la alternativa y por qué no se eligió. Incluir trade-offs reales.}

<!-- Repetir bloque anterior si hay más de una alternativa -->

## Consecuencias

### Positivas

{Lista de beneficios concretos, o "Ninguna identificada" si no hay.}

### Negativas / Trade-offs aceptados

{Al menos un ítem. Esta sección nunca puede quedar vacía.}
```

---

## No-goals

- No propone ni sugiere decisiones.
- No juzga si una decisión es correcta u óptima.
- No edita ni actualiza ADRs existentes.
- No elimina ADRs.
- No valida la precisión técnica del contenido.

---

## Criterio de éxito

El skill termina exitosamente cuando:
1. Existe un archivo bien formado en `docs/adr/` con el número correlativo correcto.
2. El archivo contiene al menos una alternativa real con trade-offs (no "no hacer nada" vacío).
3. El archivo contiene al menos una consecuencia negativa explícita.
4. El contexto nombra una restricción o problema concreto del proyecto.
