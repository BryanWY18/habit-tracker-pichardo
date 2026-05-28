Se añadió este fragmento a mano en "docs/adr/0004-", línea 29:

"Esta decisión supersede la Ronda 3 de la spec ('Rutas: Minimal + endpoints REST'). Las rutas /api/* listadas en spec.md no se implementan en el scope inicial; el acceso directo a Supabase desde el cliente las hace innecesarias. Si en el futuro se requieren (ej. webhooks, integración externa), se crean en ese momento"