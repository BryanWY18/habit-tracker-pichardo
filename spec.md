# Habit Tracker — Especificación inicial

## Resumen y objetivo

Aplicación web de escritorio para que usuarios registren, gestionen y revisen hábitos diarios o semanales de forma simple y sin distracciones. Este documento describe el alcance mínimo viable (2–3 semanas de trabajo) y los criterios de aceptación para el MVP, basado en el brief inicial en `insumos/brief.md`.

## Público objetivo

Usuarios de escritorio que desean una herramienta simple para mantener rutinas, sin gamificación ni notificaciones intrusivas.

## Alcance (MVP)

1) Autenticación básica
Usuarios pueden registrarse e iniciar sesión con email y contraseña. Gestión de sesión y recuperación mínima (logout). (Implementación: Supabase Auth.)

2) Gestión de hábitos (CRUD)
Crear, editar, listar y eliminar hábitos con: nombre, descripción breve, frecuencia (diaria|semanal) y categoría. Listado principal ordenado por categoría.

3) Registro de check-ins y reglas de racha
Registrar un check-in por hábito; prevenir duplicados dentro de la ventana definida; permitir desmarcar dentro de 12 horas con confirmación. Rachas calculadas hacia adelante; eliminación de hábito borra o preserva historial según reglas acordadas en la spec.

4) Interfaz principal y control rápido
Tablero de escritorio con listado por categoría, indicador de completado del día, racha actual y acción única para marcar/desmarcar. UI funcional, accesible y enfocada a la productividad.

## Criterios de aceptación (resumen ejecutivo)

- Un usuario puede crear cuenta, iniciar sesión y acceder a su listado de hábitos.
- Un usuario autenticado puede crear un hábito con los campos mínimos y verlo en el listado.
- Un usuario puede marcar un hábito como completado para el día actual y el sistema registra un check-in; no se permiten duplicados en el mismo día/semana según la frecuencia.
- El sistema permite marcar un check-in para el día anterior dentro de una ventana de gracia de 12 horas desde la medianoche local del usuario; fuera de esa ventana el registro es rechazado.
- Un usuario puede desmarcar un check-in en un plazo de 12 horas tras su creación con confirmación explícita; pasado ese plazo no es posible.

## Flujos principales de usuario

1. Registro / Login → ver tablero → crear hábito → marcar check-in.
2. Editar hábito → confirmar cambios aplican hacia adelante (no alterar historial pasado).
3. Eliminar hábito → borrar o preservar historial (reglas definidas en la spec).

## Decisiones y preguntas abiertas (deben resolverse en la spec)

- ¿Soportamos OAuth/social login en el MVP o dejamos sólo email/contraseña? (Recomendación inicial: sólo email/contraseña para reducir alcance.)
- ¿Permitir etiquetas libres por hábito además de categorías, o limitar a categorías únicamente? (Impacta UI y modelo de búsqueda/filtrado.)
- ¿La zona horaria para la ventana de gracia debe tomarse del navegador del usuario o normalizarse en UTC en backend? (Esto afecta cálculos de racha y aceptación de check-ins.)

## Extensiones opcionales (elige 1 para la fase siguiente)

- Exportar/Importar CSV: migración de hábitos e historial.
- Estadísticas semanales: gráficos simples de rachas y cumplimiento.
- Plantillas de hábitos: biblioteca de hábitos predefinidos.
- Sincronización con calendario: exportar eventos de check-in a calendario externo.
- Etiquetas y filtros avanzados: sistema de tags y filtros compuestos.

## Restricciones técnicas

- Stack: Next.js 15 (App Router), Supabase (Postgres + Auth), Vercel para deploy.
- TypeScript estricto y Tailwind CSS para estilos.
- No agregar infraestructura backend adicional en el MVP; usar Supabase para datos y auth.

## No-goals

- Diseño visual premium o trabajo extenso de branding.
- Optimización de rendimiento a gran escala.
- Cobertura completa de tests automatizados (se esperan tests manuales y unitarios mínimos).
- Experiencia móvil pulida (mobile es opcional y con prioridad baja).
- Integraciones externas como parte del MVP (sólo como extensiones opcionales).

## Entregables para la fase MVP (2–3 semanas)

- Implementación básica de autenticación y CRUD de hábitos.
- Registro y gestión de check-ins con reglas de 12 horas.
- Interfaz de tablero de escritorio funcional.
- Documentación mínima de despliegue en Vercel y configuración de Supabase.

---
