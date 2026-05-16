Brief: Habit Tracker

1. El problema

Muchas personas quieren construir rutinas saludables pero pierden el hilo por falta de simplicidad y claridad: las apps existentes suelen estar pensadas para móvil, saturadas de gamificación, notificaciones y opciones que distraen del objetivo principal —registrar y revisar hábitos—. Eso genera fricción para usuarios que sólo buscan una herramienta clara, de escritorio y enfocada en seguimiento diario.

Las soluciones actuales además manejan ventanas temporales y correcciones de manera rígida o inconsistente, lo que provoca datos erróneos y frustración cuando un usuario desea corregir un check-in reciente. Necesitamos una aplicación minimalista que haga el flujo básico sin forzar decisiones complejas de diseño.

2. Núcleo obligatorio

1) Autenticación y cuentas
Usuarios podrán registrarse e iniciar sesión (email/contraseña) y gestionar su sesión. El objetivo es privacidad y separación por usuario; la implementación técnica se delega a Supabase Auth.
Decisión abierta: ¿se debe soportar OAuth/social login en la spec o se limita sólo a email/contraseña?

2) Gestión de hábitos (CRUD)
Crear, leer, actualizar y eliminar hábitos con campos mínimos: nombre, descripción, frecuencia (diaria/semanal), categoría. Interfaz simple para listar y editar hábitos.
Decisión abierta: ¿permitimos etiquetas libres por hábito o sólo categorías predefinidas?

3) Registro de check-ins y reglas de racha
Registrar check-ins por hábito con una ventana de gracia de 12 horas para el día anterior, prevención de duplicados diarios/semanales y posibilidad de desmarcar dentro de 12 horas con confirmación.
Decisión abierta: ¿la zona horaria debe tomarse del navegador del usuario o centralizarse en UTC y ajustar la UI?

4) Visualización y control mínimo
Un tablero que muestre el listado por categoría, el estado (completado hoy), racha actual y botón único para marcar/desmarcar. Interfaz enfocada a desktop, accesible y sin animaciones complejas.

3. Extensiones (elige máximo 1)

| Extensión | Descripción |
|---|---|
| Exportar/Importar CSV | Exportar historial de check-ins y hábitos; importar desde CSV para migración. |
| Estadísticas semanales | Panel con gráficos simples (rachas, porcentaje de cumplimiento semanal). |
| Plantillas de hábitos | Biblioteca de plantillas preconfiguradas que el usuario puede aplicar al crear un hábito. |
| Sincronización de calendario | Sincronizar check-ins con un calendario externo (opcional, por ejemplo Google Calendar). |
| Etiquetas y filtros avanzados | Sistema de etiquetas multiple y filtros compuestos para explorar hábitos. |
| Modo responsive básico | Hacer la UI usable en móvil con prioridad mínima (no diseño móvil premium). |

4. Restricciones técnicas

- Stack fijo: Next.js 15 (App Router), Supabase (Postgres + Auth), Vercel para deploy. 
- Código en TypeScript estricto y estilos con Tailwind CSS. 
- No introducir dependencias de backend adicionales ni infra fuera de Supabase en el alcance inicial.

5. Lo que NO se evalúa

- Diseño visual premium o exploración de marca (UI será funcional y simple). 
- Performance a escala o optimizaciones avanzadas de rendering. 
- Cobertura completa de tests automatizados (se esperan tests manuales y básicos). 
- Experiencia móvil pulida y layouts complejos (mobile es opcional y limitado). 
- Integración profunda con servicios externos por defecto (sólo como extensiones opcionales).
