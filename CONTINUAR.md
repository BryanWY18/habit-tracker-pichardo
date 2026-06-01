# CONTINUAR — Guía de build

## Primer comando

```powershell
git checkout develop && git pull origin develop
git checkout -b feat/t-01-init-nextjs
```

---

## Loop por tarea (repetir T-01 → T-14)

```
1. git checkout develop && git pull origin develop
2. git checkout -b feat/t-XX-nombre-corto
3. → "implementar tarea T-XX"          # agente implementer
4. → "revisar commit"                  # agente reviewer
5. Corregir lo que marque el reviewer (editar, re-stagear)
6. git add <archivos>
   git commit -m "feat(t-XX): descripción"
7. Abrir PR feat/t-XX → develop, mergear, borrar rama
```

Una tarea = una rama = un commit. El criterio de hecho de `plan.md`
es la única definición de "terminado" para cada tarea.

---

## Si el implementer se atasca dos veces en la misma tarea

1. Editar el código manualmente.
2. Documentar en `CONTEXT.md`:
   - Qué se cambió y en qué línea/archivo.
   - Por qué el agente no lo resolvió.
3. Continuar con el paso 4 del loop (reviewer).

---

## Ramas de release (deploy a Vercel)

Cuando `develop` tiene las tareas del sprint completas y los TCs pasan:

```
git checkout develop
git checkout -b release/v1.0
# ajustes de última hora (solo fix, no feat)
git checkout main && git merge release/v1.0 --no-ff
git tag v1.0.0
git checkout develop && git merge release/v1.0 --no-ff
git branch -d release/v1.0
```

Configurar las variables de producción en Vercel antes del merge a `main`.

---

## Hotfix (bug en producción)

```
git checkout main
git checkout -b hotfix/descripcion-corta
# fix mínimo
git checkout main && git merge hotfix/descripcion-corta --no-ff
git tag v1.0.1
git checkout develop && git merge hotfix/descripcion-corta --no-ff
git branch -d hotfix/descripcion-corta
```
