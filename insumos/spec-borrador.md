# Habit Tracker — Spec

## Objetivo

Crear una aplicación web de escritorio que permita a usuarios registrados gestionar hábitos simples y registrar sus check-ins diarios o semanales sin distracciones ni opciones avanzadas.

## Scope

### Qué SÍ entra
- Autenticación de usuarios con email y contraseña usando Supabase Auth.
- CRUD de hábitos con nombre, descripción, frecuencia (diaria o semanal) y categoría.
- Registro de check-ins con ventana de gracia de 12 horas para el día anterior.
- Desmarcar un check-in dentro de las 12 horas siguientes con confirmación.
- Vista de listado de hábitos por categoría, estado de completado y racha actual.
- Interfaz de escritorio simple y funcional, desarrollada con Next.js 15, TypeScript y Tailwind CSS.

### Qué NO entra
- OAuth/social login.
- Exportar/importar CSV.
- Sincronización con calendarios externos.
- Etiquetas avanzadas o filtros compuestos.
- Diseño móvil pulido o UI móvil premium.
- Gamificación, notificaciones o alertas.

## Criterios de aceptación

1. Un usuario puede registrarse con email y contraseña y queda autenticado.
2. Un usuario puede iniciar sesión con email y contraseña y acceder a su listado de hábitos.
3. Un usuario autenticado puede cerrar sesión y es redirigido al login.
4. Un visitante sin sesión que intenta acceder a una ruta protegida es redirigido al login.
5. Un usuario puede crear un hábito con nombre, descripción, frecuencia y categoría, y verlo en su listado.
6. Un usuario puede editar un hábito y los cambios se reflejan en el listado sin alterar el historial previo.
7. Un usuario puede eliminar un hábito y, si nunca logró una racha, desaparece con sus check-ins.
8. Un usuario puede registrar un check-in para hoy y el sistema actualiza el estado de completado.
9. Un usuario puede registrar un check-in para el día anterior dentro de las 12 horas de gracia.
10. Un usuario puede desmarcar un check-in dentro de las 12 horas siguientes al registro con confirmación.
11. Un hábito diario muestra una racha basada en días consecutivos completados.
12. El tablero muestra hábitos agrupados por categoría y el estado actual de cada hábito.

## No-goals

- No se implementan notificaciones push, email ni recordatorios.
- No se añade gamificación, puntos, insignias o rankings.
- No se integra con servicios externos fuera de Supabase.
- No se prioriza experiencia móvil o diseño responsive avanzado.
- No se introduce importación/exportación de datos en la fase inicial.
