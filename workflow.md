# Workflow de desarrollo con IA — Guía para alumnos

Este repo usa un workflow local basado en archivos para que el agente (Claude Code, Cursor, etc.) te ayude a construir features de punta a punta: todo vive en `.agents/` adentro del repo.

La idea es simple: vos describís qué querés construir, el agente lo descompone en historias chiquitas, planifica cada una con la cabeza puesta en el código que ya existe, y después implementa cada historia como **un commit** sobre una rama de feature compartida (la "epic").

---

## TL;DR — el flujo en 30 segundos

```
/create-prd        → describís el producto, el agente arma el PRD
/create-stories    → el PRD se rompe en N user stories (archivos)
/plan <story>      → el agente diseña la implementación de UNA story
/implement <plan>  → el agente codea, testea, COMMITEA esa story
                     repetí /plan + /implement hasta terminar
git merge epic→main → vos mergeás cuando todo está listo
```

Cada slash command (`/create-prd`, etc.) ejecuta un archivo de instrucciones que vive en `.agents/commands/`. Si querés ver exactamente qué hace cada comando, abrí ese archivo — es markdown, se lee fácil.

---

## ¿Por qué este workflow?

**Problema clásico**: le pedís a una IA "hacé X" y te tira 800 líneas de código mezclando 5 features, sin tests, sin pensar el contexto del repo. Después no sabés ni qué cambió ni por qué.

**Lo que resuelve este workflow**:

1. **Descompone primero, codea después**. La IA no escribe ni una línea de código hasta que existe un plan revisable.
2. **Piezas chicas y revisables**. Una story = una unidad de valor = un commit. Si una sale mal, revertís sin tocar las demás.
3. **Estado en archivos**. Status, planes y reports viven en el repo. Podés ver el progreso con `git log` y los `.md`.
4. **Una sola rama por épica**. No te llenás el repo de `feature/*` huérfanas. Todo el trabajo de un PRD vive en `epic/PRD-NNN-slug` y se mergea junto al final.
5. **Trazabilidad**. Cada commit linkea a su story, su plan y su report. Cualquier reviewer entiende qué se hizo y por qué.

---

## Estructura de archivos

```
.agents/
├── commands/                     # las "recetas" que ejecutan los slash commands
│   ├── create-prd.md
│   ├── create-stories.md
│   ├── plan.md
│   ├── implement.md
│   └── prime.md
│
├── PRDs/
│   └── PRD-001-user-auth/
│       ├── PRD.md                # el documento del producto
│       └── index.md              # tablero de stories (auto-regenerado)
│
├── stories/
│   └── PRD-001-user-auth/
│       ├── STORY-001-login-form.md
│       ├── STORY-002-jwt-middleware.md
│       └── STORY-003-logout.md
│
├── plans/
│   └── PRD-001-user-auth/
│       ├── STORY-001-login-form.plan.md       # plan activo
│       └── completed/
│           └── STORY-002-jwt-middleware.plan.md  # planes ya ejecutados
│
└── reports/
    └── PRD-001-user-auth/
        └── STORY-002-jwt-middleware.report.md
```

**Regla mental**: `stories/*.md` son la "verdad" (tienen el status real en frontmatter). `index.md` es solo una vista linda derivada — se regenera sola.

---

## Convenciones de IDs y branches

- **PRD ID**: `PRD-NNN-{slug}` — ej: `PRD-001-user-auth` (NNN con 3 dígitos, autoincrementa)
- **Story ID**: `STORY-NNN-{slug}` — ej: `STORY-001-login-form` (numerado por PRD, arranca en 001 cada PRD)
- **Epic branch**: `epic/PRD-NNN-{slug}` — ej: `epic/PRD-001-user-auth`
- **Base branch**: configurable por PRD (default `main`). De acá sale el epic, y acá se mergea al final.

**No hay ramas por story.** Todas las stories de un PRD se commitean directamente sobre la epic.

---

## Status de las stories

| Status | Significado |
|--------|-------------|
| `todo` | Generada, sin plan todavía |
| `in-progress` | Plan creado, work en marcha |
| `done` | Implementada y commiteada en la epic branch |
| `blocked` | Esperando a que otra story termine |

El agente actualiza el `status` en el frontmatter de cada story automáticamente al ejecutar `/plan` (→ in-progress) e `/implement` (→ done).

---

## El flujo paso a paso

### 1. `/create-prd`

**Cuándo**: arrancás un feature nuevo, todavía no escribiste nada.

**Qué hace**: tomás una conversación con el agente (o un brief) y genera un Product Requirements Document completo: executive summary, user stories iniciales, MVP scope, stack técnico, fases, riesgos.

**Qué te deja**:
- `.agents/PRDs/PRD-NNN-{slug}/PRD.md` — el doc completo
- `.agents/PRDs/PRD-NNN-{slug}/index.md` — tablero vacío

**Tip**: si tenés dudas, el agente te las pregunta antes de generar. No te asustes si te frena — es mejor ahí que después.

**Ejemplo**:
```
/create-prd
> "Quiero un sistema de auth con email/password, JWT, y logout."
```

---

### 2. `/create-stories <PRD>`

**Cuándo**: ya revisaste el PRD y querés bajarlo a tareas concretas.

**Qué hace**: lee el PRD, lo rompe en stories chicas (1-2 días de trabajo cada una), las ordena por dependencias, y crea un archivo por story.

**Qué te deja**:
- `.agents/stories/PRD-NNN-{slug}/STORY-001-{slug}.md`, `STORY-002-...`, etc.
- `index.md` actualizado con la tabla de stories

**Cada story incluye**:
- Frontmatter con metadata (`id`, `status`, `epic_branch`, `complexity`, `depends_on`, etc.)
- Description en formato "As a [user], I want [action], so that [benefit]"
- Acceptance Criteria (Given/When/Then)
- Technical Notes
- Dependencies

**Tip**: revisá las stories antes de seguir. Si alguna es muy grande, el agente te avisa, pero a veces se le pasa. Mejor que tengas 6 stories chicas que 2 monstruos.

**Ejemplo**:
```
/create-stories .agents/PRDs/PRD-001-user-auth/PRD.md
```

---

### 3. `/plan <STORY>`

**Cuándo**: querés implementar una story específica.

**Qué hace**:
1. Lee la story y verifica que las dependencias estén `done`.
2. Explora el codebase para encontrar patterns (cómo se nombran cosas, cómo se manejan errores, cómo son los tests existentes).
3. Diseña qué archivos crear/modificar y en qué orden.
4. Genera un plan detallado con tareas atómicas y referencias `file:line` a código que tiene que "espejar".
5. Cambia el status de la story de `todo` → `in-progress`.

**Qué te deja**:
- `.agents/plans/PRD-NNN-{slug}/STORY-NNN-{slug}.plan.md`
- Story frontmatter actualizado (`plan: <path>`, `status: in-progress`)
- `index.md` regenerado

**Por qué importa**: este es el paso donde el agente "piensa con el repo en la cabeza". Sin esto, te genera código genérico que no encaja con tus convenciones. **No te saltees este paso.**

**Tip**: leé el plan antes de ejecutar. Si algo no te cierra, edítalo a mano. El agente lo respeta cuando llega `/implement`.

**Ejemplo**:
```
/plan .agents/stories/PRD-001-user-auth/STORY-001-login-form.md
```

---

### 4. `/implement <plan>`

**Cuándo**: el plan ya está revisado y querés que el agente lo codee.

**Qué hace** (en fases, con validación entre cada una):

1. **PREPARE GIT**:
   - Si la epic branch no existe → la crea desde `base_branch` (`main` por default).
   - Si ya existe → te switchea a ella.
   - **Nunca crea ramas por story.**
2. **EXECUTE**: ejecuta cada task del plan, valida después de cada cambio (`npm run lint`, import checks, etc.).
3. **VALIDATE**: corre todos los checks + escribe tests + ejecuta los E2E del plan. **Hard gate**: si esto falla, no avanza.
4. **COMMIT**: hace `git add` selectivo (solo los archivos de esta story) y un commit con formato Conventional Commits. Captura el SHA.
5. **REPORT**: genera un report con todo lo que se hizo, archiva el plan en `completed/`.
6. **UPDATE STATUS**: status de la story → `done`, escribe el `commit` SHA en el frontmatter, regenera `index.md`.
7. **NEXT STEPS**: te tira los comandos para mergear la epic a main cuando termines todas las stories.

**Qué te deja**:
- Código + tests committeados en la epic branch
- `.agents/reports/PRD-NNN-{slug}/STORY-NNN-{slug}.report.md`
- Plan movido a `.agents/plans/PRD-NNN-{slug}/completed/`
- Story con `status: done` y `commit: <SHA>`

**Ejemplo**:
```
/implement .agents/plans/PRD-001-user-auth/STORY-001-login-form.plan.md
```

---

### 5. Loop: `/plan` + `/implement` por cada story restante

Repetís estos dos comandos para cada story del PRD. Todas se commitean en la misma epic branch, una atrás de otra.

```
/plan <STORY-002> && /implement <plan-002>
/plan <STORY-003> && /implement <plan-003>
...
```

(En la práctica los corrés uno por uno, revisando entre medio.)

---

### 6. Merge manual al final

Cuando todas las stories del PRD están `done`:

```bash
# Revisás el diff completo de la epic vs main
git diff main...epic/PRD-001-user-auth
git log --oneline main..epic/PRD-001-user-auth

# Si está todo OK, mergeás
git checkout main
git pull --ff-only
git merge --no-ff epic/PRD-001-user-auth
git push
```

**El agente nunca mergea automáticamente.** Vos decidís cuándo está listo.

---

### Bonus: `/prime`

**Cuándo**: arrancás una sesión nueva y querés que el agente entienda el repo + el contexto del PRD/story actual.

**Qué hace**: lee el PRD activo, las stories, el progreso, la estructura del backend/frontend, los últimos commits. No escribe nada.

**Ejemplo**:
```
/prime PRD-001
/prime STORY-001
```

---

## Reglas de oro

1. **Una story = un commit.** No metas dos stories en el mismo commit ni partas una en varios.
2. **No te saltees `/plan`.** Sin plan, el agente te tira código genérico.
3. **Revisá antes de ejecutar.** El plan, las stories y el PRD son tuyos — editalos a mano si hace falta.
4. **`stories/*.md` es la verdad.** Si tocás algo manualmente, tocá el frontmatter de la story, no el `index.md` (se regenera).
5. **No mergees a main hasta terminar la épica.** Mantené todo el trabajo del PRD junto en la epic branch hasta que esté completo.
6. **Si el E2E falla, no avances.** El agente está obligado a frenar — no lo presiones.

---

## Errores comunes y cómo evitarlos

| Síntoma | Causa probable | Cómo arreglar |
|---------|----------------|---------------|
| El agente quiere borrar archivos no relacionados | `git add -A` o `git add .` | Pedirle staging selectivo. El comando `/implement` ya lo hace bien. |
| Stories enormes que no terminan más | `/create-stories` no las partió | Editá el archivo a mano, partilas en 2-3, regenerá el index |
| Plan que no respeta convenciones del repo | Te saltaste `/plan` o lo corriste sin contexto | Volvé a correr `/plan`, leé las "Patterns to Follow" |
| `index.md` desincronizado | Editaste un story a mano y no regeneraste | Cualquier `/plan` o `/implement` siguiente lo regenera. O pedile al agente "regenerá el index" |
| Branch `epic/...` con commits sueltos sin story | Commiteaste a mano sobre la epic | OK pero anotá el SHA en alguna story, sino se pierde la traza |
| Conflictos al mergear epic→main | Main avanzó mientras laburabas en la epic | `git merge main` adentro de la epic primero, resolvé, después mergeás al revés |

---

## Glosario rápido

- **PRD** (Product Requirements Document): el "qué y por qué" del feature.
- **Story** (User Story): una unidad de valor entregable, formato "As a / I want / So that".
- **Plan**: diseño técnico de cómo implementar UNA story.
- **Report**: resumen post-mortem de qué se hizo en una story.
- **Epic branch**: rama git compartida donde se commitean todas las stories de un PRD.
- **Acceptance Criteria**: lo que tiene que cumplirse para considerar una story `done`.
- **Frontmatter**: el bloque `---` arriba de cada `.md` con metadata YAML.

---

## ¿Por dónde arranco?

1. Cloneá el repo y leé el `AGENTS.md` raíz (te explica conventions del backend/frontend).
2. Pensá un feature chiquito (algo que normalmente harías en 2-4 días).
3. Corré `/create-prd` y conversá con el agente para definirlo.
4. `/create-stories` para bajarlo a tareas.
5. `/plan` la primera story, revisalo.
6. `/implement` esa story.
7. Mirá el diff (`git show`), entendé qué pasó.
8. Repetí con la próxima story.

Si te trabás, abrí el archivo del comando en `.agents/commands/` y leé qué hace exactamente. **Todo el workflow es transparente**.
