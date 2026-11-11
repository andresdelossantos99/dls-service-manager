# GIT WORKFLOW — Git Bash, VS Code, GPT y Claude

> Procedimiento obligatorio de DLS Service Manager. Repositorio público; aplicación y datos privados. **Claude implementa, GPT audita, el responsable aprueba y hace commit.**

## Entorno

- Sistema operativo: Windows; terminal: **Git Bash** dentro de VS Code.
- Gestor de versiones: Git y GitHub; rama principal: `main`.
- Estilo de commits: **Conventional Commits en español** (prefijo técnico estándar y descripción en español).
- No dar por sincronizado el estado local con GitHub: el remoto no contiene modificaciones sin publicar.

## Responsabilidades

| Participante | Hace | No hace sin autorización |
| --- | --- | --- |
| **GPT** | Revisa requisitos, prepara prompts para Claude, inspecciona código/diffs, valida resultados de pruebas y recomienda o rechaza un commit | Editar código, afirmar que ejecutó pruebas que no ejecutó, aprobar sin evidencia |
| **Claude** | Implementa y corrige código, documenta cambios y aporta los resultados de validación | Commit, push, merge, cambios fuera del alcance |
| **Responsable** | Decide alcance, resuelve dudas, aprueba y ejecuta commits/push | — |

Si GPT no puede ejecutar pruebas en el repositorio local, debe solicitar los comandos y resultados, revisarlos y dejar esa limitación explícita. La auditoría ayuda a reducir defectos, pero no garantiza ausencia total de errores.

## 1. Inicio de sesión: inspección antes de cambios

Desde la raíz de `dls-service-manager`:

```bash
git status -sb
git log -5 --oneline
git diff
git diff --cached
git remote -v
```

**Solo cuando no haya cambios locales que puedan entrar en conflicto**, actualizar la rama en curso:

```bash
git pull --ff-only
```

Leer en orden `PROJECT_CONTEXT.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `DATA_MODEL.md`, `docs/GIT-WORKFLOW.md` y la documentación del hito. Verificar la rama activa. No usar `git pull` mecánicamente si hay modificaciones sin revisar.

## 2. GPT define la tarea de Claude

GPT prepara un prompt con:

1. Objetivo y hito exacto del `ROADMAP.md`.
2. Archivos o módulos que Claude debe revisar y puede modificar.
3. Restricciones de producto, arquitectura, datos y seguridad.
4. Comportamiento UX/UI esperado (móvil y escritorio, si aplica).
5. Criterios de aceptación y escenarios de error.
6. Pruebas y evidencia que Claude debe entregar.
7. Documentos a actualizar.

**No enviar a Claude una petición amplia que mezcle hitos sin autorización.** La decisión sobre requisitos o alcance corresponde al responsable.

## 3. Claude implementa sin hacer commit

Para trabajo no trivial, preferir una rama propia **después de verificar que la rama base esté limpia**:

```bash
git switch main
git pull --ff-only
git switch -c feat/nombre-del-hito
```

Ejemplo:

```bash
git switch -c feat/recepcion-rapida
```

Claude puede escribir código y documentación conforme al prompt aprobado, pero debe **detenerse antes de `git add`, `git commit`, `git push` o merge**, salvo instrucción explícita del usuario. Debe identificar archivos modificados/nuevos y explicar la implementación.

## 4. Revisión de GPT antes del commit

Desde Git Bash, obtener evidencia del estado local:

```bash
git status --short
git diff --check
git diff --stat
git diff
git diff --cached
```

**Archivos nuevos:** `git diff` no muestra automáticamente archivos no rastreados. Compartir también el contenido de esos archivos o un paquete de los archivos modificados/nuevos. Para comparar con la rama principal en un trabajo con commits publicados, usar `git diff main...HEAD`, pero **esto no reemplaza revisar modificaciones locales no confirmadas**.

GPT debe revisar, según alcance:

- Correspondencia con requisitos y criterios de aceptación.
- Calidad, tipos, estado, validación y manejo de errores.
- Comportamiento mobile first, temas, accesibilidad y formularios flexibles.
- Seguridad, control de acceso, exposición de secretos, RLS, privacidad y límites de IA.
- Regresiones, documentación y decisiones técnicas.
- Evidencia de pruebas, diferenciando resultados comprobados de afirmaciones del implementador.

Si el código nuevo solo está en la computadora del responsable, GPT necesita recibirlo como diff, archivos o paquete. **Que GitHub sea público no permite ver código local sin commit.**

## 5. Validaciones exigidas

Claude ejecuta estas comprobaciones y entrega la salida real; GPT puede repetirlas cuando tenga acceso al código y un entorno adecuado:

```bash
npm run lint
npx tsc --noEmit
npm run build
git diff --check
```

Agregar pruebas de las funciones afectadas cuando existan. Si una prueba no existe o no puede ejecutarse, registrarlo como pendiente, no como «aprobada».

**Resultado de la auditoría de GPT:**

- **Aprobado para proponer commit:** alcance respetado y controles relevantes revisados, con riesgos residuales explícitos.
- **Correcciones necesarias:** GPT entrega un prompt correctivo para Claude; se repite la revisión.
- **Bloqueado:** falta información, evidencia, permisos o una decisión del responsable.

Ni GPT ni Claude deben realizar el commit automáticamente por haber pasado los controles.

## 6. Commit y push: únicamente tras aprobación del responsable

Cuando el responsable confirme que acepta el cambio, comprobar el estado y preparar solo los archivos pertinentes:

```bash
git status
git add ruta/del/archivo-1 ruta/del/archivo-2
git diff --cached
git diff --cached --check
git commit -m "feat: implementar recepción rápida de equipos"
git push -u origin HEAD
```

Se prefiere `git add` selectivo frente a `git add .` para evitar incluir archivos accidentales. En ramas de funcionalidades importantes, crear PR y revisar el diff antes de fusionar con `main`.

### Convención de mensajes

```text
feat: agregar galería de fotografías
fix: corregir el guardado de acuerdos
refactor: separar servicios de Supabase
test: incorporar pruebas de recepción
docs: registrar decisiones de autenticación
chore: actualizar configuración de ESLint
```

Prefijo técnico en minúsculas (`feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`) y descripción **en español**, breve y en infinitivo.

## 7. Comandos de revisión habituales

```bash
git status -sb
git diff --stat
git diff --name-only
git diff --cached
git log --oneline --decorate -10
git show --stat HEAD
```

Si una rama contiene commits previos, comparar cambios publicados con:

```bash
git diff main...HEAD
```

Nunca asumir que esa comparación incluye los archivos de trabajo sin commit.

## 8. Seguridad en repositorio público

**No publicar:** `.env.local`, claves de OpenRouter/Supabase, tokens, logs sensibles, contraseñas, datos reales de clientes, conversaciones o fotografías privadas. Usar `.env.example` con valores ficticios y verificar el staging antes del commit.

**No ejecutar sin aprobación explícita:** `git reset --hard`, `git clean -fd`, `git push --force`, borrado de ramas con trabajo no fusionado o reescrituras de historial.

Si una credencial se filtra en el repositorio público, **revocarla/rotarla inmediatamente**; eliminar el archivo en un commit posterior no basta para deshacer la exposición.

## 9. Documentación de cada hito

- `ROADMAP.md`: estado real y próximo hito.
- `docs/DECISIONS.md`: contexto, alternativas, elección, motivos y consecuencias, con fecha.
- `docs/TECHNICAL_REPORT.md`: implementado, evidencia de pruebas, limitaciones, riesgos y tareas pendientes.
- `ARCHITECTURE.md` / `DATA_MODEL.md`: actualizar únicamente cuando cambien límites o datos.
- `CHANGELOG.md`: cambios significativos, sin afirmar funcionalidades incompletas.

## 10. Prompt de continuidad para GPT

> Actuá como arquitecto y revisor, **no como implementador**. Antes de proponer cambios, revisá el estado real de Git (`git status`, `git log -5 --oneline`, `git diff`) y leé `PROJECT_CONTEXT.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `DATA_MODEL.md` y el documento del hito. Si solo podés ver GitHub, aclarame qué cambios locales no son visibles. Identificá el hito activo y redactá un prompt acotado para que **Claude** lo implemente. Cuando Claude termine, revisá el diff y los archivos nuevos, validá las pruebas ejecutadas (o pedí sus resultados), pedí correcciones si corresponde y emití una evaluación antes del commit. **No edites código ni hagas commits por tu cuenta.** Proponé un mensaje Conventional Commit en español; yo autorizo y ejecuto el commit.

## 11. Prompt de entrega de Claude a GPT

> Implementá únicamente el hito indicado en el prompt aprobado. No hagas commits ni push. Al terminar, informá archivos modificados y nuevos, decisiones relevantes, riesgos y funcionalidades pendientes. Mostrá `git status --short`, el diff (o facilitá los archivos nuevos), y los resultados completos de `npm run lint`, `npx tsc --noEmit`, `npm run build` y pruebas afectadas. Actualizá la documentación que corresponda. Dejá el trabajo listo para que GPT lo audite y el responsable apruebe antes del commit.
