# GIT WORKFLOW — VS Code, GPT y Claude

## Regla central

Git es la fuente de verdad del código y de la documentación. Antes de cada cambio, revisar `git status`, `git log -5 --oneline` y `git diff`. No asumir que los archivos están iguales que en otra sesión.

## Inicio del trabajo

```powershell
git status
git log -5 --oneline
git pull --ff-only
```

Leer `PROJECT_CONTEXT.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `DATA_MODEL.md` y el documento del hito. Si hay cambios locales, no hacer pull ni sobrescribirlos sin revisar.

## Desarrollo por hito

1. Explicar alcance y archivos a modificar.
2. Crear rama `feat/<tema>` o `docs/<tema>` si el cambio no es trivial: `git switch -c feat/tema`.
3. Implementar cambio acotado, sin funcionalidades no solicitadas.
4. Revisar `git diff --check`, `git diff`, `npm run lint`, `npx tsc --noEmit`, `npm run build` y pruebas afectadas.
5. Actualizar `ROADMAP.md`, `docs/DECISIONS.md`, `docs/TECHNICAL_REPORT.md` cuando corresponda.
6. Commit claro y push; para trabajo colaborativo, abrir PR y revisar el diff antes de fusionar.

```powershell
git status
git diff --check
git diff
npm run lint
npx tsc --noEmit
npm run build
git add .
git commit -m "feat: describe verified change"
git push -u origin HEAD
```

## Revisión de cambios

```powershell
git status -sb
git diff --stat
git diff --name-only
git log --oneline --decorate -10
git show --stat HEAD
```

**Nunca** ejecutar `git reset --hard`, `git clean -fd`, `git push --force` o reescritura de historia sin aprobación explícita. No agregar `.env.local`, claves o datos reales.

## Prompt de continuidad

> Revisá el estado real del repositorio con `git status`, `git log -5 --oneline` y `git diff`. Leé PROJECT_CONTEXT, ROADMAP, ARCHITECTURE y DATA_MODEL. Decime cuál es el hito activo, qué está implementado, qué está pendiente y qué cambios proponés. No modifiques nada hasta que validemos el plan. Al terminar, verificá lint, TypeScript, build, pruebas y documentación, y proponé un commit.
