# Code dump: azfr-databricks-landing-zone

Generated from `/c/Users/LARBANI/repos_git/azfr-databricks-landing-zone`.

## File tree

```
.gitignore
CLAUDE.md
config/landing_config.yml
databricks.yml
docs/adr/001-two-thread-pools.md
docs/adr/002-unbounded-route-worker.md
docs/adr/003-instrumentation-before-delta-write.md
docs/adr/004-file-backend-benchmark.md
docs/adr/005-exactly-once-file-move-semantics.md
docs/adr/006-write-ahead-intent-file.md
docs/adr/007-raise-before-delete-phase-ordering.md
docs/adr/008-modification-time-as-file-identity-key.md
docs/adr/009-consecutive-run-case-analysis.md
docs/adr/010-fuse-stall-io-timeout.md
docs/adr/011-cost-simulation-webapp.md
docs/adr/012-per-route-autoloader-filename-reuse.md
docs/cost-simulation/prices-and-sources.md
docs/cost-simulation/webapp/app.py
docs/cost-simulation/webapp/README.md
Makefile
notebooks/benchmark/result-10GB/benchmark-10GB-analysis.md
notebooks/benchmark/result-10GB/benchmark-10GB-raw-output-1st-run.txt
notebooks/benchmark/result-10GB/benchmark-10GB-raw-output-2nd-run.txt
pyproject.toml
README.md
resources/__init__.py
resources/benchmark_job.py
resources/job_clusters.py
resources/landing_jobs.py
resources/targets.yml
src/azfr_databricks_landing_zone/__init__.py
src/azfr_databricks_landing_zone/config.py
src/azfr_databricks_landing_zone/file_copier.py
src/azfr_databricks_landing_zone/file_event.py
src/azfr_databricks_landing_zone/file_intent.py
src/azfr_databricks_landing_zone/fs_backend.py
src/azfr_databricks_landing_zone/instrumentation.py
src/azfr_databricks_landing_zone/main.py
src/azfr_databricks_landing_zone/timing.py
tests/__init__.py
tests/integration/__init__.py
tests/integration/test_process_batch.py
tests/unit/__init__.py
tests/unit/conftest.py
tests/unit/test_config.py
tests/unit/test_file_copier.py
tests/unit/test_file_event.py
tests/unit/test_file_intent.py
tests/unit/test_main.py
tests/unit/test_resolve_copiers.py
typings/__builtins__.pyi
uv.lock
```

## File contents

###### FILE: .gitignore ######

```gitignore
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
cover/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
.pybuilder/
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
# .python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock

# UV
#   Similar to Pipfile.lock, it is generally recommended to include uv.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#uv.lock

# poetry
#   Similar to Pipfile.lock, it is generally recommended to include poetry.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#   https://python-poetry.org/docs/basic-usage/#commit-your-poetrylock-file-to-version-control
#poetry.lock

# pdm
#   Similar to Pipfile.lock, it is generally recommended to include pdm.lock in version control.
#pdm.lock
#   pdm stores project-wide configurations in .pdm.toml, but it is recommended to not include it
#   in version control.
#   https://pdm.fming.dev/latest/usage/project/#working-with-version-control
.pdm.toml
.pdm-python
.pdm-build/

# PEP 582; used by e.g. github.com/David-OConnor/pyflow and github.com/pdm-project/pdm
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# pytype static type analyzer
.pytype/

# Cython debug symbols
cython_debug/

# PyCharm
#  JetBrains specific template is maintained in a separate JetBrains.gitignore that can
#  be found at https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore
#  and can be added to the global gitignore or merged into this file.  For a more nuclear
#  option (not recommended) you can uncomment the following to ignore the entire idea folder.
#.idea/

# Abstra
# Abstra is an AI-powered process automation framework.
# Ignore directories containing user credentials, local state, and settings.
# Learn more at https://abstra.io/docs
.abstra/

# Visual Studio Code
#  Visual Studio Code specific template is maintained in a separate VisualStudioCode.gitignore 
#  that can be found at https://github.com/github/gitignore/blob/main/Global/VisualStudioCode.gitignore
#  and can be added to the global gitignore or merged into this file. However, if you prefer, 
#  you could uncomment the following to ignore the enitre vscode folder
.vscode/

# Ruff stuff:
.ruff_cache/

# PyPI configuration file
.pypirc

# Cursor
#  Cursor is an AI-powered code editor. `.cursorignore` specifies files/directories to
#  exclude from AI features like autocomplete and code analysis. Recommended for sensitive data
#  refer to https://docs.cursor.com/context/ignore-files
.cursorignore
.cursorindexingignore
.databricks

```

###### FILE: CLAUDE.md ######

```md
﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**azfr-databricks-landing-zone** is a Databricks Asset Bundle (DAB) that implements a file-push pattern for ingesting files from Landing Volumes to Raw Volumes in Databricks Unity Catalog. Each file is logged as a LOADED or FAILED event in a per-schema Delta audit table (`_file_event_`).

### Key Characteristics

- **File-streaming ingestion**: Uses Databricks Autoloader (CloudFiles) to stream files from source volumes, copy them to timestamped partitioned targets, and record outcomes.
- **Exactly-once delivery guarantees**: Three-layer guarantee via Autoloader checkpoint + idempotent file ops + write-ahead intent store.
- **Concurrent route processing**: Supports multiple independent routes (source → target pairs) running in parallel via Spark foreachBatch and ThreadPoolExecutor.
- **Crash-safety**: Persists intent mappings to JSON before any copy starts; robust to mid-batch failures and retries.
- **Production-ready**: Deployed to Databricks via `databricks bundle` CLI; runs as a scheduled job triggered by file arrival.

### Technology Stack

- **Python 3.12** with `uv` package manager
- **PySpark** (Databricks Connect) for Spark integration
- **Pydantic** for configuration validation and type safety
- **pytest** for unit and end-to-end tests
- **Databricks CLI** for bundle deployment

### Cost Simulation Companion

The repository includes a Streamlit cost simulation app at
`docs/cost-simulation/webapp/app.py`.

It compares these architectures:

- `A`: 1 Job, 1 Autoloader
- `B`: 1 Job, N Autoloader
- `C`: N Job, N Autoloader (1 autoloader each)

for both compute modes: Serverless and Job Compute.

---

## Architecture & Design Patterns

### Two-Level Concurrency Model

The job runs multiple routes concurrently, and each route copies multiple files concurrently — a two-level nested model:

- **Route-worker thread pool** (unbounded): spawns one thread per active route, each running Autoloader foreachBatch
- **File-worker thread pool** (bounded, default 8): shared across all routes for concurrent file copy/delete tasks

**Why two pools?** A single pool causes thread-pool starvation deadlock: route threads occupy slots while waiting for file tasks to complete, blocking those tasks indefinitely. See `docs/adr/001-two-thread-pools.md`.

### Exactly-Once File Move Semantics

Three independent layers guarantee exactly-once delivery across crashes and retries:

1. **Autoloader checkpoint** (`{work_base}/{route_id}/checkpoint/`) — Autoloader does not advance the checkpoint until `foreachBatch` returns cleanly. Re-presents the same files on retry.

2. **Idempotent file operations** — `copy_file()` is an overwrite (safe to re-run); `delete_file()` swallows `FileNotFoundError` (idempotent). Supported backends: `os` (FUSE-mounted /Volumes/, production), `fsspec-databricks` (API-based), `dbutils` (fastest but incompatible with Autoloader).

3. **Write-ahead intent file** (`{work_base}/{route_id}/intent_store/_intent.json`) — Before any copy starts, `FileIntentStore` writes all `(source_path, modification_time_ms) → target_path` entries to JSON. On retry, stored target paths are reused instead of generating new UUIDs. After all copies, Delta writes, and deletions complete, the status is atomically updated to `SUCCESS`. Atomicity verified on UC Volumes FUSE via `os.replace()` (see `docs/adr/006-write-ahead-intent-file.md`).

See `docs/adr/005-exactly-once-file-move-semantics.md` and `docs/adr/009-consecutive-run-case-analysis.md` for exhaustive case analysis of all retry scenarios.

### Code Flow

**High-level batch processing:**

1. Autoloader reads source volume, yields micro-batches of file metadata (path, modificationTime, length).
2. `LandingMoverJob.process_batch()` is called via foreachBatch for each batch per route:
   - Resolve copy intents: read intent store, check audit for already-copied files, build FileCopiers, write-ahead new intents.
   - Execute copies concurrently via file-worker pool.
   - Write LOADED/FAILED events to Delta file-event table.
   - If any copy failed, raise to trigger Autoloader retry (don't proceed to delete).
   - Delete all source files (only when every file succeeded).
   - Mark intent store as SUCCESS.
3. Route-worker collects per-route results and raises BaseExceptionGroup if any route failed.

**Key classes:**

- `LandingMoverJob` — Orchestrates batch processing, copy execution, event recording, and source deletion for a single batch per route.
- `LandingRoute` — Configuration model for a single route (source, target, id, enabled flag); derives catalog/schema from target Volume path to locate the _file_event_ table.
- `FileCopier` — Encapsulates a single file copy task; builds idempotent target path and records LOADED/FAILED events.
- `FileIntentStore` — Manages write-ahead intent file for crash-safe batch idempotency; reads prior execution state on retry.
- `FileEventTable` — Writes LOADED/FAILED rows to Delta; queries table to find already-audited files during retry resolution.
- `FileSystemBackend` — Protocol for pluggable file operations (copy/delete); concrete implementations: NativeOsBackend (production), FsspecDatabricksBackend, DbutilsBackend (incompatible with Autoloader).
- `JobInstrumentation` — Records per-file and per-route metrics (throughput, errors) to per-batch JSON and log files; replayed by driver after foreachBatch completes.
- `BatchProcessor` — Picklable foreachBatch callable; wraps LandingMoverJob to support Spark Connect serialization (no shared SparkSession/ThreadPoolExecutor capture).

### Configuration

Routes and job settings are defined in a YAML file (default: `config/landing_config.yml`). See README.md for full schema documentation. Key fields:

- `file_backend`: "fsspec-databricks" (default, HTTP REST — avoids FUSE stall hangs), "os" (FUSE-mounted /Volumes/), or "dbutils" (unavailable in Autoloader executors). See `docs/adr/010-fuse-stall-io-timeout.md`.
- `work_base`: Base Volume path for per-route working directories (checkpoint, intent_store, log subdirectories).
- `landing_routes`: List of route definitions (id, source, target, enabled).
- `fault_inject_copy_failures` / `fault_inject_delete_failures`: Optional filenames for failure injection (integration testing only).

---

## Development Commands

### Setup & Dependencies

```bash
# Create virtual environment and install all dependencies (including dev)
uv sync

# Upgrade dependencies to latest compatible versions
make upgrade-dependencies

# Alias
make update
```

### Testing

```bash
# Run all tests (unit + integration)
uv run pytest

# Run unit tests only
uv run pytest tests/unit/

# Run integration tests only
uv run pytest tests/integration/

# Run a single test file
uv run pytest tests/unit/test_config.py

# Run a single test function
uv run pytest tests/unit/test_config.py::TestAppConfig::test_duplicate_id_raises

# Run with verbose output and show print statements
uv run pytest -vv -s tests/unit/test_config.py

# Generate test coverage report
make coverage

# Generate detailed HTML coverage + junit reports
make reports
```

### Linting & Formatting

```bash
# Run ruff linter on source and test code
uv run ruff check src tests
```

### Building & Deployment

```bash
# Build the Python wheel (required before Databricks deployment)
uv build --wheel

# Deploy to dev target (Databricks)
make databricks-deploy

# Force full file re-sync to Databricks (clears sync snapshots)
make databricks-deploy-force

# Run the job on dev target (after deploy)
databricks bundle run --target dev
```

### Running Locally

```bash
# Direct invocation (requires uv and Python 3.12)
uv run landing-mover --config-path config/landing_config.yml

# Override config path at runtime
uv run landing-mover --config-path /Volumes/<catalog>/<schema>/<volume>/_config/landing_config.yml
```

### Documentation

```bash
# Initialize API documentation project (first time only)
make init-docs

# Generate Sphinx API documentation
make docs

# Clean generated documentation
make clean-docs
```

### Cleanup

```bash
# Remove test artifacts, coverage, and docs
make clean

# Full cleanup including virtualenv
make distclean

# Remove built wheels from dist/
make clean-wheels
```

### Interactive Help

```bash
# Show all available make targets
make info
```

---

## Key Directories (Non-Exhaustive)

Use this as a stable map of the repository. Do not maintain an exhaustive file list here.

- `src/azfr_databricks_landing_zone/` — runtime code for routing, copy/move semantics, intent store, audit events, instrumentation, and entrypoint.
- `tests/unit/` — fast tests for config, copier/event/intent logic, and retry-resolution behavior.
- `tests/integration/` — end-to-end crash/retry and batch behavior tests.
- `docs/adr/` — Architecture Decision Records. See README.md for full index and descriptions
- `config/` — example/default runtime configuration.
- `notebooks/` — operational and benchmark notebooks.
- `databricks.yml` — Databricks Asset Bundle definitions.
- `pyproject.toml` — package metadata and dependency/build config.

---

## Common Development Scenarios

### Running a Single Test

```bash
# Test a specific test class
uv run pytest tests/unit/test_config.py::TestLandingRoute -vv

# Test a specific test method
uv run pytest tests/unit/test_config.py::TestAppConfig::test_duplicate_id_raises -vv

# Show print statements during test execution
uv run pytest -vv -s tests/unit/test_file_copier.py
```

### Debugging a Batch Failure

1. Check the `_file_event_` Delta table in the target catalog/schema for FAILED rows:
   ```sql
   SELECT * FROM <catalog>.<schema>._file_event_
   WHERE state = 'FAILED'
   ORDER BY event_time DESC;
   ```

2. Review instrumentation logs written to `{work_base}/{route_id}/log/`:
   - `route-{run_id}-{batch_id}.log` — human-readable per-batch logs with throughput metrics
   - `summary-{run_id}-{batch_id}.json` — JSON summary with failure details

3. Check the intent store to verify target path mapping consistency:
   - File location: `{work_base}/{route_id}/intent_store/_intent.json`
   - Expected status: `in_progress` (retry in progress) or `success` (batch completed)

### Adding a New Route

1. Edit `config/landing_config.yml` and add a new entry to `landing_routes`:
   ```yaml
   - id: source-d
     source: /Volumes/<catalog>/landing/data/source-d/
     target: /Volumes/<catalog>/<schema>/source_d/data/
     enabled: true
   ```

2. Ensure the target path matches the pattern `/Volumes/<catalog>/<schema>/...` (catalog and schema are extracted from this).

3. Deploy to Databricks:
   ```bash
   make databricks-deploy
   ```

### Failure Injection Testing

To simulate copy or delete failures during integration testing:

1. Edit `config/landing_config.yml`:
   ```yaml
   fault_inject_copy_failures:
     - some_file.bin
   fault_inject_delete_failures:
     - another_file.bin
   ```

2. Run the job. The specified files (matched by basename) will raise OSError during copy/delete, triggering Autoloader retry.

3. Integration tests use `FaultInjectingBackend` to test retry logic without real file I/O.

---

## Important Design Decisions

1. **Write-ahead intent store for target-path stability** — Target paths include random UUIDs to prevent overwrite collisions, so the mapping `(source_path, modtime_ms) → target_path` must be committed before any copy starts. Without this, a crash-then-retry generates a new UUID, orphaning the first copy (ADR-006).

2. **No delete on partial failure** — Source files are deleted only after all copies AND all Delta writes AND intent mark-success complete. If any step fails, the batch raises and Autoloader retries; sources remain in place (ADR-007).

3. **Source/modificationTime as composite key** — Files are identified by `(source_path, modification_time_ms)` across retries, not by batch_id or batch index. This handles file renames and re-uploads within the same route (ADR-008).

4. **FUSE stall hang / no `future.result()` timeout** — UC Volume FUSE kernel can stall indefinitely on ADLS network issues. `future.result()` in `_execute_copies` and `_delete_sources` has no timeout; the Databricks job never terminates. Mitigated by using `fsspec-databricks` (HTTP REST, no FUSE). Definitive fix — `future.result(timeout=FILE_IO_TIMEOUT_SECONDS)` at both call sites in `main.py` — is decided but not yet implemented (ADR-010).

---

## References

- **README.md** — User-facing overview, configuration schema, deployment instructions
- **docs/adr/** — Architecture Decision Records covering concurrency, idempotency, crash-safety, performance benchmarks, and the cost simulation app
- **Databricks Bundle documentation** — https://docs.databricks.com/dev-tools/bundles/
- **Databricks Autoloader documentation** — https://docs.databricks.com/ingestion/auto-loader/
```

###### FILE: config/landing_config.yml ######

```yml
max_file_workers: 4
file_backend: fsspec-databricks  # one of: os, fsspec-os, dbutils [but dbutils is not compatible with Autoloader]
work_base: /Volumes/azfr_dev_raw/landing/work/

# Failure injection — for integration testing in real environments only.
# List filenames (basenames) that should raise OSError during copy or delete.
# Leave commented out or set to empty lists in production.
# fault_inject_copy_failures:
#   - file_100MB_00d92262.bin
#   - file_100MB_0330667d.bin
#   - file_100MB_052840ac.bin
#   - file_100MB_0586be07.bin
#   - file_100MB_0a17f3ef.bin
# fault_inject_delete_failures:
#   - file_100MB_0586be07.bin
#   - file_100MB_0a17f3ef.bin

landing_routes:
  - id: source-001
    source: /Volumes/azfr_dev_raw/landing/source-001/
    target: /Volumes/azfr_dev_raw/source_001/data/
    enabled: true
  - id: source-002
    source: /Volumes/azfr_dev_raw/landing/source-002/
    target: /Volumes/azfr_dev_raw/source_002/data/
    enabled: true
  - id: source-003
    source: /Volumes/azfr_dev_raw/landing/source-003/
    target: /Volumes/azfr_dev_raw/source_003/data/
    enabled: true
  - id: source-004
    source: /Volumes/azfr_dev_raw/landing/source-004/
    target: /Volumes/azfr_dev_raw/source_004/data/
    enabled: true
  - id: source-005
    source: /Volumes/azfr_dev_raw/landing/source-005/
    target: /Volumes/azfr_dev_raw/source_005/data/
    enabled: true
  - id: source-006
    source: /Volumes/azfr_dev_raw/landing/source-006/
    target: /Volumes/azfr_dev_raw/source_006/data/
    enabled: true
  - id: source-007
    source: /Volumes/azfr_dev_raw/landing/source-007/
    target: /Volumes/azfr_dev_raw/source_007/data/
    enabled: true
  - id: source-008
    source: /Volumes/azfr_dev_raw/landing/source-008/
    target: /Volumes/azfr_dev_raw/source_008/data/
    enabled: true
  - id: source-009
    source: /Volumes/azfr_dev_raw/landing/source-009/
    target: /Volumes/azfr_dev_raw/source_009/data/
    enabled: true
  - id: source-010
    source: /Volumes/azfr_dev_raw/landing/source-010/
    target: /Volumes/azfr_dev_raw/source_010/data/
    enabled: true
  - id: source-011
    source: /Volumes/azfr_dev_raw/landing/source-011/
    target: /Volumes/azfr_dev_raw/source_011/data/
    enabled: true
  - id: source-012
    source: /Volumes/azfr_dev_raw/landing/source-012/
    target: /Volumes/azfr_dev_raw/source_012/data/
    enabled: true
  - id: source-013
    source: /Volumes/azfr_dev_raw/landing/source-013/
    target: /Volumes/azfr_dev_raw/source_013/data/
    enabled: true
  - id: source-014
    source: /Volumes/azfr_dev_raw/landing/source-014/
    target: /Volumes/azfr_dev_raw/source_014/data/
    enabled: true
  - id: source-015
    source: /Volumes/azfr_dev_raw/landing/source-015/
    target: /Volumes/azfr_dev_raw/source_015/data/
    enabled: true
  - id: source-016
    source: /Volumes/azfr_dev_raw/landing/source-016/
    target: /Volumes/azfr_dev_raw/source_016/data/
    enabled: true
  - id: source-017
    source: /Volumes/azfr_dev_raw/landing/source-017/
    target: /Volumes/azfr_dev_raw/source_017/data/
    enabled: true
  - id: source-018
    source: /Volumes/azfr_dev_raw/landing/source-018/
    target: /Volumes/azfr_dev_raw/source_018/data/
    enabled: true
  - id: source-019
    source: /Volumes/azfr_dev_raw/landing/source-019/
    target: /Volumes/azfr_dev_raw/source_019/data/
    enabled: true
  - id: source-020
    source: /Volumes/azfr_dev_raw/landing/source-020/
    target: /Volumes/azfr_dev_raw/source_020/data/
    enabled: true

```

###### FILE: databricks.yml ######

```yml
# This is a Databricks asset bundle definition for azfr-databricks-landing-zone.
# The Databricks extension requires databricks.yml configuration file.
# See https://docs.databricks.com/dev-tools/bundles/index.html for documentation.

bundle:
  name: azfr-databricks-landing-zone
  engine: direct

include:
  - resources/targets.yml

sync:
  include:
    - config/**
    - src/**
    - notebooks/**

artifacts:
  default:
    type: whl
    path: .  # root of your project where pyproject.toml lives
    build: "uv build --wheel"

python:
  venv_path: .venv
  resources:
    - "resources:load_resources"

```

###### FILE: docs/adr/001-two-thread-pools.md ######

```md
# ADR-001: Two Separate Thread Pools for Route and File Concurrency

## Status
Accepted — single-job concurrency design; `route_worker` deprecated for the target operating model by [ADR-011](011-cost-simulation-webapp.md)

## Context

The Landing Mover processes multiple routes concurrently, and for each route moves multiple files concurrently. This creates a two-level nested concurrency model: route threads (outer) submit file tasks (inner).

## Decision

Use two separate `ThreadPoolExecutor` instances: `route_worker` for route dispatch and `file_worker` for file moves.

## Rationale

A single shared pool would cause a **thread pool starvation deadlock**:
- Route threads occupy pool slots while waiting for their file tasks to complete
- File tasks are queued behind the route threads and can never start
- All threads block indefinitely with no exception raised and no timeout

Two pools eliminate this by giving each level its own queue. Route threads and file tasks never compete for the same slots.

## Consequences

- Two executor context managers must be managed in `run()`, with `route_worker` nested inside `file_worker` to ensure correct shutdown order (routes finish before the file pool is torn down)
- Pool sizing only applies to `file_worker` — see [ADR-002](002-unbounded-route-worker.md)
- This ADR remains the rationale for the legacy single-job, multi-route runner. Under the selected topology in [ADR-011](011-cost-simulation-webapp.md) (`N Job, N Autoloader`), route-level isolation is achieved by separate Databricks jobs rather than an in-process `route_worker`. In that model, `route_worker` is deprecated, while `file_worker` remains relevant inside each per-route job.

```

###### FILE: docs/adr/002-unbounded-route-worker.md ######

```md
# ADR-002: `route_worker` is Unbounded, `file_worker` is Capped

## Status
Accepted

## Context

The two-pool concurrency model (see [ADR-001](001-two-thread-pools.md)) requires deciding how to size each pool. Applying the same bounded limit to both pools would be overly restrictive and provide no benefit for route dispatch.

## Decision

- `file_worker` is bounded: `max_workers` is read from config
- `route_worker` is unbounded: no `max_workers` limit

## Rationale

`file_worker` is capped because file moves are I/O-bound operations that compete for network bandwidth and storage throughput. Beyond a certain concurrency level, adding more threads degrades total throughput rather than improving it. The cap is the primary concurrency control for the system.

`route_worker` is unbounded because route threads are **not** doing the heavy work — they submit tasks to `file_worker` and then wait. Each route thread consumes minimal CPU and memory (one blocked thread). Capping the pool would delay route dispatch without protecting any real resource, and since the number of active routes is small and config-controlled, the unbounded pool poses no resource risk in practice.

## Consequences

- `max_workers` in config has no effect on how many routes run concurrently — it controls only file-level I/O concurrency
- If route count grows significantly (hundreds), an explicit cap on `route_worker` should be reconsidered

```

###### FILE: docs/adr/003-instrumentation-before-delta-write.md ######

```md
# ADR-003: Record Route Instrumentation Before Writing to Delta

## Status
Accepted

## Context

After file moves complete for a route, two things happen: instrumentation metrics are recorded (`on_route_done`) and results are written to a Delta table (`FileEventTable.append_batch`). The ordering of these two operations affects the accuracy of the reported throughput.

## Decision

Scope the throughput timer to the file copy phase only (`_execute_copies`), and pass the resulting `copy_elapsed` value to `on_route_done`:

```python
# In process_batch (LandingMoverJob) — only the copy phase is timed:
with Timer() as copy_timer:
    route_results = self._execute_copies(route, file_copiers)

# In BatchProcessor.__call__ — instrumentation always fires (finally block):
instr.on_route_done(self.route.id, route_results, copy_elapsed)
```

## Rationale

`copy_elapsed` measures the wall-clock time of `_execute_copies` exclusively — it covers only the concurrent file copy phase. Intent store reads and writes, the Delta audit write (`append_batch`), and source deletion all occur outside this timed window. The MB/s figure derived from `copy_elapsed` is therefore a direct measure of file transfer throughput, unaffected by Delta write latency, storage transaction time, or any other ancillary operation.

Scoping the timer to the copy phase only ensures that route-to-route throughput comparisons are meaningful and that the metric remains stable regardless of Delta cluster load or transaction log contention.

## Consequences

- Instrumentation always reflects file copy throughput. Intent store I/O, Delta write, and source deletion all contribute to total batch wall-clock time but are not factored into the reported MB/s.
- `append_batch` failures do not affect throughput metrics.

---

## Addendum: Per-batch log and summary files

### Context

With `availableNow=True`, Autoloader typically processes all pending files in a single micro-batch. However, when a batch fails (e.g. a copy error), Autoloader retries that batch before processing new files — meaning a single streaming run can fire **two or more micro-batches**. The original design wrote a single `route-{run_id}.log` / `summary-{run_id}.json` file, which was overwritten on each `flush()` call, silently discarding logs from earlier batches.

### Decision

`flush(batch_id)` writes per-batch files:

```
route-{run_id}-{batch_id}.log
summary-{run_id}-{batch_id}.json
```

`replay_logs()` reads all batch log files in `batch_id` order and prints them sequentially. `read_summary()` scans all batch summary files and returns a single aggregated dict (`loaded`, `failed`, `loaded_bytes`, `elapsed`, and `failures` are summed/concatenated across batches).

### Rationale

- A single overwriting file loses data when multiple batches fire in the same run.
- Append mode (`"a"`) raises `OSError: ESPIPE` on Unity Catalog Volumes FUSE (no `lseek` support), so is not an option.
- Per-batch files are cheap (small JSON + text), naturally ordered by `batch_id`, and trivially aggregated.
- Stale files from previous runs are cleared by `for_route()` before the streaming query starts, so the log dir only ever contains files from the current run.

```

###### FILE: docs/adr/004-file-backend-benchmark.md ######

```md
# ADR-004: File Backend Benchmark

## Status
Accepted

## Context

The landing zone mover needs to copy files between Unity Catalog Volumes (cross-catalog moves). Three backends were implemented and benchmarked against each other on 10 GB files across a range of worker counts (1–12) on a 4-core VM:

| Backend | Implementation |
|---|---|
| `dbutils` | `dbutils.fs.cp` / `dbutils.fs.rm` via Databricks runtime RPC |
| `fsspec-databricks` | `fsspec`-compatible Databricks filesystem driver |
| `os` | Standard library `shutil.copy2` / `os.remove` over the FUSE-mounted `/Volumes/` path |

Full benchmark results and analysis are in [`notebooks/benchmark/result-10GB/benchmark-10GB-analysis.md`](../../notebooks/benchmark/result-10GB/benchmark-10GB-analysis.md).

## Decision

Use `dbutils` as the production backend for the landing zone.

## Rationale

### Peak throughput

`dbutils` reaches **354 MB/s at 8 workers** — roughly **30% faster** than the best alternatives (`fsspec-databricks` at 272 MB/s, `os` at 260 MB/s). The gap is consistent and repeatable.

### Scaling behaviour

`dbutils` scales monotonically up to 8 workers and then plateaus; it never degrades. `os` and `fsspec-databricks` both peak at 5–6 workers (≈ core count) and then actively degrade — `os` drops below its own 1-worker baseline at 12 workers. This difference reflects their respective IO paths:

- `dbutils.fs.cp` issues chunked HTTP requests directly to the Azure Data Lake Storage Gen2 REST API. Each thread gets an independent HTTP session; there is no shared kernel dispatcher.
- `os` and `fsspec-databricks` go through the FUSE kernel layer. Beyond 5–6 concurrent threads the FUSE dispatcher serializes requests, turning additional workers into contention.

This means `dbutils` continues to benefit from higher parallelism on larger VMs, whereas the FUSE-based backends hit a ceiling that tracks CPU core count regardless of VM size.

### Operational simplicity

`dbutils` is provided directly by the Databricks runtime — no extra dependency, no library installation, and no authentication setup beyond what the cluster already has. `fsspec-databricks` requires an additional package and credential wiring; `os` requires the FUSE mount to be active and stable.

### Single-worker caveat

At 1 worker, `dbutils` is slower than the alternatives (147 MB/s vs ~200 MB/s) due to JVM and HTTP connection-pool initialisation cost. This cost is a one-time fixed overhead per backend instance, not per file; it amortises completely at 2+ workers. The production configuration runs with multiple workers, so this is not a factor.

## Consequences

- Worker count for the `os` backend should be capped at approximately **CPU core count** — beyond that, the FUSE dispatcher serialises requests and additional threads add contention rather than throughput (see benchmark analysis).
- A second benchmark run executed immediately after a heavy first run showed severe Azure Storage throttling at 4+ workers. Production scheduling should avoid back-to-back large batches on the same storage account within a short window.
- **Production uses `os`, not `dbutils`.** Autoloader's `foreachBatch` dispatches work to Spark executors where `dbutils` is unavailable, ruling out the benchmark winner. `os` (2nd place) is therefore the effective production backend.

```

###### FILE: docs/adr/005-exactly-once-file-move-semantics.md ######

```md
# ADR-005: Exactly-Once File Move Semantics via Three-Layer Guarantee

## Status

Accepted — **Layer 3 superseded by [ADR-006](006-write-ahead-intent-file.md)**

Layers 1 and 2 remain in effect unchanged.

## Context

The Landing Mover is triggered by Databricks Autoloader (`cloudFiles` format, `availableNow` trigger). A batch can fail mid-execution — for example after files are physically copied but before the Delta `FileEventTable` write completes. When that happens, Autoloader has not committed its checkpoint and will re-present the same files on the next job execution.

Without countermeasures, a retry would cause:
- **Copy failures**: the source file no longer exists, causing `FileNotFoundError` and an infinite retry loop.
- **Duplicate events**: the `FileEventTable` would accumulate multiple `LOADED` rows for the same file.

## Decision

Implement exactly-once semantics through three independent, complementary layers:

1. **Autoloader checkpoint** — ensures reliable re-delivery of unprocessed files.
2. **Idempotent file operations** — ensures safe re-execution of the physical copy and delete.
3. **Write-ahead intent file** — ensures target-path stability and safe audit re-writing across retries (see [ADR-006](006-write-ahead-intent-file.md)).

## Rationale

### Layer 1 — Autoloader checkpoint

Autoloader persists discovered file offsets and committed batch IDs in a checkpoint directory (one per route, under `{work_base}/{route.id}/checkpoint`). A batch is only committed after `foreachBatch` returns without raising. If a batch fails, the checkpoint is not advanced and the same files are re-presented in the next execution with the same `batch_id`.

**Guarantee**: every file is delivered to `process_batch` at least once.

---

### Layer 2 — Idempotent file operations (`fs_backend.py`)

`copy_file()` is an overwriting copy — re-copying to the same target path is safe and produces the same result. `delete_file()` swallows `FileNotFoundError` — deleting an already-absent file is a no-op. The pre-committed target path from `FileIntentStore` ensures that retries always use the same destination, so the copy is always an idempotent overwrite.

**Guarantee**: the physical file ends up at the destination exactly once, regardless of how many times the batch runs.

---

### Layer 3 — Write-ahead intent file

See [ADR-006](006-write-ahead-intent-file.md). The `FileIntentStore` commits all `(source_path, modification_time_ms) → target_path` mappings before any copy starts, and persists a `SUCCESS` marker after all copies, Delta writes, and deletions complete.

**Guarantee**: target paths are stable across retries; event rows for a given batch are written to the `FileEventTable` exactly once.

---

## Consequences

- `{work_base}/{route.id}/checkpoint/` is required for Autoloader checkpoint persistence (Layer 1).
- All `FileSystemBackend` implementations must keep `copy_file()` safe to overwrite and `delete_file()` idempotent (Layer 2).
- `FileIntentStore` is the primary crash-safety mechanism for target-path stability (Layer 3 — see ADR-006).
- The full case analysis of how these three layers interact across all retry and recovery scenarios is in [ADR-009](009-consecutive-run-case-analysis.md).

```

###### FILE: docs/adr/006-write-ahead-intent-file.md ######

```md
# ADR-006: Write-Ahead Intent File for Crash-Safe Idempotent Batch Retry

## Status

Accepted — implements Layer 3 of [ADR-005](005-exactly-once-file-move-semantics.md)

## Context

Target paths include a random UUID component to prevent overwrite collisions across simultaneous ingestion jobs on the same target volume. This means the batch outcome — specifically, which target path each source file is copied to — is not stable across retries of the same `batch_id` unless the mapping is persisted before the first copy starts.

Without persistence:
- On the first execution of batch `#N`, file `a.csv` is copied to `.../a-<uuid1>.csv`.
- If the batch crashes after the copy but before the `FileEventTable` write, the Autoloader checkpoint is not advanced.
- On retry, `foreachBatch` is called again with `batch_id = N` and the same source files.
- `FileCopier.from_metadata()` generates a **new UUID** → `.../a-<uuid2>.csv`. The first target path is now an orphan.
- The audit table records neither path. The orphaned copy at `<uuid1>` is invisible.

## Decision

Use a **write-ahead intent file** (`FileIntentStore`) that commits the target path for every file *before* the first copy starts.

### Mechanics

1. `FileIntentStore.for_route(work_base, route_id)` resolves a per-route path:
   ```
   {work_base}/{route_id}/intent_store/_intent.json
   ```

2. Before any copy starts, `write_in_progress()` serialises all `(source_path, modification_time_ms, target_path)` entries as a JSON array and atomically replaces the file using the temp-file + `os.replace()` pattern:
   ```python
   json.dump(entries, tmp)
   os.replace(tmp_path, path)   # atomic on POSIX and NTFS
   ```

3. On retry, `read()` returns the prior `{(source_path, modtime_ms): target_path}` mapping. The `_resolve_copiers()` phase re-uses the stored target path instead of generating a new UUID. The copy is then an idempotent overwrite of the same destination.

4. If the target path is already present as a `LOADED` row in the `FileEventTable` (detected via `get_loaded_target_paths()`), the file is skipped entirely — no copy, no event.

5. On successful completion of all copies, the Delta write, and source deletion, `mark_success()` atomically updates the intent file to `SUCCESS` status — same entries, same format, only the `status` field changes. The file is **not** removed.

6. If Autoloader replays the batch before Spark commits the checkpoint (i.e., `foreachBatch` returned cleanly but Spark crashed before writing the checkpoint), `_resolve_copiers` detects `SUCCESS + full batch overlap` and returns `None` immediately — no copies, no Delta writes, no deletions. This allows unlimited crash-safe replays until Spark commits (see ADR-009 case A-7).

### Why a flat JSON file per route, not a shared Delta table

The intent store is accessed inside `foreachBatch`, which runs concurrently for up to 150 routes simultaneously. Centralising intent state in a shared Delta table would introduce:

- **Transaction log contention**: every route competing for the same Delta commit slot.
- **Hot-partition writes**: all intent rows landing in the same table at once.
- **Infrastructure coupling**: a secondary managed table that must exist before the first run.

A flat JSON file per route has zero cross-route contention (files are independent), requires no provisioning, and leverages ADLS Gen2 FUSE atomicity via `os.replace()` — which is available because Unity Catalog Volumes are FUSE-mounted at `/Volumes/`.

### Why atomic temp-file write

A partial JSON file left by an in-progress or crashed write would cause `json.loads()` to raise and block all retries. Writing to a `.tmp` sibling and calling `os.replace()` ensures the reader always sees either the old complete file or the new complete file, never a partial one.

### Atomicity of `os.replace` on UC Volumes FUSE — verification

The entire idempotency guarantee rests on `os.replace` being atomic at the FUSE layer. Two distinct layers are involved:

1. **ADLS Gen2 storage API** — atomicity is guaranteed by the hierarchical namespace (HNS). HNS was introduced precisely to provide atomic rename at object-storage scale; it updates a single directory entry rather than copying and deleting blobs. This is [documented by Microsoft](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-namespace#the-benefits-of-a-hierarchical-namespace).

2. **Databricks UC Volumes FUSE driver** — Unity Catalog Volumes are FUSE-mounted at `/Volumes/`. The FUSE driver is the layer that translates `rename(2)` syscalls into ADLS Gen2 REST API calls. A FUSE driver could in principle implement rename as copy+unlink (non-atomic), even if the underlying storage supports atomic rename natively. The FUSE driver is also known to have other quirks on this platform (e.g. `lseek`/`ESPIPE` is not supported).

**Verified on 2026-06-17:** The smoke test notebook [`notebooks/smoke_test_uc_volume_atomicity.ipynb`](../../notebooks/smoke_test_uc_volume_atomicity.ipynb) was executed on a Databricks cluster against a real UC Volume:

- **Test 1** (no `EXDEV`): `os.replace` completed without raising — the FUSE driver does not reject in-directory rename.
- **Test 2** (no torn reads): 2000 concurrent `os.replace` cycles with a parallel reader produced zero `JSONDecodeError` — the rename is observed as atomic from the Python reader's perspective.

The FUSE driver correctly delegates to the ADLS Gen2 atomic rename API. `FileIntentStore` is safe to use on this platform.

## Consequences

- `FileIntentStore` is the crash-safety mechanism for target-path stability.
- A new per-route directory `{work_base}/{route_id}/intent_store/` is created alongside the existing `{work_base}/{route_id}/checkpoint/`.
- Stale intent files (from a permanently aborted batch) are overwritten on the next successful run of the same route — the new batch’s `write_in_progress()` atomically replaces whatever was left.
- The intent file adds one synchronous JSON read and one synchronous JSON write per batch, per route. For typical batch sizes (tens to low hundreds of files), this is negligible compared to copy latency.
- `_resolve_copiers()` may issue a Delta scan (`get_loaded_target_paths`) on retry to avoid re-copying already-committed files. This scan is skipped entirely on the first execution of a batch (happy path).

```

###### FILE: docs/adr/007-raise-before-delete-phase-ordering.md ######

```md
# ADR-007: Raise-Before-Delete Phase Ordering in `process_batch`

## Status

Accepted

## Context

`process_batch` performs three distinct operations for every Autoloader batch:

1. **Copy** — physical file transfer from source to target Volume.
2. **Audit write** — append `LOADED` or `FAILED` events to the `FileEventTable` Delta table.
3. **Source deletion** — remove source files once they are safely copied and recorded.

Autoloader's exactly-once checkpoint guarantee applies at the `foreachBatch` boundary: if `foreachBatch` raises, the checkpoint is **not** advanced and the batch is re-presented on the next job execution with the same files and the same `batch_id`. This makes the choice of *when to raise* and *when to delete* a correctness concern, not just a style choice.

Two orderings were considered:

**Option A — delete-before-raise** (fire-and-forget):
```
copy → audit write → delete sources → raise on failure
```
Source files are deleted regardless of copy outcome. Failed files are recorded in the audit table as terminal `FAILED` events.

**Option B — raise-before-delete** (retry-oriented):
```
copy → audit write → raise on failure → delete sources
```
Source files are only deleted after every copy and the audit write have succeeded. Any failure causes `foreachBatch` to raise, triggering an Autoloader retry.

## Decision

Use **Option B: raise-before-delete**. `process_batch` raises on the first copy failure before deleting any source file, and source deletion is only attempted when all copies in the batch have succeeded.

The full phase ordering is:

| Phase | Action | On failure |
|-------|--------|------------|
| 1 | Read intent store (`FileIntentStore.read`) | raise — batch cannot proceed safely |
| 2 | Query audit for retry-skip candidates (`get_loaded_target_paths`) | raise — cannot determine skip set |
| 3 | Build `FileCopier` list (reuse or generate target paths) | raise — cannot build work list |
| 4 | Write-ahead intent file (`FileIntentStore.write`) | raise — target paths not durably committed |
| 5 | Execute all copies concurrently | collect errors, continue to audit write |
| 6 | Append `LOADED`/`FAILED` events to `FileEventTable` | raise — audit state unknown |
| 7 | Raise if any copy in phase 5 failed | → Autoloader retries entire batch |
| 8 | Delete source files concurrently | non-fatal; log warning per file |
| 9 | Write SUCCESS intent (`FileIntentStore.mark_success`) | non-fatal |

## Rationale

### Why raise on copy failure (Option B, not A)

Under Option A, a `FAILED` event is a terminal outcome. The file is stuck at its source forever unless a human intervenes. Option B treats failures as transient: the batch retries automatically and the file gets another attempt with no manual action required. This is appropriate for infrastructure failures (network blip, throttling, brief cluster instability) which are the dominant failure mode.

### Why execute all copies before raising (phases 5 → 6 → 7, not fail-fast)

Copy tasks run concurrently in a thread pool. Interrupting mid-batch on the first failure would orphan in-flight copies. Completing all copies before raising gives every file in the batch a chance to succeed, minimises the number of files that must be retried, and produces a complete audit picture in the `FileEventTable` before the batch is declared failed.

### Why audit write precedes the raise (phase 6 before phase 7)

`FAILED` rows written to `FileEventTable` in phase 6 are visible to operators immediately, without waiting for the retry to complete. This preserves observability even on failing batches. On retry, those rows may be superseded by `LOADED` rows for the same files (the audit table is append-only and downstream reads use the latest state per file).

### Why delete only after all copies succeed (phases 7 → 8)

If phase 7 raises, the Autoloader checkpoint is not committed and the batch replays. `_resolve_copiers` on retry reads the intent store to recover the original target paths — those files may have already been successfully copied. If source deletion had already run for the successful files, the retry would see `src_exists = False, dst_exists = True` (handled by ADR-005 Layer 2) and correctly skip the copy, issuing a `LOADED` event.

Deleting sources before raising would not break correctness, but it removes the opportunity for operators to validate the target before the source disappears in failure scenarios. Keeping sources intact until the batch fully succeeds makes failure investigation safer.

### Why `mark_success` last (phase 9)

The intent file must survive until phase 8 is complete. If the job crashes between phases 7 and 9, the next execution finds an `IN_PROGRESS` intent with all files already deleted from source. `_resolve_copiers` queries Delta, detects all targets as LOADED, skips all copies, re-runs the idempotent delete (no-op), and then writes `SUCCESS`. See ADR-009 case A-6.

## Consequences

- Source files are never deleted when a batch has copy failures. On prolonged failure, files accumulate at the source. The source Volume must be monitored for growth anomalies.
- Exactly-once source deletion is not guaranteed at the phase level: a crash between phases 8 and 9 may leave an intent file referencing files that have already been deleted. This is safe — the next run detects them via the audit table and skips them.
- `_delete_sources` accepts `list[str]` (source paths) rather than `list[FileCopyResult]` because by phase 8 the outcome has been gated: all results are known-successful.

```

###### FILE: docs/adr/008-modification-time-as-file-identity-key.md ######

```md
# ADR-008: `modificationTime` as the Stable File Identity Key

## Status

Accepted

## Context

`FileIntentStore` maps each file in a batch to a pre-committed target path (see [ADR-006](006-write-ahead-intent-file.md)). On Autoloader batch retry, the intent store is read to recover the original target path and avoid generating a new random UUID.

This lookup requires a **stable key** that:

1. Uniquely identifies a specific version of a file (not just its path).
2. Is present in the Autoloader row delivered to `foreachBatch`.
3. Is stable across job executions — the same physical file must produce the same key on every retry of the same batch.

Three candidates were considered:

| Candidate | Stable? | Unique? | Notes |
|-----------|---------|---------|-------|
| `source_path` alone | ✓ | ✗ | Two successive uploads of different content to the same path would collide |
| `(source_path, batch_id)` | ✗ | ✓ | `batch_id` resets to `0` if the Autoloader checkpoint is deleted; collides with a genuinely new file |
| `(source_path, modification_time_ms)` | ✓ | ✓ | Reflects the file's content version; stable across checkpoint deletion |

`batch_id` was explicitly rejected because it is a property of the Autoloader checkpoint, not of the file itself. If the checkpoint directory is deleted and re-created (a supported recovery path), `batch_id` restarts at `0`. A file uploaded on the original batch `#0` and a file uploaded after checkpoint reset would share the same `(source_path, 0)` key despite being different objects.

## Decision

Use `(source_path, modification_time_ms)` as the composite key in `FileIntentStore`.

- `source_path` — the full Volume path of the source file, as reported by Autoloader.
- `modification_time_ms` — the file's last-modified timestamp in **milliseconds since the Unix epoch**, as reported by Autoloader's `modificationTime` column. When `modificationTime` is `None` (Autoloader did not populate it), the value is coerced to `0`.

The same composite key is used by `FileCopier.from_metadata()` to build the target path:

```
{target_base}/year=YYYY/month=MM/day=DD/{YYYYMMDDHHmmssSSS}-{8hex}/{filename}
```

The `year`, `month`, and `day` components are derived from the **wall-clock time at ingestion** (not the file's `modification_time`). The unique directory segment combines a millisecond-precision wall-clock timestamp with 8 hex characters from a random UUID to prevent collisions between concurrent copies. Target paths are intentionally opaque — consumers locate files via the `_file_event_` table, not by directory scan.

## Rationale

### `modificationTime` as a content-version proxy

Object stores (ADLS Gen2) update `modificationTime` on every write. If a file is re-uploaded at the same path with new content, its `modificationTime` changes. The composite key `(source_path, modtime_ms)` therefore correctly identifies the new version as a distinct file, while the previous version (if it had already been ingested) would still be found in the audit table under its old key.

### Stability across checkpoint deletion

Unlike `batch_id`, `modificationTime` is an intrinsic property of the file. It does not change when the Autoloader checkpoint is deleted, when the job is re-run from scratch, or when the route configuration is modified. This makes it safe to use as a durable intent key.

### Partitioning by wall-clock ingestion time

Target paths use the wall-clock time at which the job runs for the `year/month/day` partition. This simplifies the path-generation logic:

- No special handling is needed for files where `modificationTime` is `None`.
- Files from the same batch always land in the same partition since all copies run within the same job execution window.
- The unique directory prefix (`{YYYYMMDDHHmmssSSS}-{8hex}`) prevents collisions between concurrent ingestion jobs targeting the same Volume.
- Target paths are opaque: since consumers always use the `_file_event_` table as their lookup, the partition date is an operational convenience rather than a semantic guarantee about source data.

## Consequences

- `FileMetadata.modification_time` is typed `int | None`. Call sites coerce `None` to `0` with `modification_time or 0` before building intent keys. The value is never used for partition path generation.
- The target path structure is fixed at `year=YYYY/month=MM/day=DD/{YYYYMMDDHHmmssSSS}-{8hex}/{filename}`. Changing the partition scheme would invalidate all existing intent files and require a coordinated migration.
- `get_loaded_target_paths()` is the safeguard against re-copying files whose `modificationTime` matches the intent store but whose `source_path` has been re-uploaded since ingestion. The audit table is the source of truth; the intent store is a transient recovery aid.

```

###### FILE: docs/adr/009-consecutive-run-case-analysis.md ######

```md
# ADR-009: Consecutive-Run Case Analysis for `process_batch`

## Status

Accepted

## Context

`process_batch` is invoked by Autoloader's `foreachBatch` API. Two key properties of that contract define the retry model:

1. **Autoloader retries on raise** — if `foreachBatch` raises, the Spark checkpoint is _not_ advanced. The next job execution re-presents the exact same files with the same `batch_id`.
2. **Autoloader advances on clean return** — if `foreachBatch` returns without raising, Spark commits the checkpoint. The next execution delivers a new batch.

This ADR enumerates every possible state that Run 1 can leave the system in, and how Run 2 responds in each case. The goal is to verify that the `_resolve_copiers` state machine covers all paths without data loss, duplication, or an unrecoverable hang.

> **Implementation note (added after `ignoreMissingFiles` was introduced):** Cases A-6 and A-7 never reach `_resolve_copiers` in production. Because `spark.sql.files.ignoreMissingFiles=true` is set in `BatchProcessor.__call__`, Spark silently drops rows for already-deleted source files before `df.collect()` returns. When all sources are gone, `df.collect()` returns `[]` and `process_batch` handles the case entirely in an early-exit branch (P0 below). Case A-5 (partial deletion) does reach `_resolve_copiers`, but only with the files still present — not the full original batch.

## The State Machine

`_resolve_copiers` executes four sequential steps:

1. **Early exit** — if `SUCCESS + full overlap`, return `None` (success replay).
2. **Assign targets** — for each file, reuse the target path from the snapshot if the key is present; otherwise generate a fresh opaque path.
3. **Determine `loaded_targets`** — the set of target paths that can be skipped:
   - `SUCCESS` (any overlap) → targets of overlapping keys are guaranteed LOADED; no Delta query needed.
   - `IN_PROGRESS + overlap` → query Delta to find which overlapping targets are already LOADED.
   - Absent / `IN_PROGRESS` with no overlap → `loaded_targets = ∅`.
4. **Build lists** — iterate over all files; append every file to `full_entries`; append to `file_copiers` only when the target is not in `loaded_targets`.

The resulting dispatch summary:

| Snapshot status | Overlap | `loaded_targets` source | `file_copiers` |
|---|---|---|---|
| absent | — | ∅ | all files |
| IN_PROGRESS | none | ∅ | all files |
| IN_PROGRESS | partial or full | Delta query | non-LOADED files |
| SUCCESS | full | (early exit — return `None`) | — |
| SUCCESS | partial | overlapping snapshot entries | non-overlapping files |
| SUCCESS | none | ∅ | all files |

In all cases, `full_entries` includes every file delivered by `foreachBatch`, including files whose target is already LOADED and will therefore not be re-copied. When `_resolve_copiers` returns `None` (SUCCESS + full overlap), the intent already carries full entries written by a previous run — by the `mark_success` call that completed the previous batch. When it returns a tuple, `write_in_progress(full_entries)` writes them now. Either way, the intent always records a complete source→target mapping for the batch.

---

## Phase Map of `process_batch`

The phases, in order, with their crash labels:

```
★ START
│
├─ [P0]  df.collect() → if rows == []:                   ← ignoreMissingFiles dropped all sources
│          if intent == SUCCESS (non-empty): replay=True, return []
│          else:                             mark_success([]), return []
│
│  (P0 handles A-6 and A-7 in production; control never reaches P1 for those cases)
│
├─ [P1]  _resolve_copiers → write_in_progress(entries)   ← intent = IN_PROGRESS
│
│  (if SUCCESS full overlap: return None immediately — no further phases)
│  (this None-return path is exercised by tests; in production P0 intercepts first)
│
├─ [P2]  _execute_copies(file_copiers)                   ← copies running concurrently
├─ [P3]  append_batch(events)                            ← Delta write (LOADED / FAILED)
├─ [P4]  raise if any copy failed                        ← raises → Autoloader retries
├─ [P5]  _delete_sources(all source paths)               ← raises on any failure
├─ [P6]  mark_success(entries)                           ← intent = SUCCESS
│
★ RETURN → Spark commits checkpoint
```

A crash or raise at any phase causes Autoloader to retry with the same files.

---

## Part A — Same-Batch Retry Cases

Run 1 raised or crashed. Autoloader re-presents the identical set of `(source_path, modification_time)` keys.

---

### A-1 — Crash before P1 (no intent written)

**Run 1 left:**
- Intent file: absent (or a stale `SUCCESS`/`IN_PROGRESS` from a completely different batch with no key overlap)
- Sources: present
- Delta: no events for this batch

**Run 2 branch:** Fresh start

**Run 2 actions:**
1. Generates new target paths for every file.
2. Writes `IN_PROGRESS` intent with all entries.
3. Executes all copies.
4. Writes LOADED/FAILED events to Delta.
5. Raises if any copy failed (→ Autoloader retries again).
6. Deletes all sources.
7. Writes `SUCCESS` intent.

**Outcome:** Normal execution path. No special handling needed.

---

### A-2 — Crash after P1, before any copy completes

**Run 1 left:**
- Intent: `IN_PROGRESS`, full entries (all target paths pre-committed)
- Sources: present
- Delta: no events for this batch

**Run 2 branch:** Retry path (IN_PROGRESS + full overlap)

**Run 2 actions:**
1. Reads intent → full overlap, status `IN_PROGRESS`.
2. Queries Delta for LOADED target paths → returns `∅` (no events yet).
3. Re-queues all files with their **original target paths** from the intent file.
4. Re-writes `IN_PROGRESS` intent (same entries, refreshed).
5. Continues from P2 — copies, events, raise-on-failure, delete, `mark_success`.

**Outcome:** All files are copied to the same destination as would have been used had Run 1 completed. No orphan paths.

---

### A-3 — Crash during P2 (partial copies) or P3 (failure during Delta write)

**Run 1 left:**
- Intent: `IN_PROGRESS`, full entries
- Sources: present
- Delta: no events — P3 was not reached (P2 crash), or the Delta write failed atomically leaving nothing committed (P3 crash)

**Run 2 branch:** IN_PROGRESS + full overlap → Delta query for `loaded_targets`

**Run 2 actions:**
1. Reads intent → full overlap, status `IN_PROGRESS`.
2. Queries Delta for LOADED target paths → returns `∅`.
3. All files re-queued with their **original target paths** from the intent file.
4. Re-writes `IN_PROGRESS` intent with **full_entries**.
5. Executes all copies, writes LOADED/FAILED events, raises if any copy failed.
6. Deletes sources, writes `SUCCESS` intent.

**Outcome:** All files are re-copied to the same target paths. No duplication can occur since target paths are preserved in the intent file.

---

### A-4 — P4 raises (copy failures detected)

**Run 1 left:**
- Intent: `IN_PROGRESS`, full entries
- Sources: present (delete phase never reached)
- Delta: all events written — some `LOADED`, some `FAILED`

**Run 2 branch:** IN_PROGRESS + full overlap → Delta query for `loaded_targets`

**Run 2 actions:**
1. Queries Delta for LOADED target paths → returns the successfully copied files.
2. Skips `LOADED` files.
3. Re-queues `FAILED` files (and any not yet attempted) with their **original target paths**.
4. Re-writes `IN_PROGRESS` with **full_entries** — all batch files including the LOADED-skipped ones.
5. Executes copies for the re-queued files; writes new `LOADED` or `FAILED` events.
6. Raises again if any copy still fails (→ further retries).

**Outcome:** Each retry narrows the work to only the files that have not yet succeeded. The `FAILED` rows from prior attempts remain in Delta alongside the eventual `LOADED` rows — consumers read the latest state per file.

---

### A-5 — P5 raises (deletion failure)

**Run 1 left:**
- Intent: `IN_PROGRESS`, full entries
- Sources: partially deleted (some gone, some present — deletion stopped on error)
- Delta: all `LOADED` events written

**Run 2 entry path:** `ignoreMissingFiles=true` causes Spark to drop rows for already-deleted files before `df.collect()` returns. Run 2 therefore receives only the **still-present** files, not the full original batch.

**Run 2 branch:** IN_PROGRESS + overlap on remaining files → Delta query for `loaded_targets`

**Run 2 actions:**
1. `df.collect()` returns only the still-present source files (already-deleted files silently dropped).
2. `_resolve_copiers` sees only the surviving files as `batch_keys`; overlapping keys match those files in the IN_PROGRESS snapshot.
3. Queries Delta for LOADED target paths → returns the target paths of the surviving files (all were LOADED before P5 was reached).
4. All surviving files skipped → `file_copiers = []`.
5. Writes `IN_PROGRESS` intent with `full_entries` for the **surviving files only** — entries for already-deleted files are no longer present in the batch and are not included.
6. No copies, no Delta write.
7. Calls `_delete_sources` for the surviving source paths.

   **`delete_file` is idempotent.** Any other error (transient network failure, storage service unavailable) will still raise — causing Autoloader to retry again until deletion succeeds.

8. Writes `SUCCESS` intent.

**Outcome:** Sources eventually deleted, batch completes cleanly. The intent after P6 contains only the surviving files — entries for files deleted in Run 1 are gone, which is safe because those files are already LOADED and their sources already absent.

> **Note:** If all sources were deleted in Run 1 (i.e., P5 failed on the very last file after all others succeeded), `df.collect()` returns `[]` and the case falls into A-6 instead.

---

### A-6 — Crash between P5 and P6 (delete succeeded, `mark_success` not yet written)

This case applies regardless of how many prior attempts the batch has had.

**Run 1 left:**
- Intent: `IN_PROGRESS`, full entries
- Sources: all deleted
- Delta: all `LOADED`

**Run 2 entry path:** `ignoreMissingFiles=true` causes Spark to drop all rows (all sources are gone). `df.collect()` returns `[]`. `_resolve_copiers` is **never called**.

**Run 2 branch:** P0 empty-rows branch — `snapshot.status == IN_PROGRESS` → `else` arm

**Run 2 actions:**
1. `df.collect()` returns `[]`.
2. `process_batch` reads the intent store: `IN_PROGRESS` (not SUCCESS, not empty) → falls into the `else` arm.
3. Calls `mark_success([])` — writes `SUCCESS` intent with an empty entry list.
4. Returns `[]` immediately. No Delta query, no copy, no `_delete_sources` call.

**Outcome:** Handled cleanly. `mark_success([])` is sufficient because all sources are already deleted and all targets already LOADED — there is nothing left for a future run to reuse from the intent. The checkpoint advances on the next clean return.

> **`delete_file` idempotency is not exercised here.** Unlike the original description of this case, `_delete_sources` is not called in Run 2 because the empty-rows branch exits before reaching P1. Recovery does not depend on `delete_file` being idempotent for A-6.

---

### A-7 — Crash after P6, before Spark commits checkpoint

**Run 1 left:**
- Intent: `SUCCESS`, full entries
- Sources: deleted
- Delta: all `LOADED`

**Run 2 entry path:** `ignoreMissingFiles=true` causes Spark to drop all rows (sources are deleted). `df.collect()` returns `[]`. `_resolve_copiers` is **never called**.

**Run 2 branch:** P0 empty-rows branch — `snapshot.status == SUCCESS` and `snapshot.entries` non-empty → replay arm

**Run 2 actions:**
1. `df.collect()` returns `[]`.
2. `process_batch` reads the intent store: `SUCCESS` with non-empty entries → sets `replay = True`.
3. Returns `[]` immediately. No Delta query, no copy, no `_delete_sources`, no intent write.
4. Spark commits the checkpoint.

**Outcome:** The cleanest retry case. The `SUCCESS` state acts as a durable "done" marker that allows unlimited crash-safe replays until Spark commits.

> **The `_resolve_copiers` None-return path** (SUCCESS + full overlap) is the logical equivalent of this case and is fully covered by unit tests. In production, the P0 empty-rows branch intercepts first because all sources are already deleted. Both paths produce the same observable outcome: nothing happens and Spark commits.

---

## Part B — New-Batch Cases

Run 1 completed successfully and Spark committed the checkpoint. Run 2 delivers a **different** set of files.

---

### B-1 — New batch, no overlap with the previous batch

**Intent left by Run 1:** `SUCCESS` with the previous batch's entries.

**Run 2 branch:** SUCCESS + no overlap → **Fresh start**

**Run 2 actions:** Generates new target paths, proceeds normally.

**Outcome:** Standard execution. The stale `SUCCESS` intent is overwritten by the new `IN_PROGRESS` intent in P1.

---

### B-2 — New batch, partial overlap with the previous SUCCESS

This occurs if Autoloader re-presents some files from the prior committed batch alongside genuinely new files. Under normal operation this should not happen: Autoloader guarantees that a committed batch is never re-delivered, and that a retried batch always presents the exact same set of files. A partial overlap between a new batch and the previous `SUCCESS` intent would indicate a checkpoint anomaly or file re-delivery edge case outside of the normal Autoloader contract.

> **Note:** if a file is re-uploaded at the same path with new content, its `modification_time` changes, so `(path, modtime)` produces a different key — it does not match the previous `SUCCESS` entry and is treated as a new file.

**Run 2 branch:** SUCCESS + partial overlap → `loaded_targets` from overlapping snapshot entries (no Delta query)

**Run 2 actions:**
1. Identifies overlapping keys (files already processed by Run 1).
2. For overlapping files: reuses their target path from the snapshot; adds to `loaded_targets` — no copy, no event.
3. For new files: generates fresh target paths; adds to `file_copiers`.
4. Writes `IN_PROGRESS` intent with **full_entries** — all batch files (overlapping with their old targets, new with fresh targets).
5. Executes copies and writes events for new files only.
6. Deletes all source paths (overlapping ones are already gone — requires `delete_file` idempotency).
7. Writes `SUCCESS` intent with full_entries.

**Outcome:** Already-ingested files are not re-copied or re-audited. New files proceed normally. The full intent mapping covers all batch files, so any crash between P5 and P6 of this run falls into case A-6 and recovers cleanly.


## Summary Table

| Case | Run 1 terminated after | Intent entering Run 2 | Sources | `loaded_targets` source | Run 2 outcome |
|------|---|---|---|---|---|
| A-1 | Before P1 | Absent / stale no-overlap | Present | ∅ | Normal copy, all files |
| A-2 | P1 (intent written, no copies) | IP, full entries | Present | Delta query → ∅ | Reuse targets, copy all |
| A-3 | P2 (partial copies) or P3 (failed Delta write) | IP, full entries | Present | Delta query → ∅ | Copy all files |
| A-4 | P4 (copy failure, raised) | IP, full entries | Present | Delta query → LOADED only | Skip LOADED, retry FAILED |
| A-5 | P5 (delete failure, raised) | IP, full entries | Partial | Delta query → surviving targets | `file_copiers=[]` for survivors, re-delete survivors, mark_success |
| A-6 | P5→P6 gap (any attempt) | IP, full entries | Absent | P0 empty-rows branch (no Delta query) | `mark_success([])`, return — no copy, no delete |
| A-7 | P6→return gap (success, no checkpoint) | SUCCESS, full entries | Absent | P0 empty-rows branch (no Delta query) | replay=True, return — Spark commits |
| B-1 | Normal completion, new batch | SUCCESS, prior entries | Absent | ∅ (no overlap) | Normal copy, all files |
| B-2 | Normal completion, partial re-delivery | SUCCESS, prior entries | Absent | Overlapping snapshot entries | Copy new files only |

---

## Consequences

- **`delete_file` is idempotent.** Cases A-5 and B-2 call `delete_file` on paths that may already be absent. All three backends (`DbutilsBackend`, `FsspecDatabricksBackend`, `NativeOsBackend`) swallow `FileNotFoundError`, making the operation safe to retry. (A-6 does not call `delete_file` — it exits via the P0 empty-rows branch before reaching `_delete_sources`.)
- **`spark.sql.files.ignoreMissingFiles=true` is required for A-5, A-6, and A-7 recovery.** In these cases, Autoloader re-presents the original batch (checkpoint not yet advanced), but a partial or complete set of source files have already been deleted. Without this option, `df.collect()` raises `CLOUD_FILE_SOURCE_FILE_NOT_FOUND` when Spark attempts to read the binary metadata of already-deleted files — crashing `foreachBatch` before any retry logic can execute. With the option set, Spark silently drops missing rows. For A-6 and A-7 this means `df.collect()` returns `[]`, and `process_batch` handles completion entirely in the P0 empty-rows branch without ever calling `_resolve_copiers`. For A-5, only the still-present sources are delivered; `_resolve_copiers` processes that surviving subset, skips their copies (all LOADED in Delta), and deletes them. This config must be applied via `spark.conf.set()` inside `BatchProcessor.__call__` on `df.sparkSession` — the cloned session that Spark Connect creates for the foreachBatch worker process. Setting it on the driver session has no effect: under Spark Connect, `foreachBatch` is serialised over gRPC and executed in a separate worker process whose session is not inherited from the driver.
- **The intent carries the full batch mapping for cases that go through `_resolve_copiers`.** For A-1 through A-4 and B-1/B-2, `_resolve_copiers` writes `full_entries` — every file in the batch — to `write_in_progress` before any copy starts, and the same list is passed to `mark_success`. For A-6, `process_batch` calls `mark_success([])` directly (empty list) because all sources are already deleted and there is no surviving batch to map. This is safe: a `SUCCESS` intent with an empty entry list will not be mistaken for a full-overlap replay by a future run (the `overlapping_keys == batch_keys` check requires actual overlapping entries).
- **Delta `FAILED` rows accumulate across retries** (case A-4). Each retry appends new `LOADED` or `FAILED` rows for the re-queued files; prior `FAILED` rows are not removed. The `FileEventTable` is append-only; consumers must reduce to the latest event per `(source_path, modification_time)`. `LOADED` rows are not duplicated — `append_batch` is atomic and `get_loaded_target_paths` prevents re-writing events for already-committed files.
- **The `SUCCESS` status is the stable termination marker.** Once written, it absorbs unlimited Autoloader replays until Spark commits the checkpoint (case A-7). In production, the P0 empty-rows branch detects `SUCCESS` with non-empty entries and returns immediately (`replay=True`) — no copy, no Delta write, no intent mutation. The gap between `mark_success` and the Spark commit is the only phase with no data-loss risk.

```

###### FILE: docs/adr/010-fuse-stall-io-timeout.md ######

```md
# ADR-010: UC Volume FUSE Stalls Cause Infinite Job Hangs

## Status

Root cause revised. Fix decided but **not yet implemented** — see Decision section.

- The `max_concurrent_routes` constraint was removed in the N-jobs migration (see below).
- The original incident (2026-06-18) was attributed to Spark executor slot exhaustion. A
  second incident (2026-07-10) with the N-jobs architecture revealed the true root cause:
  **UC Volume FUSE kernel stalls**, not Spark slot exhaustion. See Incidents section.

## Context

### Original architecture (single job, N concurrent routes)

The two-pool concurrency model (ADR-001, ADR-002) ran up to `max_concurrent_routes`
Autoloader streaming queries concurrently inside a single Databricks Job. Each query was
managed by a `route-worker` thread calling `query.awaitTermination()`.

Inside each stream, PySpark called `foreachBatch` with the micro-batch DataFrame.
`BatchProcessor.__call__` submitted two Spark actions against the driver's active SparkSession:

1. `df.collect()` — materialises the list of incoming files for the batch.
2. `FileEventTable.append_batch()` — appends Delta table rows via `df.write.saveAsTable()`.

### Current architecture (N jobs, one route each)

`max_concurrent_routes` has been removed from `AppConfig`. The bundle now generates one
Databricks Job per route (see `resources/landing_jobs.py`). Each job:

- Is triggered independently by a `FileArrivalTrigger` on its own source Volume.
- Runs with `--route-id <id>` so only one Autoloader stream is active per job run.
- Has `max_concurrent_runs: 1` to prevent self-overlap.

Route-level concurrency is now entirely managed by the Databricks Jobs scheduler, not by
the application. The route-worker thread pool (ADR-001, ADR-002) is no longer needed for
concurrency and has been removed.

On a `local[N]` cluster the JVM runs driver and executors in the same process with a fixed
number of executor threads. The relationship between active Autoloader streams and executor
thread consumption is not fully understood, but the incident below suggests that running too
many concurrent streams against the **same JVM** can exhaust available executor capacity
and silently block Spark actions inside `foreachBatch`.

## Incidents

### Incident 1 — 2026-06-18 (single-job architecture, misdiagnosed)

Observed with `max_concurrent_routes: 4` on a `local[4]` single-node cluster
(4-core VM, JVM heap `-Xms7850m -Xmx7850m`). Only 3 routes were active.

- All file copies completed successfully.
- `FileEventTable.append_batch()` never returned.
- GC logs showed a healthy JVM — no memory pressure, no GC pause.
- Job was terminated manually after ~90 minutes.
- Resolved after reducing to `max_concurrent_routes: 3`.

Originally attributed to Spark executor slot exhaustion. The 2026-07-10 incident suggests
the actual cause may have been a FUSE stall during the `df.collect()` or Delta write that
preceded the copy; reducing route count reduced FUSE concurrency and the stall did not recur.

### Incident 2 — 2026-07-10 (N-jobs architecture, root cause identified)

Observed with N-jobs on a **shared** `local[4]` cluster, `file_backend: os`, 10 jobs
triggered in parallel. 2–3 jobs hung every time.

- The streaming batch 0 offset was committed (backfill scan completed).
- The intent store was written (`in_progress`, all 8 × 100 MB files pending).
- Log directory was empty — no file moves had completed.
- `future.result()` in `_execute_copies` was blocking forever with no timeout.

**Root cause:** The UC Volume FUSE kernel driver can stall indefinitely on Azure ADLS network
issues. `shutil.copy()` (NativeOsBackend) blocks at the `read()`/`write()` syscall level
with no built-in timeout. Because `future.result()` was called without a timeout, the
`foreachBatch` thread — and therefore `query.awaitTermination()` — never returned.

This is distinct from Spark slot exhaustion: Spark itself was healthy; the hang was entirely
in the Python file-copy thread.

**Mitigated (not fixed):** Switched default `file_backend` to `fsspec-databricks` (HTTP REST,
not FUSE). With 20 parallel jobs and `fsspec-databricks`, all runs succeeded — no hangs
observed. This is consistent with the FUSE hypothesis: HTTP REST does not go through the
FUSE kernel driver, so the syscall-level stall does not occur.

However, `fsspec-databricks` does not eliminate the risk: HTTP requests can also stall
indefinitely (no default socket timeout in the underlying client). The definitive fix is a
per-file timeout on `future.result()` — see Decision section below.

## Decision

### Current mitigation — `fsspec-databricks` as default backend

`file_backend: fsspec-databricks` is now the default in `config/landing_config.yml`.
HTTP REST avoids the FUSE kernel layer entirely, eliminating the observed hang vector.
20 parallel jobs with `fsspec-databricks` completed successfully in both test runs.

This is a **mitigation**, not a fix. HTTP stalls can still occur (no socket timeout in the
underlying client), and the `os` backend remains available for environments where FUSE
performance is preferred. The code is still vulnerable to indefinite blocking regardless of
backend.

### Planned fix — `future.result(timeout=N)` (NOT YET IMPLEMENTED)

The definitive fix is to add a per-file deadline to every `future.result()` call in
`LandingMoverJob`. This converts a silent infinite hang into a FAILED event, which Autoloader
retries on the next run.

**Exact call sites to fix (both in `main.py`):**

1. `LandingMoverJob._execute_copies()` — iterates `futures.items()`, calls `future.result()`.
2. `LandingMoverJob._delete_sources()` — iterates `futures.items()`, calls `future.result()`.

**Implementation sketch:**

```python
from concurrent.futures import TimeoutError as FutureTimeoutError

FILE_IO_TIMEOUT_SECONDS = 900  # 15 min — floor throughput ~11 MB/s; safe even under heavy ADLS throttling
# TODO: expose as AppConfig field (e.g. file_io_timeout_seconds: int = 900).
#  Wire into BatchProcessor.__init__ and pass through to LandingMoverJob._execute_copies / _delete_sources.

# In _execute_copies:
for future, copier in futures.items():
    try:
        elapsed, error, _ = future.result(timeout=FILE_IO_TIMEOUT_SECONDS)
    except FutureTimeoutError:
        elapsed = FILE_IO_TIMEOUT_SECONDS
        error = TimeoutError(f"Copy timed out after {FILE_IO_TIMEOUT_SECONDS}s")
    if error:
        ...  # existing FAILED-event path handles it

# In _delete_sources:
for future, path in futures.items():
    try:
        _, error, _ = future.result(timeout=FILE_IO_TIMEOUT_SECONDS)
    except FutureTimeoutError:
        error = TimeoutError(f"Delete timed out after {FILE_IO_TIMEOUT_SECONDS}s")
    if error:
        errors.append((path, error))
```

**Important caveat:** A timeout on `future.result()` unblocks the *calling* thread but does
**not** kill the stuck *worker* thread — a thread blocked in a kernel syscall (FUSE read/write)
cannot be interrupted from Python. The stuck thread will remain parked until the cluster
restarts, consuming one `max_file_workers` slot for the rest of the job run. With
`max_file_workers: 2` this leaves only one slot available. This is acceptable — the job
progresses rather than hanging forever — but operators should be aware that a timed-out file
does not free its worker thread.

`FILE_IO_TIMEOUT_SECONDS = 900` (15 min) is derived from a minimum acceptable throughput of
**~11 MB/s** (10 GB ÷ 900 s), well below any realistic ADLS floor even under heavy throttling.
ADR-004 benchmark: `fsspec-databricks` at 1 worker sustains ~200 MB/s peak; 900 s gives
~80× headroom, ensuring the timeout only fires on a true stall, not a degraded transfer.

When implementing, expose this as an `AppConfig` field (`file_io_timeout_seconds: int = 900`)
so operators can tune per environment without code changes.

### Cluster topology (shared vs. dedicated)

The original Spark slot starvation concern (see Incident 1) still applies when jobs share a
`local[N]` cluster — multiple concurrent Autoloader streams against the same JVM can exhaust
executor threads. With N-jobs on dedicated clusters (one cluster per job, current default),
there is no cross-job contention and any number of routes can run in parallel.

## Consequences

- `max_concurrent_routes` removed from `AppConfig`; the N-jobs model makes it obsolete.
- Default `file_backend` changed to `fsspec-databricks` as a mitigation for FUSE stalls.
- The `future.result(timeout=N)` fix is tracked here and has not been implemented yet.
- For dedicated-cluster deployments: no slot starvation risk; FUSE stall risk mitigated by
  `fsspec-databricks`.
- For shared-cluster deployments on `local[4]`: keep simultaneously triggered jobs ≤ 3
  (Spark slot starvation risk persists regardless of backend).

```

###### FILE: docs/adr/011-cost-simulation-webapp.md ######

```md
# ADR-011: Maintain a Lightweight Streamlit Cost Simulation App

## Status

Accepted — Architecture C (N Job, N Autoloader) on Serverless selected.

## Context

The project needed a rough cost estimate for three ingestion architectures:

- `A`: 1 Job, 1 Autoloader
- `B`: 1 Job, N Autoloader
- `C`: N Job, N Autoloader (1 autoloader each)

and two compute modes (Serverless and Job Compute), without running Databricks jobs for
every what-if scenario.

The main difficulty was estimation under batching assumptions. Small changes in route-level
and cross-route batching can significantly change trigger counts and monthly ranking.

Because of this, a spreadsheet-only approach was not enough. We needed an interactive tool
to quickly test assumptions and compare outcomes.

## Decision

Creation of an interactive Streamlit cost simulation app in `docs/cost-simulation/webapp/app.py` as a
lightweight decision-support tool.

The app will:
- Reuse the documented pricing model and formulas from `docs/cost-simulation/prices-and-sources.md`.
- Expose interactive controls for scenario, batching, and file-count assumptions.
- Provide comparison visualizations and a ranked analysis table for all 3x2 combinations:
	three architectures (`A`, `B`, `C`) x two compute modes (Serverless, Job Compute).
- Remain documentation/support tooling (not production runtime logic for ingestion jobs).

## Simulation Results

The app was run under the **Medium scenario** (batched per route: 55%, batched across
routes: 25%). Results ranked by monthly cost:

![Cost comparison across architectures A, B, C — Serverless vs Job Compute](img/011-cost-comparison-architectures-A-B-C.png)

| Rank | Architecture | Compute | Cost / month | Triggers / day | Δ vs best |
|---|---|---|---|---|---|
| 1 | A — 1 Job, 1 Autoloader | Serverless | $228.37 | 745.87 | — |
| 2 | C — N Job, N Autoloader | Serverless | $257.37 | 994.50 | +12.70% |
| 3 | B — 1 Job, N Autoloader | Serverless | $849.93 | 745.87 | +272.18% |
| 4 | A — 1 Job, 1 Autoloader | Job Compute | $971.32 | 745.87 | +325.34% |
| 5 | C — N Job, N Autoloader | Job Compute | $1,241.20 | 994.50 | +443.51% |
| 6 | B — 1 Job, N Autoloader | Job Compute | $1,682.39 | 745.87 | +636.71% |

Key findings:

- **Switching to Serverless compute is the single biggest cost lever.** All three Serverless
  options rank above all three Job Compute options. The compute mode gap dwarfs
  architecture-level differences.
- **Architecture A is the cheapest Serverless option** at $228.37/month.
- **Architecture C is only 12.7% more expensive than A on Serverless** ($257.37/month).
- **Architecture B is the most expensive Serverless option** — 272% above A — due to a higher
  trigger count (994.50 vs 745.87 triggers/day) caused by N independent Autoloaders firing
  separately within the same job.

## Architecture Decision: C (N Job, N Autoloader)

**Architecture C is selected** as the target operating model.

Decision rationale:

- **A is cheaper but operationally more complex**:  files from multiple routes arrive unsorted 
  and require additional mapping logic to write to the correct `file_event` table within a Spark Streaming Job.
  Furthermore, A doesn't provides full isolation per routes.
- **C is more straightforward**: each job has one Autoloader and writes to one dedicated
  `_file_event_` table, which removes cross-route mapping complexity.
- **B is discarded on price alone**: under the Medium scenario on Serverless,
  B is +272.18% vs A and is not cost-competitive.

Despite A being 12.7% cheaper, C is preferred for simplicity and route-level isolation.

| Criterion | A (1 Job, 1 Autoloader) ✗ | B (1 Job, N Autoloader) ✗ | C (N Job, N Autoloader) ✓ |
|---|---|---|---|
| Topology | 1 job with 1 Autoloader handling all routes | 1 job with N Autoloaders (one per route) | N jobs with N Autoloaders (one per route) |
| Serverless monthly cost | $228.37 (cheapest) | $849.93 (most expensive) | $257.37 |
| Isolation | Route failures share blast radius | Full isolation per route | Full isolation per route |


## Consequences

- Architecture and cost conversations can be validated quickly with a shared, executable model.
- Assumptions become explicit and easier to challenge in reviews.
- Formula drift risk is reduced by keeping the app and pricing reference in the same repository.
- Outputs are indicative for planning, but not billing-grade accounting.
- The chosen architecture (C, Serverless) trades a 12.7% cost premium over A for
  operational simplicity and full route isolation.

```

###### FILE: docs/adr/012-per-route-autoloader-filename-reuse.md ######

```md
# ADR-012: Per-Route Autoloader Filename Reuse

## Status

Accepted

## Context

Some data sources send corrected files. There are two common patterns:

**Pattern A: Same filename, replaced with corrections**
- Day 1: `sales_data_2024_01.csv` uploaded to Landing (processed as LOADED).
- Day 2: Sender discovers errors and resends `sales_data_2024_01.csv` (overwriting the Day 1 file).
- **Problem**: By default, Autoloader processes files exactly once by path — the corrected file is **not** re-processed, leaving stale data in Raw.

**Pattern B: Different filename with new extraction timestamp**
- Day 1: `sales_data_2024_01_15_20240115000500.csv` uploaded.
- Day 2: `sales_data_2024_01_15_20240115000800.csv` uploaded (new timestamp in name, different filename).
- **No problem**: Each filename is unique, so Autoloader sees each as a new file and processes it naturally. `allow_filename_reuse=false` (default) works fine.

This ADR addresses **Pattern A only**. Pattern B works out-of-the-box with immutable filenames.

Pattern A is **not compatible with Autoloader's default configuration**. By default, Autoloader tracks each file by path and prevents reprocessing the same filename — the corrected file is silently skipped. Enabling `allowOverwrites=true` allows Autoloader to reprocess a file when its modification time changes, but introduces a side effect in file notification mode: Autoloader can discover the same physical file through two channels simultaneously:

1. **File notification** (event-driven) — reports the file with one timestamp (event time from queue)
2. **Periodic directory listing** (daily, or at least once in file events mode) — may report the same file with a different timestamp (file modification time from storage)

Because Autoloader's file identity is `(path, timestamp)`, these two discovery channels create **duplicate metadata entries** for the same single physical file. Autoloader then attempts to ingest the identical file twice, causing a failure on the second attempt — only one physical copy of the file exists in Landing, and it was already deleted after the first ingestion.

To prevent this, `allow_filename_reuse=true` also disables FileEvent (`cloudFiles.useManagedFileEvents = "false"`) and switches to **directory listing only** — a single discovery channel with a single timestamp source, which eliminates spurious duplicates.

Per the [Autoloader FAQ](https://learn.microsoft.com/en-gb/azure/databricks/ingestion/cloud-object-storage/auto-loader/faq):

> In file notification mode, Auto Loader might identify new files through both file notifications and directory listing. Because file notification event time and file modification time can differ, Auto Loader could receive two different timestamps and ingest the same file twice, even if the file hasn't been updated.

The result is **spurious duplicates of unchanged file content**, violating exactly-once semantics.

## Decision

Add a per-route boolean flag `allow_filename_reuse` (default `false`) to `LandingRoute` configuration. When enabled for a specific route:

1. **Disable FileEvent**: set `cloudFiles.useManagedFileEvents = "false"` to use directory listing only (single discovery channel, single timestamp source).
2. **Enable allowOverwrites**: set `cloudFiles.allowOverwrites = "true"` to reprocess files when modification time changes.
3. **Acknowledge downstream responsibility**: the user must de-duplicate at the Raw→next-layer boundary using a downstream MERGE on business keys. Autoloader will reprocess the entire file when its timestamp changes; the Raw layer absorbs both old and new rows, and de-duplication happens downstream.

The Raw layer itself remains **immutable** — each version of a file is written to a new target path, never overwritten. Corrections appear inside a new target path; downstream logic reconciles them.

## Rationale

### Why per-route, not app-level?

Different sources have different stability guarantees:
- **Immutable filenames** (Pattern B): Each extraction uses a unique filename (e.g., with `extractionTimestamp`). No corrections needed; Autoloader handles these out-of-the-box.
- **Overwritten filenames** (Pattern A): Same filename is resent with corrections. Requires `allow_filename_reuse=true` for that specific route.

A per-route flag allows fine-grained control: routes with immutable filenames keep the default (safe, exactly-once by default), while routes with overwritten filenames can opt-in to correction support.

### Why switch to directory listing mode?

Directory listing avoids the dual-discovery issue entirely. With only one discovery channel (periodic listing) and one timestamp source (file modification time on storage), the same file cannot be discovered twice with different timestamps. This eliminates spurious duplicates.

### Why does Raw stay immutable?

Even with `allowOverwrites=true`, the job writes each file version to a new timestamped partition. Corrections are appended as new data, not merged in place. This preserves the audit trail (both old and corrected rows are visible in Raw) and shifts de-duplication responsibility to downstream layers, where business logic is available.

### Checkpoint invalidation and recovery

Toggling `useManagedFileEvents` can invalidate Autoloader's FileEvent continuation token. Depending on checkpoint and runtime state, the next run **may** raise `CF_MANAGED_FILE_EVENTS_INVALID_CONTINUATION_TOKEN` and require a full directory re-listing. This is typically a one-time recovery cost when it occurs (it may not occur). Use the recovery toggle only when the error is observed. The recovery sequence is:

1. If the run fails with `CF_MANAGED_FILE_EVENTS_INVALID_CONTINUATION_TOKEN`, set `allow_filename_reuse_recovery: true` on the affected route (this injects `cloudFiles.listOnStart = "true"` and `cloudFiles.validateOptions = "false"` - see References)
2. Restart the stream
3. After a successful micro-batch, set `allow_filename_reuse_recovery: false` and restart again

## Implications

### Positives

- **Targeted**: only affects routes that need it; default behavior unchanged for immutable sources.
- **Safe in directory listing mode**: eliminates spurious duplicates.
- **Preserves Raw immutability**: corrections are new partitions, not overwrites.
- **Shifts responsibility clearly**: Autoloader handles file re-ingestion; user handles de-duplication downstream.

### Negatives

- **User responsibility**: de-duplication is not automatic; users must implement downstream MERGE logic on business keys.
- **One-time operational cost**: first toggle triggers full re-listing (plan for ingestion spike).
- **Requires downstream coordination**: raw layer de-duplication requires knowledge of business keys in target schemas.

## Example Usage

### Source with immutable filenames (default, no flag needed)

```yaml
landing_routes:
  - id: daily-sales-reports
    source: /Volumes/catalog/landing/daily_sales/
    target: /Volumes/catalog/raw/daily_sales/
    enabled: true
    # allow_filename_reuse: false (default)
```

Each file has a unique name with extraction timestamp embedded: `sales_data_extraction_2024_01_15.csv`, `sales_data_extraction_2024_01_16.csv`, etc. Autoloader processes each as a new file naturally.

### Source with overwritten filenames (corrections via same filename)

```yaml
landing_routes:
  - id: sales-corrections
    source: /Volumes/catalog/landing/sales_corrections/
    target: /Volumes/catalog/raw/sales_corrections/
    enabled: true
    allow_filename_reuse: true  # Sender overwrites same filename with corrections
```

Same filename (`sales_data_2024_01.csv`) is resent when corrections are needed:

1. Autoloader detects the file via directory listing (modification time changed).
2. File is copied to a new timestamped partition in Raw.
3. `LOADED` event is recorded with timestamp of new partition.
4. **Downstream job**: MERGE on `(date, sales_id)` to keep only the latest version of each row.

### Source with immutable files (default)

```yaml
  - id: immutable-source
    source: /Volumes/catalog/landing/immutable/
    target: /Volumes/catalog/raw/immutable/
    enabled: true
    # allow_filename_reuse: false (default)
```

Files are processed exactly once by path; resend is not expected.

## References

- [Autoloader FAQ - Handling Overwrites](https://learn.microsoft.com/en-gb/azure/databricks/ingestion/cloud-object-storage/auto-loader/faq)
- [Auto Loader options - File notification (`cloudFiles.listOnStart`)](https://learn.microsoft.com/en-gb/azure/databricks/spark/api-options#al-file-notification)
- [ADR-005: Exactly-Once File Move Semantics](005-exactly-once-file-move-semantics.md)
- `config.py`: `allow_filename_reuse` and `allow_filename_reuse_recovery` fields
- `main.py`: `_get_autoloader_options()` function

```

###### FILE: docs/cost-simulation/prices-and-sources.md ######

```md
# Prices and Sources (West Europe, Premium, snapshot 2026-06-24)

## 1) Official sources

- Azure VM pricing page:
  - https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux-previous/
- Azure Databricks pricing page:
  - https://azure.microsoft.com/en-us/pricing/details/databricks/
- Serverless DBU consumption by SKU
  - https://learn.microsoft.com/en-us/azure/databricks/resources/pricing
- Billable usage system table reference:
  - https://learn.microsoft.com/en-us/azure/databricks/admin/system-tables/billing

## 2) Public list prices used

These are the public list prices retained as baseline for the compute options in scope.

| SKU | Price (USD/DBU-hour) | DBU multiplier | Note |
|---|---:|---:|---|
| Automated Serverless (Jobs/SDP) | 0.50 | ×1 | All-inclusive (no separate VM or startup charge) |
| Jobs Compute (classic clusters) | 0.30 | ×1 | DBU charge only — VM, startup, and idle billed separately and needs to be included |


### VM pricing (classic compute only)

| VM | Spec | Price (USD/hour) |
|---|---|---:|
| Standard D4ds v5 | 4 vCPU · 16 GiB · 150 GiB temp SSD | 0.2720 |

Pay-as-you-go, no Hybrid Benefit, West Europe region.

**Derived all-in classic rate:** $0.2720 + 1.0 × $0.30 = **$0.572/h**



## 3) Confirmed interpretation rules

- **Billing models are not comparable on a per-DBU basis.** Classic is additive
  (DBU charge + VM charge + startup time billed). Serverless is all-inclusive at $0.50/DBU;
  no VM, no startup overhead, no idle charge. The correct comparison is total cost per job run.
- Serverless Jobs are billed under the Automated Serverless SKU.
- Serverless Jobs use DBU multiplier, assumeted to be 1.0 effective (an SQL query against the `system.billing.usage` table is needed to get the exact value).
- ADP applies service-level uplifts on top of public list prices. Insert the confirmed ADP
  uplift factors in the simulation before making a final budget decision.
- For Managed file events, notification-resource count does not scale with the number of
  Autoloader queries (root query vs many per subpath), unlike the classic "one queue per
  stream" model. The expected delta between 1 and 150 queries is compute and stream
  management overhead, not notification-resource count. Current docs present file events as a
  cost reducer (mainly by lowering cloud-storage listing costs) but do not define explicit
  billing semantics for the Managed file events service itself, *so treat Managed file events
  service billing as unspecified rather than free*.

## 4) Cost formulas by compute type

### 4.1 Real comparison

- Job Cluster: `0.30 per DBU + VM + startup`
  - Startup : **7 minutes** per run.
- Serverless: `0.50 per DBU, no VM, no startup`

### 4.2 Definitions

- `t_work_s`: run execution time in seconds (work only)
- `t_startup_s`: classic startup time in seconds (here: `7 × 60 = 420 s`)
- `vm_rate_h`: VM price per hour (`0.2720`)
- `dbu_rate_jobs_h`: Jobs Compute DBU price (`0.30`)
- `dbu_rate_srv_h`: Serverless DBU price (`0.50`)
- `dbu_mult_jobs`: Jobs Compute DBU multiplier (here: `1.0`)
- `dbu_mult_srv`: Serverless effective DBU multiplier (here: `1.0`)
- `t_copy`: file copy time per file (from §6: `4 s` small / `14 s` medium / `55 s` large)
- `t_delta_write`: Delta table append time per route per run (estimate: `5 s`)
- `t_delete`: source file delete time per file (estimate: `1 s`)
- `n_routes_batch`: number of routes with files in a single run
- `n_files_batch`: number of files processed in a single run
- `n_parallel_autoloader_case_b`: number of Case B Autoloader queries processed in parallel (default: `3`)

### 4.3 Job Cluster formulas

#### 4.3.1 Literal formulas

```
cost_startup(t_startup_s) = vm_rate_h × (t_startup_s / 3600)
cost_work(t_work_s) = (vm_rate_h × (t_work_s / 3600)) + ((t_work_s / 3600) × dbu_rate_jobs_h × dbu_mult_jobs)
cost_run_classic(t_work_s, t_startup_s) = cost_work(t_work_s) + cost_startup(t_startup_s)
```

#### 4.3.2 Numeric formulas (with dbu_mult_jobs = 1.0, t_startup_s = 420)

```
cost_startup(420) = 0.2720 × (420 / 3600) = 0.03173 $ per run
cost_work(t_work_s) = (0.2720 × (t_work_s / 3600)) + ((t_work_s / 3600) × 0.30 × 1.0)
cost_run_classic(t_work_s, 420) = cost_work(t_work_s) + 0.03173
```

### 4.4 Serverless formulas

#### 4.4.1 Literal formula

```
cost_run_serverless(t_work_s) = (t_work_s / 3600) × dbu_rate_srv_h × dbu_mult_srv
```

#### 4.4.2 Numeric formula (with dbu_rate_srv_h = 0.50, dbu_mult_srv = 1.0)

```
cost_run_serverless(t_work_s) = (t_work_s / 3600) × 0.5 × 1.0
```

## 5) Architecture case definitions

| Case | Description |
|---|---|
| **A** | 1 Job · 1 FileTrigger on root · 1 Autoloader query scanning the whole root |
| **B** | 1 Job · 1 FileTrigger on root · N Autoloader queries (1 per subfolder, concurrent) — **current setup** |
| **C** | N Jobs · N FileTriggers (1 per subfolder) · 1 Autoloader query per job |

Difference between A and C:
- A is more efficient at batching: 1 trigger for all routes
- C triggers several routes in parallel

## 6) Measured run durations

File transfer times on a single-node D4ds v5 cluster (Databricks Unity Catalog Volumes):

| File size | Transfer time |
|---|---:|
| 100 MB | 4 s |
| 1 GB | 14 s |
| 10 GB | 55 s |

**Suggested file size distribution:**

| Tier | File size | Share | Transfer time |
|---|---:|---:|---:|
| Small | 100 MB | 60 % | 4 s |
| Medium | 1 GB | 30 % | 14 s |
| Large | 10 GB | 10 % | 55 s |

Weighted average transfer time: `0.60×4 + 0.30×14 + 0.10×55 = 2.4 + 4.2 + 5.5 = 12.1 s`

Empirical measurements for case B when a route has no data :

| Scenario | Duration |
|---|---:|
| 1 route, no data | 28 s |
| Overhead per additional route | +4 s |

The 28 s baseline is attributed to Job initialization, not to the Autoloader scan itself. Scan time difference between 1 subfolder and the full root is considered insignificant.


**Empirical timing formula:**

Case A:
```
t_work_A = 28 + n_files_batch × t_copy + n_routes_batch × t_delta_write + n_files_batch × t_delete   [seconds]
```

Case B (wave-based parallelism):
```
t_work_B(N_routes, n_parallel_autoloader_case_b) =
28 + ceil(N_routes / n_parallel_autoloader_case_b) × 4
+ n_files_batch × t_copy + n_routes_batch × t_delta_write + n_files_batch × t_delete   [seconds]
```

Case C (`n_routes_batch = 1` always — one route per job):
```
t_work_C = 28 + n_files_batch × t_copy + t_delta_write + n_files_batch × t_delete   [seconds]
```

The difference between A and C, is that A will batch the processing, while C will trigger individually for each route.


## 7) Workload model

| Group | Routes | Files / route / day | Total File / day |
|---|---:|---:|---:|
| Small | 10 | 1 | 10 |
| Medium | 120 | 10 | 1200 |
| Large  | 20 | 50 | 1000 |
| **Total** | **150** | — | **2210** |

To this, we need to add 2 factors:

- **% batched per route (`p_r`)**: files from the *same route* arrive close enough together that one trigger picks them all up. This is an intra-route mechanism — driven by the arrival density of a single producer. Applies to Cases A, B, and C.
- **% batched across route (`p_a`)**: files from *different routes* arrive close enough together that the single root-level trigger picks all of them up in one run. This is an inter-route mechanism — only possible when one trigger watches the whole root (Cases A and B). Case C is architecturally immune: each trigger watches only its own subfolder.

The two mechanisms are **statistically independent** (same-route bursts are not correlated with cross-route coincidences). The probability that any given file arrival fires a new trigger is therefore:

```
P(triggers) = (1 - p_r) × (1 - p_a)    [Cases A, B]
P(triggers) = (1 - p_r)                 [Case C]
```

### 7.1 Suggested batching values

Use four scenario levels for each batching factor:

| Scenario | % batched per route (p_r) | % batched across route (p_a) |
|---|---:|---:|
| Low batching | 30% | 10% |
| Medium batching | 55% | 25% |
| High batching | 75% | 50% |

Interpretation:

- 0% means no batching effect (about one trigger per file event).
- Higher percentages mean stronger batching (fewer triggers than files).
- `p_a` is not necessarily higher than `p_r`: cross-route coincidence requires files from *different* producers to land within the same trigger window, which is structurally harder than same-route bursts. In the Low and Medium scenarios `p_r > p_a`; only under heavy burst traffic (High) does cross-route batching dominate.

### 7.2 Trigger formulas with batching

- Case A: `triggers_day = total_files_day × (1 - batched_per_route) × (1 - batched_across)`
- Case B: `triggers_day = total_files_day × (1 - batched_per_route) × (1 - batched_across)` — same single FileTrigger on root as A
- Case C: `triggers_day = total_files_day × (1 - batched_per_route)`

With `total_files_day = 2210`, the resulting trigger ranges are:

Triggers are capped at **1440/day** (1 per minute — the physical minimum run cadence). Values above this are unrealisable and are clamped in the simulation.

| Scenario | Case A & B triggers/day | A/B frequency | Case C triggers/day |
|---|---:|---:|---:|
| Low    | 2210 × 0.70 × 0.90 = 1392 | every ~1.0 min | 2210 × 0.70 = 1547 → **capped 1440** |
| Medium | 2210 × 0.45 × 0.75 =  745 | every ~1.9 min | 2210 × 0.45 =  995 |
| High   | 2210 × 0.25 × 0.50 =  276 | every ~5.2 min | 2210 × 0.25 =  553 |

### 7.3 Per-case cost formulas

These combine §6 timing, §7.2 trigger counts, and §4 cost functions.

#### Case A — 1 job, 1 Autoloader query on root

Literal:
```
t_work_A = 28 + n_files_batch × t_copy + n_routes_batch × t_delta_write + n_files_batch × t_delete
triggers_day_A(p_r, p_a) = total_files_day × (1 - p_r) × (1 - p_a)
cost_day_A(p_r, p_a) = triggers_day_A(p_r, p_a) × cost_run(t_work_A, t_startup_s)
cost_month_A = cost_day_A × 30
```

Numeric (by file tier, baseline n_routes_batch = 1, n_files_batch = 1):
```
t_work_A (small)  = 28 + 1×4 + 1×5 + 1×1 =  38 s  →  classic: 0.03777 $/run  |  serverless: 0.00528 $/run
t_work_A (medium) = 28 + 1×14 + 1×5 + 1×1 =  48 s  →  classic: 0.03936 $/run  |  serverless: 0.00667 $/run
t_work_A (large)  = 28 + 1×55 + 1×5 + 1×1 =  89 s  →  classic: 0.04587 $/run  |  serverless: 0.01236 $/run
Each additional file in batch: +(t_copy + t_delete) s/run; each additional route: +t_delta_write s/run
triggers_day_A = 2210 × (1 - p_r) × (1 - p_a)
```

#### Case B — 1 job, N Autoloaders with bounded parallel runs

Literal:
```
t_work_B(N_routes, n_parallel_autoloader_case_b) =
28 + ceil(N_routes / n_parallel_autoloader_case_b) × 4
+ n_files_batch × t_copy + n_routes_batch × t_delta_write + n_files_batch × t_delete
triggers_day_B(p_r, p_a) = total_files_day × (1 - p_r) × (1 - p_a)
cost_day_B(p_r, p_a) = triggers_day_B(p_r, p_a) × cost_run(t_work_B, t_startup_s)
cost_month_B = cost_day_B × 30
```

Numeric (N_routes = 150, n_parallel_autoloader_case_b = 3, by file tier, baseline n_routes_batch = 1, n_files_batch = 1):
```
t_work_B (small)  = 28 + 200 + 1×4 + 1×5 + 1×1 = 238 s  →  classic: 0.06956 $/run  |  serverless: 0.03306 $/run
t_work_B (medium) = 28 + 200 + 1×14 + 1×5 + 1×1 = 248 s  →  classic: 0.07114 $/run  |  serverless: 0.03444 $/run
t_work_B (large)  = 28 + 200 + 1×55 + 1×5 + 1×1 = 289 s  →  classic: 0.07764 $/run  |  serverless: 0.04014 $/run
Each additional file in batch: +(t_copy + t_delete) s/run; each additional route: +t_delta_write s/run
triggers_day_B = 2210 × (1 - p_r) × (1 - p_a)
```

#### Case C — 150 independent jobs, 1 Autoloader each

Literal:
```
t_work_C = 28 + n_files_batch × t_copy + t_delta_write + n_files_batch × t_delete
triggers_day_C(p_r) = total_files_day × (1 - p_r)
cost_day_C(p_r) = triggers_day_C(p_r) × cost_run(t_work_C, t_startup_s)
cost_month_C = cost_day_C × 30
```

Numeric (by file tier, baseline n_files_batch = 1; n_routes_batch = 1 always):
```
t_work_C (small)  = 28 + 1×4 + 5 + 1×1 =  38 s  →  classic: 0.03777 $/run  |  serverless: 0.00528 $/run
t_work_C (medium) = 28 + 1×14 + 5 + 1×1 =  48 s  →  classic: 0.03936 $/run  |  serverless: 0.00667 $/run
t_work_C (large)  = 28 + 1×55 + 5 + 1×1 =  89 s  →  classic: 0.04587 $/run  |  serverless: 0.01236 $/run
Each additional file batched per route: +(t_copy + t_delete) s/run
triggers_day_C = 2210 × (1 - p_r)
```

> Note: C and A have identical `t_work` at equal `n_files_batch`, and DBU metering is not affected by parallelism, so the formula `triggers_day × cost_run` is correct for both. The 28 s overhead is Job initialization and applies equally to A and C.
>
> **Deriving batch sizes for simulation:**
> - `n_files_batch_AB = 1 / ((1 - p_r) × (1 - p_a))` — both suppression mechanisms reduce triggers, so each trigger absorbs more files
> - `n_routes_batch_A = n_files_batch_AB × (1 - p_r) = 1 / (1 - p_a)` — per-route-batched files share a route already counted, so only across-route batching drives route diversity per batch
> - `n_files_batch_C = 1 / (1 - p_r)` — only per-route batching applies; always 1 route per batch
>
> One unmodelled factor: Case C may require up to 150 warm cluster instances in parallel to avoid concurrent startup costs, while A needs only 1. Pool idle/reservation cost scales with pool size and is not modelled here.


```

###### FILE: docs/cost-simulation/webapp/app.py ######

```py
from __future__ import annotations

from dataclasses import dataclass
import math
from pathlib import Path
from typing import Any

import pandas as pd
import streamlit as st

try:
    import plotly.graph_objects as go
    _plotly_available = True
except ImportError:
    go = None
    _plotly_available = False


@dataclass(frozen=True)
class ScenarioBatching:
    per_route: float
    across_route: float


DEFAULTS: dict[str, Any] = {
    "pricing": {
        "vm_rate_h": 0.2720,
        "dbu_rate_jobs_h": 0.30,
        "dbu_rate_srv_h": 0.50,
        "dbu_mult_jobs": 1.0,
        "dbu_mult_srv": 1.0,
    },
    "timing": {
        "t_startup_s": 420.0,
        "t_init_s": 28.0,
        "t_route_overhead_s": 4.0,
        "t_delta_write": 5.0,
        "t_delete": 1.0,
    },
    "workload": {
        "routes_small": 10,
        "routes_medium": 120,
        "routes_large": 20,
        "files_per_route_small": 1,
        "files_per_route_medium": 10,
        "files_per_route_large": 50,
        "n_routes_total_for_case_b": 150,
        "n_parallel_autoloader_case_b": 3,
    },
    "file_tiers": {
        "share_small": 0.60,
        "share_medium": 0.30,
        "share_large": 0.10,
        "t_copy_small": 4.0,
        "t_copy_medium": 14.0,
        "t_copy_large": 55.0,
    },
    "batching": {
        "Low": ScenarioBatching(per_route=0.30, across_route=0.10),
        "Medium": ScenarioBatching(per_route=0.55, across_route=0.25),
        "High": ScenarioBatching(per_route=0.75, across_route=0.50),
    },
    "active_scenario": "Medium",
}


CASE_LABELS: dict[str, str] = {
    "A": "1 Job, 1 Autoloader",
    "B": "1 Job, N Autoloader",
    "C": "N Job, N Autoloader",
}

REVERSE_CASE_LABELS: dict[str, str] = {v: k for k, v in CASE_LABELS.items()}


MINUTES_PER_DAY = 1440


def _clamp_pct(value: float) -> float:
    return max(0.0, min(0.95, value))


def _files_day(cfg: dict[str, Any]) -> int:
    w = cfg["workload"]
    return int(
        w["routes_small"] * w["files_per_route_small"]
        + w["routes_medium"] * w["files_per_route_medium"]
        + w["routes_large"] * w["files_per_route_large"]
    )


def _weighted_copy_time(cfg: dict[str, Any]) -> float:
    ft = cfg["file_tiers"]
    share_sum = ft["share_small"] + ft["share_medium"] + ft["share_large"]
    if share_sum <= 0:
        return 0.0

    s_small = ft["share_small"] / share_sum
    s_medium = ft["share_medium"] / share_sum
    s_large = ft["share_large"] / share_sum

    return (
        s_small * ft["t_copy_small"]
        + s_medium * ft["t_copy_medium"]
        + s_large * ft["t_copy_large"]
    )


def _compute_rows(cfg: dict[str, Any]) -> list[dict[str, Any]]:
    files_day = _files_day(cfg)
    t_copy = _weighted_copy_time(cfg)
    timing = cfg["timing"]
    pricing = cfg["pricing"]
    workload = cfg["workload"]

    scenario_name = cfg["active_scenario"]
    scenario = cfg["batching"][scenario_name]
    batched_per_route = _clamp_pct(scenario.per_route)
    batched_across = _clamp_pct(scenario.across_route)

    survival_ab = (1.0 - batched_per_route) * (1.0 - batched_across)
    n_files_batch_ab = 1.0 / max(1e-9, survival_ab)
    n_files_batch_c = 1.0 / max(1e-9, (1.0 - batched_per_route))

    n_routes_batch_a = 1.0 / max(1e-9, (1.0 - batched_across))
    n_routes_batch_c = 1.0

    triggers_ab_raw = files_day * (1.0 - batched_per_route) * (1.0 - batched_across)
    triggers_c_raw = files_day * (1.0 - batched_per_route)
    triggers_ab = min(triggers_ab_raw, float(MINUTES_PER_DAY))
    triggers_c = min(triggers_c_raw, float(MINUTES_PER_DAY))

    def t_work(
        case: str,
    ) -> tuple[float, float, float, float, float, float, float, float, int, int]:
        t_init = timing["t_init_s"]
        t_route = 0.0
        route_waves = 0
        parallel_runs = 1

        if case == "B":
            parallel_runs = max(
                1,
                int(workload["n_parallel_autoloader_case_b"]),
            )
            route_waves = math.ceil(workload["n_routes_total_for_case_b"] / parallel_runs)
            t_route = route_waves * timing["t_route_overhead_s"]
            n_files_batch = n_files_batch_ab
            n_routes_batch = float(route_waves)
        elif case == "A":
            n_files_batch = n_files_batch_ab
            n_routes_batch = n_routes_batch_a
        else:
            n_files_batch = n_files_batch_c
            n_routes_batch = n_routes_batch_c

        t_copy_total = n_files_batch * t_copy
        t_delta_total = n_routes_batch * timing["t_delta_write"]
        t_delete_total = n_files_batch * timing["t_delete"]
        total = t_init + t_route + t_copy_total + t_delta_total + t_delete_total
        return (
            total,
            t_init,
            t_route,
            t_copy_total,
            t_delta_total,
            t_delete_total,
            n_files_batch,
            n_routes_batch,
            route_waves,
            parallel_runs,
        )

    rows: list[dict[str, Any]] = []

    for case in ("A", "B", "C"):
        case_triggers_raw = triggers_ab_raw if case in ("A", "B") else triggers_c_raw
        case_triggers = triggers_ab if case in ("A", "B") else triggers_c
        triggers_capped = case_triggers_raw > MINUTES_PER_DAY
        (
            tw,
            t_init,
            t_route,
            t_copy_total,
            t_delta_total,
            t_delete_total,
            n_files_batch,
            n_routes_batch,
            route_waves,
            parallel_runs,
        ) = t_work(case)

        shared_fields = {
            "Case": case,
            "Scenario": scenario_name,
            "t_work_s": round(tw, 3),
            "n_files_batch": round(n_files_batch, 3),
            "n_routes_batch": round(n_routes_batch, 3),
            "triggers_day": round(case_triggers, 2),
            "t_copy_component_s": round(t_copy_total, 3),
            "t_delta_component_s": round(t_delta_total, 3),
            "t_delete_component_s": round(t_delete_total, 3),
            "t_init_component_s": round(t_init, 3),
            "t_route_component_s": round(t_route, 3),
            "files_day_total": files_day,
            "case_b_route_waves_used": route_waves,
            "case_b_parallel_runs_used": parallel_runs,
            "n_routes_total_for_case_b": workload["n_routes_total_for_case_b"],
            "t_route_overhead_s": timing["t_route_overhead_s"],
            "batched_per_route_used": batched_per_route,
            "batched_across_used": batched_across,
            "batching_factor_used": (
                survival_ab if case in ("A", "B") else (1.0 - batched_per_route)
            ),
            "triggers_capped": triggers_capped,
        }

        cost_run_srv = (
            (tw / 3600.0)
            * pricing["dbu_rate_srv_h"]
            * pricing["dbu_mult_srv"]
        )
        rows.append({
            **shared_fields,
            "Compute": "Serverless",
            "cost_run_usd": cost_run_srv,
            "cost_day_usd": case_triggers * cost_run_srv,
            "cost_month_usd": case_triggers * cost_run_srv * 30.0,
        })

        cost_work = (pricing["vm_rate_h"] * (tw / 3600.0)) + (
            (tw / 3600.0) * pricing["dbu_rate_jobs_h"] * pricing["dbu_mult_jobs"]
        )
        cost_startup = pricing["vm_rate_h"] * (timing["t_startup_s"] / 3600.0)
        cost_run_job = cost_work + cost_startup
        rows.append({
            **shared_fields,
            "Compute": "Job Compute",
            "cost_run_usd": cost_run_job,
            "cost_day_usd": case_triggers * cost_run_job,
            "cost_month_usd": case_triggers * cost_run_job * 30.0,
        })

    return rows


def _fmt_money(value: float) -> str:
    return f"${value:,.4f}"


def _display_case(case_code: str) -> str:
    return CASE_LABELS.get(case_code, case_code)


def _reset_shared_controls_state() -> None:
    st.session_state["shared_active_scenario"] = DEFAULTS["active_scenario"]

    default_batching = DEFAULTS["batching"][DEFAULTS["active_scenario"]]
    st.session_state["shared_batched_per_route"] = float(
        default_batching.per_route * 100.0
    )
    st.session_state["shared_batched_across_route"] = float(
        default_batching.across_route * 100.0
    )

    st.session_state["shared_files_small"] = int(
        DEFAULTS["workload"]["files_per_route_small"]
    )
    st.session_state["shared_files_medium"] = int(
        DEFAULTS["workload"]["files_per_route_medium"]
    )
    st.session_state["shared_files_large"] = int(
        DEFAULTS["workload"]["files_per_route_large"]
    )
    st.session_state["shared_parallel_case_b"] = int(
        DEFAULTS["workload"]["n_parallel_autoloader_case_b"]
    )


def _reset_settings_controls_state() -> None:
    st.session_state["settings_active_scenario"] = DEFAULTS["active_scenario"]
    for name, batching in DEFAULTS["batching"].items():
        st.session_state[f"batch_per_route_{name}"] = float(
            _clamp_pct(batching.per_route) * 100.0
        )
        st.session_state[f"batch_across_{name}"] = float(
            _clamp_pct(batching.across_route) * 100.0
        )
    for field in ("routes_small", "routes_medium", "routes_large",
                  "files_per_route_small", "files_per_route_medium", "files_per_route_large",
                  "n_parallel_autoloader_case_b", "n_routes_total_for_case_b"):
        st.session_state[f"settings_workload_{field}"] = DEFAULTS["workload"][field]
    for field, value in DEFAULTS["timing"].items():
        st.session_state[f"settings_timing_{field}"] = value
    for field, value in DEFAULTS["pricing"].items():
        st.session_state[f"settings_pricing_{field}"] = value
    for field, value in DEFAULTS["file_tiers"].items():
        st.session_state[f"settings_file_tiers_{field}"] = value


def _reset_defaults_state() -> None:
    for key, value in DEFAULTS.items():
        st.session_state[f"cfg_{key}"] = value

    _reset_shared_controls_state()
    _reset_settings_controls_state()


def _on_shared_reset() -> None:
    _reset_shared_controls_state()
    _reset_settings_controls_state()


def _on_settings_workload_change() -> None:
    for field, shared_key in (
        ("files_per_route_small", "shared_files_small"),
        ("files_per_route_medium", "shared_files_medium"),
        ("files_per_route_large", "shared_files_large"),
        ("n_parallel_autoloader_case_b", "shared_parallel_case_b"),
    ):
        widget_key = f"settings_workload_{field}"
        st.session_state[shared_key] = int(st.session_state[widget_key])


def _on_settings_batching_change(name: str) -> None:
    active_scenario = st.session_state.get("shared_active_scenario", DEFAULTS["active_scenario"])
    if name == active_scenario:
        st.session_state["shared_batched_per_route"] = float(
            st.session_state[f"batch_per_route_{name}"]
        )
        st.session_state["shared_batched_across_route"] = float(
            st.session_state[f"batch_across_{name}"]
        )


def _on_settings_active_scenario_change() -> None:
    scenario_name = st.session_state["settings_active_scenario"]
    st.session_state["shared_active_scenario"] = scenario_name
    scenario = _load_batching_for_scenario(scenario_name)
    st.session_state["shared_batched_per_route"] = float(scenario.per_route * 100.0)
    st.session_state["shared_batched_across_route"] = float(scenario.across_route * 100.0)


def _ensure_shared_controls_state(cfg: dict[str, Any]) -> None:
    if "shared_files_small" not in st.session_state:
        st.session_state["shared_files_small"] = int(
            cfg["workload"]["files_per_route_small"]
        )
    if "shared_files_medium" not in st.session_state:
        st.session_state["shared_files_medium"] = int(
            cfg["workload"]["files_per_route_medium"]
        )
    if "shared_files_large" not in st.session_state:
        st.session_state["shared_files_large"] = int(
            cfg["workload"]["files_per_route_large"]
        )
    if "shared_parallel_case_b" not in st.session_state:
        st.session_state["shared_parallel_case_b"] = int(
            cfg["workload"]["n_parallel_autoloader_case_b"]
        )
    if "shared_active_scenario" not in st.session_state:
        st.session_state["shared_active_scenario"] = cfg["active_scenario"]

    active_scenario = st.session_state["shared_active_scenario"]
    active = cfg["batching"][active_scenario]

    if "shared_batched_per_route" not in st.session_state:
        st.session_state["shared_batched_per_route"] = float(
            _clamp_pct(active.per_route) * 100.0
        )
    if "shared_batched_across_route" not in st.session_state:
        st.session_state["shared_batched_across_route"] = float(
            _clamp_pct(active.across_route) * 100.0
        )
    for name, batching in cfg["batching"].items():
        if f"batch_per_route_{name}" not in st.session_state:
            st.session_state[f"batch_per_route_{name}"] = float(
                _clamp_pct(batching.per_route) * 100.0
            )
        if f"batch_across_{name}" not in st.session_state:
            st.session_state[f"batch_across_{name}"] = float(
                _clamp_pct(batching.across_route) * 100.0
            )


def _load_batching_for_scenario(scenario_name: str) -> ScenarioBatching:
    batching = st.session_state.get("cfg_batching", DEFAULTS["batching"])
    scenario = batching.get(scenario_name, DEFAULTS["batching"][scenario_name])
    return ScenarioBatching(
        per_route=_clamp_pct(float(scenario.per_route)),
        across_route=_clamp_pct(float(scenario.across_route)),
    )


def _sync_shared_controls_to_tab(prefix: str) -> None:
    st.session_state[f"{prefix}_files_small"] = int(st.session_state["shared_files_small"])
    st.session_state[f"{prefix}_files_medium"] = int(st.session_state["shared_files_medium"])
    st.session_state[f"{prefix}_files_large"] = int(st.session_state["shared_files_large"])
    st.session_state[f"{prefix}_parallel_case_b"] = int(
        st.session_state["shared_parallel_case_b"]
    )
    st.session_state[f"{prefix}_active_scenario"] = st.session_state[
        "shared_active_scenario"
    ]
    st.session_state[f"{prefix}_batched_per_route"] = float(
        st.session_state["shared_batched_per_route"]
    )
    st.session_state[f"{prefix}_batched_across_route"] = float(
        st.session_state["shared_batched_across_route"]
    )


def _copy_tab_value_to_shared(prefix: str, field_name: str) -> None:
    st.session_state[f"shared_{field_name}"] = st.session_state[
        f"{prefix}_{field_name}"
    ]


def _on_tab_scenario_change(prefix: str) -> None:
    scenario_name = st.session_state[f"{prefix}_active_scenario"]
    st.session_state["shared_active_scenario"] = scenario_name
    scenario = _load_batching_for_scenario(scenario_name)
    st.session_state["shared_batched_per_route"] = float(scenario.per_route * 100.0)
    st.session_state["shared_batched_across_route"] = float(
        scenario.across_route * 100.0
    )


def _render_shared_controls(cfg: dict[str, Any], prefix: str) -> None:
    _sync_shared_controls_to_tab(prefix)

    st.caption("Workload per route")
    fs1, fs2, fs3 = st.columns(3)
    fs1.slider(
        "Small files",
        min_value=0,
        max_value=100,
        step=1,
        key=f"{prefix}_files_small",
        on_change=_copy_tab_value_to_shared,
        args=(prefix, "files_small"),
    )
    fs2.slider(
        "Medium files",
        min_value=0,
        max_value=100,
        step=1,
        key=f"{prefix}_files_medium",
        on_change=_copy_tab_value_to_shared,
        args=(prefix, "files_medium"),
    )
    fs3.slider(
        "Large files",
        min_value=0,
        max_value=100,
        step=1,
        key=f"{prefix}_files_large",
        on_change=_copy_tab_value_to_shared,
        args=(prefix, "files_large"),
    )

    st.caption("Batching for selected scenario")
    scenario_options = ["Low", "Medium", "High"]
    c1, c2, c3 = st.columns([2.2, 3, 3])
    c1.selectbox(
        "Scenario",
        scenario_options,
        key=f"{prefix}_active_scenario",
        help="Shared between Analysis and Comparison.",
        on_change=_on_tab_scenario_change,
        args=(prefix,),
    )
    c2.slider(
        "Batched per route (%)",
        min_value=0.0,
        max_value=95.0,
        step=1.0,
        key=f"{prefix}_batched_per_route",
        on_change=_copy_tab_value_to_shared,
        args=(prefix, "batched_per_route"),
    )
    c3.slider(
        "Batched across route (%)",
        min_value=0.0,
        max_value=95.0,
        step=1.0,
        key=f"{prefix}_batched_across_route",
        on_change=_copy_tab_value_to_shared,
        args=(prefix, "batched_across_route"),
    )


def _render_reference_intro(rows: list[dict[str, Any]]) -> None:
    case_b_row = next((r for r in rows if r["Case"] == "B"), None)
    n_routes = case_b_row["n_routes_total_for_case_b"] if case_b_row else 0
    parallel_runs = case_b_row["case_b_parallel_runs_used"] if case_b_row else 0
    route_waves = case_b_row["case_b_route_waves_used"] if case_b_row else 0
    t_route_overhead_s = case_b_row["t_route_overhead_s"] if case_b_row else 0
    t_route_overhead = case_b_row["t_route_component_s"] if case_b_row else 0

    st.markdown("### Architecture Overview")
    st.markdown(
        f"""
**Three compute architectures** are compared across **2 compute modes** (Serverless vs Job Compute):

- **1 Job, 1 Autoloader**: Single-route batching. Each job processes one route's files in a single batch.
  Route overhead is minimal (28s init + per-file copy/delete + Delta write per 1 file).

- **1 Job, N Autoloader**: All-routes batching. A single job processes all {n_routes} routes together in one massive batch.
    Main driver of cost difference: wave-based route overhead = ceil(N_routes / parallel_runs) × t_route_overhead_s.
    With current settings N_routes={n_routes} and parallel_runs={parallel_runs}, overhead is {route_waves} × {t_route_overhead_s:.1f}s = {t_route_overhead:.1f}s.
  Higher latency per trigger, but amortizes startup cost across {n_routes} routes.

- **N Job, N Autoloader (1 autoloader each)**: Per-route independent jobs. Each route runs its own dedicated Spark job continuously.
    No route-overhead penalty (only 1 route per job). Similar t_work to 1 Job, 1 Autoloader, but managed as independent workloads.

**Key Differentiator (1 Job, 1 Autoloader vs 1 Job, N Autoloader)**: batching strategy.
1 Job, N Autoloader's long tail overhead makes it more expensive per trigger unless startup costs dominate (unlikely in serverless).
    """
    )

    st.markdown("### Example Cost Table (Current Scenario)")
    st.caption("Showing representative costs for all 6 combinations (3 architectures × 2 compute modes).")

    example_data = []
    for row in rows:
        example_data.append(
            {
                "Case": _display_case(row["Case"]),
                "Compute": row["Compute"],
                "Cost/run": _fmt_money(row["cost_run_usd"]),
                "Cost/day": _fmt_money(row["cost_day_usd"]),
                "Cost/month": _fmt_money(row["cost_month_usd"]),
                "Triggers/day": f"{row['triggers_day']:.1f}",
                "t_work(s)": f"{row['t_work_s']:.1f}",
            }
        )
    st.dataframe(example_data, use_container_width=True, hide_index=True)


def _render_analysis(cfg: dict[str, Any], rows: list[dict[str, Any]]) -> None:
    st.subheader("Analysis (3 architectures x 2 compute modes)")

    st.caption(
        "Quick live controls: adjust key values here and the table updates immediately."
    )

    _render_shared_controls(cfg, "analysis")

    sort_col, desc_col, parallel_col, reset_col = st.columns([3, 1, 1.5, 0.8])
    sort_metric = sort_col.selectbox(
        "Sort by",
        ["cost_month_usd", "cost_day_usd", "cost_run_usd", "triggers_day"],
        index=0,
    )
    desc_col.write("")
    sort_desc = desc_col.toggle("Sort descending", value=False)
    parallel_col.number_input(
        "Case B parallel runs",
        min_value=1,
        max_value=500,
        step=1,
        key="analysis_parallel_case_b",
        on_change=_copy_tab_value_to_shared,
        args=("analysis", "parallel_case_b"),
    )
    reset_col.write("")
    reset_col.write("")
    reset_col.button(
        "Reset",
        key="analysis_reset_controls",
        on_click=_on_shared_reset,
        use_container_width=True,
    )

    sorted_rows = sorted(rows, key=lambda r: r[sort_metric], reverse=sort_desc)
    best = min(rows, key=lambda r: r["cost_month_usd"])

    ranked_rows: list[dict[str, Any]] = []
    for idx, row in enumerate(sorted_rows, start=1):
        delta = row["cost_month_usd"] - best["cost_month_usd"]
        delta_pct = (
            (delta / best["cost_month_usd"] * 100.0)
            if best["cost_month_usd"]
            else 0.0
        )
        ranked_rows.append(
            {
                "Rank": idx,
                "Case": _display_case(row["Case"]),
                "Compute": row["Compute"],
                "Scenario": row["Scenario"],
                "Cost / run": _fmt_money(row["cost_run_usd"]),
                "Cost / day": _fmt_money(row["cost_day_usd"]),
                "Cost / month": _fmt_money(row["cost_month_usd"]),
                "Triggers / day": (
                    f"{row['triggers_day']:.2f} [capped]"
                    if row["triggers_capped"]
                    else f"{row['triggers_day']:.2f}"
                ),
                "Delta vs best": _fmt_money(delta),
                "Delta % vs best": f"{delta_pct:.2f}%",
            }
        )

    case_colors = {
        _display_case("A"): "#5ba3d0",
        _display_case("B"): "#5eb85e",
        _display_case("C"): "#ffb347",
    }
    bold_cols = ["Triggers / day", "Delta vs best", "Delta % vs best"]

    def color_and_style_row(row: Any) -> list[str]:
        case_val = row.get("Case", "")
        bg_color = case_colors.get(case_val, "")
        styles = []
        for col_name in row.index:
            if col_name in bold_cols:
                style = "font-weight: bold"
            else:
                style = ""
            if bg_color:
                style += f";background-color: {bg_color};color: white" if style else f"background-color: {bg_color};color: white"
            styles.append(style)
        return styles

    df_ranked = pd.DataFrame(ranked_rows)
    styled = df_ranked.style.apply(color_and_style_row, axis=1)
    st.dataframe(styled, use_container_width=True, hide_index=True)

    best_monthly = _fmt_money(best["cost_month_usd"])
    st.caption(
        f"Best currently: {_display_case(best['Case'])} / "
        f"{best['Compute']} at {best_monthly} per month"
    )

    st.markdown("### Detailed Breakdown")
    options = [f"{_display_case(r['Case'])} / {r['Compute']}" for r in sorted_rows]
    selected = st.selectbox("Select a row", options)
    selected_row = sorted_rows[options.index(selected)]

    c1, c2, c3, c4 = st.columns(4)
    c1.metric("t_work", f"{selected_row['t_work_s']:.2f} s")
    c2.metric("n_files_batch", f"{selected_row['n_files_batch']:.3f}")
    c3.metric("n_routes_batch", f"{selected_row['n_routes_batch']:.3f}")
    c4.metric("triggers/day", f"{selected_row['triggers_day']:.2f}")

    route_component_label = "Route overhead (B only)"
    if selected_row["Case"] == "B":
        route_component_label = (
            "Route overhead (B only, "
            f"waves={selected_row['case_b_route_waves_used']}, "
            f"parallel={selected_row['case_b_parallel_runs_used']})"
        )

    st.table(
        [
            {
                "Component": "Initialization",
                "Seconds": f"{selected_row['t_init_component_s']:.3f}",
            },
            {
                "Component": route_component_label,
                "Seconds": f"{selected_row['t_route_component_s']:.3f}",
            },
            {
                "Component": "Copy",
                "Seconds": f"{selected_row['t_copy_component_s']:.3f}",
            },
            {
                "Component": "Delta write",
                "Seconds": f"{selected_row['t_delta_component_s']:.3f}",
            },
            {
                "Component": "Delete",
                "Seconds": f"{selected_row['t_delete_component_s']:.3f}",
            },
        ]
    )

    st.code(
        "t_work = init + route_overhead + copy + delta_write + delete\n"
        f"      = {selected_row['t_init_component_s']:.3f} + "
        f"{selected_row['t_route_component_s']:.3f} + "
        f"{selected_row['t_copy_component_s']:.3f} + "
        f"{selected_row['t_delta_component_s']:.3f} + "
        f"{selected_row['t_delete_component_s']:.3f}\n"
        f"      = {selected_row['t_work_s']:.3f} s",
        language="text",
    )

    st.markdown("### Computation Inputs Used")
    st.table(
        [
            {
                "Input": "total_files_day",
                "Value used": f"{selected_row['files_day_total']}",
            },
            {
                "Input": "batched_per_route",
                "Value used": f"{selected_row['batched_per_route_used']:.4f}",
            },
            {
                "Input": "batched_across_route",
                "Value used": f"{selected_row['batched_across_used']:.4f}",
            },
            {
                "Input": "batching_factor_used_for_case",
                "Value used": f"{selected_row['batching_factor_used']:.4f}",
            },
            {
                "Input": "case_b_parallel_runs",
                "Value used": f"{selected_row['case_b_parallel_runs_used']}",
            },
            {
                "Input": "case_b_route_waves",
                "Value used": f"{selected_row['case_b_route_waves_used']}",
            },
            {
                "Input": "n_files_batch",
                "Value used": f"{selected_row['n_files_batch']:.4f}",
            },
            {
                "Input": "n_routes_batch",
                "Value used": f"{selected_row['n_routes_batch']:.4f}",
            },
            {
                "Input": "triggers_day",
                "Value used": f"{selected_row['triggers_day']:.2f}",
            },
        ]
    )


@st.cache_data
def _compute_comparison_series(
    cfg: dict,
    case_a: str,
    compute_a: str,
    case_b: str,
    compute_b: str,
    x_mode: str,
) -> tuple[list[float], list[float], list[float]]:
    x_values: list[float] = []
    y_values_a: list[float] = []
    y_values_b: list[float] = []

    n_steps = 20

    def _cfg_base() -> dict:
        return {
            "pricing": dict(cfg["pricing"]),
            "timing": dict(cfg["timing"]),
            "workload": dict(cfg["workload"]),
            "file_tiers": dict(cfg["file_tiers"]),
            "batching": dict(cfg["batching"]),
            "active_scenario": cfg["active_scenario"],
        }

    def _append_costs(cfg_scaled: dict) -> None:
        rows = _compute_rows(cfg_scaled)
        y_values_a.append(
            next((r["cost_month_usd"] for r in rows if r["Case"] == case_a and r["Compute"] == compute_a), 0.0)
        )
        y_values_b.append(
            next((r["cost_month_usd"] for r in rows if r["Case"] == case_b and r["Compute"] == compute_b), 0.0)
        )

    if x_mode == "file_pct":
        pct_steps = sorted(set([round(150 * i / n_steps) for i in range(n_steps + 1)] + [100]))
        for pct in pct_steps:
            x_values.append(float(pct))
            scale_factor = pct / 100.0
            cfg_scaled = _cfg_base()
            cfg_scaled["workload"]["files_per_route_small"] = max(
                0, int(cfg["workload"]["files_per_route_small"] * scale_factor)
            )
            cfg_scaled["workload"]["files_per_route_medium"] = max(
                0, int(cfg["workload"]["files_per_route_medium"] * scale_factor)
            )
            cfg_scaled["workload"]["files_per_route_large"] = max(
                0, int(cfg["workload"]["files_per_route_large"] * scale_factor)
            )
            _append_costs(cfg_scaled)

    else:  # route_count
        baseline_routes = float(
            cfg["workload"]["routes_small"]
            + cfg["workload"]["routes_medium"]
            + cfg["workload"]["routes_large"]
        )
        for total_routes in [round(baseline_routes * i / n_steps) for i in range(1, n_steps + 1)]:
            x_values.append(float(total_routes))
            scale_factor = total_routes / baseline_routes
            cfg_scaled = _cfg_base()
            cfg_scaled["workload"]["routes_small"] = max(
                0, int(cfg["workload"]["routes_small"] * scale_factor)
            )
            cfg_scaled["workload"]["routes_medium"] = max(
                0, int(cfg["workload"]["routes_medium"] * scale_factor)
            )
            cfg_scaled["workload"]["routes_large"] = max(
                0, int(cfg["workload"]["routes_large"] * scale_factor)
            )
            cfg_scaled["workload"]["n_routes_total_for_case_b"] = max(
                1,
                cfg_scaled["workload"]["routes_small"]
                + cfg_scaled["workload"]["routes_medium"]
                + cfg_scaled["workload"]["routes_large"],
            )
            _append_costs(cfg_scaled)

    return x_values, y_values_a, y_values_b


def _render_comparison(cfg: dict[str, Any], rows: list[dict[str, Any]]) -> None:
    st.subheader("Cost Comparison (2 architecture/compute pairs)")

    st.caption(
        "Select two (architecture, compute) pairs and visualize cost trends across file scale or route count. Shared controls here also drive the Analysis tab."
    )

    _render_shared_controls(cfg, "comparison")

    cp_col, reset_col = st.columns([2, 0.8])
    cp_col.number_input(
        "Case B parallel runs",
        min_value=1,
        max_value=500,
        step=1,
        key="comparison_parallel_case_b",
        on_change=_copy_tab_value_to_shared,
        args=("comparison", "parallel_case_b"),
    )
    reset_col.write("")
    reset_col.write("")
    reset_col.button(
        "Reset",
        key="comparison_reset_controls",
        on_click=_on_shared_reset,
        use_container_width=True,
    )

    st.caption(f"Scenario: {cfg['active_scenario']}")

    st.markdown("### Compare Two Pairs")
    pair_options = [
        f"{_display_case('A')} / Serverless",
        f"{_display_case('A')} / Job Compute",
        f"{_display_case('B')} / Serverless",
        f"{_display_case('B')} / Job Compute",
        f"{_display_case('C')} / Serverless",
        f"{_display_case('C')} / Job Compute",
    ]

    col1, col2 = st.columns(2)
    with col1:
        pair_a = st.selectbox(
            "First pair", pair_options, index=0, key="comparison_pair_a"
        )
    with col2:
        pair_b = st.selectbox(
            "Second pair",
            pair_options,
            index=4,
            key="comparison_pair_b",
        )

    case_label_a, compute_a = pair_a.split(" / ")
    case_label_b, compute_b = pair_b.split(" / ")
    case_a = REVERSE_CASE_LABELS.get(case_label_a, case_label_a)
    case_b = REVERSE_CASE_LABELS.get(case_label_b, case_label_b)

    triggers_sample = next(
        (r["triggers_day"] for r in rows if r["Case"] == case_a and r["Compute"] == compute_a),
        0.0,
    )
    st.metric(f"Triggers / Day ({case_label_a})", f"{triggers_sample:.1f}")

    tab_file_pct, tab_route_count = st.tabs(
        ["By File % Increase", "By Route Count"]
    )

    with tab_file_pct:
        st.markdown("**X-axis**: File count percentage (0% = no files, 100% = default workload, 150% = 1.5x workload)")
        x_vals, y_vals_a, y_vals_b = _compute_comparison_series(
            cfg,
            case_a,
            compute_a,
            case_b,
            compute_b,
            "file_pct",
        )

        file_min = st.session_state.get("comparison_file_min", 80)
        file_max = st.session_state.get("comparison_file_max", 150)
        file_min = max(0, min(150, int(file_min)))
        file_max = max(file_min, min(150, int(file_max)))

        if not _plotly_available:
            st.error("Plotly not installed. Please install it: pip install plotly")
        else:
            fig = go.Figure()
            fig.add_trace(
                go.Scatter(
                    x=x_vals,
                    y=y_vals_a,
                    mode="lines+markers",
                    name=pair_a,
                    line=dict(color="#1f77b4", width=2),
                    marker=dict(size=6),
                )
            )
            fig.add_trace(
                go.Scatter(
                    x=x_vals,
                    y=y_vals_b,
                    mode="lines+markers",
                    name=pair_b,
                    line=dict(color="#2ca02c", width=2),
                    marker=dict(size=6),
                )
            )
            fig.update_layout(
                title=f"Cost Comparison: {pair_a} vs {pair_b}",
                xaxis_title="File Increase (%)",
                yaxis_title="Cost per month (USD)",
                hovermode="x unified",
                template="plotly_white",
            )
            fig.update_xaxes(range=[file_min, file_max])
            st.plotly_chart(fig, use_container_width=True)

            st.markdown("**Displayed X-axis range (File %)**")
            fmin_col, fmax_col, fapply_col = st.columns([2, 2, 1])
            with fmin_col:
                file_min_input = st.number_input(
                    "Min %",
                    min_value=0,
                    max_value=150,
                    value=file_min,
                    step=1,
                    key="comparison_file_min_input",
                )
            with fmax_col:
                file_max_input = st.number_input(
                    "Max %",
                    min_value=0,
                    max_value=150,
                    value=file_max,
                    step=1,
                    key="comparison_file_max_input",
                )
            with fapply_col:
                if st.button("Apply", key="comparison_file_apply"):
                    applied_min = int(file_min_input)
                    applied_max = int(file_max_input)
                    if applied_min > applied_max:
                        st.warning("Min cannot be greater than Max. Values were swapped.")
                        applied_min, applied_max = applied_max, applied_min
                    st.session_state["comparison_file_min"] = applied_min
                    st.session_state["comparison_file_max"] = applied_max
                    st.rerun()

    with tab_route_count:
        st.markdown("**X-axis**: Total route count (10 to 200; scales all route tiers proportionally)")
        x_vals, y_vals_a, y_vals_b = _compute_comparison_series(
            cfg,
            case_a,
            compute_a,
            case_b,
            compute_b,
            "route_count",
        )

        route_min = st.session_state.get("comparison_route_min", 50)
        route_max = st.session_state.get("comparison_route_max", 200)
        route_min = max(10, min(200, int(route_min)))
        route_max = max(route_min, min(200, int(route_max)))

        if not _plotly_available:
            st.error("Plotly not installed. Please install it: pip install plotly")
        else:
            fig = go.Figure()
            fig.add_trace(
                go.Scatter(
                    x=x_vals,
                    y=y_vals_a,
                    mode="lines+markers",
                    name=pair_a,
                    line=dict(color="#1f77b4", width=2),
                    marker=dict(size=6),
                )
            )
            fig.add_trace(
                go.Scatter(
                    x=x_vals,
                    y=y_vals_b,
                    mode="lines+markers",
                    name=pair_b,
                    line=dict(color="#2ca02c", width=2),
                    marker=dict(size=6),
                )
            )
            fig.update_layout(
                title=f"Cost Comparison: {pair_a} vs {pair_b}",
                xaxis_title="Total Routes",
                yaxis_title="Cost per month (USD)",
                hovermode="x unified",
                template="plotly_white",
            )
            fig.update_xaxes(range=[route_min, route_max])
            st.plotly_chart(fig, use_container_width=True)

            st.markdown("**Displayed X-axis range (Routes)**")
            rmin_col, rmax_col, rapply_col = st.columns([2, 2, 1])
            with rmin_col:
                route_min_input = st.number_input(
                    "Min routes",
                    min_value=10,
                    max_value=200,
                    value=route_min,
                    step=1,
                    key="comparison_route_min_input",
                )
            with rmax_col:
                route_max_input = st.number_input(
                    "Max routes",
                    min_value=10,
                    max_value=200,
                    value=route_max,
                    step=1,
                    key="comparison_route_max_input",
                )
            with rapply_col:
                if st.button("Apply", key="comparison_route_apply"):
                    applied_min = int(route_min_input)
                    applied_max = int(route_max_input)
                    if applied_min > applied_max:
                        st.warning("Min cannot be greater than Max. Values were swapped.")
                        applied_min, applied_max = applied_max, applied_min
                    st.session_state["comparison_route_min"] = applied_min
                    st.session_state["comparison_route_max"] = applied_max
                    st.rerun()


@st.cache_data
def _load_reference_sections(reference_path_str: str) -> tuple[str, dict[str, str]]:
    content = Path(reference_path_str).read_text(encoding="utf-8")
    lines = content.splitlines()
    headings: list[tuple[str, int]] = []
    for idx, line in enumerate(lines):
        if line.startswith("## "):
            headings.append((line[3:].strip(), idx))
    section_map: dict[str, str] = {"Full document": content}
    for i, (title, start) in enumerate(headings):
        end = headings[i + 1][1] if i + 1 < len(headings) else len(lines)
        section_map[title] = "\n".join(lines[start:end]).strip()
    return content, section_map


def _render_reference(cfg: dict[str, Any], rows: list[dict[str, Any]]) -> None:
    st.subheader("Reference Document")
    st.caption(
        "Rendered directly from docs/cost-simulation/prices-and-sources.md"
    )

    _render_reference_intro(rows)

    reference_path = Path(__file__).resolve().parents[1] / "prices-and-sources.md"

    if not reference_path.exists():
        st.error(f"Reference file not found: {reference_path}")
        return

    content, section_map = _load_reference_sections(str(reference_path))

    quick_default = "4) Cost formulas by compute type"
    if quick_default not in section_map:
        quick_default = "Full document"

    st.markdown("### Quick Navigation")
    selected = st.selectbox(
        "Jump to section",
        list(section_map.keys()),
        index=list(section_map.keys()).index(quick_default),
    )

    show_full = st.toggle("Always show full document below", value=False)

    st.download_button(
        label="Download Reference Markdown",
        data=content,
        file_name="prices-and-sources.md",
        mime="text/markdown",
    )

    st.markdown("### Selected Content")
    st.markdown(section_map[selected])

    if show_full:
        st.markdown("---")
        st.markdown("### Full Document")
        st.markdown(content)


def _render_settings(cfg: dict[str, Any]) -> None:
    st.subheader("Settings")
    st.caption("Main controls first. Open Advanced only if you want full tuning.")

    st.markdown("### Main Controls")
    st.session_state["settings_active_scenario"] = st.session_state.get(
        "shared_active_scenario", DEFAULTS["active_scenario"]
    )
    cfg["active_scenario"] = st.selectbox(
        "Active batching scenario",
        ["Low", "Medium", "High"],
        key="settings_active_scenario",
        on_change=_on_settings_active_scenario_change,
    )

    st.markdown("#### Batching scenarios")
    # A slider drag on Analysis/Comparison must be reflected here before number_input reads its key.
    active_scenario_name = st.session_state["shared_active_scenario"]
    if f"batch_per_route_{active_scenario_name}" in st.session_state:
        st.session_state[f"batch_per_route_{active_scenario_name}"] = float(
            st.session_state["shared_batched_per_route"]
        )
        st.session_state[f"batch_across_{active_scenario_name}"] = float(
            st.session_state["shared_batched_across_route"]
        )

    bcols = st.columns(3)
    for i, name in enumerate(["Low", "Medium", "High"]):
        with bcols[i]:
            st.markdown(f"**{name}**")
            per_route = st.number_input(
                f"{name} per-route %",
                min_value=0.0,
                max_value=95.0,
                step=1.0,
                key=f"batch_per_route_{name}",
                on_change=_on_settings_batching_change,
                args=(name,),
            )
            across_route = st.number_input(
                f"{name} across-route %",
                min_value=0.0,
                max_value=95.0,
                step=1.0,
                key=f"batch_across_{name}",
                on_change=_on_settings_batching_change,
                args=(name,),
            )
            cfg["batching"][name] = ScenarioBatching(
                per_route=per_route / 100.0, across_route=across_route / 100.0
            )

    st.markdown("#### Workload")
    w = cfg["workload"]
    col1, col2, col3 = st.columns(3)
    w["routes_small"] = col1.number_input(
        "Small routes", min_value=0, value=int(w["routes_small"]), step=1,
        key="settings_workload_routes_small",
    )
    w["routes_medium"] = col2.number_input(
        "Medium routes", min_value=0, value=int(w["routes_medium"]), step=1,
        key="settings_workload_routes_medium",
    )
    w["routes_large"] = col3.number_input(
        "Large routes", min_value=0, value=int(w["routes_large"]), step=1,
        key="settings_workload_routes_large",
    )

    col4, col5, col6 = st.columns(3)
    st.session_state["settings_workload_files_per_route_small"] = int(st.session_state["shared_files_small"])
    w["files_per_route_small"] = col4.number_input(
        "Files/day per small route",
        min_value=0,
        step=1,
        key="settings_workload_files_per_route_small",
        on_change=_on_settings_workload_change,
    )
    st.session_state["settings_workload_files_per_route_medium"] = int(st.session_state["shared_files_medium"])
    w["files_per_route_medium"] = col5.number_input(
        "Files/day per medium route",
        min_value=0,
        step=1,
        key="settings_workload_files_per_route_medium",
        on_change=_on_settings_workload_change,
    )
    st.session_state["settings_workload_files_per_route_large"] = int(st.session_state["shared_files_large"])
    w["files_per_route_large"] = col6.number_input(
        "Files/day per large route",
        min_value=0,
        step=1,
        key="settings_workload_files_per_route_large",
        on_change=_on_settings_workload_change,
    )
    st.session_state["settings_workload_n_parallel_autoloader_case_b"] = int(st.session_state["shared_parallel_case_b"])
    w["n_parallel_autoloader_case_b"] = st.number_input(
        "Case B parallel Autoloader runs",
        min_value=1,
        step=1,
        key="settings_workload_n_parallel_autoloader_case_b",
        on_change=_on_settings_workload_change,
    )

    with st.expander("Advanced controls"):
        st.markdown("#### Timing")
        t = cfg["timing"]
        t["t_init_s"] = st.number_input(
            "t_init_s", min_value=0.0, value=float(t["t_init_s"]),
            key="settings_timing_t_init_s",
        )
        t["t_route_overhead_s"] = st.number_input(
            "t_route_overhead_s (B)",
            min_value=0.0,
            value=float(t["t_route_overhead_s"]),
            key="settings_timing_t_route_overhead_s",
        )
        t["t_delta_write"] = st.number_input(
            "t_delta_write", min_value=0.0, value=float(t["t_delta_write"]),
            key="settings_timing_t_delta_write",
        )
        t["t_delete"] = st.number_input(
            "t_delete", min_value=0.0, value=float(t["t_delete"]),
            key="settings_timing_t_delete",
        )
        t["t_startup_s"] = st.number_input(
            "t_startup_s (Job Compute)", min_value=0.0, value=float(t["t_startup_s"]),
            key="settings_timing_t_startup_s",
        )

        st.markdown("#### Pricing")
        p = cfg["pricing"]
        p["vm_rate_h"] = st.number_input(
            "vm_rate_h", min_value=0.0, value=float(p["vm_rate_h"]),
            key="settings_pricing_vm_rate_h",
        )
        p["dbu_rate_jobs_h"] = st.number_input(
            "dbu_rate_jobs_h", min_value=0.0, value=float(p["dbu_rate_jobs_h"]),
            key="settings_pricing_dbu_rate_jobs_h",
        )
        p["dbu_rate_srv_h"] = st.number_input(
            "dbu_rate_srv_h", min_value=0.0, value=float(p["dbu_rate_srv_h"]),
            key="settings_pricing_dbu_rate_srv_h",
        )
        p["dbu_mult_jobs"] = st.number_input(
            "dbu_mult_jobs", min_value=0.0, value=float(p["dbu_mult_jobs"]),
            key="settings_pricing_dbu_mult_jobs",
        )
        p["dbu_mult_srv"] = st.number_input(
            "dbu_mult_srv", min_value=0.0, value=float(p["dbu_mult_srv"]),
            key="settings_pricing_dbu_mult_srv",
        )

        st.markdown("#### File tier model")
        ft = cfg["file_tiers"]
        ft["share_small"] = st.number_input(
            "share_small", min_value=0.0, value=float(ft["share_small"]), step=0.01,
            key="settings_file_tiers_share_small",
        )
        ft["share_medium"] = st.number_input(
            "share_medium", min_value=0.0, value=float(ft["share_medium"]), step=0.01,
            key="settings_file_tiers_share_medium",
        )
        ft["share_large"] = st.number_input(
            "share_large", min_value=0.0, value=float(ft["share_large"]), step=0.01,
            key="settings_file_tiers_share_large",
        )
        ft["t_copy_small"] = st.number_input(
            "t_copy_small", min_value=0.0, value=float(ft["t_copy_small"]),
            key="settings_file_tiers_t_copy_small",
        )
        ft["t_copy_medium"] = st.number_input(
            "t_copy_medium", min_value=0.0, value=float(ft["t_copy_medium"]),
            key="settings_file_tiers_t_copy_medium",
        )
        ft["t_copy_large"] = st.number_input(
            "t_copy_large", min_value=0.0, value=float(ft["t_copy_large"]),
            key="settings_file_tiers_t_copy_large",
        )

        cfg["workload"]["n_routes_total_for_case_b"] = st.number_input(
            f"N_routes for {_display_case('B')}",
            min_value=1,
            value=int(cfg["workload"]["n_routes_total_for_case_b"]),
            step=1,
            key="settings_workload_n_routes_total_for_case_b",
        )
        _actual_routes = int(w["routes_small"]) + int(w["routes_medium"]) + int(w["routes_large"])
        if int(cfg["workload"]["n_routes_total_for_case_b"]) != _actual_routes:
            st.warning(
                f"N_routes for {_display_case('B')} ({cfg['workload']['n_routes_total_for_case_b']}) "
                f"differs from the actual route tier sum ({_actual_routes}). "
                f"Cases A and C model {_actual_routes} routes; Case B models "
                f"{cfg['workload']['n_routes_total_for_case_b']}."
            )

    if st.button("Reset defaults"):
        _reset_defaults_state()
        st.rerun()


def _get_cfg() -> dict[str, Any]:
    cfg: dict[str, Any] = {}

    for key, value in DEFAULTS.items():
        state_key = f"cfg_{key}"
        if state_key not in st.session_state:
            st.session_state[state_key] = value
        cfg[key] = st.session_state[state_key]

    cfg = {
        "pricing": dict(cfg["pricing"]),
        "timing": dict(cfg["timing"]),
        "workload": dict(cfg["workload"]),
        "file_tiers": dict(cfg["file_tiers"]),
        "batching": dict(cfg["batching"]),
        "active_scenario": cfg["active_scenario"],
    }

    return cfg


def _save_cfg(cfg: dict[str, Any]) -> None:
    for key in DEFAULTS:
        st.session_state[f"cfg_{key}"] = cfg[key]


def main() -> None:
    st.set_page_config(page_title="Databricks Cost Simulator", layout="wide")
    st.title("Databricks Cost Simulator (3 architectures x 2 compute modes)")
    st.caption(
        "Architectures: 1 Job, 1 Autoloader | 1 Job, N Autoloader | "
        "N Job, N Autoloader"
    )

    cfg = _get_cfg()
    _ensure_shared_controls_state(cfg)

    tab_analysis, tab_comparison, tab_reference, tab_settings = st.tabs(
        ["Analysis", "Comparison", "Reference", "Settings"]
    )

    with tab_settings:
        _render_settings(cfg)

    rows = _compute_rows(cfg)

    with tab_analysis:
        _render_analysis(cfg, rows)

    with tab_comparison:
        _render_comparison(cfg, rows)

    with tab_reference:
        _render_reference(cfg, rows)

    _save_cfg(cfg)


if __name__ == "__main__":
    main()

```


###### FILE: docs/cost-simulation/webapp/README.md ######

```md
# Streamlit Cost Simulator (3x2)

This app compares:

- Cases: `A`, `B`, `C`
- Compute modes: `Serverless`, `Job Compute`

## Run locally

```bash
uv sync --group dev
make cost-simulator
```

## Tabs

- `Analysis`: sortable comparison table (6 rows), ranked by monthly cost by default.
- `Comparison`: line charts comparing architectures.
- `Reference`: formulas, constants, and assumptions.
- `Settings`: main controls first (batching/workload), advanced controls in an expander.

## Notes

- Formulas and defaults are aligned with `docs/cost-simulation/prices-and-sources.md`.
- The app uses one Job Compute model (no pool/cold split).
- Batch derivation follows the note in section 7.3 of the pricing document.

```

###### FILE: Makefile ######

```
SHELL = /bin/bash
.PHONY: all venv update upgrade-dependencies test coverage cov tox reports clean-reports init-docs docs clean-docs clean distclean clean-wheels cost-simulator

all: info

info:  ## Show this information
   @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

REPORT_OUTPUT_DIR = ./target/report
DOCUMENT_OUTPUT_DIR = ./doc/_build

.venv:
   uv sync --all-extras --all-groups --no-group docs

venv: .venv ## Create virtual env

sync-dependencies: ## Sync dependencies
   uv sync --all-extras --all-groups --no-group docs

update: sync-dependencies ## Alias of sync-dependencies

upgrade-dependencies: ## Upgrade dependencies
   uv sync --upgrade --all-extras --all-groups --no-group docs

test: venv  ## Run tests
   uv run pytest $(PYTEST_OPTIONS)

coverage: venv  ## Run tests and compute test coverage
   uv run coverage run -m pytest --durations=0 $(PYTEST_OPTIONS) && \
   uv run coverage report

cov: coverage

reports: clean-reports  ## Create coverage report
   uv run coverage run -m pytest \
      --durations=0 \
      --html=$(REPORT_OUTPUT_DIR)/test/pytests.html --self-contained-html \
      --junitxml=$(REPORT_OUTPUT_DIR)/test/junit.xml $(PYTEST_OPTIONS) && \
   uv run coverage html -d $(REPORT_OUTPUT_DIR)/coverage_html && \
   uv run coverage xml -o $(REPORT_OUTPUT_DIR)/coverage.xml

clean-reports:  ## Clean reports
   rm -rf $(REPORT_OUTPUT_DIR)

init-docs: venv ## Initialize API document project
   @if [ ! -f "docs/index.rst" ]; then \
      uv run --all-groups sphinx-apidoc \
         -F -T -M --separate \
         -H "`uv run --group docs azfr-pyproject-name`" \
         -A "`uv run --group docs azfr-pyproject-author`" \
         -t docs/_templates \
         -o docs \
         src && \
         git add docs; \
   fi

docs: venv init-docs clean-docs  ## Generate API documents
   uv run --all-groups sphinx-apidoc -T -M --separate -t docs/_templates -o docs src && \
   uv run --all-groups $(MAKE) -C docs html; \

clean-docs:  ## Clean API documents
   rm -rf $(DOCUMENT_OUTPUT_DIR)

deploy:  venv ## Deploy library packages
   uv build && \
   uv publish \
      --publish-url $(PYPI_REPO_URL) \
      --username $(PYPI_REPO_USER) \
      --password $(PYPI_REPO_PASS) \
      dist/*

deploy-docs: docs  ## Deploy API documents
   uv run --group docs azfr-document-deploy \
      -i docs/_build/html \
      -o $(DOC_REPO_ROOT) \
      -n `uv run --group docs azfr-pyproject-name` \
      -u $(DOC_REPO_USER) \
      -p $(DOC_REPO_PASS) \
      -c /etc/ssl/certs/ca-certificates.crt

clean-wheels:  ## Delete built wheel files from dist/
   rm -f dist/*.whl

databricks-deploy: clean-wheels venv ## Deploy bundle to Databricks (dev target)
   databricks bundle deploy --target dev

databricks-deploy-force: clean-wheels venv  ## Deploy bundle to Databricks (dev target), forcing full file re-sync
   rm -f .databricks/bundle/dev/sync-snapshots/*.json
   databricks bundle deploy --target dev

cost-simulator: venv  ## Run Streamlit cost simulator (docs-local tool)
   uv run streamlit run docs/cost-simulation/webapp/app.py

clean: clean-reports clean-docs  ## Clean artifacts and temporary files
   find src tests -name '*.pyc' -exec rm -f {} +
   find src tests -name '*.pyo' -exec rm -f {} +
   find src tests -name '*~' -exec rm -f {} +
   rm -rf .coverage

distclean: clean  ## Clean all files including virtualenv
   rm -rf .pytest_cache .tox .eggs ./src/*.egg-info build dist
   rm -rf .venv

```

###### FILE: notebooks/benchmark/result-10GB/benchmark-10GB-analysis.md ######

```md
# Benchmark Results — Analysis (10 GB files, cross-catalog)

---

![Throughput chart — 10 GB files, cross-catalog](./throughput_chart.png)

---

## Environment

| Parameter | Value |
|---|---|
| Cluster VM | 4-core worker node |
| File size | 10 GB each |
| Move type | Cross-catalog (Unity Catalog Volume → Volume) |
| Worker counts swept | 1, 2, 3, 4, 5, 6, 7, 8, 10, 12 |
| Repetitions per config | 3 (`n_runs=3`) |

---

## Key Observations

### `dbutils` is the clear winner overall

- **Peak:** 354.5 MB/s at 8 workers
- **Plateau:** 6–12 workers at ~336–354 MB/s — very flat and stable
- **Scales well:** 1 worker = 147 MB/s → 4 workers = 312 MB/s → 8+ workers = ~350 MB/s
- **Very low std at peak:** std=2.3 at 10 workers, std=4.4 at 8 workers
- **Single-worker penalty:** 147 MB/s vs `os`/`fsspec` at 192–201 MB/s — points to JVM/RPC setup overhead, but this is completely overwhelmed by superior parallelization

### `os` and `fsspec-databricks`: increase then degrade

- Both peak around 5–6 workers at roughly 255–271 MB/s
- Beyond that, throughput **actively degrades** — unlike `dbutils` which plateaus
- `os` degrades sharply at 8+ workers with high variance; `fsspec-databricks` degrades more gradually but with worse variance at high worker counts

---

## The Story

`dbutils` dominates with a 354 MB/s peak — about **30% faster** than the alternatives. What's striking is that it scales linearly all the way through 8 workers before plateauing, **never degrading like the others**. This suggests it's leveraging server-side parallelism effectively, probably through Azure Data Lake's chunked operations. Meanwhile, `os` and `fsspec` hit their ceiling around 5–6 workers because the **FUSE mount layer becomes the bottleneck**, serializing requests beyond that point.

There's an interesting quirk with `dbutils` at 1 worker — it's actually slower than the single-worker performance of `os` and `fsspec` (147 MB/s vs 192–201 MB/s), which points to JVM and RPC overhead. But that overhead gets absorbed as soon as you add parallelism. There's also noticeable high variability in certain configurations — particularly `os` and `fsspec` at 4 workers (warm-up effect visible across runs), and `fsspec` at higher worker counts shows similar patterns.

Looking at the run-by-run breakdown, there's a **consistent pattern where run 1 is noticeably slower** than subsequent runs. This shows up dramatically in `dbutils` at 7 workers, where wall time progressively improves from 210 → 192 → 156 s — that progressive improvement strongly suggests **JVM JIT compilation** kicking in. The same first-run penalty appears in `os` and `fsspec` at 4 workers, likely from FUSE connection establishment or Azure storage edge-cache warmup. Each (backend, workers) combination pays an upfront cost on the first execution. This is why `n_runs=3` matters — a single run would overestimate wall time.

---

## Summary

### 1. Overall winner: `dbutils`

| Backend | Peak MB/s | At workers | std |
|---|---|---|---|
| `dbutils` | **354.5** | 8 | 4.4 |
| `fsspec-databricks` | 271.6 | 5 | 11.7 |
| `os` | 260.4 | 5 | 3.2 |

`dbutils` is ~30% faster at peak than the other two. This is significant and consistent across all runs.

---

### 2. Scaling behavior — fundamentally different per backend

The cluster VM has **4 cores**, which is key context: `dbutils` continuing to scale well beyond 4 workers confirms the bottleneck is **network/Azure API latency**, not CPU — threads spend most of their time waiting on IO, so oversubscription still helps. Conversely, `os`/`fsspec` degrading at 5–6 workers is consistent with the FUSE kernel dispatcher serializing requests under core pressure.

**`dbutils` — monotonically scales, then plateaus:**

More workers always helps, up to 8, then flat. It never degrades. This suggests `dbutils.fs.cp` uses the Databricks REST API with chunked multipart upload — each worker gets its own HTTP session to Azure storage, scaling efficiently without a shared bottleneck.

**`os` and `fsspec` — scale to 5–6 workers, then degrade:**

The ceiling is the **FUSE mount layer** — beyond ~5–6 concurrent threads, the kernel FUSE dispatcher serializes requests and contention exceeds any parallelism gain. Adding more workers actively hurts `os` at 12 workers (191 MB/s — below the 1-worker baseline of 201 MB/s).

**Rule of thumb on this 4-core VM:** `dbutils` peaks at 8 workers (2× CPU count); `os`/`fsspec` peak at 5–6 (CPU count + a few). These ratios may generalize to other VM sizes.

---

### 3. The `dbutils` 1-worker anomaly

`dbutils` at 1 worker is slower than `os` and `fsspec` (147 vs ~200 MB/s). This is the **JVM/RPC startup cost** — the first call initializes the Spark JVM context and HTTP connection pool. It amortizes completely at 2+ workers. The cost is a one-time fixed overhead per backend instance, not per file.

---

### 4. "Run 1 is slower" pattern

Visible across multiple configurations:

| Backend | Workers | Run 1 (s) | Run 2 (s) | Run 3 (s) |
|---|---|---|---|---|
| `dbutils` | 7 | 210 | 192 | **156** — progressive JIT |
| `os` | 4 | 176 | 147 | 137 |
| `fsspec` | 4 | 168 | 133 | 137 |

For `dbutils`, this is JVM JIT compilation — the JVM optimizes hot paths across runs. For `os`/`fsspec`, it's likely FUSE connection establishment or Azure edge-cache warmup.

---

### 5. High-variance configurations to distrust

| Backend | Workers | std | Cause |
|---|---|---|---|
| `os` | 4 | 25.6 | Run 1 outlier (176s vs ~140s) |
| `fsspec` | 4 | 25.9 | Same pattern |
| `fsspec` | 10 | 31.3 | Run 3 spike (368s vs ~490s avg) |
| `fsspec` | 12 | 27.3 | Run 3 collapse (580s) |

These high-std configurations warrant caution for throughtput estimation. Investigate the cause before using the average: if the variance is a known warm-up effect (run 1 always slower), `min` may be a better estimator than `mean`; if it looks random, consider re-running with more repetitions.

---

### 6. Second-run invalidation — Azure Storage throttling

A second benchmark run was executed immediately after the first. Performance collapsed sharply at 4+ workers:

| Backend | Workers | Run 1 avg MB/s | Run 2 avg MB/s |
|---|---|---|---|
| `fsspec-databricks` | 4 | 263.5 | **80.6** |
| `fsspec-databricks` | 5 | 271.6 | **75.6** |
| `os` | 4 | 250.3 | **64.0** |
| `os` | 5 | 260.4 | **68.1** |

**Most likely cause: Azure Storage throttling (HTTP 429).** The first benchmark moved hundreds of gigabytes across two catalogs. By the time run 2 started, the Azure ADLS Gen2 storage account had likely exhausted its per-account IOPS or bandwidth quota and started returning throttle responses. The backends retry transparently, which explains the massively inflated wall times (400–1400s instead of 130–600s) and the flat throughput ceiling regardless of worker count.

**Why 1–2 workers survive:** Below the throttle trigger rate, requests complete normally. At 4+ concurrent workers, the aggregate request rate crosses the throttle threshold.

**Run 2 results at 4+ workers are not valid** — they measure throttle recovery speed, not IO backend throughput.

**Mitigation for future runs:**
- Azure ADLS Gen2 burst quotas typically reset within 1 hour, but sustained heavy traffic (several hours of multi-worker IO) can extend that window significantly
- Wait at least **1–2 hours between IO backend runs**, ideally until the next day, to ensure a clean quota state before each backend sweep
- Alternatively, use a dedicated storage account for benchmarking isolated from other workloads

---

### 7. Recommendations

> Results measured on a **4-core VM** with **10 GB files**.

| Goal | Backend | Workers | Expected throughput |
|---|---|---|---|
| **Recommended** | `dbutils` | 8 | ~354 MB/s, std=4.4 |
| If `dbutils` unavailable | `os` | 5 | ~260 MB/s, std=3.2 |
| Avoid | `os` | ≥8 | Adding more workers hurts: 12 workers (191 MB/s) is slower than 1 worker (201 MB/s) |
| Avoid | `fsspec-databricks` | ≥8 | Degradation with high variance; throughput less reliable |

```

###### FILE: notebooks/benchmark/result-10GB/benchmark-10GB-raw-output-1st-run.txt ######

```txt

================================================================
  Backend: os
================================================================

  workers= 1  (1 files × 9.3 GB = 9.3 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            48.2              198.0
  2            46.9              203.4
  3            46.9              203.3
   avg         47.3              201.6   std=2.5  [198.0–203.4]

  workers= 2  (2 files × 9.3 GB = 18.6 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            95.8              199.2
  2            97.6              195.5
  3            92.2              206.9
   avg         95.2              200.5   std=4.7  [195.5–206.9]

  workers= 3  (3 files × 9.3 GB = 27.9 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           145.7              196.4
  2           144.1              198.5
  3           138.8              206.1
   avg        142.9              200.3   std=4.2  [196.4–206.1]

  workers= 4  (4 files × 9.3 GB = 37.3 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           176.8              215.8
  2           147.8              258.1
  3           137.7              276.9
   avg        154.1              250.3   std=25.6  [215.8–276.9]

  workers= 5  (5 files × 9.3 GB = 46.6 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           182.6              261.2
  2           185.7              256.8
  3           181.2              263.1
   avg        183.2              260.4   std=2.6  [256.8–263.1]

  workers= 6  (6 files × 9.3 GB = 55.9 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           216.7              264.0
  2           228.8              250.1
  3           227.0              252.1
   avg        224.2              255.4   std=6.1  [250.1–264.0]

  workers= 7  (7 files × 9.3 GB = 65.2 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           270.5              246.8
  2           304.2              219.4
  3           275.7              242.2
   avg        283.5              236.1   std=12.0  [219.4–246.8]

  workers= 8  (8 files × 9.3 GB = 74.5 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           412.1              185.1
  2           330.1              231.2
  3           340.4              224.1
   avg        360.9              213.5   std=20.3  [185.1–231.2]

  workers=10  (10 files × 9.3 GB = 93.1 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           436.4              218.5
  2           430.3              221.6
  3           379.8              251.1
   avg        415.5              230.4   std=14.7  [218.5–251.1]

  workers=12  (12 files × 9.3 GB = 111.8 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           565.6              202.3
  2           642.1              178.2
  3           588.5              194.5
   avg        598.7              191.7   std=10.0  [178.2–202.3]

  ── All configs for 'os' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        5      260.4     183.2     3.2   [256.8–263.1]
   2        6      255.4     224.2     7.5   [250.1–264.0]
   3        4      250.3     154.1    31.3   [215.8–276.9]
   4        7      236.1     283.5    14.7   [219.4–246.8]
   5       10      230.4     415.5    18.0   [218.5–251.1]
   6        8      213.5     360.9    24.8   [185.1–231.2]
   7        1      201.6      47.3     3.1   [198.0–203.4]
   8        2      200.5      95.2     5.8   [195.5–206.9]
   9        3      200.3     142.9     5.1   [196.4–206.1]
  10       12      191.7     598.7    12.3   [178.2–202.3]

================================================================
  Backend: fsspec-databricks
================================================================

  workers= 1  (1 files × 9.3 GB = 9.3 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            49.8              191.5
  2            49.9              191.0
  3            48.6              196.0
   avg         49.4              192.8   std=2.2  [191.0–196.0]

  workers= 2  (2 files × 9.3 GB = 18.6 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            94.9              200.9
  2            92.7              205.7
  3            96.5              197.6
   avg         94.7              201.4   std=3.3  [197.6–205.7]

  workers= 3  (3 files × 9.3 GB = 27.9 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           145.3              196.9
  2           148.5              192.7
  3           142.7              200.5
   avg        145.5              196.7   std=3.2  [192.7–200.5]

  workers= 4  (4 files × 9.3 GB = 37.3 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           168.0              227.1
  2           133.9              284.9
  3           137.0              278.4
   avg        146.3              263.5   std=25.9  [227.1–284.9]

  workers= 5  (5 files × 9.3 GB = 46.6 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           168.1              283.7
  2           183.1              260.4
  3           176.2              270.7
   avg        175.8              271.6   std=9.5  [260.4–283.7]

  workers= 6  (6 files × 9.3 GB = 55.9 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           218.5              261.9
  2           221.2              258.7
  3           234.2              244.3
   avg        224.6              255.0   std=7.7  [244.3–261.9]

  workers= 7  (7 files × 9.3 GB = 65.2 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           262.2              254.6
  2           264.1              252.8
  3           325.6              205.0
   avg        284.0              237.5   std=23.0  [205.0–254.6]

  workers= 8  (8 files × 9.3 GB = 74.5 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           295.9              257.8
  2           301.1              253.4
  3           303.0              251.8
   avg        300.0              254.3   std=2.5  [251.8–257.8]

  workers=10  (10 files × 9.3 GB = 93.1 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           476.0              200.4
  2           510.8              186.7
  3           368.4              258.8
   avg        451.7              215.3   std=31.3  [186.7–258.8]

  workers=12  (12 files × 9.3 GB = 111.8 GB)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           436.8              262.0
  2           470.4              243.3
  3           580.9              197.0
   avg        496.0              234.1   std=27.3  [197.0–262.0]

  ── All configs for 'fsspec-databricks' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        5      271.6     175.8    11.7   [260.4–283.7]
   2        4      263.5     146.3    31.7   [227.1–284.9]
   3        6      255.0     224.6     9.4   [244.3–261.9]
   4        8      254.3     300.0     3.1   [251.8–257.8]
   5        7      237.5     284.0    28.1   [205.0–254.6]
   6       12      234.1     496.0    33.5   [197.0–262.0]
   7       10      215.3     451.7    38.3   [186.7–258.8]
   8        2      201.4      94.7     4.1   [197.6–205.7]
   9        3      196.7     145.5     3.9   [192.7–200.5]
  10        1      192.8      49.4     2.8   [191.0–196.0]


================================================================
  Backend: dbutils
================================================================

  workers= 1  (1 files × 9.3 GB = 9.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            68.9              138.3
  2            62.8              151.8
  3            62.9              151.6
   avg         64.9              147.2   std=7.7  [138.3–151.8]

  workers= 2  (2 files × 9.3 GB = 18.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            85.9              221.9
  2            77.6              245.7
  3            83.4              228.7
   avg         82.3              232.1   std=12.3  [221.9–245.7]

  workers= 3  (3 files × 9.3 GB = 27.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           102.2              280.0
  2           112.1              255.2
  3           114.6              249.8
   avg        109.6              261.7   std=16.1  [249.8–280.0]

  workers= 4  (4 files × 9.3 GB = 37.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           123.1              309.9
  2           122.6              311.2
  3           120.8              315.8
   avg        122.2              312.3   std=3.1  [309.9–315.8]

  workers= 5  (5 files × 9.3 GB = 46.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           149.5              318.9
  2           147.0              324.5
  3           137.3              318.7
   avg        144.6              320.7   std=3.3  [318.7–324.5]

  workers= 6  (6 files × 9.3 GB = 55.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           169.5              337.6
  2           164.1              348.7
  3           144.1              341.6
   avg        159.2              342.6   std=5.6  [337.6–348.7]

  workers= 7  (7 files × 9.3 GB = 65.2 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           210.9              316.5
  2           192.6              346.7
  3           156.6              347.5
   avg        186.7              336.9   std=17.7  [316.5–347.5]

  workers= 8  (8 files × 9.3 GB = 74.5 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           216.5              352.4
  2           212.2              359.5
  3           169.2              351.5
   avg        199.3              354.5   std=4.4  [351.5–359.5]

  workers=10  (10 files × 9.3 GB = 93.1 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           274.1              347.9
  2           270.6              352.4
  3           207.5              351.0
   avg        250.7              350.4   std=2.3  [347.9–352.4]

  workers=12  (12 files × 9.3 GB = 111.8 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           339.6              337.0
  2           324.7              352.5
  3           261.8              351.1
   avg        308.7              346.9   std=8.6  [337.0–352.5]

  ── All configs for 'dbutils' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        8      354.5     199.3     4.4   [351.5–359.5]
   2       10      350.4     250.7     2.3   [347.9–352.4]
   3       12      346.9     308.7     8.6   [337.0–352.5]
   4        6      342.6     159.2     5.6   [337.6–348.7]
   5        7      336.9     186.7    17.7   [316.5–347.5]
   6        5      320.7     144.6     3.3   [318.7–324.5]
   7        4      312.3     122.2     3.1   [309.9–315.8]
   8        3      261.7     109.6    16.1   [249.8–280.0]
   9        2      232.1      82.3    12.3   [221.9–245.7]
  10        1      147.2      64.9     7.7   [138.3–151.8]




=============================
==== Summary
=============================


  ── All configs for 'os' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        5      260.4     183.2     3.2   [256.8–263.1]
   2        6      255.4     224.2     7.5   [250.1–264.0]
   3        4      250.3     154.1    31.3   [215.8–276.9]
   4        7      236.1     283.5    14.7   [219.4–246.8]
   5       10      230.4     415.5    18.0   [218.5–251.1]
   6        8      213.5     360.9    24.8   [185.1–231.2]
   7        1      201.6      47.3     3.1   [198.0–203.4]
   8        2      200.5      95.2     5.8   [195.5–206.9]
   9        3      200.3     142.9     5.1   [196.4–206.1]
  10       12      191.7     598.7    12.3   [178.2–202.3]


  ── All configs for 'fsspec-databricks' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        5      271.6     175.8    11.7   [260.4–283.7]
   2        4      263.5     146.3    31.7   [227.1–284.9]
   3        6      255.0     224.6     9.4   [244.3–261.9]
   4        8      254.3     300.0     3.1   [251.8–257.8]
   5        7      237.5     284.0    28.1   [205.0–254.6]
   6       12      234.1     496.0    33.5   [197.0–262.0]
   7       10      215.3     451.7    38.3   [186.7–258.8]
   8        2      201.4      94.7     4.1   [197.6–205.7]
   9        3      196.7     145.5     3.9   [192.7–200.5]
  10        1      192.8      49.4     2.8   [191.0–196.0]
  
 
  ── All configs for 'dbutils' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        8      354.5     199.3     4.4   [351.5–359.5]
   2       10      350.4     250.7     2.3   [347.9–352.4]
   3       12      346.9     308.7     8.6   [337.0–352.5]
   4        6      342.6     159.2     5.6   [337.6–348.7]
   5        7      336.9     186.7    17.7   [316.5–347.5]
   6        5      320.7     144.6     3.3   [318.7–324.5]
   7        4      312.3     122.2     3.1   [309.9–315.8]
   8        3      261.7     109.6    16.1   [249.8–280.0]
   9        2      232.1      82.3    12.3   [221.9–245.7]
  10        1      147.2      64.9     7.7   [138.3–151.8]


```

###### FILE: notebooks/benchmark/result-10GB/benchmark-10GB-raw-output-2nd-run.txt ######

```txt

================================================================
  Backend: dbutils : missing
================================================================

During the first run, I tested manually os and fsspec-databricks.
Then I made a loop to run all 3 backends in a single command, starting by dbutils.
This second Run displays an anomaly for os and fsspec-databricks starting a workers=4.
They have likely been rate-limited.
So I took the output for dbutils and put it inside the first run.


================================================================
  Backend: fsspec-databricks
================================================================

  workers= 1  (1 files × 9.3 GB = 9.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            51.6              184.9
  2            46.1              206.8
  3            45.9              207.9
   avg         47.9              199.9   std=13.0  [184.9–207.9]

  workers= 2  (2 files × 9.3 GB = 18.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            82.0              232.7
  2            75.0              254.2
  3            75.2              253.5
   avg         77.4              246.8   std=12.2  [232.7–254.2]

  workers= 3  (3 files × 9.3 GB = 27.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           164.0              174.4
  2           173.0              165.4
  3           147.4              194.1
   avg        161.5              178.0   std=14.7  [165.4–194.1]

  workers= 4  (4 files × 9.3 GB = 37.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           476.6               80.0
  2           524.1               72.8
  3           428.1               89.1
   avg        476.3               80.6   std=8.2  [72.8–89.1]

  workers= 5  (5 files × 9.3 GB = 46.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           584.3               81.6
  2           658.4               72.4
  3           600.9               72.8
   avg        614.5               75.6   std=5.2  [72.4–81.6]

  workers= 6  (6 files × 9.3 GB = 55.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           684.6               83.6
  2           646.5               88.5
  3           627.2               78.5
   avg        652.8               83.5   std=5.0  [78.5–88.5]

  workers= 7  (7 files × 9.3 GB = 65.2 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           694.3               96.1
  2           760.9               87.7
  3           663.9               82.0
   avg        706.4               88.6   std=7.1  [82.0–96.1]

  workers= 8  (8 files × 9.3 GB = 74.5 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           821.5               92.9
  2           797.3               95.7
  3           777.9               76.5
   avg        798.9               88.4   std=10.4  [76.5–95.7]

  workers=10  (10 files × 9.3 GB = 93.1 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1          1125.9               84.7
  2          1160.4               82.2
  3           910.6               80.0
   avg       1065.6               82.3   std=2.4  [80.0–84.7]

  workers=12  (12 files × 9.3 GB = 111.8 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1          1309.9               87.4
  2          1433.7               79.8
  3          1196.8               76.8
   avg       1313.5               81.3   std=5.5  [76.8–87.4]

  ── All configs for 'fsspec-databricks' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        2      246.8      77.4    12.2   [232.7–254.2]
   2        1      199.9      47.9    13.0   [184.9–207.9]
   3        3      178.0     161.5    14.7   [165.4–194.1]
   4        7       88.6     706.4     7.1   [82.0–96.1]
   5        8       88.4     798.9    10.4   [76.5–95.7]
   6        6       83.5     652.8     5.0   [78.5–88.5]
   7       10       82.3    1065.6     2.4   [80.0–84.7]
   8       12       81.3    1313.5     5.5   [76.8–87.4]
   9        4       80.6     476.3     8.2   [72.8–89.1]
  10        5       75.6     614.5     5.2   [72.4–81.6]

================================================================
  Backend: os
================================================================

  workers= 1  (1 files × 9.3 GB = 9.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            49.7              192.0
  2            52.5              181.7
  3            45.2              211.2
   avg         49.1              195.0   std=15.0  [181.7–211.2]

  workers= 2  (2 files × 9.3 GB = 18.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1            82.9              230.0
  2            69.9              272.8
  3            73.9              258.0
   avg         75.6              253.6   std=21.7  [230.0–272.8]

  workers= 3  (3 files × 9.3 GB = 27.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           151.4              189.0
  2           175.0              163.5
  3           175.0              163.5
   avg        167.1              172.0   std=14.7  [163.5–189.0]

  workers= 4  (4 files × 9.3 GB = 37.3 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           586.6               65.0
  2           700.4               54.5
  3           527.1               72.4
   avg        604.7               64.0   std=9.0  [54.5–72.4]

  workers= 5  (5 files × 9.3 GB = 46.6 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           694.8               68.6
  2           678.1               70.3
  3           669.8               65.3
   avg        680.9               68.1   std=2.5  [65.3–70.3]

  workers= 6  (6 files × 9.3 GB = 55.9 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           845.5               67.7
  2           818.9               69.9
  3           731.6               67.3
   avg        798.7               68.3   std=1.4  [67.3–69.9]

  workers= 7  (7 files × 9.3 GB = 65.2 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1           763.4               87.5
  2          1085.6               61.5
  3           924.7               58.9
   avg        924.6               69.3   std=15.8  [58.9–87.5]

  workers= 8  (8 files × 9.3 GB = 74.5 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1          1013.4               75.3
  2          1038.3               73.5
  3           871.4               68.3
   avg        974.4               72.4   std=3.6  [68.3–75.3]

  workers=10  (10 files × 9.3 GB = 93.1 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1          1473.2               64.7
  2          1376.1               69.3
  3          1464.0               49.8
   avg       1437.8               61.3   std=10.2  [49.8–69.3]

  workers=12  (12 files × 9.3 GB = 111.8 GB, 3 slices)
  run      wall (s)  throughput (MB/s)
  ------ ---------- ------------------
  1          1660.5               68.9
  2          1589.5               72.0
  3          1474.9               62.3
   avg       1575.0               67.7   std=5.0  [62.3–72.0]

  ── All configs for 'os' (10 GB), ranked by avg throughput ──
   #  workers   avg MB/s  wall (s)     std        [min–max]
  ──  ─────── ────────── ───────── ─────── ────────────────
   1        2      253.6      75.6    21.7   [230.0–272.8]
   2        1      195.0      49.1    15.0   [181.7–211.2]
   3        3      172.0     167.1    14.7   [163.5–189.0]
   4        8       72.4     974.4     3.6   [68.3–75.3]
   5        7       69.3     924.6    15.8   [58.9–87.5]
   6        6       68.3     798.7     1.4   [67.3–69.9]
   7        5       68.1     680.9     2.5   [65.3–70.3]
   8       12       67.7    1575.0     5.0   [62.3–72.0]
   9        4       64.0     604.7     9.0   [54.5–72.4]
  10       10       61.3    1437.8    10.2   [49.8–69.3]




```

###### FILE: pyproject.toml ######

```toml
[project]
name = "azfr-databricks-landing-zone"
description = "File mover for Databricks Unity Catalog Volumes — routes files from landing to raw zones and records LOADED/FAILED events in a Delta audit table."
readme = "README.md"
authors = [
    { name = "Allianz Technology" }
]

dynamic = ["version"]
requires-python = "==3.12.*"

dependencies = [
    "pydantic",
    "pyyaml",
    "uuid6",
    "fsspec-databricks"
]

[dependency-groups]
dev = [
    "pytest>=8.4.2",
    "databricks-connect",
    "streamlit>=1.46.0",
    "plotly>=5.0",
    "pandas>=2.0",
    "databricks-bundles>=0.275.0",
]

[project.urls]
Homepage = "https://github.developer.allianz.io/azf-h1-datascience/azfr-databricks-landing-zone"

[project.scripts]
landing-mover = "azfr_databricks_landing_zone.main:main"

[build-system]
requires = ["hatchling", "hatch-vcs"]
build-backend = "hatchling.build"

[tool.hatch.version]
source = "vcs"

[[tool.uv.index]]
url = "https://nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz/repository/pypi-public/simple"
default = true

[tool.pytest]
testpaths = ["tests"]

[tool.coverage.run]
source = [
    "src",
]

[tool.ruff]
exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".ipynb_checkpoints",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pyenv",
    ".pytest_cache",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    ".vscode",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "site-packages",
    "venv",
]

line-length = 88
indent-width = 4

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "TC", "ANN"]
ignore = ["ANN101", "ANN102"]

```

###### FILE: README.md ######

```md
# azfr-databricks-landing-zone

Databricks Asset Bundle that moves files from **Landing** Volumes to **Raw** Volumes and records each outcome in a per-schema `_file_event_` Delta table.

> This project implements the file-push pattern described in the ADP architecture documentation:
> [1. File-push pattern](https://cmp.allianz.net/spaces/AZFRDP/pages/2689286279/1.+File-push+pattern)

---

## Overview

For each enabled route defined in the YAML configuration file the job starts an Autoloader streaming query (`availableNow=True`) and, for every micro-batch:

1. Detect new files in the `source` Volume via Autoloader (CloudFiles).
2. Copy each file to `target`, under a timestamped opaque sub-directory (`year=YYYY/month=MM/day=DD/<YYYYMMDDHHmmssSSS>-<8hex>/`).
3. Append a `LOADED` or `FAILED` row to the `_file_event_` Delta table derived from the target path (`<catalog>.<schema>._file_event_`).
4. Delete source files only when **all** copies in the batch succeeded.

Individual file failures are recorded without aborting other files or routes. When any failure occurs the batch raises so that Autoloader retries it — the write-ahead intent store (`intent_store/`) and Autoloader checkpoint guarantee exactly-once delivery across retries and crashes. The job raises a `BaseExceptionGroup` at the end if any route's streaming query failed; failed files remain in the source and will be retried on the next run.

---

## Architecture Decision Records

Key design decisions are documented in [`docs/adr/`](docs/adr/).

| # | Title | Summary |
|---|-------|---------|
| [001](docs/adr/001-two-thread-pools.md) | Two separate thread pools | Route dispatch and file I/O use separate `ThreadPoolExecutor` instances to prevent thread-pool starvation deadlock |
| [002](docs/adr/002-unbounded-route-worker.md) | Route worker is unbounded | `route_worker` has no `max_workers` cap; only `file_worker` is bounded because route threads just wait |
| [003](docs/adr/003-instrumentation-before-delta-write.md) | Timer scoped to copy phase only | Throughput (MB/s) measures `_execute_copies` exclusively; per-batch log files preserve metrics across retries |
| [004](docs/adr/004-file-backend-benchmark.md) | File backend benchmark (`dbutils` vs `fsspec-databricks` vs `os`) | `dbutils` is 30% faster than FUSE-based alternatives, but cannot be used with Autoloader (executors); `os` is the production backend |
| [005](docs/adr/005-exactly-once-file-move-semantics.md) | Exactly-once file move semantics | Three-layer guarantee: Autoloader checkpoint + idempotent file ops + write-ahead intent file |
| 📌 [006](docs/adr/006-write-ahead-intent-file.md) | Write-ahead intent file | `FileIntentStore` commits all target paths before any copy starts, making retries deterministic and crash-safe |
| [007](docs/adr/007-raise-before-delete-phase-ordering.md) | Raise-before-delete phase ordering | Source files are only deleted after all copies and the Delta write succeed; failures trigger Autoloader retry |
| [008](docs/adr/008-modification-time-as-file-identity-key.md) | `modificationTime` as file identity key | `(source_path, modification_time_ms)` is the stable composite key used by `FileIntentStore` across retries |
| 📌 [009](docs/adr/009-consecutive-run-case-analysis.md) | Consecutive-run case analysis | Exhaustive analysis of all 9 crash and retry scenarios proving correctness of the state machine |
| [010](docs/adr/010-fuse-stall-io-timeout.md) | UC Volume FUSE stalls cause infinite job hangs | FUSE kernel can stall indefinitely on ADLS network issues; `future.result()` has no timeout and never returns. Mitigated by switching to `fsspec-databricks`; definitive fix (`future.result(timeout=N)`) not yet implemented |
| [011](docs/adr/011-cost-simulation-webapp.md) | Cost simulation webapp | Use a lightweight Streamlit app as decision-support for comparing A/B/C architecture cost under shared assumptions |
| [012](docs/adr/012-per-route-autoloader-filename-reuse.md) | Per-route Autoloader filename reuse | By default Autoloader prevents reprocessing the same filename; `allow_filename_reuse=true` opts in per route. Enabling `allowOverwrites` alone causes double discovery in file notification mode, so FileEvent is also disabled in favour of directory listing only to eliminate spurious duplicates; Raw layer stays immutable; downstream MERGE handles de-duplication |

---

## Cost Simulation App

The repository includes a Streamlit app for interactive cost what-if analysis:

```bash
make cost-simulator
```

The app compares these ingestion architectures:

- `A`: 1 Job, 1 Autoloader
- `B`: 1 Job, N Autoloader
- `C`: N Job, N Autoloader (1 autoloader each)

Each case is evaluated for both compute modes (Serverless and Job Compute).

---

## Configuration

Routes and job settings are defined in a YAML file (example: `config/landing_config.yml`).

```yaml
max_concurrent_routes: 4      # Max simultaneous Autoloader queries (one per route)
max_file_workers: 8           # Max concurrent file-copy threads across all routes
file_backend: os              # "os" for FUSE-mounted Volumes (required for Autoloader); "dbutils" and "fsspec-databricks" also available but not compatible with Autoloader
work_base: /Volumes/<catalog>/landing/work/  # Per-route working dirs (checkpoint/, intent_store/, log/)

landing_routes:
  - id: source-A                          # Unique identifier
    source: /Volumes/<catalog>/landing/data/source-a/
    target: /Volumes/<catalog>/source-a/data/
    enabled: true                         # Set to false to skip without removing the route
    # allow_filename_reuse: false         # Set to true only if sender can resend same filename with corrections
  - id: source-B
    source: /Volumes/<catalog>/landing/data/source-a/
    target: /Volumes/<catalog>/source-b/data/
    enabled: true

# Failure injection — for integration testing in real environments only.
# List filenames (basenames) that should raise OSError during copy or delete.
# Omit or leave empty in production.
# fault_inject_copy_failures:
#   - some_file.bin
# fault_inject_delete_failures:
#   - some_file.bin
```

The `work_base` directory is created automatically and organised as follows:

```
<work_base>/<route-id>/
├── checkpoint/     # Autoloader checkpoint (tracks which files have been seen)
├── intent_store/   # Write-ahead intent files (guarantees exactly-once delivery across retries)
└── log/            # Per-batch structured log files with throughput metrics
```

**Constraints enforced at load time:**
- Route `id` values must be unique.
- `target` must be a Unity Catalog Volume path (`/Volumes/<catalog>/<schema>/...`) — `catalog` and `schema` are derived from it automatically to determine the `_file_event_` table location.
- `file_backend: "dbutils"` is not compatible with Autoloader: `foreachBatch` runs in a subprocess where `dbutils` is unavailable. Use `"os"` (default for production).

**Per-route field: `allow_filename_reuse` (optional, default: `false`)**

Set to `true` only if the data source can resend the same filename with corrections (e.g., corrected CSV after initial upload). When enabled:

- Autoloader switches to **directory listing mode** (one-time discovery channel) and enables `allowOverwrites`.
- Files are re-processed when their modification time changes (sender resend detected).
- **Raw layer remains immutable** — corrections are written as new timestamped partitions, never overwritten.
- **User responsibility**: de-duplicate downstream via MERGE on business keys (both old and corrected rows appear in Raw initially).

⚠️ **Important**: Toggling this flag on an existing route invalidates Autoloader's checkpoint and triggers a one-time full directory re-listing (see [ADR-012](docs/adr/012-per-route-autoloader-filename-reuse.md) for recovery steps). Plan for an ingestion spike on the first run after toggling.

---

## Deployment

The bundle is deployed with the [Databricks CLI](https://docs.databricks.com/dev-tools/cli/index.html):

```bash
# Deploy to the dev target (default)
databricks bundle deploy --target dev

# Run the job
databricks bundle run --target dev
```

The `config_path` variable controls which config file the job reads. It can be overridden at deploy or run time:

```bash
databricks bundle run --target dev \
  --var config_path=/Volumes/<catalog>/<schema>/<volume>/_config/landing_config.yml
```

---

## Running locally / in a Databricks Job

**Python script task (Databricks Job):**

Pass `config_path` as `--config-path` in the task's *Parameters* field:

```json
["--config-path", "/Volumes/<catalog>/<schema>/<volume>/_config/landing_config.yml"]
```


---

## Development

**Requirements:** Python 3.12, [uv](https://github.com/astral-sh/uv)

```bash
# Install dependencies (including dev extras)
uv sync

# Run tests
uv run pytest

# Lint
uv run ruff check src tests
```

---

## Table — `_file_event_`

One table per `(<catalog>, <schema>)` pair derived from the route `target` path. Each row records a single file outcome.

To investigate failed files, query the table:

```sql
SELECT * FROM <catalog>.<schema>._file_event_
WHERE state = 'FAILED'
ORDER BY event_time DESC;
```

Possible `state` values: `LOADED`, `FAILED`, `INVALIDATED`, `REVALIDATED`, `DELETED`.
```

###### FILE: resources/__init__.py ######

```py
from pathlib import Path

from databricks.bundles.core import Bundle, Resources

from azfr_databricks_landing_zone.config import load_config
from resources.benchmark_job import benchmark_job
from resources.landing_jobs import create_landing_job


def load_resources(bundle: Bundle) -> Resources:
    resources = Resources()
    
    resources.add_resource("benchmark_job", benchmark_job)

    # Dynamically generate one landing job per route defined in the local config.
    config_path = Path(__file__).parent.parent / "config" / "landing_config.yml"
    config = load_config(str(config_path))

    for route in config.landing_routes:
        key = f"landing_job_{route.id.replace('-', '_')}"
        resources.add_resource(key, create_landing_job(route))

    return resources

```

###### FILE: resources/benchmark_job.py ######

```py
from databricks.bundles.jobs import Job, JobParameterDefinition, Library, NotebookTask, Task
from databricks.bundles.jobs._models.source import Source

from resources.job_clusters import JOB_CLUSTER, JOB_CLUSTER_KEY

benchmark_job = Job(
    name="Benchmark File Move",
    job_clusters=[JOB_CLUSTER],
    parameters=[
        JobParameterDefinition(name="io_backend", default="dbutils"),
        JobParameterDefinition(name="source_folder", default="/Volumes/azfr_dev_raw/sample_raw/benchmark/data/benchmark-10GB/"), # Better to copy the source data in a different volume than the target folder
        JobParameterDefinition(name="target_folder", default="/Volumes/azfr_dev_raw/sample_raw/benchmark/active-transfers/"),
        JobParameterDefinition(name="results_folder", default="/Volumes/azfr_dev_raw/sample_raw/benchmark/results/"),
        JobParameterDefinition(name="n_runs", default="3"),
        JobParameterDefinition(name="worker_counts", default="1,2,3,4,5,6,7,8,10,12"),
    ],
    tasks=[
        Task(
            task_key="benchmark-file-move",
            job_cluster_key=JOB_CLUSTER_KEY,
            notebook_task=NotebookTask(
                notebook_path="${workspace.file_path}/notebooks/benchmark/benchmark_file_move",
                source=Source.WORKSPACE,
                base_parameters={
                    "io_backend": "{{job.parameters.io_backend}}",
                    "source_folder": "{{job.parameters.source_folder}}",
                    "target_folder": "{{job.parameters.target_folder}}",
                    "results_folder": "{{job.parameters.results_folder}}",
                    "n_runs": "{{job.parameters.n_runs}}",
                    "worker_counts": "{{job.parameters.worker_counts}}",
                },
            ),
            libraries=[Library(whl="./dist/*.whl")],
        )
    ],
)

```

###### FILE: resources/job_clusters.py ######

```py
from databricks.bundles.jobs import ClusterSpec, JobCluster
from databricks.bundles.jobs._models.data_security_mode import DataSecurityMode
from databricks.bundles.jobs._models.kind import Kind
from databricks.bundles.jobs._models.runtime_engine import RuntimeEngine

JOB_CLUSTER_KEY = "azfr-landing-job-cluster"

JOB_CLUSTER = JobCluster(
    job_cluster_key=JOB_CLUSTER_KEY,
    new_cluster=ClusterSpec(
        spark_version="17.3.x-scala2.13",
        node_type_id="Standard_D4ds_v5",
        spark_env_vars={
            "PYSPARK_PYTHON": "/databricks/python3/bin/python3",
        },
        data_security_mode=DataSecurityMode.DATA_SECURITY_MODE_AUTO,
        runtime_engine=RuntimeEngine.STANDARD,
        kind=Kind.CLASSIC_PREVIEW,
        is_single_node=True,
    ),
)
```

###### FILE: resources/landing_jobs.py ######

```py
from databricks.bundles.jobs import (
    FileArrivalTriggerConfiguration, Job, Library, PythonWheelTask,
    QueueSettings, Task, TriggerSettings,
)
from databricks.bundles.jobs._models.pause_status import PauseStatus

from azfr_databricks_landing_zone.config import LandingRoute
from resources.job_clusters import JOB_CLUSTER, JOB_CLUSTER_KEY


def create_landing_job(route: LandingRoute) -> Job:
    return Job(
        name=f"Landing - {route.id}",
        trigger=TriggerSettings(
            file_arrival=FileArrivalTriggerConfiguration(url=route.source),
            pause_status=PauseStatus.UNPAUSED if route.enabled else PauseStatus.PAUSED,
        ),
        queue=QueueSettings(enabled=True),
        max_concurrent_runs=1,
        job_clusters=[JOB_CLUSTER],
        tasks=[
            Task(
                task_key=f"landing-{route.id}",
                job_cluster_key=JOB_CLUSTER_KEY,
                python_wheel_task=PythonWheelTask(
                    package_name="azfr_databricks_landing_zone",
                    entry_point="landing-mover",
                    parameters=[
                        "--config-path",
                        "${var.config_path}",
                        "--route-id",
                        route.id,
                    ],
                ),
                libraries=[Library(whl="./dist/*.whl")],
            )
        ],
    )

```

###### FILE: resources/targets.yml ######

```yml
variables:
  config_path:
    description: Path to the landing config YAML file

targets:
  dev:
    mode: development
    default: true
    workspace:
      host: https://adb-4260984301853003.3.azuredatabricks.net
      artifact_path: "/Volumes/azfr_dev_raw/landing/artifact/${workspace.current_user.userName}"
    variables:
      config_path: "/Volumes/azfr_dev_raw/landing/config/landing_config.yml"

```

###### FILE: src/azfr_databricks_landing_zone/config.py ######

```py
"""Configuration models and loader for the Databricks landing zone."""
import re
import yaml
from collections import Counter

from pydantic import BaseModel, Field, PrivateAttr, model_validator
from typing import Literal


VOLUME_PATH_RE = re.compile(r"^/Volumes/(?P<catalog>[^/]+)/(?P<schema>[^/]+)/(?P<volume>[^/]+)/")


def extract_from_volume_path(target: str) -> tuple[str, str] | None:
    """Return (catalog, schema) from a /Volumes/<catalog>/<schema>/<volume>/... path, or None."""
    m = VOLUME_PATH_RE.match(target)
    if m:
        return m.group("catalog"), m.group("schema")
    return None


class LandingRoute(BaseModel):
    """Configuration for a single landing zone route.

    catalog and schema_name are inferred from the target Databricks Volume path
    (/Volumes/<catalog>/<schema>/...).
    """
    id: str = Field(description="Unique identifier for the route.")
    source: str = Field(description="Source directory to scan for incoming files.")
    target: str = Field(description="Root destination directory under a Unity Catalog Volume (/Volumes/<catalog>/<schema>/<volume>/...). catalog and schema are inferred from this path.")
    enabled: bool = Field(default=True, description="When False the route is skipped entirely at job startup.")
    allow_filename_reuse: bool = Field(default=False, description="Enable same-filename correction ingestion for this route (sets allowOverwrites and disables managed file events). See ADR-012 Decision and Rationale.")
    allow_filename_reuse_recovery: bool = Field(default=False, description="Temporary one-run continuation-token recovery mode. Disable after one successful run. See ADR-012 (Checkpoint invalidation and recovery)..")
    _catalog: str = PrivateAttr()
    _schema: str = PrivateAttr()

    @model_validator(mode="after")
    def validate_volume_target(self) -> "LandingRoute":
        """Validate that target is a Unity Catalog Volume path and extract catalog/schema from it."""
        extracted = extract_from_volume_path(self.target)
        if extracted is None:
            raise ValueError(
                "catalog and schema could not be derived from field 'target'; "
                "use a Databricks Volume path (/Volumes/<catalog>/<schema>/<volume>/path/...)"
            )
        self._catalog = extracted[0]
        self._schema = extracted[1]
        return self

    @property
    def file_event_table(self) -> str:
        """Return the fully qualified event log table name (<catalog>.<schema>._file_event_)."""
        return f"{self._catalog}.{self._schema}._file_event_"


class AppConfig(BaseModel):
    """Root configuration holding all landing routes; validates uniqueness of route IDs."""
    max_file_workers: int = Field(ge=1, description="Maximum number of concurrent file-worker threads shared across all active routes.")
    file_backend: Literal["dbutils", "fsspec-databricks", "os"] = Field(description="File system backend to use for copy and delete operations. Note: 'dbutils' is not compatible with Autoloader (foreachBatch runs on executors where dbutils is not available).")
    work_base: str = Field(description="Base Volume path for per-route working directories. Each route gets {work_base}/{route.id}/ containing 'checkpoint/' (Autoloader), 'intent_store/' (idempotency state), and 'log/' (per-run instrumentation).")
    landing_routes: list[LandingRoute] = Field(default_factory=list, description="Ordered list of routes to process. Routes with enabled=False are skipped.")
    fault_inject_copy_failures: set[str] | None = Field(default=None, description="Filenames (basenames only) for which copy_file should raise OSError. Used for failure-injection testing in real environments. Omit (None) in production.")
    fault_inject_delete_failures: set[str] | None = Field(default=None, description="Filenames (basenames only) for which delete_file should raise OSError. Used for failure-injection testing in real environments. Omit (None) in production.")

    @model_validator(mode="after")
    def check_unique_ids(self) -> "AppConfig":
        """Raise if any landing route id is duplicated."""
        ids = [c.id for c in self.landing_routes]
        duplicates = {i for i, c in Counter(ids).items() if c > 1}
        if duplicates:
            raise ValueError(f"Duplicate ids found: {duplicates}")
        return self

    @property
    def active_and_disabled_routes(self) -> tuple[list[LandingRoute], list[LandingRoute]]:
        """Return (active_routes, disabled_routes) partitioned in a single pass."""
        active, disabled = [], []
        for route in self.landing_routes:
            (active if route.enabled else disabled).append(route)
        return active, disabled


def load_config(path: str) -> AppConfig:
    """Load and validate landing zone configuration from a YAML file."""
    with open(path, "r", encoding="utf-8") as f:
        raw = yaml.safe_load(f) or {}

    return AppConfig.model_validate(raw)

```

###### FILE: src/azfr_databricks_landing_zone/file_copier.py ######

```py
from dataclasses import dataclass
from pathlib import Path
from uuid import uuid4

from azfr_databricks_landing_zone.file_event import FileEvent, utc_now
from azfr_databricks_landing_zone.fs_backend import FileMetadata, FileSystemBackend


def _build_target_path(base: str, filename: str) -> str:
    """
    Build the full target path for a single file.

    Structure:
      {base}/year=YYYY/month=MM/day=DD/{YYYYMMDDHHmmssSSS}-{8hex}/{filename}

    The directory is based on the current wall-clock time, not the file's
    modification time.  The path is intentionally opaque — consumers locate
    files via the `_file_event_` table, not directory scans.  Idempotency on
    retry is guaranteed by the FileIntentStore, which persists the generated
    target path before any copy starts.
    """
    now = utc_now()
    partition = f"year={now.year}/month={now.month:02d}/day={now.day:02d}"
    ts_str = now.strftime("%Y%m%d%H%M%S") + f"{now.microsecond // 1000:03d}"
    unique_dir = f"{ts_str}-{uuid4().hex[:8]}"
    return Path(base, partition, unique_dir, filename).as_posix()


@dataclass
class FileCopier:
    """
    Encapsulates a single file copy task: source path, target path, and file metadata.

    Call run() to execute the copy (raises on failure).
    Use loaded_event() or failed_event() to build the corresponding FileEvent.
    """

    metadata: FileMetadata
    source_path: str
    target_path: str
    fs: FileSystemBackend

    @classmethod
    def from_metadata(cls, metadata: FileMetadata, target_base: str, fs: FileSystemBackend) -> "FileCopier":
        """Build a FileCopier from a FileMetadata, a route target base, and a backend.

        The target path is an opaque wall-clock-stamped path with a random
        suffix.  It is written to the FileIntentStore before any copy starts,
        so retries reuse the same path rather than generating a new one.
        """
        return cls(
            metadata=metadata,
            source_path=metadata.path,
            target_path=_build_target_path(target_base, metadata.name),
            fs=fs,
        )

    def run(self) -> None:
        """Copy the file from source to target. Creates intermediate directories. Raises on failure."""
        self.fs.copy_file(self.source_path, self.target_path)

    @property
    def intent_entry(self) -> tuple[str, int, str]:
        """Return the ``(source_path, modification_time_ms, target_path)`` tuple for the intent store."""
        return (self.metadata.path, self.metadata.modification_time, self.target_path)

    def loaded_event(self) -> FileEvent:
        return FileEvent.loaded(utc_now(), self.target_path, self.metadata)

    def failed_event(self) -> FileEvent:
        return FileEvent.failed(utc_now(), self.target_path, self.metadata)


@dataclass
class FileCopyResult:
    """The outcome of a single FileCopier.run() call, enriched with route context."""

    route_id: str
    event: FileEvent
    source_path: str
    error: Exception | None = None



```

###### FILE: src/azfr_databricks_landing_zone/file_event.py ######

```py
from dataclasses import dataclass
from datetime import datetime, timezone
from enum import Enum

from pyspark.sql import Row, SparkSession
from pyspark.sql.functions import col
from pyspark.sql.types import (
    LongType,
    StringType,
    StructField,
    StructType,
    TimestampType,
)
from uuid6 import uuid7
from pyspark.errors.exceptions.base import AnalysisException

from azfr_databricks_landing_zone.fs_backend import FileMetadata

_TABLE_OR_VIEW_NOT_FOUND = "TABLE_OR_VIEW_NOT_FOUND"


class FileState(str, Enum):
    LOADED = "LOADED"
    FAILED = "FAILED"
    INVALIDATED = "INVALIDATED"
    REVALIDATED = "REVALIDATED"
    DELETED = "DELETED"


@dataclass
class FileEvent:
    event_id: str
    event_time: datetime
    path: str
    state: FileState
    size: int | None
    creation_time: datetime | None
    modification_time: datetime | None

    @property
    def filename(self) -> str:
        """The filename extracted from *path* (last path segment)."""
        return self.path.rsplit("/", 1)[-1]

    @classmethod
    def create(cls, event_time: datetime, path: str, state: FileState, metadata: FileMetadata) -> "FileEvent":
        return cls(
            event_id=str(uuid7()),
            event_time=event_time,
            path=path,
            state=state,
            size=metadata.size,
            creation_time=ms_epoch_to_utc(metadata.creation_time),
            modification_time=ms_epoch_to_utc(metadata.modification_time),
        )

    @classmethod
    def loaded(cls, event_time: datetime, path: str, metadata: FileMetadata) -> "FileEvent":
        return cls.create(event_time, path, FileState.LOADED, metadata)

    @classmethod
    def failed(cls, event_time: datetime, path: str, metadata: FileMetadata) -> "FileEvent":
        return cls.create(event_time, path, FileState.FAILED, metadata)

    def to_row(self) -> Row:
        return Row(
            event_id=self.event_id,
            event_time=self.event_time,
            path=self.path,
            state=self.state.value,
            size=self.size,
            creation_time=self.creation_time,
            modification_time=self.modification_time,
        )


class FileEventTable:
    """Writes FileEvent rows to a Delta table."""

    _SCHEMA = StructType(
        [
            StructField("event_id", StringType(), nullable=False),
            StructField("event_time", TimestampType(), nullable=False),
            StructField("path", StringType(), nullable=False),
            StructField("state", StringType(), nullable=False),
            StructField("size", LongType(), nullable=True),
            StructField("creation_time", TimestampType(), nullable=True),
            StructField("modification_time", TimestampType(), nullable=True),
        ]
    )

    def __init__(self, spark: SparkSession, table: str) -> None:
        self._spark = spark
        self._table = table

    def append_batch(self, events: list[FileEvent]) -> None:
        """Append multiple event rows to the Delta table in a single write.

        Raises on failure so the caller can decide whether to retry.
        """
        if not events:
            return

        rows = [e.to_row() for e in events]
        df = self._spark.createDataFrame(rows, schema=self._SCHEMA)
        df.write.format("delta").mode("append").saveAsTable(self._table)

    def get_loaded_target_paths(self, target_paths: set[str]) -> set[str]:
        """Return the subset of *target_paths* that already have a LOADED event.

        Used during retry resolution to skip files successfully audited in a
        previous batch attempt. Returns an empty set when the table does not
        yet exist or *target_paths* is empty.
        """
        if not target_paths:
            return set()
        try:
            rows = (
                self._spark.table(self._table)
                .filter(
                    (col("state") == FileState.LOADED.value)
                    & col("path").isin(list(target_paths))
                )
                .select("path")
                .collect()
            )
            return {row.path for row in rows}
        except AnalysisException as e:
            # _TABLE_OR_VIEW_NOT_FOUND is the Databricks runtime condition code.
            # Also match on the stringified exception as a fallback for runtime
            # versions where the condition string has changed.
            if e.getCondition() == _TABLE_OR_VIEW_NOT_FOUND or _TABLE_OR_VIEW_NOT_FOUND in str(e):
                return set()
            raise


def utc_now() -> datetime:
    return datetime.now(tz=timezone.utc)


def ms_epoch_to_utc(ms: int | None) -> datetime | None:
    """Convert a millisecond epoch integer (as returned by dbutils.fs.ls) to UTC datetime."""
    if ms is None:
        return None
    return datetime.fromtimestamp(ms / 1000.0, tz=timezone.utc)

```

###### FILE: src/azfr_databricks_landing_zone/file_intent.py ######

```py
"""Write-ahead intent store for per-batch copy idempotency.

Persists ``(source_path, modification_time_ms) → target_path`` mappings as a
JSON file at ``{work_base}/{route_id}/intent_store/_intent.json``.

The file carries a ``status`` field with two possible values:

- ``"in_progress"``: written before any copy starts (write-ahead guarantee).
  Signals that a previous batch attempt did not complete successfully.
  On retry the caller reuses the pre-committed target paths so that copies
  are idempotent overwrites.

- ``"success"``: written after source files are deleted and before
  ``foreachBatch`` returns.  If Autoloader replays the batch (checkpoint not
  yet committed), the caller detects this status and skips all work — no
  copies, no Delta writes, no deletes — letting Spark commit cleanly.

Writes are atomic: content is first written to a ``.tmp`` sibling and then
``os.replace``-d into place.  Readers always see either the old file or the
new one — never a partial write.  A leftover ``.tmp`` from a crash is ignored.
"""
import json
import os
from dataclasses import dataclass
from enum import Enum


class IntentStatus(Enum):
    IN_PROGRESS = "in_progress"
    SUCCESS = "success"


@dataclass
class IntentSnapshot:
    """Parsed content of an intent file."""

    status: IntentStatus
    entries: dict[tuple[str, int], str]  # {(source_path, modtime_ms): target_path}


class FileIntentStore:
    """Crash-safe intent store for a single route."""

    def __init__(self, path: str, tmp_path: str) -> None:
        self._path = path
        self._tmp_path = tmp_path

    @classmethod
    def for_route(cls, work_base: str, route_id: str) -> "FileIntentStore":
        """Return a store rooted at ``{work_base}/{route_id}/intent_store/``."""
        base = f"{work_base}/{route_id}/intent_store"
        return cls(path=f"{base}/_intent.json", tmp_path=f"{base}/_intent.json.tmp")

    def read(self) -> IntentSnapshot | None:
        """Return the current snapshot, or ``None`` when no file exists."""
        if not os.path.exists(self._path):
            return None
        with open(self._path, encoding="utf-8") as f:
            data: dict = json.load(f)
        entries = {
            (entry["source_path"], entry["modification_time_ms"]): entry["target_path"]
            for entry in data.get("entries", [])
        }
        return IntentSnapshot(status=IntentStatus(data["status"]), entries=entries)

    def write_in_progress(self, entries: list[tuple[str, int, str]]) -> None:
        """Atomically write an ``in_progress`` intent file with *entries*.

        Each entry is ``(source_path, modification_time_ms, target_path)``.
        Called before any file copy starts (write-ahead guarantee).
        """
        self._atomic_write(IntentStatus.IN_PROGRESS, entries)

    def mark_success(self, entries: list[tuple[str, int, str]]) -> None:
        """Atomically write a ``success`` intent file with *entries*.

        Called after all source files are deleted and before ``foreachBatch``
        returns.  If Autoloader replays this batch before the Spark checkpoint
        is committed, ``_resolve_copiers`` detects the ``success`` status and
        skips all work without touching Delta or the file system.
        """
        self._atomic_write(IntentStatus.SUCCESS, entries)

    def _atomic_write(self, status: IntentStatus, entries: list[tuple[str, int, str]]) -> None:
        data = {
            "status": status.value,
            "entries": [
                {
                    "source_path": source,
                    "modification_time_ms": modtime_ms,
                    "target_path": target,
                }
                for source, modtime_ms, target in entries
            ],
        }
        os.makedirs(os.path.dirname(self._path), exist_ok=True)
        with open(self._tmp_path, "w", encoding="utf-8") as f:
            json.dump(data, f)
        os.replace(self._tmp_path, self._path)

```

###### FILE: src/azfr_databricks_landing_zone/fs_backend.py ######

```py
"""File-system backend abstractions for file-move operations (copy and delete)."""
import os
import shutil
from dataclasses import dataclass
from typing import TYPE_CHECKING, Protocol

if TYPE_CHECKING:
    from databricks.sdk.dbutils import FileInfo as DbFileInfo
    from databricks.sdk.runtime.dbutils_stub import FileInfo as StubFileInfo
    AnyFileInfo = StubFileInfo | DbFileInfo


def _strip_dbfs_prefix(path: str) -> str:
    """Normalize a path to /Volumes/... — removes 'dbfs:' prefix if present."""
    if path.startswith("dbfs:"):
        return path[len("dbfs:"):]
    return path


@dataclass
class FileMetadata:
    """Portable file metadata — decoupled from the Databricks SDK FileInfo type."""

    path: str
    name: str
    size: int
    modification_time: int  # milliseconds since epoch
    creation_time: int | None  # milliseconds since epoch

    @classmethod
    def from_file_info(cls, file_info: "AnyFileInfo") -> "FileMetadata":
        """Build a FileMetadata from a Databricks SDK FileInfo (dbutils.fs.ls result)."""
        return cls(
            path=_strip_dbfs_prefix(file_info.path),
            name=file_info.name,
            size=file_info.size,
            modification_time=file_info.modificationTime,
            creation_time=getattr(file_info, "creationTime", None),
        )

    @classmethod
    def from_autoloader_row(cls, row: object) -> "FileMetadata":
        """Build a FileMetadata from an Autoloader binaryFile row (path, modificationTime, length)."""
        path = _strip_dbfs_prefix(row.path)  # type: ignore[union-attr]
        return cls(
            path=path,
            name=os.path.basename(path),
            size=row.length,  # type: ignore[union-attr]
            modification_time=int(row.modificationTime.timestamp() * 1000),  # type: ignore[union-attr]
            creation_time=None,
        )


class FileSystemBackend(Protocol):
    """Structural interface for file-system operations used by FileCopier."""

    def copy_file(self, source_path: str, target_path: str) -> None:
        """Copy *source_path* to *target_path*, creating any missing parent directories."""
        ...

    def delete_file(self, path: str) -> None:
        """Delete the file at *path*. Must be idempotent — must not raise if the file is already absent."""
        ...


class DbutilsBackend:
    """IO backend that delegates to Databricks dbutils.fs.

    Not compatible with Autoloader: ``foreachBatch`` runs on executors where
    ``dbutils`` is not available.  Use ``NativeOsBackend`` instead.
    """

    def __init__(self) -> None:
        from databricks.sdk.runtime import dbutils  # imported lazily — not needed for other backends
        self._dbutils = dbutils

    def copy_file(self, source_path: str, target_path: str) -> None:
        target_parent = os.path.dirname(target_path)
        self._dbutils.fs.mkdirs(target_parent)
        self._dbutils.fs.cp(source_path, target_path)

    def delete_file(self, path: str) -> None:
        self._dbutils.fs.rm(path)  # returns False (not raises) when the file is absent — idempotent


class FsspecDatabricksBackend:
    """IO backend that uses the fsspec Databricks filesystem driver."""

    def __init__(self) -> None:
        from fsspec_databricks import DatabricksFileSystem  # imported lazily — not a hard dependency for other backends
        self._fs = DatabricksFileSystem()

    def copy_file(self, source_path: str, target_path: str) -> None:
        target_parent = os.path.dirname(target_path)
        self._fs.makedirs(target_parent, exist_ok=True)
        self._fs.copy(source_path, target_path)

    def delete_file(self, path: str) -> None:
        # The fsspec Databricks driver raises FileNotFoundError when the file is absent.
        # Other error types (PermissionError, IOError) are intentionally not caught here
        # and propagate to the caller — they signal a genuine access failure rather than
        # an expected missing-file condition.
        try:
            self._fs.rm(path)
        except FileNotFoundError:
            pass


class NativeOsBackend:
    """IO backend that uses the standard-library os/shutil modules (FUSE-mounted /Volumes/ paths)."""

    def copy_file(self, source_path: str, target_path: str) -> None:
        target_parent = target_path.rsplit("/", 1)[0]
        os.makedirs(target_parent, exist_ok=True)
        shutil.copy(source_path, target_path)

    def delete_file(self, path: str) -> None:
        try:
            os.remove(path)
        except FileNotFoundError:
            pass


class FaultInjectingBackend:
    """Wraps any FileSystemBackend and raises OSError for specific filenames.

    Used to simulate partial copy or delete failures in integration tests.
    Matches on the *basename* of the path so callers don't need to know the
    full source/target directory structure.

    Example::

        fs = FaultInjectingBackend(
            NativeOsBackend(),
            copy_failures={"file_100MB_00d92262.bin", "file_100MB_0330667d.bin"},
            delete_failures={"file_100MB_0586be07.bin"},
        )
    """

    def __init__(
        self,
        backend: FileSystemBackend,
        copy_failures: set[str] | None,
        delete_failures: set[str] | None,
    ) -> None:
        self._backend = backend
        self._copy_failures: set[str] = copy_failures or set()
        self._delete_failures: set[str] = delete_failures or set()

    def copy_file(self, source_path: str, target_path: str) -> None:
        if os.path.basename(source_path) in self._copy_failures:
            raise OSError(f"[FaultInjectingBackend] Simulated copy failure for {source_path!r}")
        self._backend.copy_file(source_path, target_path)

    def delete_file(self, path: str) -> None:
        if os.path.basename(path) in self._delete_failures:
            raise OSError(f"[FaultInjectingBackend] Simulated delete failure for {path!r}")
        self._backend.delete_file(path)


_BACKENDS: dict[str, type[FileSystemBackend]] = {
    "dbutils": DbutilsBackend,
    "fsspec-databricks": FsspecDatabricksBackend,
    "os": NativeOsBackend,
}


def create_fs_backend(
    name: str,
    copy_failures: set[str] | None = None,
    delete_failures: set[str] | None = None,
) -> FileSystemBackend:
    """Return an instantiated FileSystemBackend for the given *name*.

    Valid values: ``"dbutils"``, ``"fsspec-databricks"``, ``"os"``.
    Raises ``ValueError`` for unknown names.

    When *copy_failures* or *delete_failures* are non-empty, the backend is
    wrapped in a ``FaultInjectingBackend`` that raises ``OSError`` for the
    named files (matched by basename). Intended for integration testing in
    real environments only — leave both sets empty in production.
    """
    cls = _BACKENDS.get(name)
    if cls is None:
        raise ValueError(f"Unknown io_backend '{name}'. Valid options: {list(_BACKENDS)}")
    backend: FileSystemBackend = cls()
    if copy_failures or delete_failures:
        return FaultInjectingBackend(backend, copy_failures=copy_failures, delete_failures=delete_failures)
    return backend

```

###### FILE: src/azfr_databricks_landing_zone/instrumentation.py ######

```py
import json
import os
import pathlib

from azfr_databricks_landing_zone.file_copier import FileCopyResult


def mb_per_sec(size_bytes: int, elapsed: float) -> float:
    return size_bytes / elapsed / 1_000_000 if elapsed > 0 else float('inf')


_PARTIAL_BATCH_WARNING = (
    "CHECKPOINT NOT COMMITTED — all files will be retried on next run; "
    "files with status LOADED will not be re-copied and are ready to be consumed."
)


class JobInstrumentation:
    """Receives lifecycle hooks from LandingMoverJob and computes / prints all metrics.

    When *log_file* is provided, log lines are appended to that file instead of
    being printed to stdout.  This is required when the instrumentation runs inside
    a Spark Connect foreachBatch subprocess, whose stdout is not forwarded to the
    notebook cell output.  The driver process then reads the file after
    ``awaitTermination()`` and re-prints its contents.
    """

    def __init__(self, log_dir: str, run_id: str) -> None:
        self._log_dir = log_dir
        self._run_id = run_id
        self._log_buffer: list[str] = []
        self._summary: dict | None = None

    @classmethod
    def for_route(cls, log_dir: str, run_id: str) -> "JobInstrumentation":
        """Create a JobInstrumentation for *run_id* inside *log_dir*.

        Creates *log_dir* if absent, removes any files left by previous runs,
        then returns a fresh instance bound to *log_dir* and *run_id*.
        Per-batch log and summary files are created lazily by flush().
        """
        os.makedirs(log_dir, exist_ok=True)
        for entry in os.scandir(log_dir):
            os.remove(entry.path)
        return cls(log_dir=log_dir, run_id=run_id)

    def _log(self, line: str) -> None:
        """Buffer *line* for later writing via flush().

        Unity Catalog Volumes FUSE does not support lseek, so append mode ("a") raises
        OSError ESPIPE.  Instead, lines are accumulated in memory and flushed in one
        write via flush().
        """
        self._log_buffer.append(line)

    def flush(self, batch_id: int) -> None:
        """Write buffered log lines and summary to per-batch files.

        Each Autoloader micro-batch writes to its own
        ``route-{run_id}-{batch_id}.log`` / ``summary-{run_id}-{batch_id}.json``
        so that logs and metrics from multiple batches in the same run
        (e.g. a retry followed by new files) are all preserved.
        """
        log_file = f"{self._log_dir}/route-{self._run_id}-{batch_id}.log"
        with open(log_file, "w", encoding="utf-8") as f:
            if self._log_buffer:
                f.write("\n".join(self._log_buffer) + "\n")
        if self._summary is not None:
            summary_file = f"{self._log_dir}/summary-{self._run_id}-{batch_id}.json"
            with open(summary_file, "w", encoding="utf-8") as f:
                json.dump(self._summary, f)
        self._log_buffer = []
        self._summary = None

    def replay_logs(self, route_id: str) -> None:
        """Print all per-batch log files to stdout in batch order.

        Called on the driver after ``awaitTermination()`` to surface logs written
        by the foreachBatch subprocess.  Prints a fallback message when no log
        files were produced (e.g. the worker was killed before flush() ran).
        """
        prefix = f"route-{self._run_id}-"
        log_files = sorted(
            (e.path for e in os.scandir(self._log_dir) if e.name.startswith(prefix) and e.name.endswith(".log")),
            key=lambda p: int(os.path.basename(p)[len(prefix):-4]),
        )
        if not log_files:
            print(f"[{route_id}] No new files.")
            return
        for path in log_files:
            with open(path, encoding="utf-8") as f:
                print(f.read(), end="")

    def read_summary(self) -> "dict | None":
        """Aggregate and return summaries from all per-batch summary files for this route.

        All summary files are scoped to *log_dir*, which is a per-route directory,
        Returns ``None`` when no summary files exist (route processed zero files).
        When multiple batches ran (e.g. a retry followed by new files), their counts
        and elapsed times are summed so the caller sees a single total.
        """
        prefix = f"summary-{self._run_id}-"
        summaries = [
            json.loads(pathlib.Path(e.path).read_text(encoding="utf-8"))
            for e in os.scandir(self._log_dir)
            if e.name.startswith(prefix) and e.name.endswith(".json")
        ]
        if not summaries:
            return None
        return {
            "route_id": summaries[0]["route_id"],
            "table": summaries[0]["table"],
            "loaded": sum(s["loaded"] for s in summaries),
            "failed": sum(s["failed"] for s in summaries),
            "loaded_bytes": sum(s["loaded_bytes"] for s in summaries),
            "elapsed": sum(s["elapsed"] for s in summaries),
            "copy_failures": [f for s in summaries for f in s.get("copy_failures", [])],
            "delete_failures": [f for s in summaries for f in s.get("delete_failures", [])],
            "partial": any(s.get("partial", False) for s in summaries),
        }

    def on_file_loaded(self, route_id: str, filename: str, path: str, size_bytes: int, elapsed: float) -> None:
        mbs = mb_per_sec(size_bytes, elapsed)
        self._log(f"[{route_id}] LOADED  {filename} \u2192 {path} ({elapsed:.3f}s, {mbs:.1f} MB/s)")

    def on_file_failed(self, route_id: str, filename: str, target_path: str, elapsed: float, error: Exception) -> None:
        self._log(f"[{route_id}] FAILED  {filename}: {error} ({elapsed:.3f}s)")
        self._log(f"[{route_id}]   target: {target_path}")

    def on_file_delete_failed(self, route_id: str, path: str, error: Exception) -> None:
        self._log(f"[{route_id}] DELETE FAILED  {path}: {error}")

    def on_route_done(self, route_id: str, table: str, route_results: list[FileCopyResult], elapsed: float, *, delete_failures: list[tuple[str, Exception]] | None = None, partial: bool = False, replay: bool = False) -> None:
        if not route_results:
            if replay:
                self._log(f"[{route_id}] Batch already committed (SUCCESS replay) — skipped.")
            else:
                self._log(f"[{route_id}] No new files.")
            return
        loaded = sum(1 for r in route_results if r.error is None)
        failed = len(route_results) - loaded
        loaded_bytes = sum(r.event.size or 0 for r in route_results if r.error is None)
        mbs = mb_per_sec(loaded_bytes, elapsed)
        if partial:
            self._log(
                f"[{route_id}] {loaded} copied, {failed} failed \u2014 {loaded_bytes / 1_000_000:.1f} MB in {elapsed:.1f}s ({mbs:.1f} MB/s)"
                f" ({_PARTIAL_BATCH_WARNING})"
            )
        else:
            self._log(f"[{route_id}] {loaded} loaded, {failed} failed \u2014 {loaded_bytes / 1_000_000:.1f} MB in {elapsed:.1f}s ({mbs:.1f} MB/s)")
        self._summary = {
            "route_id": route_id,
            "table": table,
            "loaded": loaded,
            "failed": failed,
            "partial": partial,
            "loaded_bytes": loaded_bytes,
            "elapsed": elapsed,
            "copy_failures": [
                {"filename": r.event.filename, "error": str(r.error)}
                for r in route_results if r.error is not None
            ],
            "delete_failures": [{"path": p, "error": str(e)} for p, e in (delete_failures or [])],
        }


def print_job_metrics(summaries: list[dict], elapsed: float) -> None:
    """Print aggregate throughput across all routes."""
    total_loaded_bytes = sum(s["loaded_bytes"] for s in summaries)
    total_loaded = sum(s["loaded"] for s in summaries)
    mbs = mb_per_sec(total_loaded_bytes, elapsed)
    print(f"Job done \u2014 {total_loaded_bytes / 1_000_000:.1f} MB in {elapsed:.1f}s ({mbs:.1f} MB/s, {total_loaded} file(s) loaded)")


def print_job_summary(summaries: list[dict]) -> None:
    """Print aggregate counts and any per-file failure details, grouped by route."""
    total = sum(s["loaded"] + s["failed"] for s in summaries)
    total_loaded = sum(s["loaded"] for s in summaries)
    total_failed = sum(s["failed"] for s in summaries)
    print(f"Summary \u2014 {total} file(s): {total_loaded} loaded, {total_failed} failed")

    for s in summaries:
        copy_failures = s.get("copy_failures", [])
        delete_failures = s.get("delete_failures", [])
        partial = s.get("partial", False)
        if not copy_failures and not delete_failures and not partial:
            continue

        print(f"Route [{s['route_id']}] (table: {s['table']}): {s['failed']} failed")
        if partial:
            print(f"  WARNING: {_PARTIAL_BATCH_WARNING}")
        for f in copy_failures:
            print(f"  COPY FAILED  {f['filename']}: {f['error']}")
        for df in delete_failures:
            print(f"  DELETE FAILED  {df['path']}: {df['error']}")


def on_job_done(summaries: list[dict], elapsed: float) -> int:
    """Print job metrics and summary. Returns the total number of copy failures."""
    if not summaries:
        return 0
    print_job_metrics(summaries, elapsed)
    print_job_summary(summaries)
    return sum(s["failed"] for s in summaries)

```

###### FILE: src/azfr_databricks_landing_zone/main.py ######

```py
import argparse
from pathlib import PurePosixPath
import sys
import time
from concurrent.futures import ThreadPoolExecutor

from pyspark.sql import DataFrame, SparkSession
from pyspark.sql.types import LongType, StringType, StructField, StructType, TimestampType, BinaryType

from azfr_databricks_landing_zone.config import LandingRoute, load_config
from azfr_databricks_landing_zone.file_event import FileEventTable
from azfr_databricks_landing_zone.file_intent import FileIntentStore, IntentStatus
from azfr_databricks_landing_zone.fs_backend import FileSystemBackend, FileMetadata, create_fs_backend
from azfr_databricks_landing_zone.instrumentation import JobInstrumentation, on_job_done
from azfr_databricks_landing_zone.timing import Timer, timed_call
from azfr_databricks_landing_zone.file_copier import FileCopier, FileCopyResult


AUTOLOADER_SCHEMA = StructType([
  StructField("path", StringType(), False),
  StructField("modificationTime", TimestampType(), False),
  StructField("length", LongType(), False),
  StructField("content", BinaryType(), True),
])


def get_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(add_help=False)
    parser.add_argument("--config-path", dest="config_path", default=None)
    parser.add_argument("--route-id", dest="route_id", default=None)
    args, _ = parser.parse_known_args()
    if not args.config_path:
        raise ValueError("Missing required config path. Provide it with '--config-path <path-to-yml>'.")
    return args


def _resolve_route(config, route_id: str | None) -> LandingRoute:
    if route_id is None:
        raise ValueError("route_id is required. It should be passed from the job definition.")

    route = next((r for r in config.landing_routes if r.id == route_id), None)
    if route is None:
        raise ValueError(f"Route '{route_id}' not found in config.")
    return route


def _get_autoloader_options(route: LandingRoute) -> dict[str, str]:
    """Return Autoloader options dict based on route configuration.
    
    If allow_filename_reuse is True, enables directory listing mode and allowOverwrites
    to support senders that resend the same filename with corrections. See ADR-012.
    """
    options = {
        "cloudFiles.format": "binaryFile",
        "cloudFiles.includeExistingFiles": "true",
    }
    
    if route.allow_filename_reuse:
        options["cloudFiles.useManagedFileEvents"] = "false"
        options["cloudFiles.allowOverwrites"] = "true"
    else:
        options["cloudFiles.useManagedFileEvents"] = "true"

    if route.allow_filename_reuse_recovery:
        options["cloudFiles.listOnStart"] = "true"
        options["cloudFiles.validateOptions"] = "false"

    return options


class LandingMoverJob:
    """Orchestrates concurrent file moves (copy then delete) across all active routes."""

    def __init__(self, spark: SparkSession, fs: FileSystemBackend, shared_executor: ThreadPoolExecutor, work_base: str, instrumentation: JobInstrumentation) -> None:
        self._spark: SparkSession = spark
        self._fs: FileSystemBackend = fs
        self._shared_executor: ThreadPoolExecutor = shared_executor
        self._work_base: str = work_base
        self._instrumentation: JobInstrumentation = instrumentation

    def process_batch(self, df: DataFrame, batch_id: int, route: LandingRoute) -> list[FileCopyResult]:
        """Process a single Autoloader micro-batch for *route*.

        1. Resolve copy intents (read intent store, check audit for retry files,
           build FileCopiers, write-ahead new intents).
        2. Execute copies concurrently.
        3. Write LOADED/FAILED events to the Delta file-event table.
        4. Raise if any copy failed so Autoloader retries the batch.
        5. Delete all source files (only reached when every file is LOADED).
        6. Mark intent store as successful.
        """
        copy_elapsed = 0.0
        replay = False
        has_errors = False
        copy_results: list[FileCopyResult] = []
        delete_errors: list[tuple[str, Exception]] = []
        
        try:
            rows = df.collect()
            if not rows:
                intent_store = FileIntentStore.for_route(self._work_base, route.id)
                snapshot = intent_store.read()
                if snapshot and snapshot.status == IntentStatus.SUCCESS and snapshot.entries:
                    # ignoreMissingFiles dropped all rows because sources were already deleted
                    # after a completed batch (ADR-009 A-5/A-7). Treat as success replay.
                    replay = True
                else:
                    intent_store.mark_success([])
                return []
            files = [FileMetadata.from_autoloader_row(row) for row in rows]
            resolved = self._resolve_copiers(route, files)
            if resolved is None:
                # Success replay: intent file is already SUCCESS, nothing to do.
                replay = True
                return []
            file_copiers, mark_success_entries = resolved

            with Timer() as copy_timer:
                copy_results = self._execute_copies(route, file_copiers)
            copy_elapsed = copy_timer.elapsed

            FileEventTable(self._spark, route.file_event_table).append_batch([r.event for r in copy_results])
            has_errors = any(r.error for r in copy_results)
            if has_errors:
                # Return early — do not delete sources or mark_success.
                # BatchProcessor will raise after instrumentation so Autoloader retries the batch.
                return copy_results

            delete_errors = self._delete_sources(route, [f.path for f in files])
            for path, error in delete_errors:
                self._instrumentation.on_file_delete_failed(route.id, path, error)
            if not delete_errors:
                FileIntentStore.for_route(self._work_base, route.id).mark_success(mark_success_entries)
            if delete_errors:
                raise RuntimeError(f"[{route.id}] Failed to delete {len(delete_errors)} source file(s) — see logs for details.")

            return copy_results
        finally:
            partial = sys.exc_info()[1] is not None or has_errors
            self._instrumentation.on_route_done(
                route.id, route.file_event_table, copy_results, copy_elapsed,
                delete_failures=delete_errors,
                partial=partial, replay=replay,
            )

    def _resolve_copiers(self, route: LandingRoute, files: list[FileMetadata]) -> tuple[list[FileCopier], list[tuple[str, int, str]]] | None:
        """Build the list of FileCopier tasks for this batch, or ``None`` if already complete.

        Returns ``None`` when the intent is SUCCESS and covers the full batch — the batch
        completed but Autoloader crashed before committing the checkpoint.

        Otherwise returns ``(file_copiers, full_entries)`` where ``file_copiers`` contains
        only files that still need copying, and ``full_entries`` covers every file in the
        batch so the intent always retains the complete mapping (see ADR-009 A-6).
        """
        intent_store = FileIntentStore.for_route(self._work_base, route.id)
        snapshot = intent_store.read()
        batch_keys = {(f.path, f.modification_time) for f in files}
        overlapping_keys = batch_keys & snapshot.entries.keys() if snapshot else set()

        # Full success replay — batch already completed, waiting for Spark checkpoint commit.
        if snapshot and snapshot.status == IntentStatus.SUCCESS and overlapping_keys == batch_keys:
            return None

        # Assign a target path to every file: reuse from the snapshot when available,
        # generate a fresh opaque path otherwise.
        targets: dict[tuple[str, int], str] = {}
        for f in files:
            key = (f.path, f.modification_time)
            targets[key] = snapshot.entries[key] if (snapshot and key in snapshot.entries) else FileCopier.from_metadata(f, route.target, self._fs).target_path

        # Determine which targets are already LOADED and can be skipped.
        if snapshot and snapshot.status == IntentStatus.SUCCESS:
            # SUCCESS guarantees all overlapping files are LOADED — no Delta query needed.
            loaded_targets = {snapshot.entries[k] for k in overlapping_keys}
        elif snapshot and snapshot.status == IntentStatus.IN_PROGRESS and overlapping_keys:
            # IN_PROGRESS doesn't guarantee LOADED — query Delta to find which ones completed.
            loaded_targets = FileEventTable(self._spark, route.file_event_table).get_loaded_target_paths(
                {snapshot.entries[k] for k in overlapping_keys}
            )
        else:
            # Fresh start or IN_PROGRESS with no overlap — no prior work to skip.
            loaded_targets = set()

        # Build entries for all files, and copiers only for non-LOADED files.
        file_copiers: list[FileCopier] = []
        full_entries: list[tuple[str, int, str]] = []
        for f in files:
            key = (f.path, f.modification_time)
            target = targets[key]
            full_entries.append((f.path, f.modification_time, target))
            if target not in loaded_targets:
                file_copiers.append(FileCopier(f, f.path, target, self._fs))

        intent_store.write_in_progress(full_entries)
        return file_copiers, full_entries

    def _execute_copies(self, route: LandingRoute, file_copiers: list[FileCopier]) -> list[FileCopyResult]:
        """Submit all copies concurrently and collect results."""
        futures = {self._shared_executor.submit(timed_call(copier.run)): copier for copier in file_copiers}
        route_results: list[FileCopyResult] = []
        for future, copier in futures.items():
            elapsed, error, _ = future.result()
            if error:
                event = copier.failed_event()
                self._instrumentation.on_file_failed(route.id, copier.metadata.name, copier.target_path, elapsed, error)
                route_results.append(FileCopyResult(route.id, event, source_path=copier.source_path, error=error))
            else:
                event = copier.loaded_event()
                self._instrumentation.on_file_loaded(route.id, event.filename, event.path, copier.metadata.size, elapsed)
                route_results.append(FileCopyResult(route.id, event, source_path=copier.source_path))
        return route_results

    def _delete_sources(self, route: LandingRoute, source_paths: list[str]) -> list[tuple[str, Exception]]:
        """Delete source files concurrently. Returns any deletion errors."""
        if not source_paths:
            return []

        futures = {
            self._shared_executor.submit(timed_call(lambda path=p: self._fs.delete_file(path))): p
            for p in source_paths
        }
        errors: list[tuple[str, Exception]] = []
        for future, path in futures.items():
            _, error, _ = future.result()
            if error:
                errors.append((path, error))
        return errors


class BatchProcessor:
    """Picklable foreachBatch callable for Spark Connect compatibility.

    Spark Connect serializes the foreachBatch function over gRPC, so it must not
    capture any un-picklable objects (SparkSession, dbutils, ThreadPoolExecutor).
    This class stores only simple config values and reconstructs heavy objects
    lazily inside __call__, obtaining the SparkSession from ``df.sparkSession``
    as required by the Spark Connect API.
    """

    def __init__(self, work_base: str, file_backend: str, route: LandingRoute, max_file_workers: int, instr: JobInstrumentation, copy_failures: set[str] | None, delete_failures: set[str] | None) -> None:
        self.work_base = work_base
        self.file_backend = file_backend
        self.route = route
        self.max_file_workers = max_file_workers
        self.instr = instr
        self.copy_failures = copy_failures
        self.delete_failures = delete_failures

    def __call__(self, df: DataFrame, batch_id: int) -> None:
        spark = df.sparkSession
        spark.conf.set("spark.sql.files.ignoreMissingFiles", "true")  # see ADR-009 A-5/A-6; must be set on the cloned foreachBatch session
        fs = create_fs_backend(self.file_backend, self.copy_failures, self.delete_failures)
        try:
            with ThreadPoolExecutor(max_workers=self.max_file_workers, thread_name_prefix=f"file-worker-{self.route.id}") as executor:
                job = LandingMoverJob(spark, fs, executor, self.work_base, self.instr)
                route_results = job.process_batch(df, batch_id, self.route)

            # Raise after instrumentation so the summary is always recorded correctly.
            # Raising here causes Autoloader to not advance the checkpoint (A-4 retry path).
            if any(r.error for r in route_results):
                failures = sum(1 for r in route_results if r.error)
                raise RuntimeError(
                    f"Batch {batch_id} on route '{self.route.id}' had {failures} copy failure(s) — "
                    "Autoloader will retry. Check FAILED rows in the file-event table."
                )
        finally:
            self.instr.flush(batch_id)


def run(config_path: str, route_id: str) -> None:
    config = load_config(config_path)
    print(f"Work base:             {config.work_base}")
    print(f"Max file workers:      {config.max_file_workers}")
    print(f"IO backend: {config.file_backend}")

    route = _resolve_route(config, route_id)
    print(f"Processing route: {route.id}")

    spark = SparkSession.getActiveSession()
    if spark is None:
        raise RuntimeError("No active SparkSession found. Ensure a SparkSession is initialized before calling run().")
    route_errors: list[tuple[str, BaseException]] = []
    run_id = spark.conf.get("spark.databricks.job.runId", str(int(time.time())))
    
    def _start_and_wait(route: LandingRoute) -> tuple[tuple[str, BaseException] | None, dict | None]:
        route_work_dir = PurePosixPath(config.work_base) / route.id
        checkpoint_dir = str(route_work_dir / "checkpoint")
        log_dir = str(route_work_dir / "log")
        instr = JobInstrumentation.for_route(log_dir, run_id)
        
        autoloader_opts = _get_autoloader_options(route)
        read_stream = spark.readStream.format("cloudFiles")
        for key, value in autoloader_opts.items():
            read_stream = read_stream.option(key, value)
        
        query = (
            read_stream
            .schema(AUTOLOADER_SCHEMA)
            .load(route.source)
            .select("path", "modificationTime", "length")  # don't load file content
            .writeStream
            .trigger(availableNow=True)
            .option("checkpointLocation", checkpoint_dir)
            .foreachBatch(BatchProcessor(config.work_base, config.file_backend, route, config.max_file_workers, instr,
                                         config.fault_inject_copy_failures, config.fault_inject_delete_failures))
            .start()
        )
        result: tuple[str, BaseException] | None = None
        try:
            query.awaitTermination()
        except Exception as exc:
            result = (route.id, exc)

        instr.replay_logs(route.id)
        summary = instr.read_summary()

        return result, summary

    with Timer() as job_timer:
        summaries: list[dict] = []
        result, summary = _start_and_wait(route)
        if result is not None:
            route_errors.append(result)
        if summary is not None:
            summaries.append(summary)

    total_copy_failures = on_job_done(summaries, job_timer.elapsed)
    if route_errors:
        msg = f"Landing Mover completed with {len(route_errors)} route failure(s)."
        if total_copy_failures > 0:
            msg += " Review FAILED rows in the _file_event_ tables."
        raise BaseExceptionGroup(msg, [error for _, error in route_errors])


def main():
    args = get_args()
    run(args.config_path, args.route_id)


if __name__ == "__main__":
    main()

```

###### FILE: src/azfr_databricks_landing_zone/timing.py ######

```py
from time import perf_counter
from typing import Callable, TypeVar
from dataclasses import dataclass, field

T = TypeVar("T")


@dataclass
class Timer:
    """Context manager that measures elapsed wall-clock time for a block of code.

    Exceptions propagate normally and the block result is returned directly — no exception wrapping.
    For timing a callable where exceptions must be captured as values, use `timed_call()` instead.
    """

    elapsed: float = field(default=0.0, init=False)
    _start: float = field(default=0.0, init=False, repr=False)

    def __enter__(self) -> "Timer":
        self._start = perf_counter()
        return self

    def __exit__(self, *_: object) -> None:
        self.elapsed = perf_counter() - self._start


def timed_call(fn: Callable[[], T]) -> Callable[[], tuple[float, Exception | None, T | None]]:
    """Decorator that measures elapsed wall-clock time for a callable.

    Exceptions are captured as values and returned as (elapsed_seconds, exception_or_None, result_or_None) —
    the future always resolves cleanly without raising.
    For timing a block of code where exceptions should propagate normally, use `Timer` instead.
    """
    def wrapper() -> tuple[float, Exception | None, T | None]:
        t0 = perf_counter()
        try:
            result = fn()
            return perf_counter() - t0, None, result
        except Exception as exc:
            return perf_counter() - t0, exc, None
    return wrapper

```

###### FILE: tests/integration/test_process_batch.py ######

```py
"""Tests for LandingMoverJob.process_batch — ADR-009 full coverage.

Combines two complementary test strategies:

1. **P0 / single-run phase-map tests** — drive one process_batch call and
   verify the intent, FS, and Delta state left behind at each exit point.

2. **Two-run integration tests** — Run 1 leaves a specific state; Run 2 is
   called on the same job and must recover correctly.  Uses InMemoryEventTable
   so Delta state (LOADED / FAILED events) carries across calls.

ADR-009 case → test class mapping:

  P0 branch (empty rows)
  ──────────────────────────────────────────────────────────
  TestP0NoIntent              no prior intent → mark_success([])
  TestP0InProgress (A-6)      IN_PROGRESS + empty rows → mark_success([])
  TestP0SuccessReplay (A-7)   SUCCESS + entries + empty rows → replay, no mutation
  TestP0SuccessEmpty          SUCCESS + no entries → mark_success([]) (not a replay)

  Full pipeline — single run
  ──────────────────────────────────────────────────────────
  TestHappyPath (A-1)         P1→P6 nominal: all files copied, intent=SUCCESS
  TestResolveCopierNoneReturn SUCCESS full-overlap reached via rows path

  Two-run retry scenarios (InMemoryEventTable carries Delta state across runs)
  ──────────────────────────────────────────────────────────
  TestRetryAfterP1OrP3Crash (A-2/A-3)   IN_PROGRESS + Delta empty → reuse targets
  TestTwoRunCopyFailureThenSuccess (A-4) partial failure → Run 2 copies only FAILED
  TestRetryWhenAllLoaded (A-5)           all LOADED in Delta → no copies, delete+success
  TestTwoRunSuccessReplay (A-7 rows)     SUCCESS full-overlap via rows → no side effects
  TestNewBatchNoOverlap (B-1)            new files after committed batch
  TestNewBatchPartialOverlap (B-2)       partial re-delivery alongside new files

  Single-run failure paths
  ──────────────────────────────────────────────────────────
  TestCopyFailure (A-4 P4)    has_errors → early return, intent IN_PROGRESS, no delete
  TestDeleteFailure (A-5 P5)  _delete_sources raises, intent IN_PROGRESS, P3 already ran
"""

import datetime
import pytest
from concurrent.futures import ThreadPoolExecutor
from unittest.mock import MagicMock, patch

from azfr_databricks_landing_zone.config import LandingRoute
from azfr_databricks_landing_zone.file_event import FileState
from azfr_databricks_landing_zone.file_intent import FileIntentStore, IntentStatus
from azfr_databricks_landing_zone.fs_backend import FaultInjectingBackend
from azfr_databricks_landing_zone.instrumentation import JobInstrumentation
from azfr_databricks_landing_zone.main import LandingMoverJob


# ---------------------------------------------------------------------------
# Shared constants
# ---------------------------------------------------------------------------

ROUTE = LandingRoute(
    id="route-pb",
    source="/Volumes/cat/sch/vol/source",
    target="/Volumes/cat/sch/vol/target",
)

MOD_A = 1_700_000_000_000
MOD_B = 1_700_000_001_000
MOD_C = 1_700_000_002_000

PATH_A = "/Volumes/cat/sch/vol/source/a.csv"
PATH_B = "/Volumes/cat/sch/vol/source/b.csv"
PATH_C = "/Volumes/cat/sch/vol/source/c.csv"

# Pre-committed target paths — used only when tests pre-seed the intent store.
TARGET_A = "/Volumes/cat/sch/vol/target/2023/01/aaaa/a.csv"
TARGET_B = "/Volumes/cat/sch/vol/target/2023/01/bbbb/b.csv"

ENTRY_A = (PATH_A, MOD_A, TARGET_A)
ENTRY_B = (PATH_B, MOD_B, TARGET_B)


# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------

class InMemoryEventTable:
    """In-memory substitute for the Delta-backed FileEventTable.

    Tracks LOADED target paths so get_loaded_target_paths() returns the
    correct subset across multiple process_batch calls in the same test.
    """

    def __init__(self):
        self._loaded: set[str] = set()
        self.append_calls: list = []

    def append_batch(self, events: list) -> None:
        self.append_calls.append(list(events))
        for event in events:
            if event.state == FileState.LOADED:
                self._loaded.add(event.path)

    def get_loaded_target_paths(self, candidate_paths: set[str]) -> set[str]:
        return self._loaded & candidate_paths


def make_row(path: str, modtime_ms: int, size: int = 100):
    row = MagicMock()
    row.path = path
    row.modificationTime = datetime.datetime.fromtimestamp(
        modtime_ms / 1000, tz=datetime.timezone.utc
    )
    row.length = size
    return row


def make_df(*rows):
    df = MagicMock()
    df.collect.return_value = list(rows)
    return df


def make_job(tmp_path, fs=None) -> LandingMoverJob:
    return LandingMoverJob(
        spark=MagicMock(),
        fs=fs or MagicMock(),
        shared_executor=ThreadPoolExecutor(max_workers=2),
        work_base=str(tmp_path),
        instrumentation=JobInstrumentation(log_dir=str(tmp_path), run_id="pb-test"),
    )


def make_store(tmp_path) -> FileIntentStore:
    return FileIntentStore.for_route(str(tmp_path), ROUTE.id)


# ===========================================================================
# P0 — empty-rows branch
# ===========================================================================

class TestP0NoIntent:
    """df.collect() returns [] and there is no prior intent file."""

    def test_returns_empty_list(self, tmp_path):
        result = make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        assert result == []

    def test_writes_success_intent_with_empty_entries(self, tmp_path):
        make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        snapshot = make_store(tmp_path).read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.SUCCESS
        assert snapshot.entries == {}

    def test_no_copy_or_delete_performed(self, tmp_path):
        fs = MagicMock()
        make_job(tmp_path, fs=fs).process_batch(make_df(), batch_id=1, route=ROUTE)
        fs.copy_file.assert_not_called()
        fs.delete_file.assert_not_called()


# ---------------------------------------------------------------------------
# A-6: empty rows + IN_PROGRESS intent
# ---------------------------------------------------------------------------

class TestP0InProgress:
    """A-6: all sources already deleted; intent is IN_PROGRESS; df.collect() returns [].

    Run 2 must call mark_success([]) and return [] without touching Delta or the FS.
    """

    def test_returns_empty_list(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        result = make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        assert result == []

    def test_intent_becomes_success_with_empty_entries(self, tmp_path):
        """mark_success([]) must overwrite the IN_PROGRESS intent."""
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        snapshot = make_store(tmp_path).read()
        assert snapshot.status == IntentStatus.SUCCESS
        assert snapshot.entries == {}

    def test_no_delta_interaction(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_cls:
            make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        mock_cls.assert_not_called()

    def test_no_copy_or_delete_performed(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        fs = MagicMock()
        make_job(tmp_path, fs=fs).process_batch(make_df(), batch_id=1, route=ROUTE)
        fs.copy_file.assert_not_called()
        fs.delete_file.assert_not_called()


# ---------------------------------------------------------------------------
# A-7: empty rows + SUCCESS intent with entries (success replay)
# ---------------------------------------------------------------------------

class TestP0SuccessReplay:
    """A-7: mark_success already ran; Spark has not committed the checkpoint yet.

    df.collect() returns [] (sources deleted); Run N must return [] without
    touching the intent, Delta, or the file system.
    """

    def test_returns_empty_list(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        result = make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        assert result == []

    def test_intent_not_overwritten(self, tmp_path):
        """The SUCCESS intent must survive the replay unchanged."""
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        snapshot = make_store(tmp_path).read()
        assert snapshot.status == IntentStatus.SUCCESS
        assert len(snapshot.entries) == 2

    def test_no_delta_interaction(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_cls:
            make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        mock_cls.assert_not_called()

    def test_no_copy_or_delete_performed(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        fs = MagicMock()
        make_job(tmp_path, fs=fs).process_batch(make_df(), batch_id=1, route=ROUTE)
        fs.copy_file.assert_not_called()
        fs.delete_file.assert_not_called()

    def test_unlimited_replays_are_safe(self, tmp_path):
        """Each replay must leave intent intact so the next replay also works."""
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        job = make_job(tmp_path)
        for _ in range(3):
            assert job.process_batch(make_df(), batch_id=1, route=ROUTE) == []
        snapshot = make_store(tmp_path).read()
        assert snapshot.status == IntentStatus.SUCCESS
        assert len(snapshot.entries) == 2


# ---------------------------------------------------------------------------
# P0 edge case: SUCCESS with empty entries is not a replay
# ---------------------------------------------------------------------------

class TestP0SuccessEmpty:
    """SUCCESS intent with an empty entry list must NOT be treated as a replay.

    The overlapping_keys == batch_keys check requires actual entries to match,
    so an empty SUCCESS never triggers the replay path (ADR-009 Consequences).
    """

    def test_returns_empty_list(self, tmp_path):
        make_store(tmp_path).mark_success([])
        result = make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        assert result == []

    def test_writes_success_intent_again(self, tmp_path):
        """Falls into the else branch and calls mark_success([]) again — safe no-op."""
        make_store(tmp_path).mark_success([])
        make_job(tmp_path).process_batch(make_df(), batch_id=1, route=ROUTE)
        snapshot = make_store(tmp_path).read()
        assert snapshot.status == IntentStatus.SUCCESS
        assert snapshot.entries == {}


# ===========================================================================
# Full pipeline — single run
# ===========================================================================

class TestHappyPath:
    """A-1: no intent → P1→P6 nominal run: copies, Delta append, delete, mark_success."""

    def test_returns_results_for_all_files(self, tmp_path):
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results = make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert len(results) == 2

    def test_results_have_no_errors(self, tmp_path):
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results = make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert all(r.error is None for r in results)

    def test_all_events_are_loaded(self, tmp_path):
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results = make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert all(r.event.state == FileState.LOADED for r in results)

    def test_intent_is_success_after_pipeline(self, tmp_path):
        """P6 must have run: intent is SUCCESS."""
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert make_store(tmp_path).read().status == IntentStatus.SUCCESS

    def test_intent_entries_cover_all_source_paths(self, tmp_path):
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        snapshot = make_store(tmp_path).read()
        assert {k[0] for k in snapshot.entries} == {PATH_A, PATH_B}

    def test_sources_deleted(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert fs.delete_file.call_count == 2

    def test_delta_append_called_with_loaded_events(self, tmp_path):
        """P3: one append_batch call covering all LOADED events."""
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert len(event_table.append_calls) == 1
        assert len(event_table.append_calls[0]) == 2
        assert all(e.state == FileState.LOADED for e in event_table.append_calls[0])

    def test_target_paths_start_with_route_target(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        for call in fs.copy_file.call_args_list:
            _, target = call.args
            assert target.startswith(ROUTE.target)


class TestResolveCopierNoneReturn:
    """process_batch handles None from _resolve_copiers: returns [] without side effects.

    In production this path is pre-empted by P0 (A-7: empty rows after SUCCESS).
    Still reachable when df delivers rows for a batch already committed (SUCCESS +
    full key overlap) — e.g. Spark Connect re-sends the same batch over gRPC.
    """

    def test_returns_empty_list(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            result = make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert result == []

    def test_intent_not_overwritten(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        snapshot = make_store(tmp_path).read()
        assert snapshot.status == IntentStatus.SUCCESS
        assert len(snapshot.entries) == 2

    def test_no_copy_or_delete_performed(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        fs = MagicMock()
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        fs.copy_file.assert_not_called()
        fs.delete_file.assert_not_called()


# ===========================================================================
# Two-run integration tests (InMemoryEventTable carries Delta state)
# ===========================================================================

class TestRetryAfterP1OrP3Crash:
    """A-2 / A-3: IN_PROGRESS intent with Delta empty — Run 2 copies to original target paths.

    A-2: crash after P1 (intent written, no copies started).
    A-3: crash during P2 or failed-atomic Delta write in P3.
    Both leave IN_PROGRESS intent and Delta empty — identical Run 2 behaviour.
    """

    def test_original_target_paths_reused(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        fs = MagicMock()
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        targets_used = {c.args[1] for c in fs.copy_file.call_args_list}
        assert targets_used == {TARGET_A, TARGET_B}

    def test_intent_ends_as_success(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert make_store(tmp_path).read().status == IntentStatus.SUCCESS


class TestTwoRunCopyFailureThenSuccess:
    """A-4: Run 1 fails for file B; Run 2 retries only B and completes successfully."""

    def test_run2_only_copies_the_failed_file(self, tmp_path):
        event_table = InMemoryEventTable()
        inner_fs = MagicMock()
        fs = FaultInjectingBackend(inner_fs, copy_failures={"b.csv"}, delete_failures=None)
        job = make_job(tmp_path, fs=fs)
        rows = make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B))

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results_1 = job.process_batch(rows, 0, ROUTE)
            assert any(r.error for r in results_1)
            assert make_store(tmp_path).read().status == IntentStatus.IN_PROGRESS

            fs._copy_failures.clear()
            job.process_batch(rows, 0, ROUTE)

        # Run 1: A forwarded to inner (B rejected before reaching inner).
        # Run 2: only B forwarded to inner (A already LOADED, skipped).
        assert inner_fs.copy_file.call_count == 2

    def test_intent_is_success_after_run2(self, tmp_path):
        event_table = InMemoryEventTable()
        inner_fs = MagicMock()
        fs = FaultInjectingBackend(inner_fs, copy_failures={"b.csv"}, delete_failures=None)
        job = make_job(tmp_path, fs=fs)
        rows = make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B))

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(rows, 0, ROUTE)
            fs._copy_failures.clear()
            job.process_batch(rows, 0, ROUTE)

        assert make_store(tmp_path).read().status == IntentStatus.SUCCESS


class TestRetryWhenAllLoaded:
    """A-5: all files are LOADED in Delta — Run 2 skips copies, deletes sources, marks success."""

    def test_no_copies_when_all_loaded(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        event_table._loaded.update({TARGET_A, TARGET_B})
        fs = MagicMock()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        fs.copy_file.assert_not_called()

    def test_sources_deleted_even_when_all_loaded(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        event_table._loaded.update({TARGET_A, TARGET_B})
        fs = MagicMock()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert fs.delete_file.call_count == 2

    def test_intent_ends_as_success(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        event_table._loaded.update({TARGET_A, TARGET_B})
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert make_store(tmp_path).read().status == IntentStatus.SUCCESS

    def test_no_extra_events_written(self, tmp_path):
        """append_batch is called with an empty list when file_copiers=[] — no new events."""
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        event_table._loaded.update({TARGET_A, TARGET_B})
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path).process_batch(
                make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE
            )
        assert all(len(batch) == 0 for batch in event_table.append_calls)


class TestTwoRunSuccessReplay:
    """A-7 (rows path): Run 1 completes cleanly; Run 2 receives the same rows
    (checkpoint not yet committed) and must produce no additional side effects.

    This exercises the _resolve_copiers None-return path (SUCCESS + full key
    overlap) rather than the P0 empty-rows branch tested by TestP0SuccessReplay.
    """

    def test_run2_produces_no_additional_copies_or_deletes(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        job = make_job(tmp_path, fs=fs)
        rows = make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B))

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(rows, 0, ROUTE)
            copies_r1 = fs.copy_file.call_count
            deletes_r1 = fs.delete_file.call_count
            events_r1 = len(event_table.append_calls)

            job.process_batch(rows, 0, ROUTE)

        assert fs.copy_file.call_count == copies_r1
        assert fs.delete_file.call_count == deletes_r1
        assert len(event_table.append_calls) == events_r1


class TestNewBatchNoOverlap:
    """B-1: Run 1 completes. Run 2 delivers entirely new files — treated as a fresh start."""

    def test_all_new_files_are_copied(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        job = make_job(tmp_path, fs=fs)

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE)
            fs.copy_file.reset_mock()
            job.process_batch(make_df(make_row(PATH_C, MOD_C)), 1, ROUTE)

        assert fs.copy_file.call_count == 1
        assert "c.csv" in fs.copy_file.call_args.args[0]

    def test_intent_ends_as_success(self, tmp_path):
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job = make_job(tmp_path)
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE)
            job.process_batch(make_df(make_row(PATH_C, MOD_C)), 1, ROUTE)
        assert make_store(tmp_path).read().status == IntentStatus.SUCCESS


class TestNewBatchPartialOverlap:
    """B-2: Run 1 completes. Run 2 re-delivers file A alongside new file C.
    Only C must be copied; A must reuse its original target path; delete_file
    is called for all paths (A is already gone — idempotency required).
    """

    def test_only_new_file_copied(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        job = make_job(tmp_path, fs=fs)

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE)
            fs.copy_file.reset_mock()
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_C, MOD_C)), 1, ROUTE)

        assert fs.copy_file.call_count == 1
        assert "c.csv" in fs.copy_file.call_args.args[0]

    def test_overlapping_file_reuses_original_target(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        job = make_job(tmp_path, fs=fs)

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE)
            target_a_run1 = next(
                args[1] for args, _ in fs.copy_file.call_args_list if "a.csv" in args[0]
            )
            fs.copy_file.reset_mock()
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_C, MOD_C)), 1, ROUTE)

        run2_sources = [args[0] for args, _ in fs.copy_file.call_args_list]
        assert not any("a.csv" in s for s in run2_sources)
        assert make_store(tmp_path).read().entries.get((PATH_A, MOD_A)) == target_a_run1

    def test_delete_called_for_all_sources_including_already_gone(self, tmp_path):
        event_table = InMemoryEventTable()
        fs = MagicMock()
        job = make_job(tmp_path, fs=fs)

        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_B, MOD_B)), 0, ROUTE)
            fs.delete_file.reset_mock()
            job.process_batch(make_df(make_row(PATH_A, MOD_A), make_row(PATH_C, MOD_C)), 1, ROUTE)

        deleted = {c.args[0] for c in fs.delete_file.call_args_list}
        assert PATH_A in deleted  # already deleted in Run 1 — idempotent no-op
        assert PATH_C in deleted


# ===========================================================================
# Single-run failure paths
# ===========================================================================

class TestCopyFailure:
    """A-4 P4: has_errors → early return before P5/P6.

    Intent must stay IN_PROGRESS so Autoloader can retry.
    Delta FAILED events must be appended (P3 ran before P4 check).
    """

    def test_returns_results_containing_error(self, tmp_path):
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results = make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
            )
        assert len(results) == 1
        assert results[0].error is not None

    def test_failed_event_state(self, tmp_path):
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            results = make_job(tmp_path, fs=fs).process_batch(
                make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
            )
        assert results[0].event.state == FileState.FAILED

    def test_intent_stays_in_progress(self, tmp_path):
        """P6 must not run: intent stays IN_PROGRESS."""
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(make_df(make_row(PATH_A, MOD_A)), 0, ROUTE)
        assert make_store(tmp_path).read().status == IntentStatus.IN_PROGRESS

    def test_sources_not_deleted(self, tmp_path):
        """P5 must not run when copies failed."""
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(make_df(make_row(PATH_A, MOD_A)), 0, ROUTE)
        fs.delete_file.assert_not_called()

    def test_delta_append_called_with_failed_events(self, tmp_path):
        """P3 ran before P4: FAILED events appended to Delta."""
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            make_job(tmp_path, fs=fs).process_batch(make_df(make_row(PATH_A, MOD_A)), 0, ROUTE)
        assert len(event_table.append_calls) == 1
        assert all(e.state == FileState.FAILED for e in event_table.append_calls[0])


class TestDeleteFailure:
    """A-5 P5: _delete_sources raises RuntimeError → process_batch propagates it.

    P3 (Delta append) must have completed before P5 ran.
    Intent stays IN_PROGRESS because P6 (mark_success) was never reached.
    """

    def test_raises_runtime_error(self, tmp_path):
        fs = MagicMock()
        fs.delete_file.side_effect = OSError("permission denied")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            with pytest.raises(RuntimeError, match="Failed to delete"):
                make_job(tmp_path, fs=fs).process_batch(
                    make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
                )

    def test_intent_stays_in_progress(self, tmp_path):
        """P6 must not run when delete raises."""
        fs = MagicMock()
        fs.delete_file.side_effect = OSError("permission denied")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            try:
                make_job(tmp_path, fs=fs).process_batch(
                    make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
                )
            except RuntimeError:
                pass
        assert make_store(tmp_path).read().status == IntentStatus.IN_PROGRESS

    def test_delta_was_appended_before_delete(self, tmp_path):
        """P3 ran before P5: LOADED events were appended even though delete failed."""
        fs = MagicMock()
        fs.delete_file.side_effect = OSError("permission denied")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            try:
                make_job(tmp_path, fs=fs).process_batch(
                    make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
                )
            except RuntimeError:
                pass
        assert len(event_table.append_calls) == 1
        assert all(e.state == FileState.LOADED for e in event_table.append_calls[0])

    def test_delete_was_attempted(self, tmp_path):
        """Verify P5 ran (delete_file was called) before RuntimeError propagated."""
        fs = MagicMock()
        fs.delete_file.side_effect = OSError("permission denied")
        event_table = InMemoryEventTable()
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            try:
                make_job(tmp_path, fs=fs).process_batch(
                    make_df(make_row(PATH_A, MOD_A)), 0, ROUTE
                )
            except RuntimeError:
                pass
        fs.delete_file.assert_called()

    def test_partial_delete_survivors_skipped_on_retry(self, tmp_path):
        """A-5 retry: surviving source already LOADED → _resolve_copiers returns file_copiers=[].

        Simulates: Run 1 copied A + B (both LOADED in Delta), deleted A, then
        crashed on B's delete.  Run 2 receives only B (ignoreMissingFiles dropped A).
        _resolve_copiers must return file_copiers=[] for B (already LOADED).
        """
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        event_table = InMemoryEventTable()
        event_table._loaded.add(TARGET_B)

        from azfr_databricks_landing_zone.fs_backend import FileMetadata
        job = make_job(tmp_path)
        surviving_files = [
            FileMetadata(path=PATH_B, name="b.csv", size=200, modification_time=MOD_B, creation_time=None),
        ]
        with patch("azfr_databricks_landing_zone.main.FileEventTable", return_value=event_table):
            file_copiers, full_entries = job._resolve_copiers(ROUTE, surviving_files)

        assert file_copiers == []
        assert len(full_entries) == 1
        assert full_entries[0][0] == PATH_B

```

###### FILE: tests/unit/conftest.py ######

```py
"""
Stub out the Databricks SDK runtime before any test imports.

The SDK's runtime/__init__.py instantiates RemoteDbUtils() at module level,
which requires Databricks credentials. Since unit tests run locally without
a cluster, we replace the entire runtime module tree with mocks.
"""
import sys
from types import ModuleType
from unittest.mock import MagicMock


def _stub_databricks_runtime() -> None:
    # Only stub if the real module hasn't already been successfully imported
    if "databricks.sdk.runtime" in sys.modules:
        return

    # Build a minimal FileInfo stub that matches the fields used in production code
    class FileInfo:
        def __init__(self, path: str, name: str, size: int, modificationTime: int):
            self.path = path
            self.name = name
            self.size = size
            self.modificationTime = modificationTime

    dbutils_stub = ModuleType("databricks.sdk.runtime.dbutils_stub")
    dbutils_stub.FileInfo = FileInfo  # type: ignore[attr-defined]
    dbutils_stub.dbutils = MagicMock()  # type: ignore[attr-defined]

    runtime = ModuleType("databricks.sdk.runtime")
    runtime.dbutils = MagicMock()  # type: ignore[attr-defined]

    sys.modules["databricks.sdk.runtime"] = runtime
    sys.modules["databricks.sdk.runtime.dbutils_stub"] = dbutils_stub


_stub_databricks_runtime()

```

###### FILE: tests/unit/test_config.py ######

```py
import pytest
from pydantic import ValidationError

from azfr_databricks_landing_zone.config import AppConfig, LandingRoute, load_config


MINIMAL_CONFIG = {
    "id": "data-source-A",
    "source": "/Volumes/azfr_dev_raw/landing/landing_volume/data-source-A/",
    "target": "/Volumes/azfr_dev_raw/my_schema/raw_files/data-source-A/",
}


class TestLandingRoute:
    def test_file_event_table(self):
        config = LandingRoute(**{**MINIMAL_CONFIG, "target": "/Volumes/dev/my_schema/raw_files/data-source-A/"})
        assert config.file_event_table == "dev.my_schema._file_event_"

    def test_allow_filename_reuse_defaults_to_false(self):
        config = LandingRoute(**MINIMAL_CONFIG)
        assert config.allow_filename_reuse is False

    def test_allow_filename_reuse_can_be_set_true(self):
        config = LandingRoute(**{**MINIMAL_CONFIG, "allow_filename_reuse": True})
        assert config.allow_filename_reuse is True

    def test_allow_filename_reuse_recovery_defaults_to_false(self):
        config = LandingRoute(**MINIMAL_CONFIG)
        assert config.allow_filename_reuse_recovery is False

    def test_allow_filename_reuse_recovery_can_be_set_true(self):
        config = LandingRoute(**{**MINIMAL_CONFIG, "allow_filename_reuse_recovery": True})
        assert config.allow_filename_reuse_recovery is True

    def test_raises_if_non_volume_target(self):
        with pytest.raises(ValidationError, match="catalog and schema could not be derived"):
            LandingRoute(id="data-source-A", source="/data/source", target="/data/target")


class TestAppConfig:
    def test_duplicate_id_raises(self):
        config = LandingRoute(**MINIMAL_CONFIG)
        duplicate = LandingRoute(**{**MINIMAL_CONFIG, "source": "/Volumes/azfr_dev_raw/landing/landing_volume/other"})
        with pytest.raises(ValidationError, match="Duplicate ids"):
            AppConfig(max_file_workers=10, file_backend="dbutils", work_base="/Volumes/x/y/work/", landing_routes=[config, duplicate])


class TestLoadConfig:
    def test_load_config(self, tmp_path):
        config_file = tmp_path / "config.yml"
        config_file.write_text(
            """
max_file_workers: 10
file_backend: dbutils
work_base: /Volumes/catalog/schema/work/
landing_routes:
  - id: data-source-A
    source: /Volumes/azfr_dev_raw/landing/landing_volume/data-source-A
    target: /Volumes/azfr_dev_raw/my_schema/raw_files/data-source-A/
""",
            encoding="utf-8",
        )
        app_config = load_config(str(config_file))
        assert app_config.landing_routes[0].id == "data-source-A"
        assert app_config.work_base == "/Volumes/catalog/schema/work/"

    def test_work_base_is_required(self, tmp_path):
        config_file = tmp_path / "config.yml"
        config_file.write_text(
            """
max_file_workers: 10
file_backend: dbutils
work_base: /Volumes/catalog/schema/work/
landing_routes:
  - id: data-source-A
    source: /Volumes/azfr_dev_raw/landing/landing_volume/data-source-A
    target: /Volumes/azfr_dev_raw/my_schema/raw_files/data-source-A/
""",
            encoding="utf-8",
        )
        app_config = load_config(str(config_file))
        assert app_config.work_base == "/Volumes/catalog/schema/work/"

```

###### FILE: tests/unit/test_file_copier.py ######

```py
from unittest.mock import MagicMock, call

import pytest

from azfr_databricks_landing_zone.file_event import FileState
from azfr_databricks_landing_zone.file_copier import FileCopier, FileCopyResult
from azfr_databricks_landing_zone.fs_backend import FileMetadata


def make_metadata(path: str = "/Volumes/cat/sch/vol/src/file.csv", size: int = 1024) -> FileMetadata:
    return FileMetadata(
        path=path,
        name="file.csv",
        size=size,
        modification_time=1_700_000_000_000,
        creation_time=None,
    )


TARGET_BASE = "/Volumes/cat/sch/vol/target"


class TestFileCopierRun:
    def test_calls_copy_file_not_delete(self):
        fs = MagicMock()
        copier = FileCopier(
            metadata=make_metadata(),
            source_path="/Volumes/cat/sch/vol/src/file.csv",
            target_path="/Volumes/cat/sch/vol/target/year=2024/month=01/day=01/abc/file.csv",
            fs=fs,
        )
        copier.run()

        fs.copy_file.assert_called_once_with(copier.source_path, copier.target_path)
        fs.delete_file.assert_not_called()

    def test_run_raises_on_copy_failure(self):
        fs = MagicMock()
        fs.copy_file.side_effect = OSError("disk full")
        copier = FileCopier(
            metadata=make_metadata(),
            source_path="/Volumes/cat/sch/vol/src/file.csv",
            target_path="/Volumes/cat/sch/vol/target/year=2024/month=01/day=01/abc/file.csv",
            fs=fs,
        )
        with pytest.raises(OSError, match="disk full"):
            copier.run()


class TestFileCopierFromMetadata:
    def test_source_path_matches_metadata(self):
        fs = MagicMock()
        metadata = make_metadata(path="/Volumes/cat/sch/vol/src/file.csv")
        copier = FileCopier.from_metadata(metadata, TARGET_BASE, fs)
        assert copier.source_path == "/Volumes/cat/sch/vol/src/file.csv"

    def test_target_path_starts_with_base(self):
        fs = MagicMock()
        copier = FileCopier.from_metadata(make_metadata(), TARGET_BASE, fs)
        assert copier.target_path.startswith(TARGET_BASE)

    def test_target_path_ends_with_filename(self):
        fs = MagicMock()
        copier = FileCopier.from_metadata(make_metadata(), TARGET_BASE, fs)
        assert copier.target_path.endswith("/file.csv")


class TestFileCopierEvents:
    def _make_copier(self) -> FileCopier:
        fs = MagicMock()
        return FileCopier(
            metadata=make_metadata(),
            source_path="/Volumes/cat/sch/vol/src/file.csv",
            target_path="/Volumes/cat/sch/vol/target/year=2024/month=01/day=01/abc/file.csv",
            fs=fs,
        )

    def test_loaded_event_state(self):
        assert self._make_copier().loaded_event().state == FileState.LOADED

    def test_failed_event_state(self):
        assert self._make_copier().failed_event().state == FileState.FAILED

    def test_loaded_event_path_is_target(self):
        copier = self._make_copier()
        assert copier.loaded_event().path == copier.target_path

    def test_failed_event_path_is_target(self):
        copier = self._make_copier()
        assert copier.failed_event().path == copier.target_path


class TestFileCopyResult:
    def _make_event(self):
        from azfr_databricks_landing_zone.file_event import FileEvent
        from datetime import datetime, timezone
        return FileEvent.loaded(
            datetime(2024, 1, 1, tzinfo=timezone.utc),
            "/Volumes/cat/sch/vol/target/file.csv",
            make_metadata(),
        )

    def test_has_source_path(self):
        event = self._make_event()
        result = FileCopyResult(
            route_id="route-a",
            event=event,
            source_path="/Volumes/cat/sch/vol/src/file.csv",
        )
        assert result.source_path == "/Volumes/cat/sch/vol/src/file.csv"

    def test_defaults_no_error(self):
        result = FileCopyResult(
            route_id="route-a",
            event=self._make_event(),
            source_path="/src/file.csv",
        )
        assert result.error is None

    def test_error_stored(self):
        err = ValueError("oops")
        result = FileCopyResult(
            route_id="route-a",
            event=self._make_event(),
            source_path="/src/file.csv",
            error=err,
        )
        assert result.error is err


class TestFsBackendSplit:
    """Verify copy_file and delete_file are separate operations on NativeOsBackend."""

    def test_copy_file_does_not_delete(self, tmp_path):
        import os
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend

        src = tmp_path / "src" / "file.txt"
        src.parent.mkdir()
        src.write_text("hello")
        dst = tmp_path / "dst" / "file.txt"

        backend = NativeOsBackend()
        backend.copy_file(str(src), str(dst))

        assert os.path.exists(str(dst)), "destination file should exist after copy"
        assert os.path.exists(str(src)), "copy_file must not delete the source"

    def test_delete_file_removes_it(self, tmp_path):
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend

        src = tmp_path / "file.txt"
        src.write_text("hello")

        backend = NativeOsBackend()
        backend.delete_file(str(src))

        assert not src.exists()

    def test_copy_then_delete_equivalent_to_old_move(self, tmp_path):
        import os
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend

        src = tmp_path / "src" / "file.txt"
        src.parent.mkdir()
        src.write_text("data")
        dst = tmp_path / "dst" / "file.txt"

        backend = NativeOsBackend()
        backend.copy_file(str(src), str(dst))
        backend.delete_file(str(src))

        assert os.path.exists(str(dst)), "destination file should exist after copy"
        assert not os.path.exists(str(src)), "source should be gone after delete"

```

###### FILE: tests/unit/test_file_event.py ######

```py
from datetime import datetime, timezone
from unittest.mock import MagicMock

import pytest

from azfr_databricks_landing_zone.file_event import FileEvent, FileEventTable, FileState, ms_epoch_to_utc
from azfr_databricks_landing_zone.fs_backend import FileMetadata


def make_file_info(size: int = 1024, modification_time: int = 1_700_000_000_000):
    return FileMetadata(
        path="dbfs:/Volumes/azfr_dev_work/test_schema/landing/source-A/data_file.csv",
        name="data_file.csv",
        size=size,
        modification_time=modification_time,
        creation_time=None,
    )


NOW = datetime(2024, 6, 1, 12, 0, 0, tzinfo=timezone.utc)
PATH = "/Volumes/catalog/schema/raw_files/year=2024/month=06/day=01/file.csv"


class TestMsEpochToUtc:
    def test_none_returns_none(self):
        assert ms_epoch_to_utc(None) is None

    def test_zero_returns_epoch(self):
        result = ms_epoch_to_utc(0)
        assert result == datetime(1970, 1, 1, 0, 0, 0, tzinfo=timezone.utc)

    def test_known_value(self):
        result = ms_epoch_to_utc(1_700_000_000_000)
        assert result == datetime.fromtimestamp(1_700_000_000, tz=timezone.utc)

    def test_result_is_utc_aware(self):
        result = ms_epoch_to_utc(1_000_000_000_000)
        assert result.tzinfo == timezone.utc


class TestFileEventLoaded:
    def test_state_is_loaded(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        assert event.state == FileState.LOADED

    def test_path_is_set(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        assert event.path == PATH

    def test_event_time_is_set(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        assert event.event_time == NOW

    def test_event_id_is_non_empty_string(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        assert isinstance(event.event_id, str)
        assert len(event.event_id) > 0

    def test_event_ids_are_unique(self):
        e1 = FileEvent.loaded(NOW, PATH, make_file_info())
        e2 = FileEvent.loaded(NOW, PATH, make_file_info())
        assert e1.event_id != e2.event_id

    def test_size_from_file_info(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info(size=4096))
        assert event.size == 4096

    def test_modification_time_from_file_info(self):
        event = FileEvent.loaded(NOW, PATH, make_file_info(modification_time=1_700_000_000_000))
        assert event.modification_time == ms_epoch_to_utc(1_700_000_000_000)



class TestFileEventFailed:
    def test_state_is_failed(self):
        event = FileEvent.failed(NOW, PATH, make_file_info())
        assert event.state == FileState.FAILED

    def test_path_is_set(self):
        event = FileEvent.failed(NOW, PATH, make_file_info())
        assert event.path == PATH


class TestFileEventToRow:
    def setup_method(self):
        self.event = FileEvent.loaded(NOW, PATH, make_file_info(size=512))
        self.row = self.event.to_row()

    def test_all_fields_present(self):
        assert set(self.row.asDict().keys()) == {
            "event_id", "event_time", "path", "state", "size",
            "creation_time", "modification_time",
        }

    def test_state_is_string_not_enum(self):
        assert self.row.state == "LOADED"
        assert isinstance(self.row.state, str)

    def test_path_matches(self):
        assert self.row.path == PATH

    def test_size_matches(self):
        assert self.row.size == 512

    def test_event_id_matches(self):
        assert self.row.event_id == self.event.event_id


class TestFileEventTable:
    TABLE = "catalog.schema._file_event_"

    def _make_writer_mock(self):
        """Return a chainable MagicMock that simulates df.write.format(...).mode(...).option(...).saveAsTable()."""
        writer = MagicMock()
        writer.format.return_value = writer
        writer.mode.return_value = writer
        writer.option.return_value = writer
        return writer

    def _make_spark_mock(self, writer):
        mock_df = MagicMock()
        mock_df.write = writer
        spark = MagicMock()
        spark.createDataFrame.return_value = mock_df
        return spark

    def test_append_batch_empty_does_not_call_spark(self):
        spark = MagicMock()
        FileEventTable(spark, self.TABLE).append_batch([])
        spark.createDataFrame.assert_not_called()

    def test_append_batch_without_txn_options_calls_save_table(self):
        writer = self._make_writer_mock()
        spark = self._make_spark_mock(writer)
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        FileEventTable(spark, self.TABLE).append_batch([event])
        writer.saveAsTable.assert_called_once_with(self.TABLE)
        writer.option.assert_not_called()

    def test_append_batch_raises_on_spark_failure(self):
        writer = self._make_writer_mock()
        writer.saveAsTable.side_effect = RuntimeError("Delta write failed")
        spark = self._make_spark_mock(writer)
        event = FileEvent.loaded(NOW, PATH, make_file_info())
        with pytest.raises(RuntimeError, match="Delta write failed"):
            FileEventTable(spark, self.TABLE).append_batch([event])

```

###### FILE: tests/unit/test_file_intent.py ######

```py
import json
import os

import pytest

from azfr_databricks_landing_zone.file_intent import FileIntentStore, IntentSnapshot, IntentStatus


def make_store(tmp_path) -> FileIntentStore:
    return FileIntentStore.for_route(str(tmp_path), "route-a")


ENTRY_A = ("/source/a.csv", 1_700_000_000_000, "/target/2023/a.csv")
ENTRY_B = ("/source/b.csv", 1_700_000_001_000, "/target/2023/b.csv")


class TestFileIntentStoreForRoute:
    def test_builds_correct_paths(self, tmp_path):
        store = FileIntentStore.for_route(str(tmp_path), "my-route")
        assert store._path == f"{tmp_path}/my-route/intent_store/_intent.json"
        assert store._tmp_path == f"{tmp_path}/my-route/intent_store/_intent.json.tmp"


class TestFileIntentStoreRead:
    def test_returns_none_when_no_file(self, tmp_path):
        assert make_store(tmp_path).read() is None

    def test_ignores_tmp_file_when_no_main_file(self, tmp_path):
        store = make_store(tmp_path)
        os.makedirs(os.path.dirname(store._path), exist_ok=True)
        with open(store._tmp_path, "w") as f:
            json.dump({"status": "in_progress", "entries": [{"source_path": "/x", "modification_time_ms": 1, "target_path": "/y"}]}, f)
        assert store.read() is None

    def test_returns_in_progress_snapshot_after_write(self, tmp_path):
        store = make_store(tmp_path)
        store.write_in_progress([ENTRY_A, ENTRY_B])
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.IN_PROGRESS
        assert snapshot.entries == {
            ("/source/a.csv", 1_700_000_000_000): "/target/2023/a.csv",
            ("/source/b.csv", 1_700_000_001_000): "/target/2023/b.csv",
        }

    def test_returns_success_snapshot_after_mark_success(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A])
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.SUCCESS
        assert snapshot.entries == {("/source/a.csv", 1_700_000_000_000): "/target/2023/a.csv"}


class TestFileIntentStoreWrite:
    def test_creates_parent_directories(self, tmp_path):
        store = make_store(tmp_path)
        store.write_in_progress([ENTRY_A])
        assert os.path.exists(store._path)

    def test_tmp_file_not_left_behind(self, tmp_path):
        store = make_store(tmp_path)
        store.write_in_progress([ENTRY_A])
        assert not os.path.exists(store._tmp_path)

    def test_overwrites_previous_content(self, tmp_path):
        store = make_store(tmp_path)
        store.write_in_progress([ENTRY_A])
        store.write_in_progress([ENTRY_B])
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.entries == {("/source/b.csv", 1_700_000_001_000): "/target/2023/b.csv"}

    def test_write_always_sets_in_progress_status(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A])
        store.write_in_progress([ENTRY_B])  # overwrite with new batch
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.IN_PROGRESS

    def test_write_empty_list(self, tmp_path):
        store = make_store(tmp_path)
        store.write_in_progress([])
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.IN_PROGRESS
        assert snapshot.entries == {}

    def test_roundtrip_preserves_large_modtime(self, tmp_path):
        large_modtime = 9_999_999_999_999
        store = make_store(tmp_path)
        store.write_in_progress([("/source/x.parquet", large_modtime, "/target/x.parquet")])
        snapshot = store.read()
        assert snapshot is not None
        assert snapshot.entries[("/source/x.parquet", large_modtime)] == "/target/x.parquet"


class TestFileIntentStoreMarkSuccess:
    def test_promotes_status_to_success(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A])
        assert store.read().status == IntentStatus.SUCCESS

    def test_preserves_entries(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A, ENTRY_B])
        snapshot = store.read()
        assert snapshot.entries == {
            ("/source/a.csv", 1_700_000_000_000): "/target/2023/a.csv",
            ("/source/b.csv", 1_700_000_001_000): "/target/2023/b.csv",
        }

    def test_tmp_file_not_left_behind(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A])
        assert not os.path.exists(store._tmp_path)

    def test_idempotent(self, tmp_path):
        store = make_store(tmp_path)
        store.mark_success([ENTRY_A])
        store.mark_success([ENTRY_A])  # second call must not raise or corrupt
        assert store.read().status == IntentStatus.SUCCESS


```

###### FILE: tests/unit/test_main.py ######

```py
import sys

import pytest

from azfr_databricks_landing_zone.config import LandingRoute
from azfr_databricks_landing_zone.main import _resolve_route, get_args, _get_autoloader_options


MINIMAL_ROUTE = {
    "id": "data-source-A",
    "source": "/Volumes/azfr_dev_raw/landing/landing_volume/data-source-A/",
    "target": "/Volumes/azfr_dev_raw/my_schema/raw_files/data-source-A/",
}


class TestGetAutoloaderOptions:
    def test_default_route_uses_managed_file_events(self):
        route = LandingRoute(**MINIMAL_ROUTE)

        assert _get_autoloader_options(route) == {
            "cloudFiles.format": "binaryFile",
            "cloudFiles.includeExistingFiles": "true",
            "cloudFiles.useManagedFileEvents": "true",
        }

    def test_allow_filename_reuse_enables_allow_overwrites(self):
        route = LandingRoute(**{**MINIMAL_ROUTE, "allow_filename_reuse": True})

        assert _get_autoloader_options(route) == {
            "cloudFiles.format": "binaryFile",
            "cloudFiles.includeExistingFiles": "true",
            "cloudFiles.useManagedFileEvents": "false",
            "cloudFiles.allowOverwrites": "true",
        }

    def test_allow_overwrites_absent_when_filename_reuse_disabled(self):
        route = LandingRoute(**MINIMAL_ROUTE)

        options = _get_autoloader_options(route)
        assert "cloudFiles.allowOverwrites" not in options

    def test_recovery_mode_default_false_does_not_set_recovery_options(self):
        route = LandingRoute(**MINIMAL_ROUTE)

        options = _get_autoloader_options(route)
        assert "cloudFiles.listOnStart" not in options
        assert "cloudFiles.validateOptions" not in options

    def test_recovery_mode_true_sets_recovery_options(self):
        route = LandingRoute(**{**MINIMAL_ROUTE, "allow_filename_reuse_recovery": True})

        options = _get_autoloader_options(route)
        assert options["cloudFiles.listOnStart"] == "true"
        assert options["cloudFiles.validateOptions"] == "false"


class TestResolveRoute:
    def test_missing_route_id_raises(self):
        config = type("Config", (), {"landing_routes": [LandingRoute(**MINIMAL_ROUTE)]})()

        with pytest.raises(ValueError, match="route_id is required"):
            _resolve_route(config, None)

    def test_route_not_found_raises(self):
        config = type("Config", (), {"landing_routes": [LandingRoute(**MINIMAL_ROUTE)]})()

        with pytest.raises(ValueError, match="Route 'missing' not found in config"):
            _resolve_route(config, "missing")

    def test_resolves_matching_route(self):
        route_a = LandingRoute(**MINIMAL_ROUTE)
        route_b = LandingRoute(**{**MINIMAL_ROUTE, "id": "data-source-B"})
        config = type("Config", (), {"landing_routes": [route_a, route_b]})()

        resolved = _resolve_route(config, "data-source-B")

        assert resolved is route_b


class TestGetArgs:
    def test_config_path_parsed(self, monkeypatch):
        monkeypatch.setattr(sys, "argv", ["prog", "--config-path", "/some/config.yml"])

        args = get_args()

        assert args.config_path == "/some/config.yml"
        assert args.route_id is None

    def test_route_id_parsed(self, monkeypatch):
        monkeypatch.setattr(sys, "argv", ["prog", "--config-path", "/some/config.yml", "--route-id", "source-a"])

        args = get_args()

        assert args.route_id == "source-a"

    def test_missing_config_path_raises(self, monkeypatch):
        monkeypatch.setattr(sys, "argv", ["prog"])

        with pytest.raises(ValueError, match="config path"):
            get_args()

```

###### FILE: tests/unit/test_resolve_copiers.py ######

```py
"""Unit tests for LandingMoverJob._resolve_copiers — covers ADR-009 cases A-1 through A-7 and B-1/B-2."""

from concurrent.futures import ThreadPoolExecutor
from unittest.mock import MagicMock, patch

from azfr_databricks_landing_zone.config import LandingRoute
from azfr_databricks_landing_zone.file_intent import FileIntentStore, IntentStatus
from azfr_databricks_landing_zone.fs_backend import FileMetadata
from azfr_databricks_landing_zone.instrumentation import JobInstrumentation
from azfr_databricks_landing_zone.main import LandingMoverJob


ROUTE = LandingRoute(
    id="route-a",
    source="/Volumes/cat/sch/vol/source",
    target="/Volumes/cat/sch/vol/target",
)

MOD_A = 1_700_000_000_000
MOD_B = 1_700_000_001_000

FILE_A = FileMetadata(path="/Volumes/cat/sch/vol/source/a.csv", name="a.csv", size=100, modification_time=MOD_A, creation_time=None)
FILE_B = FileMetadata(path="/Volumes/cat/sch/vol/source/b.csv", name="b.csv", size=200, modification_time=MOD_B, creation_time=None)

TARGET_A = "/Volumes/cat/sch/vol/target/2023/01/aaaa/a.csv"
TARGET_B = "/Volumes/cat/sch/vol/target/2023/01/bbbb/b.csv"

ENTRY_A = (FILE_A.path, MOD_A, TARGET_A)
ENTRY_B = (FILE_B.path, MOD_B, TARGET_B)


def make_job(tmp_path) -> LandingMoverJob:
    return LandingMoverJob(
        spark=MagicMock(),
        fs=MagicMock(),
        shared_executor=ThreadPoolExecutor(max_workers=1),
        work_base=str(tmp_path),
        instrumentation=JobInstrumentation(log_dir=str(tmp_path), run_id="test-run"),
    )


def make_store(tmp_path) -> FileIntentStore:
    return FileIntentStore.for_route(str(tmp_path), ROUTE.id)


# ---------------------------------------------------------------------------
# A-1: no snapshot — fresh start
# ---------------------------------------------------------------------------

class TestFreshStart:
    """A-1: no intent file — all files get fresh target paths."""

    def test_returns_all_files_in_copiers(self, tmp_path):
        file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(file_copiers) == 2

    def test_full_entries_covers_all_files(self, tmp_path):
        _, full_entries = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        keys = {(e[0], e[1]) for e in full_entries}
        assert keys == {(FILE_A.path, MOD_A), (FILE_B.path, MOD_B)}

    def test_write_in_progress_is_called(self, tmp_path):
        make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        snapshot = make_store(tmp_path).read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.IN_PROGRESS

    def test_stale_in_progress_no_overlap_treated_as_fresh(self, tmp_path):
        make_store(tmp_path).write_in_progress([("/other/x.csv", 999, "/target/x.csv")])
        file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(file_copiers) == 2


# ---------------------------------------------------------------------------
# A-2: IN_PROGRESS + full overlap, Delta returns ∅ — all files re-queued
# ---------------------------------------------------------------------------

class TestInProgressDeltaEmpty:
    """A-2: crash before any copy completed — all files re-queued with original target paths."""

    def test_all_files_in_copiers(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = set()
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(file_copiers) == 2

    def test_original_target_paths_reused(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = set()
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert {c.target_path for c in file_copiers} == {TARGET_A, TARGET_B}


# ---------------------------------------------------------------------------
# A-3 / A-4: IN_PROGRESS + full overlap, Delta returns a subset
# ---------------------------------------------------------------------------

class TestInProgressDeltaPartial:
    """A-3/A-4: some files already LOADED — only non-LOADED files re-queued."""

    def test_loaded_file_skipped_in_copiers(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = {TARGET_A}
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(file_copiers) == 1
        assert file_copiers[0].target_path == TARGET_B

    def test_full_entries_still_covers_all_files(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = {TARGET_A}
            _, full_entries = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(full_entries) == 2


# ---------------------------------------------------------------------------
# A-5 / A-6: IN_PROGRESS + full overlap, Delta returns all targets
# ---------------------------------------------------------------------------

class TestInProgressDeltaAll:
    """A-5/A-6: all files already LOADED — no copies needed, full_entries still complete."""

    def test_file_copiers_is_empty(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = {TARGET_A, TARGET_B}
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert file_copiers == []

    def test_full_entries_still_covers_all_files(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = {TARGET_A, TARGET_B}
            _, full_entries = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(full_entries) == 2

    def test_intent_written_with_full_entries(self, tmp_path):
        make_store(tmp_path).write_in_progress([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            mock_fet.return_value.get_loaded_target_paths.return_value = {TARGET_A, TARGET_B}
            make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        snapshot = make_store(tmp_path).read()
        assert snapshot is not None
        assert len(snapshot.entries) == 2


# ---------------------------------------------------------------------------
# A-7: SUCCESS + full overlap — success replay, returns None
# ---------------------------------------------------------------------------

class TestSuccessReplay:
    """A-7: batch already completed (SUCCESS + full overlap) — returns None immediately."""

    def test_returns_none(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        result = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert result is None

    def test_no_delta_query(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
            mock_fet.return_value.get_loaded_target_paths.assert_not_called()

    def test_intent_not_overwritten(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A, ENTRY_B])
        make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        snapshot = make_store(tmp_path).read()
        assert snapshot is not None
        assert snapshot.status == IntentStatus.SUCCESS


# ---------------------------------------------------------------------------
# B-1: SUCCESS + no overlap — fresh start, no Delta query
# ---------------------------------------------------------------------------

class TestSuccessNoOverlap:
    """B-1: stale SUCCESS from a different batch — treated as fresh start."""

    def test_all_files_in_copiers(self, tmp_path):
        make_store(tmp_path).mark_success([("/other/x.csv", 999, "/target/x.csv")])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
            mock_fet.return_value.get_loaded_target_paths.assert_not_called()
        assert len(file_copiers) == 2


# ---------------------------------------------------------------------------
# B-2: SUCCESS + partial overlap — overlapping files skipped, no Delta query
# ---------------------------------------------------------------------------

class TestSuccessPartialOverlap:
    """B-2: FILE_A from previous SUCCESS batch, FILE_B is new."""

    def test_only_new_file_in_copiers(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A])
        with patch("azfr_databricks_landing_zone.main.FileEventTable") as mock_fet:
            file_copiers, _ = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
            mock_fet.return_value.get_loaded_target_paths.assert_not_called()
        assert len(file_copiers) == 1
        assert file_copiers[0].source_path == FILE_B.path

    def test_overlapping_file_reuses_original_target(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A])
        with patch("azfr_databricks_landing_zone.main.FileEventTable"):
            _, full_entries = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        entry_a = next(e for e in full_entries if e[0] == FILE_A.path)
        assert entry_a[2] == TARGET_A

    def test_full_entries_covers_both_files(self, tmp_path):
        make_store(tmp_path).mark_success([ENTRY_A])
        with patch("azfr_databricks_landing_zone.main.FileEventTable"):
            _, full_entries = make_job(tmp_path)._resolve_copiers(ROUTE, [FILE_A, FILE_B])
        assert len(full_entries) == 2


# ---------------------------------------------------------------------------
# NativeOsBackend.delete_file idempotency — A-5/A-6 safety
# ---------------------------------------------------------------------------

class TestDeleteFileIdempotent:
    """delete_file must not raise when the file is already absent."""

    def test_missing_file_does_not_raise(self, tmp_path):
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend
        NativeOsBackend().delete_file(str(tmp_path / "nonexistent.csv"))

    def test_existing_file_is_removed(self, tmp_path):
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend
        f = tmp_path / "file.csv"
        f.write_text("data")
        NativeOsBackend().delete_file(str(f))
        assert not f.exists()

    def test_double_delete_does_not_raise(self, tmp_path):
        from azfr_databricks_landing_zone.fs_backend import NativeOsBackend
        f = tmp_path / "file.csv"
        f.write_text("data")
        backend = NativeOsBackend()
        backend.delete_file(str(f))
        backend.delete_file(str(f))  # second call must not raise

```

###### FILE: typings/__builtins__.pyi ######

```pyi

from databricks.sdk.runtime import *
from pyspark.sql.session import SparkSession
from pyspark.sql.functions import udf as U
from pyspark.sql.context import SQLContext

udf = U
spark: SparkSession
sc = spark.sparkContext
sqlContext: SQLContext
sql = sqlContext.sql
table = sqlContext.table
getArgument = dbutils.widgets.getArgument

def displayHTML(html): ...

def display(input=None, *args, **kwargs): ...


from databricks.sdk.runtime import *
from pyspark.sql.session import SparkSession
from pyspark.sql.functions import udf as U
from pyspark.sql.context import SQLContext

udf = U
spark: SparkSession
sc = spark.sparkContext
sqlContext: SQLContext
sql = sqlContext.sql
table = sqlContext.table
getArgument = dbutils.widgets.getArgument

def displayHTML(html): ...

def display(input=None, *args, **kwargs): ...


```

###### FILE: uv.lock ######

```lock
version = 1
revision = 3
requires-python = "==3.12.*"

[[package]]
name = "aiohappyeyeballs"
version = "2.6.2"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/aiohappyeyeballs/2.6.2/aiohappyeyeballs-2.6.2.tar.gz", hash = "sha256:e202810ee718bd01fc6ef49e8ea53d023d5cb6b581076d7925aa499fa55dbe64" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/aiohappyeyeballs/2.6.2/aiohappyeyeballs-2.6.2-py3-none-any.whl", hash = "sha256:4708045e2d7a6c6bdf8aafa8ed39649eaf926a4543b54560659129e3365953c4" },
]

[[package]]
name = "aiohttp"
version = "3.13.5"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "aiohappyeyeballs" },
    { name = "aiosignal" },
    { name = "attrs" },
    { name = "frozenlist" },
    { name = "multidict" },
    { name = "propcache" },
    { name = "yarl" },
]
sdist = { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5.tar.gz", hash = "sha256:9d98cc980ecc96be6eb4c1994ce35d28d8b1f5e5208a23b421187d1209dbb7d1" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:023ecba036ddd840b0b19bf195bfae970083fd7024ce1ac22e9bba90464620e9" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:15c933ad7920b7d9a20de151efcd05a6e38302cbf0e10c9b2acb9a42210a2416" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:ab2899f9fa2f9f741896ebb6fa07c4c883bfa5c7f2ddd8cf2aafa86fa981b2d2" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:a60eaa2d440cd4707696b52e40ed3e2b0f73f65be07fd0ef23b6b539c9c0b0b4" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:55b3bdd3292283295774ab585160c4004f4f2f203946997f49aac032c84649e9" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:c2b2355dc094e5f7d45a7bb262fe7207aa0460b37a0d87027dcf21b5d890e7d5" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:b38765950832f7d728297689ad78f5f2cf79ff82487131c4d26fe6ceecdc5f8e" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:b18f31b80d5a33661e08c89e202edabf1986e9b49c42b4504371daeaa11b47c1" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:33add2463dde55c4f2d9635c6ab33ce154e5ecf322bd26d09af95c5f81cfa286" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:327cc432fdf1356fb4fbc6fe833ad4e9f6aacb71a8acaa5f1855e4b25910e4a9" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:7c35b0bf0b48a70b4cb4fc5d7bed9b932532728e124874355de1a0af8ec4bc88" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:df23d57718f24badef8656c49743e11a89fd6f5358fa8a7b96e728fda2abf7d3" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:02e048037a6501a5ec1f6fc9736135aec6eb8a004ce48838cb951c515f32c80b" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:31cebae8b26f8a615d2b546fee45d5ffb76852ae6450e2a03f42c9102260d6fe" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:888e78eb5ca55a615d285c3c09a7a91b42e9dd6fc699b166ebd5dee87c9ccf14" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-win32.whl", hash = "sha256:8bd3ec6376e68a41f9f95f5ed170e2fcf22d4eb27a1f8cb361d0508f6e0557f3" },
    { url = "nexus/repository/pypi-public/packages/aiohttp/3.13.5/aiohttp-3.13.5-cp312-cp312-win_amd64.whl", hash = "sha256:110e448e02c729bcebb18c60b9214a87ba33bac4a9fa5e9a5f139938b56c6cb1" },
]

[[package]]
name = "aiosignal"
version = "1.4.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "frozenlist" },
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/aiosignal/1.4.0/aiosignal-1.4.0.tar.gz", hash = "sha256:f47eecd9468083c2029cc99945502cb7708b082c232f9aca65da147157b251c7" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/aiosignal/1.4.0/aiosignal-1.4.0-py3-none-any.whl", hash = "sha256:053243f8b92b990551949e63930a839ff0cf0b0ebbe0597b0f3fb19e1a0fe82e" },
]

[[package]]
name = "altair"
version = "6.2.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "jinja2" },
    { name = "jsonschema" },
    { name = "narwhals" },
    { name = "packaging" },
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/altair/6.2.2/altair-6.2.2.tar.gz", hash = "sha256:a1ff9d9cfe81c75414641826312b9471780e19d39293ba0b012933f6b6cba0fe" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/altair/6.2.2/altair-6.2.2-py3-none-any.whl", hash = "sha256:94014f8ad8617c3cb163d1137359cd6db5ba134b9b46d93cfd8b609fd245a583" },
]

[[package]]
name = "annotated-types"
version = "0.7.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/annotated-types/0.7.0/annotated_types-0.7.0.tar.gz", hash = "sha256:aff07c09a53a08bc8cfccb9c85b05f1aa9a2a6f23728d790723543408344ce89" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/annotated-types/0.7.0/annotated_types-0.7.0-py3-none-any.whl", hash = "sha256:1f02e8b43a8fbbc3f3e0d4f0f4bfc8131bcb4eebe8849b8e5c773f3a1c582a53" },
]

[[package]]
name = "anyio"
version = "4.14.1"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "idna" },
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/anyio/4.14.1/anyio-4.14.1.tar.gz", hash = "sha256:8d648a3544c1a700e3ff78615cd679e4c5c3f149904287e73687b2596963629e" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/anyio/4.14.1/anyio-4.14.1-py3-none-any.whl", hash = "sha256:4e5533c5b8ff0a24f5d7a176cbe6877129cd183893f66b537f8f227d10527d72" },
]

[[package]]
name = "attrs"
version = "26.1.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/attrs/26.1.0/attrs-26.1.0.tar.gz", hash = "sha256:d03ceb89cb322a8fd706d4fb91940737b6642aa36998fe130a9bc96c985eff32" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/attrs/26.1.0/attrs-26.1.0-py3-none-any.whl", hash = "sha256:c647aa4a12dfbad9333ca4e71fe62ddc36f4e63b2d260a37a8b83d2f043ac309" },
]

[[package]]
name = "azfr-databricks-landing-zone"
source = { editable = "." }
dependencies = [
    { name = "fsspec-databricks" },
    { name = "pydantic" },
    { name = "pyyaml" },
    { name = "uuid6" },
]

[package.dev-dependencies]
dev = [
    { name = "databricks-bundles" },
    { name = "databricks-connect" },
    { name = "pandas" },
    { name = "plotly" },
    { name = "pytest" },
    { name = "streamlit" },
]

[package.metadata]
requires-dist = [
    { name = "fsspec-databricks" },
    { name = "pydantic" },
    { name = "pyyaml" },
    { name = "uuid6" },
]

[package.metadata.requires-dev]
dev = [
    { name = "databricks-bundles", specifier = ">=0.275.0" },
    { name = "databricks-connect" },
    { name = "pandas", specifier = ">=2.0" },
    { name = "plotly", specifier = ">=5.0" },
    { name = "pytest", specifier = ">=8.4.2" },
    { name = "streamlit", specifier = ">=1.46.0" },
]

[[package]]
name = "blinker"
version = "1.9.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/blinker/1.9.0/blinker-1.9.0.tar.gz", hash = "sha256:b4ce2265a7abece45e7cc896e98dbebe6cead56bcf805a3d23136d145f5445bf" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/blinker/1.9.0/blinker-1.9.0-py3-none-any.whl", hash = "sha256:ba0efaa9080b619ff2f3459d1d500c57bddea4a6b424b60a91141db6fd2f08bc" },
]

[[package]]
name = "cachetools"
version = "7.1.4"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/cachetools/7.1.4/cachetools-7.1.4.tar.gz", hash = "sha256:437f55a4e0c1b01a4f3077cc470e6991d47430970e36fbcb77e2be0df4fc1cd6" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/cachetools/7.1.4/cachetools-7.1.4-py3-none-any.whl", hash = "sha256:323dc4127934744db5b54eb4924482d7edafbf9554e820d1531c2e08c0e4ef54" },
]

[[package]]
name = "certifi"
version = "2026.2.25"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/certifi/2026.2.25/certifi-2026.2.25.tar.gz", hash = "sha256:e887ab5cee78ea814d3472169153c2d12cd43b14bd03329a39a9c6e2e80bfba7" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/certifi/2026.2.25/certifi-2026.2.25-py3-none-any.whl", hash = "sha256:027692e4402ad994f1c42e52a4997a9763c646b73e4096e4d5d6db8af1d6f0fa" },
]

[[package]]
name = "cffi"
version = "2.0.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "pycparser", marker = "implementation_name != 'PyPy'" },
]
sdist = { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0.tar.gz", hash = "sha256:44d1b5909021139fe36001ae048dbdde8214afa20200eda0f64c068cac5d5529" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:6d02d6655b0e54f54c4ef0b94eb6be0607b70853c45ce98bd278dc7de718be5d" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:8eca2a813c1cb7ad4fb74d368c2ffbbb4789d377ee5bb8df98373c2cc0dee76c" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:21d1152871b019407d8ac3985f6775c079416c282e431a4da6afe7aefd2bccbe" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:b21e08af67b8a103c71a250401c78d5e0893beff75e28c53c98f4de42f774062" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:1e3a615586f05fc4065a8b22b8152f0c1b00cdbc60596d187c2a74f9e3036e4e" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:81afed14892743bbe14dacb9e36d9e0e504cd204e0b165062c488942b9718037" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:3e17ed538242334bf70832644a32a7aae3d83b57567f9fd60a26257e992b79ba" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3925dd22fa2b7699ed2617149842d2e6adde22b262fcbfada50e3d195e4b3a94" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:2c8f814d84194c9ea681642fd164267891702542f028a15fc97d4674b6206187" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-win32.whl", hash = "sha256:da902562c3e9c550df360bfa53c035b2f241fed6d9aef119048073680ace4a18" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-win_amd64.whl", hash = "sha256:da68248800ad6320861f129cd9c1bf96ca849a2771a59e0344e88681905916f5" },
    { url = "nexus/repository/pypi-public/packages/cffi/2.0.0/cffi-2.0.0-cp312-cp312-win_arm64.whl", hash = "sha256:4671d9dd5ec934cb9a73e7ee9676f9362aba54f7f34910956b84d727b0d73fb6" },
]

[[package]]
name = "charset-normalizer"
version = "3.4.7"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7.tar.gz", hash = "sha256:ae89db9e5f98a11a4bf50407d4363e7b09b31e55bc117b4f7d80aab97ba009e5" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:eca9705049ad3c7345d574e3510665cb2cf844c2f2dcfe675332677f081cbd46" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:6178f72c5508bfc5fd446a5905e698c6212932f25bcdd4b47a757a50605a90e2" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:e1421b502d83040e6d7fb2fb18dff63957f720da3d77b2fbd3187ceb63755d7b" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:edac0f1ab77644605be2cbba52e6b7f630731fc42b34cb0f634be1a6eface56a" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:5649fd1c7bade02f320a462fdefd0b4bd3ce036065836d4f42e0de958038e116" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux_2_31_armv7l.whl", hash = "sha256:203104ed3e428044fd943bc4bf45fa73c0730391f9621e37fe39ecf477b128cb" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:298930cec56029e05497a76988377cbd7457ba864beeea92ad7e844fe74cd1f1" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:708838739abf24b2ceb208d0e22403dd018faeef86ddac04319a62ae884c4f15" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:0f7eb884681e3938906ed0434f20c63046eacd0111c4ba96f27b76084cd679f5" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:4dc1e73c36828f982bfe79fadf5919923f8a6f4df2860804db9a98c48824ce8d" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:aed52fea0513bac0ccde438c188c8a471c4e0f457c2dd20cdbf6ea7a450046c7" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:fea24543955a6a729c45a73fe90e08c743f0b3334bbf3201e6c4bc1b0c7fa464" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:bb6d88045545b26da47aa879dd4a89a71d1dce0f0e549b1abcb31dfe4a8eac49" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-win32.whl", hash = "sha256:2257141f39fe65a3fdf38aeccae4b953e5f3b3324f4ff0daf9f15b8518666a2c" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-win_amd64.whl", hash = "sha256:5ed6ab538499c8644b8a3e18debabcd7ce684f3fa91cf867521a7a0279cab2d6" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-cp312-cp312-win_arm64.whl", hash = "sha256:56be790f86bfb2c98fb742ce566dfb4816e5a83384616ab59c49e0604d49c51d" },
    { url = "nexus/repository/pypi-public/packages/charset-normalizer/3.4.7/charset_normalizer-3.4.7-py3-none-any.whl", hash = "sha256:3dce51d0f5e7951f8bb4900c257dad282f49190fdbebecd4ba99bcc41fef404d" },
]

[[package]]
name = "click"
version = "8.4.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "colorama", marker = "sys_platform == 'win32'" },
]
sdist = { url = "nexus/repository/pypi-public/packages/click/8.4.2/click-8.4.2.tar.gz", hash = "sha256:9a6cea6e60b17ebe0a44c5cc636d94f09bd66142c1cd7d8b4cd731c4917a15f6" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/click/8.4.2/click-8.4.2-py3-none-any.whl", hash = "sha256:e6f9f66136c816745b9d65817da91d61d957fb16e02e4dcd0552553c5a197b76" },
]

[[package]]
name = "colorama"
version = "0.4.6"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/colorama/0.4.6/colorama-0.4.6.tar.gz", hash = "sha256:08695f5cb7ed6e0531a20572697297273c47b8cae5a63ffc6d6ed5c201be6e44" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/colorama/0.4.6/colorama-0.4.6-py2.py3-none-any.whl", hash = "sha256:4f1d9991f5acc0ca119f9d443620b77f9d6b33703e51011c16baf57afb285fc6" },
]

[[package]]
name = "cryptography"
version = "46.0.7"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "cffi", marker = "platform_python_implementation != 'PyPy'" },
]
sdist = { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7.tar.gz", hash = "sha256:e4cfd68c5f3e0bfdad0d38e023239b96a2fe84146481852dffbcca442c245aa5" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-macosx_10_9_universal2.whl", hash = "sha256:ea42cbe97209df307fdc3b155f1b6fa2577c0defa8f1f7d3be7d31d189108ad4" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:b36a4695e29fe69215d75960b22577197aca3f7a25b9cf9d165dcfe9d80bc325" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:5ad9ef796328c5e3c4ceed237a183f5d41d21150f972455a9d926593a1dcb308" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_28_aarch64.whl", hash = "sha256:73510b83623e080a2c35c62c15298096e2a5dc8d51c3b4e1740211839d0dea77" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_28_ppc64le.whl", hash = "sha256:cbd5fb06b62bd0721e1170273d3f4d5a277044c47ca27ee257025146c34cbdd1" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_28_x86_64.whl", hash = "sha256:420b1e4109cc95f0e5700eed79908cef9268265c773d3a66f7af1eef53d409ef" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_31_armv7l.whl", hash = "sha256:24402210aa54baae71d99441d15bb5a1919c195398a87b563df84468160a65de" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_34_aarch64.whl", hash = "sha256:8a469028a86f12eb7d2fe97162d0634026d92a21f3ae0ac87ed1c4a447886c83" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_34_ppc64le.whl", hash = "sha256:9694078c5d44c157ef3162e3bf3946510b857df5a3955458381d1c7cfc143ddb" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-manylinux_2_34_x86_64.whl", hash = "sha256:42a1e5f98abb6391717978baf9f90dc28a743b7d9be7f0751a6f56a75d14065b" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-musllinux_1_2_aarch64.whl", hash = "sha256:91bbcb08347344f810cbe49065914fe048949648f6bd5c2519f34619142bbe85" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-musllinux_1_2_x86_64.whl", hash = "sha256:5d1c02a14ceb9148cc7816249f64f623fbfee39e8c03b3650d842ad3f34d637e" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-win32.whl", hash = "sha256:d23c8ca48e44ee015cd0a54aeccdf9f09004eba9fc96f38c911011d9ff1bd457" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp311-abi3-win_amd64.whl", hash = "sha256:397655da831414d165029da9bc483bed2fe0e75dde6a1523ec2fe63f3c46046b" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-macosx_10_9_universal2.whl", hash = "sha256:462ad5cb1c148a22b2e3bcc5ad52504dff325d17daf5df8d88c17dda1f75f2a4" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:84d4cced91f0f159a7ddacad249cc077e63195c36aac40b4150e7a57e84fffe7" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:128c5edfe5e5938b86b03941e94fac9ee793a94452ad1365c9fc3f4f62216832" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_28_aarch64.whl", hash = "sha256:5e51be372b26ef4ba3de3c167cd3d1022934bc838ae9eaad7e644986d2a3d163" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_28_ppc64le.whl", hash = "sha256:cdf1a610ef82abb396451862739e3fc93b071c844399e15b90726ef7470eeaf2" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_28_x86_64.whl", hash = "sha256:1d25aee46d0c6f1a501adcddb2d2fee4b979381346a78558ed13e50aa8a59067" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_31_armv7l.whl", hash = "sha256:cdfbe22376065ffcf8be74dc9a909f032df19bc58a699456a21712d6e5eabfd0" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_34_aarch64.whl", hash = "sha256:abad9dac36cbf55de6eb49badd4016806b3165d396f64925bf2999bcb67837ba" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_34_ppc64le.whl", hash = "sha256:935ce7e3cfdb53e3536119a542b839bb94ec1ad081013e9ab9b7cfd478b05006" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-manylinux_2_34_x86_64.whl", hash = "sha256:35719dc79d4730d30f1c2b6474bd6acda36ae2dfae1e3c16f2051f215df33ce0" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-musllinux_1_2_aarch64.whl", hash = "sha256:7bbc6ccf49d05ac8f7d7b5e2e2c33830d4fe2061def88210a126d130d7f71a85" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-musllinux_1_2_x86_64.whl", hash = "sha256:a1529d614f44b863a7b480c6d000fe93b59acee9c82ffa027cfadc77521a9f5e" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-win32.whl", hash = "sha256:f247c8c1a1fb45e12586afbb436ef21ff1e80670b2861a90353d9b025583d246" },
    { url = "nexus/repository/pypi-public/packages/cryptography/46.0.7/cryptography-46.0.7-cp38-abi3-win_amd64.whl", hash = "sha256:506c4ff91eff4f82bdac7633318a526b1d1309fc07ca76a3ad182cb5b686d6d3" },
]

[[package]]
name = "databricks-bundles"
version = "1.6.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/databricks-bundles/1.6.0/databricks_bundles-1.6.0.tar.gz", hash = "sha256:538b930524ec2ec98424ede3796383fa30dbec610d716b961ca51c3ac9852d46" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/databricks-bundles/1.6.0/databricks_bundles-1.6.0-py3-none-any.whl", hash = "sha256:55556cb97edfb56721b8754d6971c729da9fd56aaab37531fb64f89204474ee3" },
]

[[package]]
name = "databricks-connect"
version = "18.1.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "databricks-sdk" },
    { name = "googleapis-common-protos" },
    { name = "grpcio" },
    { name = "grpcio-status" },
    { name = "numpy" },
    { name = "packaging" },
    { name = "pandas" },
    { name = "py4j" },
    { name = "pyarrow" },
    { name = "setuptools" },
    { name = "six" },
    { name = "zstandard" },
]
wheels = [
    { url = "nexus/repository/pypi-public/packages/databricks-connect/18.1.2/databricks_connect-18.1.2-py2.py3-none-any.whl", hash = "sha256:b9133a113d111acd828d5fd8aad588a9f355b431d6926dbaeb3e28f4757cc8bb" },
]

[[package]]
name = "databricks-sdk"
version = "0.103.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "google-auth" },
    { name = "protobuf" },
    { name = "requests" },
]
sdist = { url = "nexus/repository/pypi-public/packages/databricks-sdk/0.103.0/databricks_sdk-0.103.0.tar.gz", hash = "sha256:bdc93a2382e5717edd39c2faa92e38606ccc48aead047fe2154243509861eb1a" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/databricks-sdk/0.103.0/databricks_sdk-0.103.0-py3-none-any.whl", hash = "sha256:eb6c1cdbe8dfe76590d049cbd03e35c45855d1bbc968d565183fa27b80ac3a76" },
]

[[package]]
name = "frozenlist"
version = "1.8.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0.tar.gz", hash = "sha256:3ede829ed8d842f6cd48fc7081d7a41001a56f1f38603f9d49bf3020d59a31ad" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:78f7b9e5d6f2fdb88cdde9440dc147259b62b9d3b019924def9f6478be254ac1" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:229bf37d2e4acdaf808fd3f06e854a4a7a3661e871b10dc1f8f1896a3b05f18b" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:f833670942247a14eafbb675458b4e61c82e002a148f49e68257b79296e865c4" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl", hash = "sha256:494a5952b1c597ba44e0e78113a7266e656b9794eec897b19ead706bd7074383" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:96f423a119f4777a4a056b66ce11527366a8bb92f54e541ade21f2374433f6d4" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:3462dd9475af2025c31cc61be6652dfa25cbfb56cbbf52f4ccfe029f38decaf8" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:c4c800524c9cd9bac5166cd6f55285957fcfc907db323e193f2afcd4d9abd69b" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:d6a5df73acd3399d893dafc71663ad22534b5aa4f94e8a2fabfe856c3c1b6a52" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:405e8fe955c2280ce66428b3ca55e12b3c4e9c336fb2103a4937e891c69a4a29" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:908bd3f6439f2fef9e85031b59fd4f1297af54415fb60e4254a95f75b3cab3f3" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:294e487f9ec720bd8ffcebc99d575f7eff3568a08a253d1ee1a0378754b74143" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:74c51543498289c0c43656701be6b077f4b265868fa7f8a8859c197006efb608" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:776f352e8329135506a1d6bf16ac3f87bc25b28e765949282dcc627af36123aa" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-win32.whl", hash = "sha256:433403ae80709741ce34038da08511d4a77062aa924baf411ef73d1146e74faf" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-win_amd64.whl", hash = "sha256:34187385b08f866104f0c0617404c8eb08165ab1272e884abc89c112e9c00746" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-cp312-cp312-win_arm64.whl", hash = "sha256:fe3c58d2f5db5fbd18c2987cba06d51b0529f52bc3a6cdc33d3f4eab725104bd" },
    { url = "nexus/repository/pypi-public/packages/frozenlist/1.8.0/frozenlist-1.8.0-py3-none-any.whl", hash = "sha256:0c18a16eab41e82c295618a77502e17b195883241c563b00f0aa5106fc4eaa0d" },
]

[[package]]
name = "fsspec"
version = "2026.4.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/fsspec/2026.4.0/fsspec-2026.4.0.tar.gz", hash = "sha256:301d8ac70ae90ef3ad05dcf94d6c3754a097f9b5fe4667d2787aa359ec7df7e4" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/fsspec/2026.4.0/fsspec-2026.4.0-py3-none-any.whl", hash = "sha256:11ef7bb35dab8a394fde6e608221d5cf3e8499401c249bebaeaad760a1a8dec2" },
]

[[package]]
name = "fsspec-databricks"
version = "0.1.10"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "aiohttp" },
    { name = "databricks-sdk" },
    { name = "fsspec" },
]
sdist = { url = "nexus/repository/pypi-public/packages/fsspec-databricks/0.1.10/fsspec_databricks-0.1.10.tar.gz", hash = "sha256:c2775b459736f05840c7c292b767e39c95d271f761fd55ea9b29477a635c0bfa" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/fsspec-databricks/0.1.10/fsspec_databricks-0.1.10-py3-none-any.whl", hash = "sha256:79829f86407f8875b0eec71f49c4c3f55f9c43f5025b651565281d44fc33c736" },
]

[[package]]
name = "gitdb"
version = "4.0.12"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "smmap" },
]
sdist = { url = "nexus/repository/pypi-public/packages/gitdb/4.0.12/gitdb-4.0.12.tar.gz", hash = "sha256:5ef71f855d191a3326fcfbc0d5da835f26b13fbcba60c32c21091c349ffdb571" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/gitdb/4.0.12/gitdb-4.0.12-py3-none-any.whl", hash = "sha256:67073e15955400952c6565cc3e707c554a4eea2e428946f7a4c162fab9bd9bcf" },
]

[[package]]
name = "gitpython"
version = "3.1.50"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "gitdb" },
]
sdist = { url = "nexus/repository/pypi-public/packages/gitpython/3.1.50/gitpython-3.1.50.tar.gz", hash = "sha256:80da2d12504d52e1f998772dc5baf6e553f8d2fcfe1fcc226c9d9a2ee3372dcc" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/gitpython/3.1.50/gitpython-3.1.50-py3-none-any.whl", hash = "sha256:d352abe2908d07355014abdd21ddf798c2a961469239afec4962e9da884858f9" },
]

[[package]]
name = "google-auth"
version = "2.49.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "cryptography" },
    { name = "pyasn1-modules" },
]
sdist = { url = "nexus/repository/pypi-public/packages/google-auth/2.49.2/google_auth-2.49.2.tar.gz", hash = "sha256:c1ae38500e73065dcae57355adb6278cf8b5c8e391994ae9cbadbcb9631ab409" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/google-auth/2.49.2/google_auth-2.49.2-py3-none-any.whl", hash = "sha256:c2720924dfc82dedb962c9f52cabb2ab16714fd0a6a707e40561d217574ed6d5" },
]

[[package]]
name = "googleapis-common-protos"
version = "1.74.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "protobuf" },
]
sdist = { url = "nexus/repository/pypi-public/packages/googleapis-common-protos/1.74.0/googleapis_common_protos-1.74.0.tar.gz", hash = "sha256:57971e4eeeba6aad1163c1f0fc88543f965bb49129b8bb55b2b7b26ecab084f1" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/googleapis-common-protos/1.74.0/googleapis_common_protos-1.74.0-py3-none-any.whl", hash = "sha256:702216f78610bb510e3f12ac3cafd281b7ac45cc5d86e90ad87e4d301a3426b5" },
]

[[package]]
name = "grpcio"
version = "1.80.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0.tar.gz", hash = "sha256:29aca15edd0688c22ba01d7cc01cb000d72b2033f4a3c72a81a19b56fd143257" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-linux_armv7l.whl", hash = "sha256:c624cc9f1008361014378c9d776de7182b11fe8b2e5a81bc69f23a295f2a1ad0" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-macosx_11_0_universal2.whl", hash = "sha256:f49eddcac43c3bf350c0385366a58f36bed8cc2c0ec35ef7b74b49e56552c0c2" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:d334591df610ab94714048e0d5b4f3dd5ad1bee74dfec11eee344220077a79de" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-manylinux2014_i686.manylinux_2_17_i686.whl", hash = "sha256:0cb517eb1d0d0aaf1d87af7cc5b801d686557c1d88b2619f5e31fab3c2315921" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:4e78c4ac0d97dc2e569b2f4bcbbb447491167cb358d1a389fc4af71ab6f70411" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:2ed770b4c06984f3b47eb0517b1c69ad0b84ef3f40128f51448433be904634cd" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:256507e2f524092f1473071a05e65a5b10d84b82e3ff24c5b571513cfaa61e2f" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:9a6284a5d907c37db53350645567c522be314bac859a64a7a5ca63b77bb7958f" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-win32.whl", hash = "sha256:c71309cfce2f22be26aa4a847357c502db6c621f1a49825ae98aa0907595b193" },
    { url = "nexus/repository/pypi-public/packages/grpcio/1.80.0/grpcio-1.80.0-cp312-cp312-win_amd64.whl", hash = "sha256:9fe648599c0e37594c4809d81a9e77bd138cc82eb8baa71b6a86af65426723ff" },
]

[[package]]
name = "grpcio-status"
version = "1.80.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "googleapis-common-protos" },
    { name = "grpcio" },
    { name = "protobuf" },
]
sdist = { url = "nexus/repository/pypi-public/packages/grpcio-status/1.80.0/grpcio_status-1.80.0.tar.gz", hash = "sha256:df73802a4c89a3ea88aa2aff971e886fccce162bc2e6511408b3d67a144381cd" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/grpcio-status/1.80.0/grpcio_status-1.80.0-py3-none-any.whl", hash = "sha256:4b56990363af50dbf2c2ebb80f1967185c07d87aa25aa2bea45ddb75fc181dbe" },
]

[[package]]
name = "h11"
version = "0.16.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/h11/0.16.0/h11-0.16.0.tar.gz", hash = "sha256:4e35b956cf45792e4caa5885e69fba00bdbc6ffafbfa020300e549b208ee5ff1" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/h11/0.16.0/h11-0.16.0-py3-none-any.whl", hash = "sha256:63cf8bbe7522de3bf65932fda1d9c2772064ffb3dae62d55932da54b31cb6c86" },
]

[[package]]
name = "httptools"
version = "0.8.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0.tar.gz", hash = "sha256:6b2a32f18d97e16e90827d7a819ffa8dbd8cc245fc4e1fa9d1095b54ef4bd999" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:880490234c10f70a9830743097e8958d6e4b9f5a0ffc24515023afeef984054d" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:5931891fb7b441b8a3853cf1b85c82c903defce084dd5f6771ca46e31bf862c5" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl", hash = "sha256:b15fc622b0f869d19207c4089a501d9bcc63ca5e071ffdd2f03f922df882dcb2" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:425f83884fd6343828d8c565f046cb72b6d19063f6924093e11bcd8e1548cd09" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:ef7c3c97f4311c7be57e2986629df89d49cb434dbff78eafcd48c2bff986b15a" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:a1afd7c9fbff0d9f5d489c4ce2768bd09c84a46ddefc7161e6aa82ae35c85745" },
    { url = "nexus/repository/pypi-public/packages/httptools/0.8.0/httptools-0.8.0-cp312-cp312-win_amd64.whl", hash = "sha256:cd96f29b4bab1d42fa6e3d008711c75e0f79e94e06827330160e3a304227f150" },
]

[[package]]
name = "idna"
version = "3.11"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/idna/3.11/idna-3.11.tar.gz", hash = "sha256:795dafcc9c04ed0c1fb032c2aa73654d8e8c5023a7df64a53f39190ada629902" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/idna/3.11/idna-3.11-py3-none-any.whl", hash = "sha256:771a87f49d9defaf64091e6e6fe9c18d4833f140bd19464795bc32d966ca37ea" },
]

[[package]]
name = "iniconfig"
version = "2.3.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/iniconfig/2.3.0/iniconfig-2.3.0.tar.gz", hash = "sha256:c76315c77db068650d49c5b56314774a7804df16fee4402c1f19d6d15d8c4730" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/iniconfig/2.3.0/iniconfig-2.3.0-py3-none-any.whl", hash = "sha256:f631c04d2c48c52b84d0d0549c99ff3859c98df65b3101406327ecc7d53fbf12" },
]

[[package]]
name = "itsdangerous"
version = "2.2.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/itsdangerous/2.2.0/itsdangerous-2.2.0.tar.gz", hash = "sha256:e0050c0b7da1eea53ffaf149c0cfbb5c6e2e2b69c4bef22c81fa6eb73e5f6173" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/itsdangerous/2.2.0/itsdangerous-2.2.0-py3-none-any.whl", hash = "sha256:c6242fc49e35958c8b15141343aa660db5fc54d4f13a1db01a3f5891b98700ef" },
]

[[package]]
name = "jinja2"
version = "3.1.6"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "markupsafe" },
]
sdist = { url = "nexus/repository/pypi-public/packages/jinja2/3.1.6/jinja2-3.1.6.tar.gz", hash = "sha256:0137fb05990d35f1275a587e9aee6d56da821fc83491a0fb838183be43f66d6d" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/jinja2/3.1.6/jinja2-3.1.6-py3-none-any.whl", hash = "sha256:85ece4451f492d0c13c5dd7c13a64681a86afae63a5f347908daf103ce6d2f67" },
]

[[package]]
name = "jsonschema"
version = "4.26.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "attrs" },
    { name = "jsonschema-specifications" },
    { name = "referencing" },
    { name = "rpds-py" },
]
sdist = { url = "nexus/repository/pypi-public/packages/jsonschema/4.26.0/jsonschema-4.26.0.tar.gz", hash = "sha256:0c26707e2efad8aa1bfc5b7ce170f3fccc2e4918ff85989ba9ffa9facb2be326" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/jsonschema/4.26.0/jsonschema-4.26.0-py3-none-any.whl", hash = "sha256:d489f15263b8d200f8387e64b4c3a75f06629559fb73deb8fdfb525f2dab50ce" },
]

[[package]]
name = "jsonschema-specifications"
version = "2025.9.1"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "referencing" },
]
sdist = { url = "nexus/repository/pypi-public/packages/jsonschema-specifications/2025.9.1/jsonschema_specifications-2025.9.1.tar.gz", hash = "sha256:b540987f239e745613c7a9176f3edb72b832a4ac465cf02712288397832b5e8d" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/jsonschema-specifications/2025.9.1/jsonschema_specifications-2025.9.1-py3-none-any.whl", hash = "sha256:98802fee3a11ee76ecaca44429fda8a41bff98b00a0f2838151b113f210cc6fe" },
]

[[package]]
name = "markupsafe"
version = "3.0.3"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3.tar.gz", hash = "sha256:722695808f4b6457b320fdc131280796bdceb04ab50fe1795cd540799ebe1698" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:d53197da72cc091b024dd97249dfc7794d6a56530370992a5e1a08983ad9230e" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:1872df69a4de6aead3491198eaf13810b565bdbeec3ae2dc8780f14458ec73ce" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:3a7e8ae81ae39e62a41ec302f972ba6ae23a5c5396c8e60113e9066ef893da0d" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:d6dd0be5b5b189d31db7cda48b91d7e0a9795f31430b7f271219ab30f1d3ac9d" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:94c6f0bb423f739146aec64595853541634bde58b2135f27f61c1ffd1cd4d16a" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:be8813b57049a7dc738189df53d69395eba14fb99345e0a5994914a3864c8a4b" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:83891d0e9fb81a825d9a6d61e3f07550ca70a076484292a70fde82c4b807286f" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:77f0643abe7495da77fb436f50f8dab76dbc6e5fd25d39589a0f1fe6548bfa2b" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-win32.whl", hash = "sha256:d88b440e37a16e651bda4c7c2b930eb586fd15ca7406cb39e211fcff3bf3017d" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-win_amd64.whl", hash = "sha256:26a5784ded40c9e318cfc2bdb30fe164bdb8665ded9cd64d500a34fb42067b1c" },
    { url = "nexus/repository/pypi-public/packages/markupsafe/3.0.3/markupsafe-3.0.3-cp312-cp312-win_arm64.whl", hash = "sha256:35add3b638a5d900e807944a078b51922212fb3dedb01633a8defc4b01a3c85f" },
]

[[package]]
name = "multidict"
version = "6.7.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1.tar.gz", hash = "sha256:ec6652a1bee61c53a3e5776b6049172c53b6aaba34f18c9ad04f82712bac623d" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:a90f75c956e32891a4eda3639ce6dd86e87105271f43d43442a3aedf3cddf172" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:3fccb473e87eaa1382689053e4a4618e7ba7b9b9b8d6adf2027ee474597128cd" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:b0fa96985700739c4c7853a43c0b3e169360d6855780021bfc6d0f1ce7c123e7" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux1_i686.manylinux_2_28_i686.manylinux_2_5_i686.whl", hash = "sha256:cb2a55f408c3043e42b40cc8eecd575afa27b7e0b956dfb190de0f8499a57a53" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:eb0ce7b2a32d09892b3dd6cc44877a0d02a33241fafca5f25c8b6b62374f8b75" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:c3a32d23520ee37bf327d1e1a656fec76a2edd5c038bf43eddfa0572ec49c60b" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:9c90fed18bffc0189ba814749fdcc102b536e83a9f738a9003e569acd540a733" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:da62917e6076f512daccfbbde27f46fed1c98fee202f0559adec8ee0de67f71a" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:bfde23ef6ed9db7eaee6c37dcec08524cb43903c60b285b172b6c094711b3961" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3758692429e4e32f1ba0df23219cd0b4fc0a52f476726fff9337d1a57676a582" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:398c1478926eca669f2fd6a5856b6de9c0acf23a2cb59a14c0ba5844fa38077e" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:c102791b1c4f3ab36ce4101154549105a53dc828f016356b3e3bcae2e3a039d3" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:a088b62bd733e2ad12c50dad01b7d0166c30287c166e137433d3b410add807a6" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:3d51ff4785d58d3f6c91bdbffcb5e1f7ddfda557727043aa20d20ec4f65e324a" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:fc5907494fccf3e7d3f94f95c91d6336b092b5fc83811720fae5e2765890dfba" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-win32.whl", hash = "sha256:28ca5ce2fd9716631133d0e9a9b9a745ad7f60bac2bccafb56aa380fc0b6c511" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-win_amd64.whl", hash = "sha256:fcee94dfbd638784645b066074b338bc9cc155d4b4bffa4adce1615c5a426c19" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-cp312-cp312-win_arm64.whl", hash = "sha256:ba0a9fb644d0c1a2194cf7ffb043bd852cea63a57f66fbd33959f7dae18517bf" },
    { url = "nexus/repository/pypi-public/packages/multidict/6.7.1/multidict-6.7.1-py3-none-any.whl", hash = "sha256:55d97cc6dae627efa6a6e548885712d4864b81110ac76fa4e534c03819fa4a56" },
]

[[package]]
name = "narwhals"
version = "2.22.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/narwhals/2.22.1/narwhals-2.22.1.tar.gz", hash = "sha256:d62920805a0a43b7ff8b54b0c0d3142d796f8a9301836ada37e573d6a33cbcd9" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/narwhals/2.22.1/narwhals-2.22.1-py3-none-any.whl", hash = "sha256:60567d774edf77db53906f89d9fbd164e66e56d66d388e1e6990f17ac33cfb53" },
]

[[package]]
name = "numpy"
version = "2.4.4"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4.tar.gz", hash = "sha256:2d390634c5182175533585cc89f3608a4682ccb173cc9bb940b2881c8d6f8fa0" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:15716cfef24d3a9762e3acdf87e27f58dc823d1348f765bbea6bef8c639bfa1b" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:23cbfd4c17357c81021f21540da84ee282b9c8fba38a03b7b9d09ba6b951421e" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-macosx_14_0_arm64.whl", hash = "sha256:8b3b60bb7cba2c8c81837661c488637eee696f59a877788a396d33150c35d842" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-macosx_14_0_x86_64.whl", hash = "sha256:e4a010c27ff6f210ff4c6ef34394cd61470d01014439b192ec22552ee867f2a8" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:f9e75681b59ddaa5e659898085ae0eaea229d054f2ac0c7e563a62205a700121" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:81f4a14bee47aec54f883e0cad2d73986640c1590eb9bfaaba7ad17394481e6e" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:62d6b0f03b694173f9fcb1fb317f7222fd0b0b103e784c6549f5e53a27718c44" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:fbc356aae7adf9e6336d336b9c8111d390a05df88f1805573ebb0807bd06fd1d" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-win32.whl", hash = "sha256:0d35aea54ad1d420c812bfa0385c71cd7cc5bcf7c65fed95fc2cd02fe8c79827" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-win_amd64.whl", hash = "sha256:b5f0362dc928a6ecd9db58868fca5e48485205e3855957bdedea308f8672ea4a" },
    { url = "nexus/repository/pypi-public/packages/numpy/2.4.4/numpy-2.4.4-cp312-cp312-win_arm64.whl", hash = "sha256:846300f379b5b12cc769334464656bc882e0735d27d9726568bc932fdc49d5ec" },
]

[[package]]
name = "packaging"
version = "26.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/packaging/26.1/packaging-26.1.tar.gz", hash = "sha256:f042152b681c4bfac5cae2742a55e103d27ab2ec0f3d88037136b6bfe7c9c5de" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/packaging/26.1/packaging-26.1-py3-none-any.whl", hash = "sha256:5d9c0669c6285e491e0ced2eee587eaf67b670d94a19e94e3984a481aba6802f" },
]

[[package]]
name = "pandas"
version = "2.3.3"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "numpy" },
    { name = "python-dateutil" },
    { name = "pytz" },
    { name = "tzdata" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3.tar.gz", hash = "sha256:e05e1af93b977f7eafa636d043f9f94c7ee3ac81af99c13508215942e64c993b" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:6d21f6d74eb1725c2efaa71a2bfc661a0689579b58e9c0ca58a739ff0b002b53" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:3fd2f887589c7aa868e02632612ba39acb0b8948faf5cc58f0850e165bd46f35" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-manylinux_2_24_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:ecaf1e12bdc03c86ad4a7ea848d66c685cb6851d807a26aa245ca3d2017a1908" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:b3d11d2fda7eb164ef27ffc14b4fcab16a80e1ce67e9f57e19ec0afaf715ba89" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:a68e15f780eddf2b07d242e17a04aa187a7ee12b40b930bfdd78070556550e98" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:371a4ab48e950033bcf52b6527eccb564f52dc826c02afd9a1bc0ab731bba084" },
    { url = "nexus/repository/pypi-public/packages/pandas/2.3.3/pandas-2.3.3-cp312-cp312-win_amd64.whl", hash = "sha256:a16dcec078a01eeef8ee61bf64074b4e524a2a3f4b3be9326420cabe59c4778b" },
]

[[package]]
name = "pillow"
version = "12.2.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0.tar.gz", hash = "sha256:a830b1a40919539d07806aa58e1b114df53ddd43213d9c8b75847eee6c0182b5" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:2d192a155bbcec180f8564f693e6fd9bccff5a7af9b32e2e4bf8c9c69dbad6b5" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:f3f40b3c5a968281fd507d519e444c35f0ff171237f4fdde090dd60699458421" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:03e7e372d5240cc23e9f07deca4d775c0817bffc641b01e9c3af208dbd300987" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:b86024e52a1b269467a802258c25521e6d742349d760728092e1bc2d135b4d76" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:7371b48c4fa448d20d2714c9a1f775a81155050d383333e0a6c15b1123dda005" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:62f5409336adb0663b7caa0da5c7d9e7bdbaae9ce761d34669420c2a801b2780" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:01afa7cf67f74f09523699b4e88c73fb55c13346d212a59a2db1f86b0a63e8c5" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:fc3d34d4a8fbec3e88a79b92e5465e0f9b842b628675850d860b8bd300b159f5" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-win32.whl", hash = "sha256:58f62cc0f00fd29e64b29f4fd923ffdb3859c9f9e6105bfc37ba1d08994e8940" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-win_amd64.whl", hash = "sha256:7f84204dee22a783350679a0333981df803dac21a0190d706a50475e361c93f5" },
    { url = "nexus/repository/pypi-public/packages/pillow/12.2.0/pillow-12.2.0-cp312-cp312-win_arm64.whl", hash = "sha256:af73337013e0b3b46f175e79492d96845b16126ddf79c438d7ea7ff27783a414" },
]

[[package]]
name = "plotly"
version = "6.8.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "narwhals" },
    { name = "packaging" },
]
sdist = { url = "nexus/repository/pypi-public/packages/plotly/6.8.0/plotly-6.8.0.tar.gz", hash = "sha256:e088e7ddc68d4f70e3d66659224727a45296d71d2b8284181862d3d8f1f0d88f" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/plotly/6.8.0/plotly-6.8.0-py3-none-any.whl", hash = "sha256:13c5c4a0f70b74cab1913eda0de49b826df5931708eb6f9c3010040614700ec8" },
]

[[package]]
name = "pluggy"
version = "1.6.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pluggy/1.6.0/pluggy-1.6.0.tar.gz", hash = "sha256:7dcc130b76258d33b90f61b658791dede3486c3e6bfb003ee5c9bfb396dd22f3" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pluggy/1.6.0/pluggy-1.6.0-py3-none-any.whl", hash = "sha256:e920276dd6813095e9377c0bc5566d94c932c33b27a3e3945d8389c374dd4746" },
]

[[package]]
name = "propcache"
version = "0.5.2"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2.tar.gz", hash = "sha256:01c4fc7480cd0598bb4b57022df55b9ca296da7fc5a8760bd8451a7e63a7d427" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:806719138ecd720339a12410fb9614ac9b2b2d3a5fdf8235d56981c36f4039ba" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:db2b80ea58eab4f86b2beec3cc8b39e8ff9276ac20e96b7cce43c8ae84cd6b5a" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:e5cbfac9f61484f7e9f3597775500cd3ebe8274e9b050c38f9525c77c97520bf" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:5dbc581d2814337da56222fab8dc5f161cd798a434e49bac27930aaef798e144" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:857187f381f88c8e2fa2fe56ab94879d011b883d5a2ee5a1b60a8cd2a06846d9" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:178b4a2cdaac1818e2bf1c5a99b94383fa73ea5382e032a48dec07dc5668dc42" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:6f328175a2cde1f0ff2c4ed8ce968b9dcfb55f3a7153f39e2957ed994da13476" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:5671d09a36b06d0fd4a3da0fccbcae360e9b1570924171a15e9e0997f0249fba" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:80168e2ebe4d3ec6599d10ad8f520304ae1cad9b6c5a95372aef1b66b7bfb53a" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:45f11346f884bc47444f6e6647131055844134c3175b629f84952e2b5cd62b64" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:8e778ebd44ef4f66ed60a0416b06b489687db264a9c0b3620362f26489492913" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:c0cb9ed24c8964e172768d455a38254c2dd8a552905729ce006cad3d3dda59b1" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:1d1ad32d9d4355e2be65574fd0bfd3677e7066b009cd5b9b2dee8aa6a6393b33" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:c80f4ba3e8f00189165999a742ee526ebeccedf6c3f7beb0c7df821e9772435a" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-win32.whl", hash = "sha256:8c7972d8f193740d9175f0998ab38717e6cd322d5935c5b0fef8c0d323fd9031" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-win_amd64.whl", hash = "sha256:d9ee8826a7d47863a08ac44e1a5f611a462eefc3a194b492da242128bec75b42" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-cp312-cp312-win_arm64.whl", hash = "sha256:2800a4a8ead6b28cccd1ec54b59346f0def7922ee1c7598e8499c733cfbb7c84" },
    { url = "nexus/repository/pypi-public/packages/propcache/0.5.2/propcache-0.5.2-py3-none-any.whl", hash = "sha256:be1ddfcbb376e3de5d2e2db1d58d6d67463e6b4f9f040c000de8e300295465fe" },
]

[[package]]
name = "protobuf"
version = "6.33.6"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6.tar.gz", hash = "sha256:a6768d25248312c297558af96a9f9c929e8c4cee0659cb07e780731095f38135" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp310-abi3-win32.whl", hash = "sha256:7d29d9b65f8afef196f8334e80d6bc1d5d4adedb449971fefd3723824e6e77d3" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp310-abi3-win_amd64.whl", hash = "sha256:0cd27b587afca21b7cfa59a74dcbd48a50f0a6400cfb59391340ad729d91d326" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp39-abi3-macosx_10_9_universal2.whl", hash = "sha256:9720e6961b251bde64edfdab7d500725a2af5280f3f4c87e57c0208376aa8c3a" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp39-abi3-manylinux2014_aarch64.whl", hash = "sha256:e2afbae9b8e1825e3529f88d514754e094278bb95eadc0e199751cdd9a2e82a2" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp39-abi3-manylinux2014_s390x.whl", hash = "sha256:c96c37eec15086b79762ed265d59ab204dabc53056e3443e702d2681f4b39ce3" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-cp39-abi3-manylinux2014_x86_64.whl", hash = "sha256:e9db7e292e0ab79dd108d7f1a94fe31601ce1ee3f7b79e0692043423020b0593" },
    { url = "nexus/repository/pypi-public/packages/protobuf/6.33.6/protobuf-6.33.6-py3-none-any.whl", hash = "sha256:77179e006c476e69bf8e8ce866640091ec42e1beb80b213c3900006ecfba6901" },
]

[[package]]
name = "py4j"
version = "0.10.9.9"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/py4j/0.10.9.9/py4j-0.10.9.9.tar.gz", hash = "sha256:f694cad19efa5bd1dee4f3e5270eb406613c974394035e5bfc4ec1aba870b879" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/py4j/0.10.9.9/py4j-0.10.9.9-py2.py3-none-any.whl", hash = "sha256:c7c26e4158defb37b0bb124933163641a2ff6e3a3913f7811b0ddbe07ed61533" },
]

[[package]]
name = "pyarrow"
version = "24.0.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0.tar.gz", hash = "sha256:85fe721a14dd823aca09127acbb06c3ca723efbd436c004f16bca601b04dcc83" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-macosx_12_0_arm64.whl", hash = "sha256:6233c9ed9ab9d1db47de57d9753256d9dcffbf42db341576099f0fd9f6bf4810" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-macosx_12_0_x86_64.whl", hash = "sha256:f7616236ec1bc2b15bfdec22a71ab38851c86f8f05ff64f379e1278cf20c634a" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-manylinux_2_28_aarch64.whl", hash = "sha256:1617043b99bd33e5318ae18eb2919af09c71322ef1ca46566cdafc6e6712fb66" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-manylinux_2_28_x86_64.whl", hash = "sha256:6165461f55ef6314f026de6638d661188e3455d3ec49834556a0ebbdbace18bb" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3b13dedfe76a0ad2d1d859b0811b53827a4e9d93a0bcb05cf59333ab4980cc7e" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:25ea65d868eb04015cd18e6df2fbe98f07e5bda2abefabcb88fce39a947716f6" },
    { url = "nexus/repository/pypi-public/packages/pyarrow/24.0.0/pyarrow-24.0.0-cp312-cp312-win_amd64.whl", hash = "sha256:295f0a7f2e242dabd513737cf076007dc5b2d59237e3eca37b05c0c6446f3826" },
]

[[package]]
name = "pyasn1"
version = "0.6.3"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pyasn1/0.6.3/pyasn1-0.6.3.tar.gz", hash = "sha256:697a8ecd6d98891189184ca1fa05d1bb00e2f84b5977c481452050549c8a72cf" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pyasn1/0.6.3/pyasn1-0.6.3-py3-none-any.whl", hash = "sha256:a80184d120f0864a52a073acc6fc642847d0be408e7c7252f31390c0f4eadcde" },
]

[[package]]
name = "pyasn1-modules"
version = "0.4.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "pyasn1" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pyasn1-modules/0.4.2/pyasn1_modules-0.4.2.tar.gz", hash = "sha256:677091de870a80aae844b1ca6134f54652fa2c8c5a52aa396440ac3106e941e6" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pyasn1-modules/0.4.2/pyasn1_modules-0.4.2-py3-none-any.whl", hash = "sha256:29253a9207ce32b64c3ac6600edc75368f98473906e8fd1043bd6b5b1de2c14a" },
]

[[package]]
name = "pycparser"
version = "3.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pycparser/3.0/pycparser-3.0.tar.gz", hash = "sha256:600f49d217304a5902ac3c37e1281c9fe94e4d0489de643a9504c5cdfdfc6b29" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pycparser/3.0/pycparser-3.0-py3-none-any.whl", hash = "sha256:b727414169a36b7d524c1c3e31839a521725078d7b2ff038656844266160a992" },
]

[[package]]
name = "pydantic"
version = "2.13.3"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "annotated-types" },
    { name = "pydantic-core" },
    { name = "typing-extensions" },
    { name = "typing-inspection" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pydantic/2.13.3/pydantic-2.13.3.tar.gz", hash = "sha256:af09e9d1d09f4e7fe37145c1f577e1d61ceb9a41924bf0094a36506285d0a84d" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pydantic/2.13.3/pydantic-2.13.3-py3-none-any.whl", hash = "sha256:6db14ac8dfc9a1e57f87ea2c0de670c251240f43cb0c30a5130e9720dc612927" },
]

[[package]]
name = "pydantic-core"
version = "2.46.3"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3.tar.gz", hash = "sha256:41c178f65b8c29807239d47e6050262eb6bf84eb695e41101e62e38df4a5bc2c" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-macosx_10_12_x86_64.whl", hash = "sha256:b11b59b3eee90a80a36701ddb4576d9ae31f93f05cb9e277ceaa09e6bf074a67" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:af8653713055ea18a3abc1537fe2ebc42f5b0bbb768d1eb79fd74eb47c0ac089" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:75a519dab6d63c514f3a81053e5266c549679e4aa88f6ec57f2b7b854aceb1b0" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:a6cd87cb1575b1ad05ba98894c5b5c96411ef678fa2f6ed2576607095b8d9789" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f80a55484b8d843c8ada81ebf70a682f3f00a3d40e378c06cf17ecb44d280d7d" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:3861f1731b90c50a3266316b9044f5c9b405eecb8e299b0a7120596334e4fe9c" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fb528e295ed31570ac3dcc9bfdd6e0150bc11ce6168ac87a8082055cf1a67395" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_31_riscv64.whl", hash = "sha256:367508faa4973b992b271ba1494acaab36eb7e8739d1e47be5035fb1ea225396" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:5ad3c826fe523e4becf4fe39baa44286cff85ef137c729a2c5e269afbfd0905d" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:ec638c5d194ef8af27db69f16c954a09797c0dc25015ad6123eb2c73a4d271ca" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-musllinux_1_1_armv7l.whl", hash = "sha256:28ed528c45446062ee66edb1d33df5d88828ae167de76e773a3c7f64bd14e976" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:aed19d0c783886d5bd86d80ae5030006b45e28464218747dcf83dabfdd092c7b" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-win32.whl", hash = "sha256:06d5d8820cbbdb4147578c1fe7ffcd5b83f34508cb9f9ab76e807be7db6ff0a4" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-win_amd64.whl", hash = "sha256:c3212fda0ee959c1dd04c60b601ec31097aaa893573a3a1abd0a47bcac2968c1" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-cp312-cp312-win_arm64.whl", hash = "sha256:f1f8338dd7a7f31761f1f1a3c47503a9a3b34eea3c8b01fa6ee96408affb5e72" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-graalpy312-graalpy250_312_native-macosx_10_12_x86_64.whl", hash = "sha256:b12dd51f1187c2eb489af8e20f880362db98e954b54ab792fa5d92e8bcc6b803" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-graalpy312-graalpy250_312_native-macosx_11_0_arm64.whl", hash = "sha256:f00a0961b125f1a47af7bcc17f00782e12f4cd056f83416006b30111d941dfa3" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-graalpy312-graalpy250_312_native-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:57697d7c056aca4bbb680200f96563e841a6386ac1129370a0102592f4dddff5" },
    { url = "nexus/repository/pypi-public/packages/pydantic-core/2.46.3/pydantic_core-2.46.3-graalpy312-graalpy250_312_native-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fd35aa21299def8db7ef4fe5c4ff862941a9a158ca7b63d61e66fe67d30416b4" },
]

[[package]]
name = "pydeck"
version = "0.9.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "jinja2" },
    { name = "numpy" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pydeck/0.9.2/pydeck-0.9.2.tar.gz", hash = "sha256:c10d9035e81ead6385264cac8d19402471f6866a15ca1f7df1400f52142bcf87" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pydeck/0.9.2/pydeck-0.9.2-py2.py3-none-any.whl", hash = "sha256:8213dfeacc5f6bfe6825f61c8ee34e3850e8a31fc43924379ec98edb34a75b25" },
]

[[package]]
name = "pygments"
version = "2.20.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pygments/2.20.0/pygments-2.20.0.tar.gz", hash = "sha256:6757cd03768053ff99f3039c1a36d6c0aa0b263438fcab17520b30a303a82b5f" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pygments/2.20.0/pygments-2.20.0-py3-none-any.whl", hash = "sha256:81a9e26dd42fd28a23a2d169d86d7ac03b46e2f8b59ed4698fb4785f946d0176" },
]

[[package]]
name = "pytest"
version = "9.0.3"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "colorama", marker = "sys_platform == 'win32'" },
    { name = "iniconfig" },
    { name = "packaging" },
    { name = "pluggy" },
    { name = "pygments" },
]
sdist = { url = "nexus/repository/pypi-public/packages/pytest/9.0.3/pytest-9.0.3.tar.gz", hash = "sha256:b86ada508af81d19edeb213c681b1d48246c1a91d304c6c81a427674c17eb91c" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pytest/9.0.3/pytest-9.0.3-py3-none-any.whl", hash = "sha256:2c5efc453d45394fdd706ade797c0a81091eccd1d6e4bccfcd476e2b8e0ab5d9" },
]

[[package]]
name = "python-dateutil"
version = "2.9.0.post0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "six" },
]
sdist = { url = "nexus/repository/pypi-public/packages/python-dateutil/2.9.0.post0/python-dateutil-2.9.0.post0.tar.gz", hash = "sha256:37dd54208da7e1cd875388217d5e00ebd4179249f90fb72437e91a35459a0ad3" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/python-dateutil/2.9.0.post0/python_dateutil-2.9.0.post0-py2.py3-none-any.whl", hash = "sha256:a8b2bc7bffae282281c8140a97d3aa9c14da0b136dfe83f850eea9a5f7470427" },
]

[[package]]
name = "python-multipart"
version = "0.0.32"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/python-multipart/0.0.32/python_multipart-0.0.32.tar.gz", hash = "sha256:be54b7f3fa167bb83e4fcd936b887b708f4e57fe75911c02aebf53efaf8d938e" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/python-multipart/0.0.32/python_multipart-0.0.32-py3-none-any.whl", hash = "sha256:ff6d3f776f16878c894e52e107296ffc890e913c611b1a4ec6c44e2821fe2e23" },
]

[[package]]
name = "pytz"
version = "2026.1.post1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pytz/2026.1.post1/pytz-2026.1.post1.tar.gz", hash = "sha256:3378dde6a0c3d26719182142c56e60c7f9af7e968076f31aae569d72a0358ee1" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pytz/2026.1.post1/pytz-2026.1.post1-py2.py3-none-any.whl", hash = "sha256:f2fd16142fda348286a75e1a524be810bb05d444e5a081f37f7affc635035f7a" },
]

[[package]]
name = "pyyaml"
version = "6.0.3"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3.tar.gz", hash = "sha256:d76623373421df22fb4cf8817020cbb7ef15c725b9d5e45f17e189bfc384190f" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:7f047e29dcae44602496db43be01ad42fc6f1cc0d8cd6c83d342306c32270196" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:fc09d0aa354569bc501d4e787133afc08552722d3ab34836a80547331bb5d4a0" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:9149cad251584d5fb4981be1ecde53a1ca46c891a79788c0df828d2f166bda28" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:5fdec68f91a0c6739b380c83b951e2c72ac0197ace422360e6d5a959d8d97b2c" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:ba1cc08a7ccde2d2ec775841541641e4548226580ab850948cbfda66a1befcdc" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:8dc52c23056b9ddd46818a57b78404882310fb473d63f17b07d5c40421e47f8e" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:41715c910c881bc081f1e8872880d3c650acf13dfa8214bad49ed4cede7c34ea" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-win32.whl", hash = "sha256:96b533f0e99f6579b3d4d4995707cf36df9100d67e0c8303a0c55b27b5f99bc5" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-win_amd64.whl", hash = "sha256:5fcd34e47f6e0b794d17de1b4ff496c00986e1c83f7ab2fb8fcfe9616ff7477b" },
    { url = "nexus/repository/pypi-public/packages/pyyaml/6.0.3/pyyaml-6.0.3-cp312-cp312-win_arm64.whl", hash = "sha256:64386e5e707d03a7e172c0701abfb7e10f0fb753ee1d773128192742712a98fd" },
]

[[package]]
name = "referencing"
version = "0.37.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "attrs" },
    { name = "rpds-py" },
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/referencing/0.37.0/referencing-0.37.0.tar.gz", hash = "sha256:44aefc3142c5b842538163acb373e24cce6632bd54bdb01b21ad5863489f50d8" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/referencing/0.37.0/referencing-0.37.0-py3-none-any.whl", hash = "sha256:381329a9f99628c9069361716891d34ad94af76e461dcb0335825aecc7692231" },
]

[[package]]
name = "requests"
version = "2.33.1"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "certifi" },
    { name = "charset-normalizer" },
    { name = "idna" },
    { name = "urllib3" },
]
sdist = { url = "nexus/repository/pypi-public/packages/requests/2.33.1/requests-2.33.1.tar.gz", hash = "sha256:18817f8c57c6263968bc123d237e3b8b08ac046f5456bd1e307ee8f4250d3517" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/requests/2.33.1/requests-2.33.1-py3-none-any.whl", hash = "sha256:4e6d1ef462f3626a1f0a0a9c42dd93c63bad33f9f1c1937509b8c5c8718ab56a" },
]

[[package]]
name = "rpds-py"
version = "2026.5.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1.tar.gz", hash = "sha256:07b24fea40541e28570e5b795a4a38fbdcd12550c06bd0748005ecc8116ca256" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-macosx_10_12_x86_64.whl", hash = "sha256:3abe24a66e57adcfa645d718063a5fa5103ecc71ddbf26d78af8f9368018ff1d" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:58b1d94308ddf0b1982f61f2eb54bf92997c9ece8a8093ef014250f4a517906c" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0fa92420128dadce7f54bd73ba1825a273e9268fe9e35dbf7e6362890efa4e08" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:ca653c6546386227cd9800d1bef6a348099acf8db4250341da6d90f663d6dfcb" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:66c93681c4729e4e3ecba31b8179fae083ff3118841672835140338b4b9867c1" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:40ff257542e04796880e011e15cd4dc21c2599975df2aaa8f2c8495ca574e1a5" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b6825cc329b290e93c5f6a9be2393118a763f6ccf6abd83704e0c102ca583644" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_31_riscv64.whl", hash = "sha256:de42116e69cb53b911cc34aee5ab98f36c597b822545045d49e938818b99e5e4" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:c0f920015df2a504bebaba6d4c31ccf3fcf942f92655c086da30b671aad19aa6" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:0408a24e44feb919423dc6d9da677cb5cddb894d2ca9e763967d156d9c60fab4" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:cea68bcd53467561ae2f96a6bdad1544299ba97b5b0ddcd5ac3d376e5c781c24" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:4be8b1d2a705cc37d08256004e1d07de143fa0075c8e85a3df020b776f62b732" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-win32.whl", hash = "sha256:6736718bd4fc49cbcb538ba30516fdbef161522acefb739657d48b97bd864fed" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-win_amd64.whl", hash = "sha256:0a7d1eec967df0e9b22614a5e177622e0c89611d03727fa0cb48e45028907870" },
    { url = "nexus/repository/pypi-public/packages/rpds-py/2026.5.1/rpds_py-2026.5.1-cp312-cp312-win_arm64.whl", hash = "sha256:1841d067089e117142d79b98aa0df2f08b52f2ecc1819dd2700636c0db74a473" },
]

[[package]]
name = "setuptools"
version = "82.0.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/setuptools/82.0.1/setuptools-82.0.1.tar.gz", hash = "sha256:7d872682c5d01cfde07da7bccc7b65469d3dca203318515ada1de5eda35efbf9" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/setuptools/82.0.1/setuptools-82.0.1-py3-none-any.whl", hash = "sha256:a59e362652f08dcd477c78bb6e7bd9d80a7995bc73ce773050228a348ce2e5bb" },
]

[[package]]
name = "six"
version = "1.17.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/six/1.17.0/six-1.17.0.tar.gz", hash = "sha256:ff70335d468e7eb6ec65b95b99d3a2836546063f63acc5171de367e834932a81" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/six/1.17.0/six-1.17.0-py2.py3-none-any.whl", hash = "sha256:4721f391ed90541fddacab5acf947aa0d3dc7d27b2e1e8eda2be8970586c3274" },
]

[[package]]
name = "smmap"
version = "5.0.3"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/smmap/5.0.3/smmap-5.0.3.tar.gz", hash = "sha256:4d9debb8b99007ae47165abc08670bd74cb74b5227dda7f643eccc4e9eb5642c" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/smmap/5.0.3/smmap-5.0.3-py3-none-any.whl", hash = "sha256:c106e05d5a61449cf6ba9a1e650227ecfb141590d2a98412103ff35d89fc7b2f" },
]

[[package]]
name = "starlette"
version = "1.3.1"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "anyio" },
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/starlette/1.3.1/starlette-1.3.1.tar.gz", hash = "sha256:05d0213193f2fbaae60e2ecb593b4add4262ad4e46536b54abe36f11a71724e0" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/starlette/1.3.1/starlette-1.3.1-py3-none-any.whl", hash = "sha256:c7372aae11c3c3f26a42df7bd626cec2f47d03483d261d369516a615a53714c6" },
]

[[package]]
name = "streamlit"
version = "1.58.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "altair" },
    { name = "anyio" },
    { name = "blinker" },
    { name = "cachetools" },
    { name = "click" },
    { name = "gitpython" },
    { name = "httptools" },
    { name = "itsdangerous" },
    { name = "numpy" },
    { name = "packaging" },
    { name = "pandas" },
    { name = "pillow" },
    { name = "protobuf" },
    { name = "pyarrow" },
    { name = "pydeck" },
    { name = "python-multipart" },
    { name = "requests" },
    { name = "starlette" },
    { name = "tenacity" },
    { name = "toml" },
    { name = "typing-extensions" },
    { name = "uvicorn" },
    { name = "watchdog", marker = "sys_platform != 'darwin'" },
    { name = "websockets" },
]
sdist = { url = "nexus/repository/pypi-public/packages/streamlit/1.58.0/streamlit-1.58.0.tar.gz", hash = "sha256:78a22e7085b053af7ce544442bf4b670771e68c509ba1bdaa056ba0708f49c3d" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/streamlit/1.58.0/streamlit-1.58.0-py3-none-any.whl", hash = "sha256:4ca8a7afc5bd16a5f176ccf4be1e34e8121cad0240becd127fb58a103ea3178d" },
]

[[package]]
name = "tenacity"
version = "9.1.4"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/tenacity/9.1.4/tenacity-9.1.4.tar.gz", hash = "sha256:adb31d4c263f2bd041081ab33b498309a57c77f9acf2db65aadf0898179cf93a" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/tenacity/9.1.4/tenacity-9.1.4-py3-none-any.whl", hash = "sha256:6095a360c919085f28c6527de529e76a06ad89b23659fa881ae0649b867a9d55" },
]

[[package]]
name = "toml"
version = "0.10.2"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/toml/0.10.2/toml-0.10.2.tar.gz", hash = "sha256:b3bda1d108d5dd99f4a20d24d9c348e91c4db7ab1b749200bded2f839ccbe68f" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/toml/0.10.2/toml-0.10.2-py2.py3-none-any.whl", hash = "sha256:806143ae5bfb6a3c6e736a764057db0e6a0e05e338b5630894a5f779cabb4f9b" },
]

[[package]]
name = "typing-extensions"
version = "4.15.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/typing-extensions/4.15.0/typing_extensions-4.15.0.tar.gz", hash = "sha256:0cea48d173cc12fa28ecabc3b837ea3cf6f38c6d1136f85cbaaf598984861466" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/typing-extensions/4.15.0/typing_extensions-4.15.0-py3-none-any.whl", hash = "sha256:f0fa19c6845758ab08074a0cfa8b7aecb71c999ca73d62883bc25cc018c4e548" },
]

[[package]]
name = "typing-inspection"
version = "0.4.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "typing-extensions" },
]
sdist = { url = "nexus/repository/pypi-public/packages/typing-inspection/0.4.2/typing_inspection-0.4.2.tar.gz", hash = "sha256:ba561c48a67c5958007083d386c3295464928b01faa735ab8547c5692e87f464" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/typing-inspection/0.4.2/typing_inspection-0.4.2-py3-none-any.whl", hash = "sha256:4ed1cacbdc298c220f1bd249ed5287caa16f34d44ef4e9c3d0cbad5b521545e7" },
]

[[package]]
name = "tzdata"
version = "2026.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/tzdata/2026.1/tzdata-2026.1.tar.gz", hash = "sha256:67658a1903c75917309e753fdc349ac0efd8c27db7a0cb406a25be4840f87f98" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/tzdata/2026.1/tzdata-2026.1-py2.py3-none-any.whl", hash = "sha256:4b1d2be7ac37ceafd7327b961aa3a54e467efbdb563a23655fbfe0d39cfc42a9" },
]

[[package]]
name = "urllib3"
version = "2.6.3"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/urllib3/2.6.3/urllib3-2.6.3.tar.gz", hash = "sha256:1b62b6884944a57dbe321509ab94fd4d3b307075e0c2eae991ac71ee15ad38ed" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/urllib3/2.6.3/urllib3-2.6.3-py3-none-any.whl", hash = "sha256:bf272323e553dfb2e87d9bfd225ca7b0f467b919d7bbd355436d3fd37cb0acd4" },
]

[[package]]
name = "uuid6"
version = "2025.0.1"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/uuid6/2025.0.1/uuid6-2025.0.1.tar.gz", hash = "sha256:cd0af94fa428675a44e32c5319ec5a3485225ba2179eefcf4c3f205ae30a81bd" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/uuid6/2025.0.1/uuid6-2025.0.1-py3-none-any.whl", hash = "sha256:80530ce4d02a93cdf82e7122ca0da3ebbbc269790ec1cb902481fa3e9cc9ff99" },
]

[[package]]
name = "uvicorn"
version = "0.49.0"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "click" },
    { name = "h11" },
]
sdist = { url = "nexus/repository/pypi-public/packages/uvicorn/0.49.0/uvicorn-0.49.0.tar.gz", hash = "sha256:ebf4271aa580d9de97f93192d4595176df6e91f9aae919ca73e4fc07df1e66a3" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/uvicorn/0.49.0/uvicorn-0.49.0-py3-none-any.whl", hash = "sha256:ba3d14c3ee7e41c6c654c46c9eb489d33213cdd30aa1696eab1374337c13f68f" },
]

[[package]]
name = "watchdog"
version = "6.0.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0.tar.gz", hash = "sha256:9ddf7c82fda3ae8e24decda1338ede66e1c99883db93711d8fb941eaa2d8c282" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_aarch64.whl", hash = "sha256:7607498efa04a3542ae3e05e64da8202e58159aa1fa4acddf7678d34a35d4f13" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_armv7l.whl", hash = "sha256:9041567ee8953024c83343288ccc458fd0a2d811d6a0fd68c4c22609e3490379" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_i686.whl", hash = "sha256:82dc3e3143c7e38ec49d61af98d6558288c415eac98486a5c581726e0737c00e" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_ppc64.whl", hash = "sha256:212ac9b8bf1161dc91bd09c048048a95ca3a4c4f5e5d4a7d1b1a7d5752a7f96f" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_ppc64le.whl", hash = "sha256:e3df4cbb9a450c6d49318f6d14f4bbc80d763fa587ba46ec86f99f9e6876bb26" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_s390x.whl", hash = "sha256:2cce7cfc2008eb51feb6aab51251fd79b85d9894e98ba847408f662b3395ca3c" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-manylinux2014_x86_64.whl", hash = "sha256:20ffe5b202af80ab4266dcd3e91aae72bf2da48c0d33bdb15c66658e685e94e2" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-win32.whl", hash = "sha256:07df1fdd701c5d4c8e55ef6cf55b8f0120fe1aef7ef39a1c6fc6bc2e606d517a" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-win_amd64.whl", hash = "sha256:cbafb470cf848d93b5d013e2ecb245d4aa1c8fd0504e863ccefa32445359d680" },
    { url = "nexus/repository/pypi-public/packages/watchdog/6.0.0/watchdog-6.0.0-py3-none-win_ia64.whl", hash = "sha256:a1914259fa9e1454315171103c6a30961236f508b9b623eae470268bbcc6a22f" },
]

[[package]]
name = "websockets"
version = "16.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0.tar.gz", hash = "sha256:5f6261a5e56e8d5c42a4497b364ea24d94d9563e8fbd44e78ac40879c60179b5" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:71c989cbf3254fbd5e84d3bff31e4da39c43f884e64f2551d14bb3c186230f00" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:8b6e209ffee39ff1b6d0fa7bfef6de950c60dfb91b8fcead17da4ee539121a79" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:86890e837d61574c92a97496d590968b23c2ef0aeb8a9bc9421d174cd378ae39" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl", hash = "sha256:9b5aca38b67492ef518a8ab76851862488a478602229112c4b0d58d63a7a4d5c" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:e0334872c0a37b606418ac52f6ab9cfd17317ac26365f7f65e203e2d0d0d359f" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:a0b31e0b424cc6b5a04b8838bbaec1688834b2383256688cf47eb97412531da1" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:485c49116d0af10ac698623c513c1cc01c9446c058a4e61e3bf6c19dff7335a2" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-win32.whl", hash = "sha256:eaded469f5e5b7294e2bdca0ab06becb6756ea86894a47806456089298813c89" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-cp312-cp312-win_amd64.whl", hash = "sha256:5569417dc80977fc8c2d43a86f78e0a5a22fee17565d78621b6bb264a115d4ea" },
    { url = "nexus/repository/pypi-public/packages/websockets/16.0/websockets-16.0-py3-none-any.whl", hash = "sha256:1637db62fad1dc833276dded54215f2c7fa46912301a24bd94d45d46a011ceec" },
]

[[package]]
name = "yarl"
version = "1.24.2"
source = { registry = "nexus/repository/pypi-public/simple" }
dependencies = [
    { name = "idna" },
    { name = "multidict" },
    { name = "propcache" },
]
sdist = { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2.tar.gz", hash = "sha256:9ac374123c6fd7abf64d1fec93962b0bd4ee2c19751755a762a72dd96c0378f8" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:b975866c184564c827e0877380f0dae57dcca7e52782128381b72feff6dfceb8" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:3b075301a2836a0e297b1b658cb6d6135df535d62efefdd60366bd589c2c82f2" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:8ae44649b00947634ab0dab2a374a638f52923a6e67083f2c156cd5cbd1a881d" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:507cc19f0b45454e2d6dcd62ff7d062b9f77a2812404e62dbdaec05b50faa035" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:c4c17bad5a530912d2111825d3f05e89bab2dd376aaa8cbc77e449e6db63e576" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:f5f0cbb112838a4a293985b6ed73948a547dadcc1ba6d2089938e7abdedceef8" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:5ec8356b8a6afcf81fc7aeeef13b1ff7a49dec00f313394bbb9e83830d32ccd7" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:7e7ebcdef69dec6c6451e616f32b622a6d4a2e92b445c992f7c8e5274a6bbc4c" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:47a55d6cf6db2f401017a9e96e5288844e5051911fb4e0c8311a3980f5e59a7d" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3065657c80a2321225e804048597ad55658a7e76b32d6f5ee4074d04c50401db" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:cb84b80d88e19ede158619b80813968713d8d008b0e2497a576e6a0557d50712" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:990de4f680b1c217e77ff0d6aa0029f9eb79889c11fb3e9a3942c7eba29c1996" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:abb8ec0323b80161e3802da3150ef660b41d0e9be2048b76a363d93eee992c2b" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:e7977781f83638a4c73e0f88425563d70173e0dfd90ac006a45c65036293ee3c" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:e30dd55825dc554ec5b66a94953b8eda8745926514c5089dfcacecb9c99b5bd1" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-win_amd64.whl", hash = "sha256:7dafe10c12ddd4d120d528c4b5599c953bd7b12845347d507b95451195bb6cad" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-cp312-cp312-win_arm64.whl", hash = "sha256:044a09d8401fcf8681977faef6d286b8ade1e2d2e9dceda175d1cfa5ca496f30" },
    { url = "nexus/repository/pypi-public/packages/yarl/1.24.2/yarl-1.24.2-py3-none-any.whl", hash = "sha256:2783d9226db8797636cd6896e4de81feed252d1db72265686c9558d97a4d94b9" },
]

[[package]]
name = "zstandard"
version = "0.25.0"
source = { registry = "nexus/repository/pypi-public/simple" }
sdist = { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0.tar.gz", hash = "sha256:7713e1179d162cf5c7906da876ec2ccb9c3a9dcbdffef0cc7f70c3667a205f0b" }
wheels = [
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:7b3c3a3ab9daa3eed242d6ecceead93aebbb8f5f84318d82cee643e019c4b73b" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:913cbd31a400febff93b564a23e17c3ed2d56c064006f54efec210d586171c00" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-manylinux2010_i686.manylinux2014_i686.manylinux_2_12_i686.manylinux_2_17_i686.whl", hash = "sha256:011d388c76b11a0c165374ce660ce2c8efa8e5d87f34996aa80f9c0816698b64" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:6dffecc361d079bb48d7caef5d673c88c8988d3d33fb74ab95b7ee6da42652ea" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:7149623bba7fdf7e7f24312953bcf73cae103db8cae49f8154dd1eadc8a29ecb" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:6a573a35693e03cf1d67799fd01b50ff578515a8aeadd4595d2a7fa9f3ec002a" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:5a56ba0db2d244117ed744dfa8f6f5b366e14148e00de44723413b2f3938a902" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:10ef2a79ab8e2974e2075fb984e5b9806c64134810fac21576f0668e7ea19f8f" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:aaf21ba8fb76d102b696781bddaa0954b782536446083ae3fdaa6f16b25a1c4b" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:1869da9571d5e94a85a5e8d57e4e8807b175c9e4a6294e3b66fa4efb074d90f6" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:809c5bcb2c67cd0ed81e9229d227d4ca28f82d0f778fc5fea624a9def3963f91" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:f27662e4f7dbf9f9c12391cb37b4c4c3cb90ffbd3b1fb9284dadbbb8935fa708" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:99c0c846e6e61718715a3c9437ccc625de26593fea60189567f0118dc9db7512" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:474d2596a2dbc241a556e965fb76002c1ce655445e4e3bf38e5477d413165ffa" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-win32.whl", hash = "sha256:23ebc8f17a03133b4426bcc04aabd68f8236eb78c3760f12783385171b0fd8bd" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-win_amd64.whl", hash = "sha256:ffef5a74088f1e09947aecf91011136665152e0b4b359c42be3373897fb39b01" },
    { url = "nexus/repository/pypi-public/packages/zstandard/0.25.0/zstandard-0.25.0-cp312-cp312-win_arm64.whl", hash = "sha256:181eb40e0b6a29b3cd2849f825e0fa34397f649170673d385f3598ae17cca2e9" },
]

```

