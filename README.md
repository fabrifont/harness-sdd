# harness-sdd — Template genérico para desarrollo con agentes de IA

Template de **Harness Engineering** para desarrollo de software con agentes
de IA siguiendo **Spec Driven Development**.

> Lo importante de este repo no es **qué** aplicación contiene, sino **cómo**
> está estructurado para que un agente de IA pueda trabajar sobre él de forma
> autónoma y verificable.

## Cómo está organizado el arnés

| Pilar                                  | Manifestación en este repo                                                       |
|----------------------------------------|----------------------------------------------------------------------------------|
| **1. El repositorio ES el sistema**    | `AGENTS.md`, `init.sh`, `feature_list.json`, `specs/`, `progress/`, `docs/`      |
| **2. Orquestación multi-agente**       | `.opencode/agents/leader.md`, `spec_author.md`, `implementer.md`, `reviewer.md` |
| **3. Spec Driven Development**         | `docs/specs.md`, EARS notation, puerta de aprobación humana en `spec_ready`      |
| **4. Supervisión y mejora**            | `CHECKPOINTS.md`, `tests/`                                                       |

## Para empezar

### 1. Si acabas de clonar el template (estado de plantilla)

El repo contiene placeholders `{{...}}` que deben rellenarse para tu stack.

Abre Opencode en la raíz y pide:
> **«Configura este proyecto para un [API REST en Python 3.11]»**

El `leader` ejecutará el protocolo de `docs/setup.md`:
- Rellenará placeholders en `feature_list.json`, `docs/`, `README.md`.
- Configurará `.gitignore` y estructura mínima del stack.
- Verificará con `./init.sh`.

**No implementes features hasta completar el setup.**

### 2. Si el repo ya está configurado

```bash
./init.sh
```

Si todo está verde, abre `AGENTS.md` y sigue desde ahí.

## Para usar con Opencode

Si te descargas el repo y abres Opencode en la raíz, ya estás dentro del
arnés: `.opencode/AGENTS.md` fuerza al modelo a actuar como `leader`
(orquesta, no edita código) y `docs/specs.md` impone el flujo Spec Driven
Development.

Receta rápida:

1. `./init.sh` — debe terminar verde.
2. Edita `feature_list.json` y define tus features con
   `status: "pending"` y `"sdd": true`.
3. Abre Opencode en la raíz del repo.
4. Pídele: **«implementa la siguiente feature pendiente»**.

Lo que ocurre, en dos fases:

**Fase 1 — Spec.** El `leader` lanza un `spec_author` que escribe
`specs/<feature>/{requirements.md, design.md, tasks.md}` y deja la feature
en `spec_ready`. Luego **para y te pide aprobación**.

Tú lees los tres archivos en tu editor:

- `requirements.md` — qué debe hacer la feature, en EARS estricto.
- `design.md` — decisiones técnicas antes de escribir código.
- `tasks.md` — checklist de pasos discretos a ejecutar.

Cuando estés conforme, dices al chat «aprobado» (o pides cambios).

**Fase 2 — Código.** El `leader` transiciona la feature a `in_progress` y
lanza `implementer` (sigue las tasks una a una marcándolas `[x]`) y
después `reviewer` (verifica trazabilidad `R<n>` ↔ test y todas las tasks
completas).

Dónde queda la traza de cada subagente:

| Archivo                                  | Quién lo escribe   | Qué contiene                                                  |
|------------------------------------------|--------------------|---------------------------------------------------------------|
| `specs/<feature>/requirements.md`        | spec_author        | EARS requirements numeradas `R1`, `R2`, ...                  |
| `specs/<feature>/design.md`              | spec_author        | Decisiones técnicas + alternativa descartada                  |
| `specs/<feature>/tasks.md`               | spec_author        | Checklist; el implementer la va marcando `[x]`                |
| `progress/current.md`                    | leader             | Plan vivo de la sesión                                        |
| `progress/impl_<feature>.md`             | implementer        | Archivos tocados + mapa `R<n> → test` + output de los tests   |
| `progress/review_<feature>.md`           | reviewer           | Checklist contra `docs/`, `specs/<feature>/` y `CHECKPOINTS.md` |
| `feature_list.json`                      | leader/implementer | `pending` → `spec_ready` → `in_progress` → `done`             |
| `progress/history.md`                    | leader             | Resumen append-only al cerrar la sesión                       |

Abre `specs/` y `progress/` en tu editor mientras el agente trabaja: cada
informe aparece en cuanto el subagente termina. Esa es la regla
anti-teléfono-descompuesto en acción — el contenido no circula por chat,
vive en disco y queda versionado.

## Estructura

```
.
├── AGENTS.md              # Mapa para agentes (divulgación progresiva)
├── CHECKPOINTS.md         # Criterios de "estado final correcto"
├── feature_list.json      # Alcance: una feature a la vez
├── init.sh                # Verificación e inicialización
├── specs/<feature>/       # Spec por feature (Kiro-style)
│   ├── requirements.md    # EARS notation
│   ├── design.md          # Decisiones técnicas
│   └── tasks.md           # Checklist de implementación
├── progress/
│   ├── current.md         # Sesión activa (estado vivo)
│   └── history.md         # Bitácora append-only
├── docs/
│   ├── setup.md           # Protocolo para configurar el template desde cero
│   ├── architecture.md    # Qué significa "buen trabajo"
│   ├── conventions.md     # Estilo, nombres, errores
│   ├── specs.md           # Proceso SDD: EARS, 3 archivos, aprobación humana
│   └── verification.md    # Cómo demostrar que funciona
├── .opencode/
│   ├── agents/            # leader, spec_author, implementer, reviewer
│   └── AGENTS.md          # Instrucciones de inicio de sesión
├── src/                   # Código de la aplicación (vacío en el template)
└── tests/                 # Tests automáticos (vacío en el template)
```

## Aprendizajes que ilustra este template

- **Setup automático desde template**: `docs/setup.md` guía al leader para
  rellenar placeholders, configurar `.gitignore` y crear estructura mínima del
  stack sin intervención humana en los detalles.
- **Divulgación progresiva** en `AGENTS.md`: el agente no recibe todas las
  reglas de golpe, recibe un mapa para buscarlas bajo demanda.
- **Una feature a la vez** validado por `init.sh` (rechaza más de un
  `in_progress` en `feature_list.json`).
- **Spec Driven Development** estilo Kiro: requirements (EARS) → design →
  tasks → code, con una puerta de aprobación humana antes de tocar código.
- **Estado en disco**, no en chat: `specs/`, `progress/current.md` y
  `history.md` sobreviven a reinicios y context windows reventadas.
- **Verificación ejecutable**: `init.sh` corre los tests reales y valida
  la presencia de specs para toda feature SDD.
- **Trazabilidad obligatoria**: cada `R<n>` se mapea a un test concreto;
  el reviewer rechaza si falta.
- **Patrón Leader-Spec-Implementer-Reviewer**: el leader no implementa,
  el spec_author no codifica, el implementer no se autoaprueba, el
  reviewer no edita código.
- **Anti teléfono-descompuesto**: los subagentes escriben sus resultados
  en archivos y solo devuelven una referencia ligera.
