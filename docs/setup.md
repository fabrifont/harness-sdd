# Setup — De template a proyecto concreto

> Protocolo para transformar este template genérico en un proyecto concreto.
> El `leader` ejecuta este protocolo cuando el usuario pide "levantar el
> repositorio", "configura el proyecto" o cuando detecta que el repo aún está
> en estado de plantilla (hay placeholders `{{...}}` sin rellenar).

---

## 1. Estado de plantilla

El repo está en **estado de plantilla** si `feature_list.json` contiene
`{{PROJECT_NAME}}` o cualquier otro placeholder sin sustituir.

En ese modo, **no se implementan features**. Primero hay que configurar el
arnés para el stack concreto.

---

## 2. Cuestionario de configuración

El leader presenta este cuestionario al humano. Todas las respuestas se usan
para rellenar placeholders y tomar decisiones técnicas iniciales. Si el
humano no responde alguna pregunta, el leader propone un valor por defecto
razonable para el stack declarado.

> **Regla:** no se avanza al Paso 3 (rellenar placeholders) hasta que el
> cuestionario esté completo o el humano explícitamente diga "usa los
> defaults".

---

### A. Identidad del proyecto

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| A1 | **Nombre del proyecto** | Se usa en `feature_list.json`, `README.md`, y archivos de config (package.json, Cargo.toml, go.mod). | `task-tracker` |
| A2 | **Descripción en una línea** | Aparece en `feature_list.json` y `README.md`. Define el dominio para que los specs sean coherentes. | "API REST para gestión de tareas personales" |
| A3 | **Dominio / problema que resuelve** | Ayuda a los subagentes a no inventar features incongruentes. | "CRUD de tareas con prioridades y etiquetas" |

---

### B. Stack tecnológico

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| B1 | **Lenguaje principal** | Determina `docs/conventions.md`, estructura de archivos, `.gitignore`. | Python, TypeScript, Go, Rust, Java, Ruby |
| B2 | **Versión mínima del runtime** | Se valida en `init.sh` y documenta requisitos. | Python 3.11, Node 20, Go 1.21, Rust 1.75 |
| B3 | **Framework / librería base** (si aplica) | Define las capas iniciales en `docs/architecture.md`. | FastAPI, Express, Flask, Axum, Actix, Spring Boot |
| B4 | **Gestor de dependencias** | Determina archivos de manifesto a crear. | pip + requirements.txt, poetry, npm, cargo, maven |
| B5 | **¿Se permiten dependencias externas?** | Si "no", el `init.sh` y `docs/architecture.md` bloquean su uso. | Sí / Solo stdlib / Solo aprobadas explícitamente |

---

### C. Tipo de proyecto y arquitectura

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| C1 | **Tipo de proyecto** | Define qué ejemplos usar en `docs/verification.md` y qué agente líder esperar. | CLI, API REST, API GraphQL, Librería, Web app, Microservicio, Desktop, Script |
| C2 | **Arquitectura deseada** | El `spec_author` respeta estas capas al diseñar. | Monolito de 3 capas (UI / Dominio / Persistencia), Hexagonal, Clean Architecture, Serverless functions |
| C3 | **Capas concretas** (lista) | Se documentan en `docs/architecture.md` como contracto. | `api.py` (rutas), `services.py` (lógica), `models.py` (dominio), `db.py` (persistencia) |
| C4 | **¿Hay interfaz de usuario?** | Si es CLI: se define parser de args. Si es Web: se define framework frontend. | No / CLI con argparse / CLI con click / Web React / Web Vue |
| C5 | **Entry point principal** | Se usa en todos los ejemplos de `docs/` y en `init.sh`. | `python -m src.cli`, `npm start`, `go run .`, `cargo run` |
| C6 | **Puerto o URL base** (si aplica) | Para APIs y servicios. | `http://localhost:8000`, `http://localhost:3000` |

---

### D. Testing y verificación

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| D1 | **Test runner** | Se documenta en `docs/verification.md` y se usa en `init.sh`. | pytest, jest, vitest, go test, cargo test, unittest |
| D2 | **Comando para ejecutar tests** | LITERAL. Se pega en `docs/verification.md` y `init.sh`. | `python3 -m pytest`, `npm test`, `go test ./...` |
| D3 | **¿Tests de integración obligatorios?** | Si "sí", el `reviewer` exige tests que invoquen el proceso real. | Sí / Solo unitarios / Opcional |
| D4 | **Política de mocking** | El `reviewer` rechaza mocks innecesarios si la política dice "entornos reales". | Tempfiles reales / In-memory / Mocks solo para red/DB externa |
| D5 | **Cobertura mínima esperada** | El `reviewer` puede usarlo como referencia (no duro). | 80% / No hay mínimo / 100% de funciones públicas |
| D6 | **¿Hay CI/CD configurado?** | Indica si el `init.sh` debe validar archivos de CI. | No / GitHub Actions / GitLab CI / Otro |

---

### E. Persistencia y datos

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| E1 | **¿Persistencia?** | Define si `docs/architecture.md` incluye capa de storage. | No (solo memoria) / JSON en disco / SQLite / PostgreSQL / MongoDB |
| E2 | **Ruta o conexión por defecto** | Para tests temporales y smoke tests. | `./data.json`, `sqlite:///./app.db`, env var `DATABASE_URL` |
| E3 | **¿Migraciones o schema versioning?** | Si "sí", el primer spec de persistencia incluye migración inicial. | No / Sí, manual / Sí, con herramienta (Alembic, Flyway) |
| E4 | **¿Datos de seed / fixtures?** | El `implementer` puede crear fixtures si se necesitan para tests. | No / JSON estático / Factory functions |

---

### F. Interfaz y experiencia de usuario

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| F1 | **Formato de salida** (CLI) | Los specs de CLI deben ser consistentes. | Texto plano / JSON / Tabla / Markdown |
| F2 | **Idioma de los mensajes de usuario** | Los specs EARS y errores usan este idioma. | Español / Inglés / Bilingüe (errores en inglés, UI en español) |
| F3 | **Manejo de argumentos** (CLI) | El `spec_author` sabe qué librería usar. | argparse (stdlib) / click / typer / cobra |
| F4 | **Autenticación** (API/Web) | Si aplica, se documenta en `docs/architecture.md`. | No / API key / JWT / OAuth2 / Basic Auth |
| F5 | **Documentación de API** (API) | El `implementer` puede generar docs automáticamente. | No / OpenAPI (Swagger UI) / GraphQL introspection / README manual |

---

### G. Calidad de código y estilo

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| G1 | **Linter / Formatter** | Se ejecuta en `init.sh` opcionalmente o se menciona en convenciones. | ruff, black, eslint, prettier, gofmt, rustfmt |
| G2 | **Guía de estilo** | Se documenta en `docs/conventions.md`. | PEP 8, Google Style, Airbnb, StandardJS, gofmt |
| G3 | **Sistema de tipos** | Afecta `docs/conventions.md` y qué tan estricto es el reviewer. | Estático (mypy, TypeScript, Rust) / Dinámico (Python sin mypy) / Gradual |
| G4 | **Longitud máxima de línea** | Se pega en `docs/conventions.md`. | 100 / 88 (black) / 120 |
| G5 | **¿Pre-commit hooks?** | Si "sí", se configuran en `.pre-commit-config.yaml` o similar. | No / Sí, con husky / Sí, con pre-commit |

---

### H. Configuración y entorno

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| H1 | **Variables de entorno obligatorias** | El `init.sh` puede validar que existan. | `DATABASE_URL`, `API_KEY`, `PORT` |
| H2 | **Archivo de configuración** | El `spec_author` sabe si diseñar parsing de config. | No / `.env` / `config.yaml` / `config.json` / CLI flags únicamente |
| H3 | **Secretos / credenciales** | Reglas de `.gitignore` y convenciones de manejo. | `.env` en `.gitignore`, nunca en código / Vault / AWS Secrets Manager |

---

### I. Build, despliegue y operaciones

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| I1 | **Build step** | Se documenta para que el `implementer` no olvide compilar. | No / `npm run build` / `cargo build --release` / `go build` |
| I2 | **Docker / Containerización** | Si "sí", se puede añadir `Dockerfile` en el setup. | No / Sí, Dockerfile / Sí, docker-compose / Sí, Kubernetes manifests |
| I3 | **Sistema de logs** | Los specs de error handling incluyen logging. | print a stderr / logging stdlib / logrus / tracing (OpenTelemetry) |
| I4 | **Observabilidad** (metrics, health checks) | Si aplica, se documentan endpoints o convenciones. | No / Health endpoint `/health` / Métricas Prometheus / Tracing |

---

### J. Reglas específicas para agentes

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| J1 | **¿Qué NO debe hacer un agente en este proyecto?** | Reglas adicionales al `CHECKPOINTS.md`. | "No toque la base de datos sin migrations" / "No añada dependencias nuevas sin aprobación" / "No modifique CI/CD" |
| J2 | **Patrones prohibidos** | El `reviewer` los detecta y rechaza. | "No usar singletons globales" / "No usar ORM para queries complejas" / "No hardcodear URLs" |
| J3 | **Decisiones ya tomadas que no se pueden cambiar** | Evita que el `spec_author` proponga alternativas inválidas. | "Usamos SQLAlchemy, no cambiar" / "Puerto fijo 8080" / "Autenticación ya está en middleware X" |
| J4 | **Librerías / herramientas ya aprobadas** | Lista blanca de dependencias. | `fastapi`, `pydantic`, `sqlalchemy`, `pytest` |
| J5 | **¿Hay código legacy o bibliotecas propietarias?** | El `spec_author` debe conocerlos para no romper compatibilidad. | No / Sí, módulo `legacy_core.py` que no se toca / Sí, dependencia interna `@corp/utils` |

---

### K. Features iniciales (opcional pero recomendado)

| # | Pregunta | Por qué importa | Ejemplo |
|---|----------|----------------|---------|
| K1 | **¿Ya tienes features definidas?** | Si "sí", se rellenan en `feature_list.json`. | No / Sí, lista en archivo adjunto / Sí, las enumero ahora |
| K2 | **¿Feature #1?** | Se convierte en el primer item de `feature_list.json` reemplazando `example_feature`. | "Endpoint POST /tasks para crear tarea" |
| K3 | **¿Feature #2?** | Segunda feature, si aplica. | "Endpoint GET /tasks para listar tareas" |
| K4 | **¿Feature #3?** | Tercera feature, si aplica. | "Comando CLI `list` para ver tareas en terminal" |

> Si el humano no tiene features definidas, se deja `example_feature` como
> placeholder instructivo en `feature_list.json`.

---

## 3. Proceso de ejecución del cuestionario

```
Humano: "Configura este proyecto para una API REST en Python 3.11 con FastAPI"
  ↓
Leader: Detecta estado de plantilla → presenta cuestionario
  ↓
Humano: Responde las preguntas (o dice "usa defaults para Python API")
  ↓
Leader: Rellena placeholders (Paso 4)
  ↓
Leader: Configura archivos (Paso 5)
  ↓
Leader: Verifica con ./init.sh
  ↓
Leader: "Setup completado. Repo listo para features."
```

---

## 4. Rellenar placeholders

Tras completar el cuestionario, el leader edita archivos sustituyendo
**todos** los `{{...}}`:

#### `feature_list.json`
- `{{PROJECT_NAME}}` → nombre del proyecto
- `{{PROJECT_DESCRIPTION}}` → descripción
- Reemplazar `example_feature` con features iniciales (K2-K4) o dejar como
  instructivo si no hay.

#### `README.md`
- Actualizar título y descripción.

#### `docs/conventions.md`
- `{{LANGUAGE}}` → lenguaje (B1)
- `{{VERSION}}` → versión mínima (B2)
- `{{STYLE_GUIDE}}` → guía de estilo (G2)
- `{{INTERPOLATION}}` → interpolación típica del lenguaje
- `{{ENTRY_POINT}}` → entry point (C5)

#### `docs/verification.md`
- `{{TEST_COMMAND}}` → comando de tests (D2)
- `{{ENTRY_POINT}}` → entry point (C5)

#### `docs/specs.md`
- `{{ENTRY_POINT}}` → entry point (C5)

#### `docs/architecture.md`
- Adaptar el ejemplo de capas al stack real. Añadir una nota con las capas
  concretas declaradas en C3.

---

## 5. Configurar `.gitignore`

El leader reescribe `.gitignore` con patrones estándar del stack.

Plantillas por stack (ejemplos):

**Python:**
```
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.venv/
.env
*.tmp
.DS_Store
```

**Node / TypeScript:**
```
node_modules/
dist/
build/
*.log
.env
*.tmp
.DS_Store
```

**Go:**
```
bin/
*.tmp
.DS_Store
```

**Rust:**
```
target/
*.tmp
.DS_Store
```

**Java / Maven:**
```
target/
*.class
*.jar
*.tmp
.DS_Store
```

**Ruby:**
```
*.gem
.bundle/
pkg/
*.tmp
.DS_Store
```

---

## 6. Configurar `init.sh` (si el stack lo requiere)

Si el stack necesita verificaciones adicionales, el leader añade secciones
a `init.sh` sin reescribirlo completamente.

Ejemplos de extensiones comunes:

**Node:**
```bash
if ! command -v node >/dev/null 2>&1; then
  fail "node no está instalado"
  exit 1
fi
ok "node -> $(node --version)"

if [ ! -d "node_modules" ]; then
  warn "node_modules no encontrado. Ejecuta 'npm install'"
fi
```

**Go:**
```bash
if ! command -v go >/dev/null 2>&1; then
  fail "go no está instalado"
  exit 1
fi
ok "go -> $(go version)"
```

**Rust:**
```bash
if ! command -v cargo >/dev/null 2>&1; then
  fail "cargo no está instalado"
  exit 1
fi
ok "cargo -> $(cargo --version)"
```

---

## 7. Crear estructura inicial mínima

Crear archivos vacíos o mínimos para que el proyecto compile/ejecute desde
día 0:

**Python:**
```
src/__init__.py
tests/__init__.py
```

**Node / TypeScript:**
```
src/index.ts           # export vacío o Hello World
package.json           # mínimo: { "name": "...", "version": "1.0.0" }
tsconfig.json          # si aplica
tests/.gitkeep         # o test mínimo
```

**Go:**
```
go.mod                 # `go mod init <nombre>`
main.go                # package main + func main()
```

**Rust:**
```
Cargo.toml             # `cargo init --name <nombre>`
src/main.rs            # fn main() {}
```

**Java / Maven:**
```
pom.xml                # mínimo
src/main/java/.gitkeep
src/test/java/.gitkeep
```

> El leader **no** implementa lógica de negocio. Solo crea la estructura
> mínima para que `init.sh` y los tests unitarios tengan algo que verificar.

---

## 8. Verificar

Ejecutar `./init.sh`. Debe terminar verde. En estado de plantilla, 0 tests
es válido, pero el entorno debe estar sano.

---

## 9. Documentar en `progress/history.md`

Añadir entrada:

```markdown
## <fecha> — Setup inicial

- **Agente:** leader
- **Stack:** <lenguaje> <version> + <framework>
- **Tipo:** <CLI/API/Librería/...>
- **Cambios:** Configurado template para <descripción>. Rellenados placeholders,
  creada estructura mínima, configurado `.gitignore`.
- **Resultado:** Entorno listo, esperando primera feature.
```

---

## 10. Reglas post-setup

1. **Una vez completado, el repo deja de estar en estado de plantilla.**
   El leader no vuelve a ejecutar este protocolo salvo que el humano pida
   re-configurar explícitamente (ej: "migrar a Rust").

2. **Si el humano pide re-configurar**, se considera una nueva sesión.
   Se documenta en `progress/history.md` y se ejecuta el cuestionario
   completo de nuevo.

3. **No se implementan features durante el setup.** Si el humano pide
   "configura y luego implementa la feature X", el leader responde:
   "Primero completo el setup, luego pasamos a features."
