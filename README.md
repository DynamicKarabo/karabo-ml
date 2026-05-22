# kb — Platform & MLOps CLI

**`pip install karabo-ml`** — one CLI to query docs, check cluster health, manage infrastructure, and run RAG pipelines from your terminal.

[![CI](https://github.com/karabo-labs/karabo-ml/actions/workflows/ci.yml/badge.svg)](https://github.com/karabo-labs/karabo-ml/actions/workflows/ci.yml)
[![PyPI version](https://img.shields.io/pypi/v/karabo-ml.svg)](https://pypi.org/project/karabo-ml/)
[![Python versions](https://img.shields.io/pypi/pyversions/karabo-ml.svg)](https://pypi.org/project/karabo-ml/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/docker-ghcr.io-blue?logo=docker)](https://github.com/karabo-labs/karabo-ml/pkgs/container/karabo-ml)
[![GitHub stars](https://img.shields.io/github/stars/karabo-labs/karabo-ml?style=social)](https://github.com/karabo-labs/karabo-ml)

```
kb rag query "how to setup ArgoCD"
kb rag chat
kb drift check
kb cluster status
kb config show
```

---

## Why `kb`?

Platform engineering shouldn't mean context-switching between docs, dashboards, and terminals. `kb` unifies three jobs into one CLI:

- **🧠 RAG assistant** — query your infrastructure docs in natural language, get answers with citations. No more grepping through 15 READMEs.
- **🔍 Drift detection** — sanity-check service health and vector collection status with a single command.
- **📊 Cluster overview** — see Docker + k3s state at a glance. JSON output for piping into your monitoring stack.

Built for engineers who value their keystrokes. Every subcommand outputs clean tables with Rich — no noise, no pagination traps.

---

## Quick Start

**pip install**
```bash
pip install karabo-ml
```
Output: Installs `kb` CLI. Run `kb --help` to verify.

**Docker**
```bash
docker run --rm ghcr.io/karabo-labs/karabo-ml --help
```
Output: CLI help text. Container exits after execution.

**From source**
```bash
git clone https://github.com/karabo-labs/karabo-ml.git
cd karabo-ml
pip install -e .
```
Output: Editable install. Run `kb --help` to verify.

---

## Commands

### `rag` — Documentation queries

| Subcommand | What it does |
|------------|-------------|
| `query` | Ask a DevOps question against indexed docs |
| `chat` | Interactive multi-turn Q&A session |

```bash
kb rag query "How do I set up ArgoCD in a k3s cluster?"
kb rag query --top-k 10 --json "What is the best way to configure Prometheus?"
kb rag chat
```

**`query`** returns a synthesised answer with citations from indexed documentation. The `--json` flag gives you the top-N relevant chunks plus the answer as structured JSON.  
**`chat`** drops you into an interactive REPL — type questions, receive answers. `/exit` to quit.

---

### `model` — RAG API lifecycle

| Subcommand | What it does |
|------------|-------------|
| `serve` | Start the RAG backend (FastAPI + Qdrant) |
| `stop`  | Tear down all services |

```bash
kb model serve                 # via docker-compose
kb model serve --build         # rebuild images first
kb model serve --profile ingest  # include ingestion pipeline
kb model stop
```

`serve` streams container logs to your terminal. API available at `http://localhost:8000`. `stop` removes containers cleanly.

---

### `drift` — Health checks

| Subcommand | What it does |
|------------|-------------|
| `check` | Verify API + Qdrant are reachable |
| `collections` | List Qdrant vector collections |

```bash
kb drift check
kb drift collections
```

Health status per service (OK / FAIL). Collections returns a table of names and metadata.

---

### `cluster` — Infrastructure monitoring

| Subcommand | What it does |
|------------|-------------|
| `status` | Show Docker + k3s cluster state |
| `logs`   | View service logs |

```bash
kb cluster status
kb cluster status --json
kb cluster logs api
kb cluster logs qdrant --tail 200
```

`status` shows a table of services with status and resource usage. `--json` for programmatic consumption. Logs tail defaults to 50 lines.

---

### `config` — Settings management

| Subcommand | What it does |
|------------|-------------|
| `init` | Create default config file |
| `show` | Print current effective configuration |
| `edit` | Open config in `$EDITOR` |

```bash
kb config init
kb config show
kb config edit
```

Config loaded in priority order:
1. `~/.kb/config.yaml` (defaults)
2. Environment variables (override file values)

```bash
kb config init
```

Example `~/.kb/config.yaml`:
```yaml
api:
  url: http://localhost:8000
  timeout: 30
qdrant:
  url: http://localhost:6333
  collection: devops_docs
logging:
  level: INFO
  file: ~/.kb/kb.log
```

Environment variables map to config keys: `KB_API_URL`, `KB_QDRANT_URL`, `KB_LOG_LEVEL`, etc.

---

### `completions` — Shell tab-completions

| Subcommand | What it does |
|------------|-------------|
| `install` | Register completions permanently |
| *(print)* | Output completions script to stdout |

```bash
kb completions install bash    # appends to ~/.bashrc
kb completions install zsh     # appends to ~/.zshrc
kb completions bash            # print script without installing
```

Completions appended to your shell rc file. Restart shell or `source` to activate.

---

## Architecture

```
kb CLI  ---- RAG API (FastAPI) ---- Qdrant (vectors)
          \                      \
           \-- OpenRouter (LLM)   \-- Web UI (HTMX)
```

Data flow: `kb rag query` → RAG API → retrieve from Qdrant → augment prompt → call OpenRouter → return answer. Web UI uses the same backend.

---

## Tech Stack

| Layer | Tool |
|-------|------|
| CLI framework | Click + Rich |
| HTTP client | httpx |
| Config | PyYAML + env vars |
| Backend API | FastAPI + Qdrant |
| LLM | OpenRouter (OpenAI-compatible) |
| Web UI | FastAPI + HTMX + Jinja2 |
| Container | Docker multi-stage |
| Orchestration | Docker Compose / k3s |
| CI/CD | GitHub Actions |
| Distribution | PyPI + GHCR |

---

## Development

```bash
# Clone and install with dev dependencies
git clone https://github.com/karabo-labs/karabo-ml.git
cd karabo-ml
pip install -e ".[dev]"

# Lint and format
ruff check src/
ruff format src/

# Build
python -m build

# Run tests
pytest

# Serve the web UI locally
kb model serve
```

> New to the project? Start with `kb --help`, then try `kb rag chat`. Contributions welcome — open an issue or PR.

---

## Ecosystem

`kb` is part of the [**karabo-labs**](https://github.com/karabo-labs) ecosystem — a growing set of tools for platform engineering, MLOps, and developer experience.

Related projects:
- [karabo-ml](https://github.com/karabo-labs/karabo-ml) — you are here
- *(more coming soon)*

---

<p align="center">
  <a href="https://github.com/karabo-labs">karabo-labs</a> ·
  <a href="https://github.com/karabo-labs/karabo-ml">karabo-ml</a> ·
  <a href="https://pypi.org/project/karabo-ml/">PyPI</a>
  <br>
  <sub>Built with ❤️ for platform engineers</sub>
</p>
