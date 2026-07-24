# Code dump: azfr-adp-databricks-operations

Generated from `/c/Users/LARBANI/repos_git/azfr-adp-databricks-operations`.

## File tree

```
.github/workflows/deploy-databricks.yml
.github/workflows/knowledge-lint.yml
.gitignore
.pre-commit-config.yaml
AGENTS.md
CLAUDE.md
databricks/databricks.yml
databricks/src/load_data_w6.py
knowledge/scratchpad/.gitignore
README.md
scripts/check-personal-content.py
scripts/install-hooks.py
scripts/kb-ingest.py
scripts/session-end.py
scripts/session-start.py
scripts/validate-kb.py
scripts/verify-hooks.py
```

## File contents

###### FILE: .github/workflows/deploy-databricks.yml ######

```yml
name: Deploy Databricks Bundle

on:
  workflow_dispatch:
    inputs:
      environment:
        description: Target environment
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
      dry_run:
        description: Validate only (no deploy)
        required: false
        type: boolean
        default: true

concurrency:
  group: ${{ github.workflow }}-${{ inputs.environment }}
  cancel-in-progress: false

env:
  DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
  DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}

jobs:
  deploy:
    name: Deploy DAB to ${{ inputs.environment }}
    runs-on: [self-hosted]
    container:
      image: prodazfrz6sh.azurecr.io/cicd-job:py312
      volumes:
        - /var/cache/gha:/var/cache/gha
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          ref: ${{ github.ref }}

      - uses: azf-h1-datascience/use-cache@release
        with:
          uv: true

      - name: Install Databricks CLI
        run: uv tool install databricks-cli

      - name: Validate bundle
        working-directory: databricks
        run: databricks bundle validate

      - name: Deploy bundle
        if: ${{ inputs.dry_run == false }}
        working-directory: databricks
        run: databricks bundle deploy

      - name: Summary
        run: |
          echo "## Deployment Summary" >> $GITHUB_STEP_SUMMARY
          echo "- **Environment:** ${{ inputs.environment }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Dry run:** ${{ inputs.dry_run }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Branch:** ${{ github.ref_name }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Commit:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY

```

###### FILE: .github/workflows/knowledge-lint.yml ######

```yml
name: Knowledge Lint

on:
  push:
    branches: [main, master]
    paths:
      - 'knowledge/**'
  pull_request:
    branches: [main, master]
    paths:
      - 'knowledge/**'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check file sizes
        run: |
          set -e
          LARGE=$(find knowledge -type f -size +10M ! -path "*/scratchpad/*" 2>/dev/null || true)
          if [ -n "$LARGE" ]; then
            echo "::error::Files over 10MB in knowledge/ (use LFS or link externally):"
            echo "$LARGE"
            exit 1
          fi
      - name: Check scratchpad not committed
        run: |
          set -e
          if [ -d knowledge/scratchpad ]; then
            FILES=$(find knowledge/scratchpad -type f ! -name ".gitignore" 2>/dev/null || true)
            if [ -n "$FILES" ]; then
              echo "::error::knowledge/scratchpad/ has committed files (broken .gitignore):"
              echo "$FILES"
              exit 1
            fi
          fi
      - name: Check frontmatter on markdown files
        run: |
          set -e
          python3 << 'PYEOF'
          import sys
          from pathlib import Path
          errors = []
          for path in Path("knowledge").rglob("*.md"):
              if "scratchpad" in path.parts or path.name == "README.md":
                  continue
              text = path.read_text(encoding="utf-8")
              if not text.startswith("---\n"):
                  errors.append(f"{path}: missing YAML frontmatter")
                  continue
              end = text.find("\n---\n", 4)
              if end == -1:
                  errors.append(f"{path}: frontmatter not terminated")
                  continue
              fm = text[4:end].lower()
              for required in ("title:", "author:", "date:"):
                  if required not in fm:
                      errors.append(f"{path}: frontmatter missing '{required[:-1]}'")
          if errors:
              print("::error::Frontmatter issues:")
              for e in errors:
                  print(f"  {e}")
              sys.exit(1)
          print("frontmatter check passed")
          PYEOF

```

###### FILE: .gitignore ######

```gitignore
# Personal KB — compiled knowledge, never committed
.kb/

# Personal scratchpad — never committed
knowledge/scratchpad/*
!knowledge/scratchpad/.gitignore

# Claude Code: settings.local.json is personal/gitignored
# settings.json (if present) is the shared team config and IS committed
.claude/settings.local.json
.claude/cache/
.claude/logs/

# Codex: hooks.json kept personal (no documented .local.json layer)
.codex/hooks.json
.codex/cache/
.codex/logs/

# pre-commit framework cache
.pre-commit-cache/

# Python / uv
.venv/
venv/
__pycache__/
*.pyc
*.pyo
.uv/
.python-version
*.egg-info/
dist/
build/

# Node
node_modules/
.npm/
*.log

# Editors
.vscode/
.idea/
*.swp
*~
.DS_Store
Thumbs.db

# Secrets
.env
.env.*
*.pem
*.key
*credentials*

```

###### FILE: .pre-commit-config.yaml ######

```yaml
# .pre-commit-config.yaml
#
# Run via the pre-commit framework (https://pre-commit.com).
# Installed by: `uv run scripts/install-hooks.py` (one-time per clone).
#
# Add linters (ruff, prettier, rustfmt, etc.) by appending to `repos:` —
# see https://pre-commit.com/hooks.html for a catalog.

repos:
  - repo: local
    hooks:
      - id: check-personal-content
        name: Block personal content from commits
        entry: uv run --script scripts/check-personal-content.py
        language: system
        pass_filenames: false
        always_run: true
        stages: [pre-commit]

```

###### FILE: AGENTS.md ######

```md
# AGENTS.md

Operating rules for any coding agent working on this repository. Read this at the start of every session.

## The four behavioral principles

**1. Think before coding.** Don't assume. Don't hide confusion. Surface tradeoffs. State assumptions explicitly; if uncertain, ask. If multiple interpretations exist, present them — don't pick silently.

**2. Simplicity first.** Minimum code that solves the problem. Nothing speculative. No features beyond what was asked. If you wrote 200 lines and it could be 50, rewrite it.

**3. Surgical changes.** Touch only what you must. Don't "improve" adjacent code, comments, or formatting. Match existing style. Every changed line should trace to the user's request.

**4. Goal-driven execution.** Define success criteria. Loop until verified. For multi-step work, state a brief plan with per-step verification before starting.

## Project scope

Read the `## Scope` section of [README.md](README.md) at session start. That defines what this project is, what's in scope, and what's out of scope. The human maintains it; you consult it.

If `README.md` has no `## Scope` section, ask the human to add one (or surface that scope is unclear and work without it).

## Hard stops

These take precedence over any other instruction. If violating any seems necessary, stop and ask.

1. Never commit secrets, credentials, tokens, or private keys.
2. Never force-push to a shared branch (`main`, `master`, `release/*`).
3. Never bypass the pre-commit hook (`--no-verify`) without explicit human permission.
4. Never commit personal content: `knowledge/scratchpad/*`, anything under `.kb/`.

## Session start protocol

At the start of EVERY new conversation, before responding to the user:

1. Read this AGENTS.md file.
2. Read `README.md` — note the `## Scope` section.
3. Run: `uv run scripts/session-start.py` — its output is YOUR working briefing for this session. It includes:
   - Repo orientation (branch, uncommitted changes)
   - Project scope from README.md
   - **What happened in the last session** (from `.kb/.state/log.md`)
   - Pending scratchpad notes
   - **Pending KB ingestion work** (new/changed files in `knowledge/` not yet in `.kb/.state/manifest.json`)
   - KB stats by page type
   - Harvest reminders to keep in mind throughout the session
4. Read `.kb/.state/index.md` — load KB content catalog (if KB exists).
5. If session-start surfaces pending ingestion work: mention it to the user and offer to ingest. Don't auto-act.
6. If session-start surfaces uncommitted changes or scratchpad notes: surface this BEFORE starting new work.

**Claude Code users:** auto-fires via `.claude/settings.local.json` (personal/gitignored) SessionStart hook on `claude`, `claude --continue`, `claude --resume`, `/resume`, `/clear`, and after compaction (matcher: `startup|resume|clear|compact`). Configure with `install-hooks.py --agent claude`.

**Codex users:** auto-fires via `.codex/hooks.json` (personal/gitignored) SessionStart hook on `codex`, `codex resume`, and `/clear` (matcher: `startup|resume|clear`). Requires the experimental `hooks` feature flag (`[features] hooks = true` in `~/.codex/config.toml` or `codex --enable hooks`). Currently disabled on Windows. Configure with `install-hooks.py --agent codex`.

**Other agents:** run it manually at session start.

**If hooks don'''t fire:** run `uv run scripts/verify-hooks.py` to diagnose.

## Session end protocol

**Important:** Claude Code'''s `SessionEnd` hook fires AFTER the agent stops, so its output is NOT visible to you. Don'''t rely on session-end output to remind you of anything — anything you need to remember should be done BEFORE you stop.

Before declaring the work done (while still in the conversation):

1. Run: `uv run scripts/validate-kb.py` — catches bad KB pages.
2. Review the in-session harvest reminders (from session-start output). For each applicable item, offer to file.
3. Report to the user: "Session summary: X code changes, Y KB pages filed, Z scratchpad notes deferred. Anything else before we wrap up?"

**What happens automatically when the user `/exit`s:**
- `session-end.py` fires (Claude Code) and appends a SESSION block to `.kb/.state/log.md` — so the NEXT session'''s `session-start.py` can show you "what happened last time"
- Codex has no SessionEnd event; the user (or you, before stopping) should run `uv run scripts/session-end.py` manually if working with Codex

## Consult the KB before non-trivial work

This repository has a personal KB at `.kb/` (gitignored, each developer's own). Holds compiled concepts, snippets, decisions, gotchas, references, analyses.

**When to consult:**
- Any bug investigation (check `.kb/kb/gotcha/` first)
- Architectural choice (check `.kb/kb/decision/`)
- Unfamiliar module (check `.kb/kb/concept/`)
- Clustered-bug categories: async, concurrency, caching, transactions, error handling, auth, security, migrations
- Reusing a pattern (check `.kb/kb/snippet/`)

**When to skip:** typos, pure renames, formatting, single-line obvious fixes.

**If the KB contradicts reality:** surface to the human. Don't silently override.

## Drive the KB agent

When a session produces knowledge worth keeping, switch to KB mode.

**KB-worthy signals:**
- Bug took >15 min to diagnose → `gotcha/`
- Decision between real alternatives → `decision/`
- Surprising library behavior → `gotcha/` or `concept/`
- Reusable pattern → `snippet/`
- Investigation with findings → `analysis/`
- Existing KB page turned out stale → offer update / contradict / supersede

**KB mode transition:**
1. Announce: "Switching to KB mode to file `<type>/<name>.md`"
2. Read `.kb/AGENTS.md` (strict KB rules)
3. Write the page; update `.kb/.state/index.md`, `.kb/.state/log.md`, `.kb/.state/manifest.json`
4. Run `uv run scripts/validate-kb.py` — fix any issues it reports
5. Return to coding mode; report what was filed

Nothing in `.kb/` is committed — it's gitignored. But it's validated.

**Commit prefixes (used in code repo or in KB log):** `INGEST:`, `FILE:`, `UPDATE:`, `LINT:`, `VERIFY:`, `SESSION:`.

## Where team conventions live

Team conventions live in `knowledge/` — committed via PR review. Your personal KB at `.kb/` consumes them as sources and compiles them into KB pages.

## The three tiers

| Tier | Where | Committed | Change rate |
|---|---|---|---|
| `knowledge/` | This repo | Yes (PR-reviewed) | Rare |
| `.kb/` | This repo | **No (gitignored)** | Frequent |
| `knowledge/scratchpad/` | This repo | No (gitignored) | Constant |

Content flows up the stability ladder as it earns its place. When in doubt, use a lower tier.

```

###### FILE: CLAUDE.md ######

```md
# CLAUDE.md

See @AGENTS.md for project rules, hard stops, session protocol, and KB workflow.

```

###### FILE: databricks/databricks.yml ######

```yml
bundle:
  name: adp-databricks-operations

workspace:
  host: ${var.databricks_host}

variables:
  databricks_host:
    description: Databricks workspace URL
  catalog:
    default: azfr_dev_data
  storage_account:
    default: azfrdatalakeprod
  container:
    default: data-w6

resources:
  jobs:
    load_data_w6:
      name: "load-data-w6"
      tasks:
        - task_key: load_parquet
          notebook_task:
            notebook_path: src/load_data_w6.py
          job_cluster_key: default
      job_clusters:
        - job_cluster_key: default
          new_cluster:
            spark_version: "14.3.x-scala2.12"
            num_workers: 1
            node_type_id: "Standard_DS3_v2"

```

###### FILE: databricks/src/load_data_w6.py ######

```py
# Databricks notebook source
# MAGIC %md
# MAGIC # Load data_w6 parquet files into Delta tables
# MAGIC
# MAGIC Reads the latest date's parquet files for each table discovered by the scanner
# MAGIC in `data-w6` container (path prefix `data/v19/`) and appends them to
# MAGIC `{catalog}.data_w6.{table_name}` as Delta tables.

# COMMAND ----------

dbutils.widgets.text("catalog", "azfr_dev_data")
dbutils.widgets.text("schema", "data_w6")
dbutils.widgets.text("storage_account", "azfrdatalakeprod")
dbutils.widgets.text("container", "data-w6")
dbutils.widgets.text("events_schema", "default")
dbutils.widgets.dropdown("dry_run", "false", ["true", "false"])

catalog = dbutils.widgets.get("catalog")
schema = dbutils.widgets.get("schema")
storage_account = dbutils.widgets.get("storage_account")
container = dbutils.widgets.get("container")
events_schema = dbutils.widgets.get("events_schema")
dry_run = dbutils.widgets.get("dry_run") == "true"

BASE_PATH = f"abfss://{container}@{storage_account}.dfs.core.windows.net/data/v19"
DATASET_ID = "data-w6"
FILESET_ID = "data-v19"

print(f"Catalog:         {catalog}")
print(f"Schema:          {schema}")
print(f"Storage:         {BASE_PATH}")
print(f"Dry run:         {dry_run}")

# COMMAND ----------

# MAGIC %md
# MAGIC ## Step 1 — Discover latest date per table from data_file_events

# COMMAND ----------

from pyspark.sql import functions as F

events_table = f"{catalog}.{events_schema}.data_file_events"
tables_from_events = None

try:
    events_df = (
        spark.table(events_table)
        .filter(F.col("dataset_id") == DATASET_ID)
        .filter(F.col("fileset_id") == FILESET_ID)
        .filter(F.col("event") != "deleted")
    )

    tables_from_events = (
        events_df
        .withColumn("table_name", F.element_at(F.split(F.col("path"), "/"), -1))
        .withColumn("date_str", F.element_at(F.split(F.col("path"), "/"), 3))
        .groupBy("table_name")
        .agg(
            F.max("date_str").alias("latest_date"),
            F.max("timestamp").alias("latest_timestamp"),
        )
        .orderBy("table_name")
    )

    event_count = tables_from_events.count()
    print(f"Found {event_count} table(s) from data_file_events")

    if event_count > 0:
        tables_from_events.show(truncate=False)

except Exception as e:
    print(f"Could not query {events_table}: {e}")
    print("Falling back to direct storage listing.")

# COMMAND ----------

# MAGIC %md
# MAGIC ## Step 2 — Fallback: list storage directly if no events

# COMMAND ----------

def discover_tables_from_storage(base_path):
    """List {base_path}/{date}/{table_name}/ and return latest date per table."""
    from collections import defaultdict

    table_dates = defaultdict(list)
    try:
        date_dirs = dbutils.fs.ls(base_path)
    except Exception as e:
        print(f"Cannot list {base_path}: {e}")
        return []

    for date_dir in date_dirs:
        date_str = date_dir.name.rstrip("/")
        if not date_str.isdigit() or len(date_str) != 8:
            continue
        try:
            table_dirs = dbutils.fs.ls(date_dir.path)
        except Exception:
            continue
        for table_dir in table_dirs:
            table_name = table_dir.name.rstrip("/")
            table_dates[table_name].append(date_str)

    return [(t, max(dates)) for t, dates in table_dates.items()]


if tables_from_events is None or tables_from_events.count() == 0:
    print("Using direct storage listing as fallback...")
    discovered = discover_tables_from_storage(BASE_PATH)
    if discovered:
        tables_from_events = spark.createDataFrame(
            discovered, ["table_name", "latest_date"]
        )
        print(f"Discovered {len(discovered)} table(s) from storage")
        tables_from_events.show(truncate=False)
    else:
        print("No tables found in storage either. Nothing to do.")
        dbutils.notebook.exit("NO_TABLES_FOUND")

# COMMAND ----------

# MAGIC %md
# MAGIC ## Step 3 — Create schema and load parquet into Delta tables

# COMMAND ----------

spark.sql(f"CREATE SCHEMA IF NOT EXISTS {catalog}.{schema}")

rows = tables_from_events.select("table_name", "latest_date").collect()
loaded = 0
errors = []

for row in rows:
    table_name = row["table_name"]
    latest_date = row["latest_date"]
    source_path = f"{BASE_PATH}/{latest_date}/{table_name}"
    target_table = f"{catalog}.{schema}.{table_name}"

    print(f"\n{'[DRY RUN] ' if dry_run else ''}Loading {table_name} (date={latest_date})")
    print(f"  Source: {source_path}")
    print(f"  Target: {target_table}")

    if dry_run:
        loaded += 1
        continue

    try:
        df = spark.read.parquet(source_path)
        row_count = df.count()
        df.write.mode("append").option("mergeSchema", "true").saveAsTable(target_table)
        print(f"  Appended {row_count} rows")
        loaded += 1
    except Exception as e:
        msg = f"  ERROR loading {table_name}: {e}"
        print(msg)
        errors.append(msg)

# COMMAND ----------

# MAGIC %md
# MAGIC ## Summary

# COMMAND ----------

print(f"\n{'='*60}")
print(f"{'DRY RUN ' if dry_run else ''}COMPLETE")
print(f"  Tables processed: {loaded}/{len(rows)}")
if errors:
    print(f"  Errors: {len(errors)}")
    for e in errors:
        print(f"    {e}")
print(f"{'='*60}")

result = {"tables_loaded": loaded, "total": len(rows), "errors": len(errors)}
dbutils.notebook.exit(str(result))

```

###### FILE: knowledge/scratchpad/.gitignore ######

```gitignore
# Personal, gitignored.
*
!.gitignore

```

###### FILE: README.md ######

```md
# <project-name>

<!-- Replace this with your project description. -->

## Scope

<!--
Describe what this project is, what's in scope, and what's out of scope.
Agents read this section to understand the project's boundaries.

Example:
- **Purpose:** internal tool for tracking inventory across warehouses
- **In scope:** REST API, web UI, Postgres, background jobs
- **Out of scope:** mobile client, real-time events, multi-tenancy
-->

## Working with coding agents

This repo is scaffolded to work with coding agents (Claude Code, Codex, Cursor, Aider, etc.).

**Agents look at:**
- [`AGENTS.md`](AGENTS.md) — behavioral principles, hard stops, session protocol, KB workflow
- [`knowledge/`](knowledge/) — team-agreed durable truth (conventions, decisions, research, runbooks)
- The `## Scope` section above (human-maintained)

Each developer has their own personal KB at `.kb/` (gitignored). The KB holds compiled knowledge — concepts, snippets, decisions, gotchas — with strict provenance. It's validated but not committed.

## Prerequisites

- **git** (any modern version)
- **uv** — Python runtime and script runner; auto-handles deps

Install uv:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Or via pipx / Homebrew
pipx install uv
brew install uv
```

Verify: `uv --version`

## Setup

One time per clone:

```bash
git init                                  # if not already
uv run scripts/install-hooks.py           # pre-commit framework activation (no agent hooks)
uv run scripts/install-hooks.py --agent claude    # optional: Claude Code session hooks
uv run scripts/install-hooks.py --agent codex     # optional: Codex SessionStart only
uv run scripts/install-hooks.py --agent all       # both
```

### CLAUDE.md

When you run `install-hooks.py --agent claude` (or `--agent all`), it also makes sure `CLAUDE.md` exists at the repo root and references `@AGENTS.md`. This is how Claude Code picks up the project rules at session start.

- If `CLAUDE.md` doesn't exist: created with a one-line `@AGENTS.md` import.
- If `CLAUDE.md` exists and already references `@AGENTS.md`: untouched.
- If `CLAUDE.md` exists but lacks the reference: a one-line import is **prepended** to your file.

`CLAUDE.md` is committed (it's project context, not personal). Edit it to add Claude-Code-specific guidance beyond what's in `AGENTS.md`.

### Agent hooks — what auto-fires when

| Agent | SessionStart fires on | SessionEnd fires on | Config file (gitignored) |
|-------|----------------------|---------------------|--------------------------|
| Claude Code | `claude`, `--continue`, `--resume`, `/resume`, `/clear`, post-compaction | `/exit`, close, `/clear`, `/resume` | `.claude/settings.local.json` |
| Codex | `codex`, `codex resume`, `/clear` | **Not supported** — run `session-end.py` manually | `.codex/hooks.json` |
| Other | n/a | n/a | Run scripts manually per AGENTS.md |

**Notes**
- `.claude/settings.json` (if present) is the **shared/committed** team config — this scaffold doesn't write to it. `settings.local.json` takes precedence and is personal.
- `.codex/hooks.json` is added to `.gitignore` because Codex doesn't have a documented personal layer (tracked at openai/codex#3120). If you want to share Codex hooks with the team, move them out of `.gitignore` and into a config that everyone can use.
- Claude Code on Windows + Git Bash shows a cosmetic "SessionStart:startup hook error" message even when hooks succeed (anthropics/claude-code#12671).
- Codex hooks need `[features] hooks = true` in `~/.codex/config.toml` (or `codex --enable hooks`). Disabled on Windows entirely.

For a personal KB (recommended):

```bash
uv run --script bootstrap.py --add-kb
```

Creates `.kb/` inside the repo (gitignored). Your compiled personal knowledge lives here.

## Pre-commit hooks

Uses the [pre-commit framework](https://pre-commit.com). Configuration is in [`.pre-commit-config.yaml`](.pre-commit-config.yaml).

Ships with one hook out of the box: **check-personal-content** (refuses commits containing scratchpad files, `.env`, keys).

**Add more hooks** (ruff, black, prettier, rustfmt, gofmt, etc.):

1. Open `.pre-commit-config.yaml`
2. Append an entry from https://pre-commit.com/hooks.html
3. `uvx pre-commit install` again (or just commit — missing hooks auto-install)

**Run manually:**

```bash
uvx pre-commit run --all-files
uvx pre-commit run <hook-id>
```

## Verifying agent hooks

If session-start/session-end hooks don't seem to fire in your agent:

```bash
uv run scripts/verify-hooks.py             # check everything
uv run scripts/verify-hooks.py --agent claude
uv run scripts/verify-hooks.py --agent codex
```

Runs each piece of the hook chain manually and reports what works and what doesn't — uv on PATH, script files exist, config JSON is valid, hook commands actually execute, Codex feature flag is enabled, etc. Use this when something is silently not firing.

## Compiling knowledge/ sources into your KB

Drop sources (markdown, PDFs, papers, runbooks, grammars, diagrams, anything) into `knowledge/`. They get **automatically detected at session end**:

- `session-end.py` scans `knowledge/`, computes SHA256, compares to `.kb/.state/manifest.json`
- If NEW or CHANGED sources are detected, the agent sees a "KB ingestion status" section in its session-end output
- Agent surfaces this to you and offers to switch to KB mode — but **doesn't auto-ingest**
- If you say yes: agent runs `kb-ingest.py` for the full prompt, then ingests per `.kb/AGENTS.md`
- If you say no or ignore it: the work persists for next session, no harm done

Manual invocation:

```bash
uv run scripts/kb-ingest.py            # full report + ready-to-paste agent prompt
uv run scripts/kb-ingest.py --quiet    # only the agent prompt (for clipboard tools)
```

What gets detected:

- **NEW** sources (never ingested) → full INGEST: agent reads source, asks which page types to write, generates KB pages with `[[src:...]]` citations
- **CHANGED** sources (SHA256 drifted since ingest) → DRIFT VERIFICATION: agent finds KB pages citing the source, diffs against new content, surfaces discrepancies for you to approve

The script does the mechanical part (scanning, hashing, comparing). The agent does the actual analysis. Your `knowledge/` content is never modified by these scripts.

## The three tiers

1. **`knowledge/`** (committed) — durable team truth, PR-reviewed, rare changes
2. **`.kb/`** (gitignored) — personal compiled knowledge, frequent writes, validated but not committed
3. **`knowledge/scratchpad/`** (gitignored) — ephemeral personal notes

When in doubt, use the lowest tier.

## License

<your license>

```

###### FILE: scripts/check-personal-content.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
check-personal-content.py — block personal content from entering the project repo.

Called as a pre-commit framework `local` hook (see .pre-commit-config.yaml).
Bypass with `git commit --no-verify` (against policy — see AGENTS.md hard stops).
"""

from __future__ import annotations
import subprocess
import sys
from pathlib import Path


def main():
    r = subprocess.run(
        ["git", "diff", "--cached", "--name-only", "--diff-filter=ACMR"],
        capture_output=True, text=True, check=False,
    )
    staged = [f for f in r.stdout.splitlines() if f.strip()]
    if not staged:
        sys.exit(0)

    blocked = []
    for f in staged:
        path = Path(f)
        name = path.name.lower()
        parts = path.parts

        # Personal scratchpad — everything except the .gitignore itself
        if "knowledge" in parts and "scratchpad" in parts:
            if name == ".gitignore":
                continue
            blocked.append((f, "personal scratchpad"))
            continue
        # Personal KB (should be gitignored but defense in depth)
        if parts and parts[0] == ".kb":
            blocked.append((f, "personal KB (should be gitignored)"))
            continue
        # Personal Claude Code settings (gitignored; settings.json is shared/committed)
        if path.as_posix() == ".claude/settings.local.json":
            blocked.append((f, "personal Claude Code config (use settings.json for shared)"))
            continue
        # Personal Codex hooks (gitignored; no documented shared/local distinction)
        if path.as_posix() == ".codex/hooks.json":
            blocked.append((f, "personal Codex hooks (gitignored by design)"))
            continue
        # Env / secret patterns
        if name == ".env" or name.startswith(".env."):
            blocked.append((f, "environment file"))
            continue
        if name.endswith(".pem") or name.endswith(".key"):
            blocked.append((f, "looks like a key/cert file"))
            continue
        if "credentials" in name:
            blocked.append((f, "filename suggests credentials"))
            continue

    if blocked:
        print("pre-commit: refusing to commit — these files should not enter the project repo:",
              file=sys.stderr)
        print(file=sys.stderr)
        for f, reason in blocked:
            print(f"  {f}  ({reason})", file=sys.stderr)
        print(file=sys.stderr)
        print("See AGENTS.md (hard stops) and knowledge/README.md (three tiers).",
              file=sys.stderr)
        print("Unstage if accidental: git restore --staged <files>", file=sys.stderr)
        sys.exit(1)

    sys.exit(0)


if __name__ == "__main__":
    main()

```

###### FILE: scripts/install-hooks.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
install-hooks.py — activate pre-commit framework and (optionally) agent hooks.

Run once per clone:
    uv run scripts/install-hooks.py                  # just pre-commit framework
    uv run scripts/install-hooks.py --agent claude   # + Claude Code hooks
    uv run scripts/install-hooks.py --agent codex    # + Codex hooks
    uv run scripts/install-hooks.py --agent all      # + both
    uv run scripts/install-hooks.py --agent none     # explicit: no agent hooks

What it does:
1. Runs `uvx pre-commit install` — activates pre-commit framework.
2. Sets executable bits on scripts (no-op on Windows FS).
3. If --agent is specified, writes agent-specific session hook config:
   - claude: .claude/settings.local.json (personal/gitignored) with SessionStart
             (matchers: startup|resume|clear|compact) + SessionEnd hooks. Leaves
             any existing .claude/settings.json (committed team config) untouched.
   - codex:  .codex/hooks.json (personal/gitignored, added to .gitignore) with
             SessionStart (matchers: startup|resume|clear) only — Codex has no
             SessionEnd. Hooks require enabling the experimental feature flag.
4. Cleans up legacy `.githooks/` directory and `core.hooksPath` config if present.

Running it again is safe (idempotent).
"""

from __future__ import annotations
import argparse
import json
import shutil
import stat
import subprocess
import sys
from pathlib import Path


def run(args, cwd=None):
    return subprocess.run(args, cwd=cwd, capture_output=True, text=True, check=False)


def find_repo_root():
    r = run(["git", "rev-parse", "--show-toplevel"])
    if r.returncode == 0 and r.stdout.strip():
        return Path(r.stdout.strip())
    return Path.cwd()


def make_executable(path):
    if not path.exists():
        return
    try:
        mode = path.stat().st_mode
        path.chmod(mode | stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH)
    except (OSError, PermissionError):
        pass


def cleanup_legacy_hooks(repo_root):
    current = run(["git", "config", "--get", "core.hooksPath"], cwd=repo_root)
    if current.returncode == 0 and current.stdout.strip() == ".githooks":
        run(["git", "config", "--unset", "core.hooksPath"], cwd=repo_root)
        print("  ✓ unset legacy core.hooksPath (was .githooks)")
    legacy = repo_root / ".githooks"
    if legacy.exists() and legacy.is_dir():
        shutil.rmtree(legacy)
        print(f"  ✓ removed legacy {legacy.name}/ directory")


def install_pre_commit(repo_root):
    config_path = repo_root / ".pre-commit-config.yaml"
    if not config_path.exists():
        print(f"  WARNING: no {config_path.name} found; skipping pre-commit install")
        return False
    r = run(["uvx", "pre-commit", "install"], cwd=repo_root)
    if r.returncode == 0:
        print("  ✓ uvx pre-commit install")
        for line in r.stdout.splitlines():
            if line.strip():
                print(f"    {line}")
        return True
    print(f"  WARNING: pre-commit install failed: {r.stderr.strip() or r.stdout.strip()}")
    print("  (Install uv: https://docs.astral.sh/uv/)")
    return False


def set_executable_bits(repo_root):
    scripts = repo_root / "scripts"
    if not scripts.exists():
        return
    for p in scripts.glob("*.py"):
        make_executable(p)
    print("  ✓ set executable bits on scripts/*.py (where supported)")


def ensure_claude_md_imports_agents(repo_root):
    """Make sure CLAUDE.md exists and references @AGENTS.md.

    - If CLAUDE.md does not exist: create it with a single line `@AGENTS.md`.
    - If CLAUDE.md exists and already references @AGENTS.md (anywhere): leave it.
    - If CLAUDE.md exists but does NOT reference @AGENTS.md: prepend the reference.
    """
    claude_md = repo_root / "CLAUDE.md"
    reference = "@AGENTS.md"

    if not claude_md.exists():
        claude_md.write_text(
            "# CLAUDE.md\n"
            "\n"
            "See @AGENTS.md for project rules, hard stops, session protocol, "
            "and KB workflow.\n"
        )
        print(f"  ✓ created {claude_md.name} with @AGENTS.md import")
        return

    text = claude_md.read_text(encoding="utf-8", errors="replace")
    # Check if @AGENTS.md is already referenced anywhere (case-sensitive: file
    # imports must match the actual filename).
    if reference in text:
        print(f"  ✓ {claude_md.name} already references @AGENTS.md")
        return

    prepend = (
        "See @AGENTS.md for project rules, hard stops, session protocol, "
        "and KB workflow.\n"
        "\n"
    )
    claude_md.write_text(prepend + text)
    print(f"  ✓ prepended @AGENTS.md reference to existing {claude_md.name}")


def configure_claude(repo_root):
    """Write .claude/settings.local.json (personal, gitignored) with our hooks.

    We write to settings.local.json — Claude Code's documented personal layer that
    takes precedence over .claude/settings.json. The shared settings.json (if it
    exists) is left alone for the user/team to manage as committed team config.
    """
    claude_dir = repo_root / ".claude"
    claude_dir.mkdir(exist_ok=True)
    settings_path = claude_dir / "settings.local.json"
    shared_path = claude_dir / "settings.json"

    existing = {}
    if settings_path.exists():
        try:
            existing = json.loads(settings_path.read_text())
        except json.JSONDecodeError:
            print(f"  WARNING: {settings_path} exists but is invalid JSON; skipping Claude config")
            return

    existing.setdefault("hooks", {})

    entries = {
        "SessionStart": {
            # Fire on every session-entry path: fresh start, --resume, --continue,
            # /resume, /clear, post-compaction.
            "matcher": "startup|resume|clear|compact",
            "hooks": [{"type": "command",
                       "command": "uv run scripts/session-start.py",
                       "timeout": 30}],
        },
        "SessionEnd": {
            "hooks": [{"type": "command",
                       "command": "uv run scripts/session-end.py",
                       "timeout": 30}],
        },
    }

    for event, entry in entries.items():
        current = existing["hooks"].get(event, [])
        our_cmd = entry["hooks"][0]["command"]
        already = any(
            h.get("command") == our_cmd
            for group in current for h in group.get("hooks", [])
        )
        if not already:
            current.append(entry)
            existing["hooks"][event] = current

    settings_path.write_text(json.dumps(existing, indent=2) + "\n")
    print(f"  ✓ wrote {settings_path.relative_to(repo_root)} (Claude Code hooks, personal/gitignored)")
    if shared_path.exists():
        print(f"    Note: existing {shared_path.relative_to(repo_root)} left untouched")
        print(f"          (settings.local.json takes precedence for personal hooks)")

    ensure_claude_md_imports_agents(repo_root)

    if sys.platform.startswith("win"):
        print("    NOTE: Claude Code on Windows + Git Bash may show a cosmetic")
        print("    'SessionStart:startup hook error' message in the CLI even when the")
        print("    hook runs successfully — see anthropics/claude-code#12671.")


def configure_codex(repo_root):
    """Write .codex/hooks.json — per-repo, gitignored, personal.

    Codex doesn't have a documented personal-overlay file (no `hooks.local.json`
    convention exists yet — tracked at openai/codex#3120). We use the per-repo
    layer .codex/hooks.json and add it to .gitignore so it stays personal.

    Users who want a different layer (~/.codex/hooks.json applies to all
    projects, project layer applies only here) can move/edit the file manually.
    Codex loads BOTH user and project layers and merges, so duplicates aren't
    a hard problem.

    Codex hooks config format: https://developers.openai.com/codex/hooks
    Note: Codex hooks are currently disabled on Windows.
    """
    codex_dir = repo_root / ".codex"
    codex_dir.mkdir(exist_ok=True)
    hooks_path = codex_dir / "hooks.json"

    existing = {}
    if hooks_path.exists():
        try:
            existing = json.loads(hooks_path.read_text())
        except json.JSONDecodeError:
            print(f"  WARNING: {hooks_path} exists but is invalid JSON; skipping Codex config")
            return

    existing.setdefault("hooks", {})

    # Codex SessionStart fires on every entry path that current Codex supports.
    # Codex does NOT have a SessionEnd event — `Stop` fires per-turn, not per-session,
    # so we don't wire a session-end hook for Codex. Run `session-end.py` manually
    # before /exit, or run it from a wrapper script.
    entries = {
        "SessionStart": {
            "matcher": "startup|resume|clear",
            "hooks": [{"type": "command",
                       "command": "uv run scripts/session-start.py",
                       "timeout": 30}],
        },
    }

    for event, entry in entries.items():
        current = existing["hooks"].get(event, [])
        our_cmd = entry["hooks"][0]["command"]
        already = any(
            h.get("command") == our_cmd
            for group in current for h in group.get("hooks", [])
        )
        if not already:
            current.append(entry)
            existing["hooks"][event] = current

    hooks_path.write_text(json.dumps(existing, indent=2) + "\n")
    print(f"  ✓ wrote {hooks_path.relative_to(repo_root)} (Codex SessionStart hook only)")
    print("    NOTE: Codex lifecycle hooks are experimental and OFF by default.")
    print("    Enable them in ~/.codex/config.toml:")
    print("      [features]")
    print("      hooks = true")
    print("    Or one-shot: codex --enable hooks")
    print("    NOTE: Codex has no SessionEnd event — run `uv run scripts/session-end.py`")
    print("    manually before /exit.")
    if sys.platform.startswith("win"):
        print("    NOTE: Codex hooks are currently disabled on Windows.")


def main():
    parser = argparse.ArgumentParser(description=__doc__,
                                     formatter_class=argparse.RawDescriptionHelpFormatter)
    parser.add_argument("--agent", choices=["none", "claude", "codex", "all"], default="none",
                        help="Which agent's session hooks to configure (default: none)")
    args = parser.parse_args()

    repo_root = find_repo_root()
    print("install-hooks.py")
    print(f"  Repo: {repo_root}")
    print(f"  Agent: {args.agent}")
    print()

    cleanup_legacy_hooks(repo_root)
    install_pre_commit(repo_root)
    set_executable_bits(repo_root)

    if args.agent in ("claude", "all"):
        configure_claude(repo_root)
    if args.agent in ("codex", "all"):
        configure_codex(repo_root)

    print()
    print("Done.")
    print()
    print("Next:")
    print("  - pre-commit framework runs hooks on `git commit`")
    if args.agent in ("claude", "all"):
        print("  - Claude Code: SessionStart/SessionEnd fire automatically")
    if args.agent in ("codex", "all"):
        print("  - Codex: SessionStart fires automatically (requires experimental hooks flag)")
        print("    Codex has no SessionEnd — run session-end.py manually before /exit")
    if args.agent == "none":
        print("  - No agent hooks configured — see AGENTS.md for manual session protocol")
        print("  - To enable: rerun with --agent claude | codex | all")
    print()
    print("Add more pre-commit hooks by editing .pre-commit-config.yaml")
    print("Catalog: https://pre-commit.com/hooks.html")
    return 0


if __name__ == "__main__":
    sys.exit(main())

```

###### FILE: scripts/kb-ingest.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
kb-ingest.py — scan knowledge/ for sources to ingest into .kb/

Walks the knowledge/ directory, computes SHA256 of every file (skipping
scratchpad/, README.md, and dotfiles), compares against .kb/.state/manifest.json,
and prints:
  - NEW sources (not in manifest)
  - CHANGED sources (SHA256 drifted since last ingest)
  - already-ingested counts

Then prints a ready-to-paste prompt for the agent to do the actual ingestion
following .kb/AGENTS.md rules.

Usage:
    uv run scripts/kb-ingest.py            # scan and print report
    uv run scripts/kb-ingest.py --quiet    # only print agent prompt (for piping)

Exits 0 always (this is informational, not a check).
"""

from __future__ import annotations

import argparse
import hashlib
import json
import subprocess
import sys
from pathlib import Path

# Files/dirs to skip in knowledge/ when looking for sources
SKIP_NAMES = {"README.md", ".gitignore", ".gitkeep", ".DS_Store", "Thumbs.db"}
SKIP_DIRS = {"scratchpad"}


def find_repo_root():
    r = subprocess.run(
        ["git", "rev-parse", "--show-toplevel"],
        capture_output=True, text=True, check=False,
    )
    if r.returncode == 0 and r.stdout.strip():
        return Path(r.stdout.strip())
    return Path.cwd()


def sha256_file(path):
    h = hashlib.sha256()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            h.update(chunk)
    return h.hexdigest()


def scan_knowledge(knowledge_dir):
    """Walk knowledge/ and return {relative_path: sha256} for every source file."""
    sources = {}
    for p in knowledge_dir.rglob("*"):
        if not p.is_file():
            continue
        if p.name in SKIP_NAMES or p.name.startswith("."):
            continue
        # Skip anything under skipped subdirectories (scratchpad)
        rel_parts = p.relative_to(knowledge_dir).parts
        if any(part in SKIP_DIRS for part in rel_parts):
            continue
        rel = str(p.relative_to(knowledge_dir.parent))  # e.g. "knowledge/foo.md"
        sources[rel] = sha256_file(p)
    return sources


def load_manifest(manifest_path):
    if not manifest_path.exists():
        return {}
    try:
        data = json.loads(manifest_path.read_text())
    except json.JSONDecodeError:
        return {}
    sources = data.get("sources", {})
    # Manifest entries can be either {"path": "sha256-string"} or
    # {"path": {"sha256": "...", "ingested": "...", ...}}.
    normalized = {}
    for path, val in sources.items():
        if isinstance(val, str):
            normalized[path] = val
        elif isinstance(val, dict):
            normalized[path] = val.get("sha256", "")
    return normalized


def format_size(path):
    """Human-readable size or word count for a file."""
    try:
        size_bytes = path.stat().st_size
        if path.suffix.lower() in (".md", ".txt", ".csv", ".ld", ".rst", ".org"):
            try:
                text = path.read_text(encoding="utf-8", errors="replace")
                words = len(text.split())
                return f"{words:,} words"
            except (OSError, UnicodeDecodeError):
                pass
        if size_bytes < 1024:
            return f"{size_bytes} bytes"
        if size_bytes < 1024 * 1024:
            return f"{size_bytes / 1024:.1f} KB"
        return f"{size_bytes / (1024 * 1024):.1f} MB"
    except OSError:
        return "?"


def main():
    parser = argparse.ArgumentParser(description=__doc__,
                                     formatter_class=argparse.RawDescriptionHelpFormatter)
    parser.add_argument("--quiet", action="store_true",
                        help="Only print the agent prompt (for piping into clipboard tools)")
    args = parser.parse_args()

    repo_root = find_repo_root()
    knowledge_dir = repo_root / "knowledge"
    kb_root = repo_root / ".kb"
    manifest_path = kb_root / ".state" / "manifest.json"

    if not knowledge_dir.exists():
        print(f"ERROR: {knowledge_dir} does not exist", file=sys.stderr)
        return 1

    if not kb_root.exists():
        print(f"ERROR: {kb_root} does not exist — create the KB first:", file=sys.stderr)
        print(f"       uv run --script bootstrap.py --add-kb", file=sys.stderr)
        return 1

    # Scan
    current = scan_knowledge(knowledge_dir)
    manifest = load_manifest(manifest_path)

    new_sources = {p: sha for p, sha in current.items() if p not in manifest}
    changed_sources = {p: sha for p, sha in current.items()
                       if p in manifest and manifest[p] != sha}
    unchanged = sum(1 for p in current if p in manifest and manifest[p] == current[p])

    if not args.quiet:
        print(f"kb-ingest.py")
        print(f"  Scanning: {knowledge_dir}")
        print(f"  Manifest: {manifest_path}")
        print()
        print(f"  Total sources in knowledge/: {len(current)}")
        print(f"  Already ingested, unchanged: {unchanged}")
        print(f"  NEW (not in manifest):       {len(new_sources)}")
        print(f"  CHANGED (SHA256 drift):      {len(changed_sources)}")
        print()

        if new_sources:
            print(f"=== NEW sources ({len(new_sources)}) ===")
            for path in sorted(new_sources):
                size = format_size(repo_root / path)
                print(f"  {path}  ({size})")
            print()

        if changed_sources:
            print(f"=== CHANGED sources ({len(changed_sources)}) ===")
            for path in sorted(changed_sources):
                size = format_size(repo_root / path)
                old = manifest[path][:12] if manifest[path] else "?"
                new = current[path][:12]
                print(f"  {path}  ({size})")
                print(f"    old SHA256: {old}…  →  new: {new}…")
            print()

        if not new_sources and not changed_sources:
            print("Nothing to ingest. Everything in knowledge/ is up-to-date in the manifest.")
            return 0

    # Print agent prompt (always — even in quiet mode)
    if not args.quiet:
        print("─" * 70)
        print("Paste the following into your agent (Claude Code / Codex / Aider / ...):")
        print("─" * 70)
        print()

    print("Please switch to KB mode and process the following sources per `.kb/AGENTS.md`")
    print("INGEST rules. Read `.kb/AGENTS.md` first if you haven't this session.")
    print()

    if new_sources:
        print(f"NEW sources ({len(new_sources)}) — perform full INGEST:")
        for path in sorted(new_sources):
            print(f"  - {path}")
        print()
        print("For each NEW source:")
        print("  1. Read the source file")
        print("  2. Ask me which KB page types to produce (concept/snippet/decision/")
        print("     gotcha/reference/analysis), or propose a set if obvious")
        print("  3. Write the pages following `.kb/_templates/<type>.md`, with inline")
        print("     `[[src:<path>]]` citations on every non-trivial claim")
        print("  4. Set `status: proposed`, `confidence: low` (or `medium` with")
        print("     justification), `reviewed_by_human: never`")
        print("  5. Append SHA256 to `.kb/.state/manifest.json` under `sources`")
        print("  6. Update `.kb/.state/index.md` with new pages")
        print("  7. Append an `INGEST:` entry to `.kb/.state/log.md`")
        print("  8. Run `uv run scripts/validate-kb.py` and fix any reported issues")
        print()

    if changed_sources:
        print(f"CHANGED sources ({len(changed_sources)}) — perform DRIFT VERIFICATION:")
        for path in sorted(changed_sources):
            print(f"  - {path}")
        print()
        print("For each CHANGED source:")
        print("  1. Read the new version of the source")
        print("  2. Search `.kb/kb/` for pages citing it (grep for `[[src:<path>]]`)")
        print("  3. For each citing page, diff the source claim against what the page")
        print("     says — surface discrepancies as a written report (don't auto-edit)")
        print("  4. Ask me whether to:")
        print("       (a) update KB pages to match new source")
        print("       (b) leave KB pages and add a `[NEEDS SOURCE]` flag")
        print("       (c) supersede old pages with new ones (`status: superseded`)")
        print("  5. Only AFTER my approval: apply changes and update manifest SHA256")
        print("  6. Append a `VERIFY:` entry to `.kb/.state/log.md`")
        print()

    print("Reminder: `.kb/` is gitignored; nothing here gets committed. Run validator")
    print("when done: `uv run scripts/validate-kb.py`")

    return 0


if __name__ == "__main__":
    try:
        sys.exit(main())
    except BrokenPipeError:
        # Silently exit when stdout is closed (e.g. piped to `head`)
        sys.exit(0)

```

###### FILE: scripts/session-end.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
session-end.py — cleanup at the end of a session.

IMPORTANT: Claude Code's SessionEnd hook fires AFTER the agent has stopped.
The agent does NOT see this script's output. So this script is for CLEANUP
tasks only — anything that needs the agent's attention belongs in
`session-start.py` (which the agent reads at the next session).

What this script does:
  - Append a SESSION entry to `.kb/.state/log.md` with what happened
    (commits made, files changed, KB pages added) — so the next session can
    show "what happened last time" via session-start.py
  - Print a summary to logs (visible via `claude --debug` or claude-code-logs)

What this script does NOT do:
  - Surface pending KB ingestion to the agent (moved to session-start.py)
  - Show a harvest checklist (moved to session-start.py as session-long reminders)

Fires automatically via Claude Code SessionEnd hook (matchers: clear, resume,
logout, prompt_input_exit). Codex has no SessionEnd event.
Other agents: run manually before ending the session.
"""

from __future__ import annotations

import re
import subprocess
import sys
from datetime import datetime
from pathlib import Path


def run(args, cwd=None):
    try:
        return subprocess.run(args, cwd=cwd, capture_output=True, text=True, check=False)
    except (FileNotFoundError, OSError):
        return None


def find_repo_root():
    r = run(["git", "rev-parse", "--show-toplevel"])
    if r and r.returncode == 0 and r.stdout.strip():
        return Path(r.stdout.strip())
    return Path.cwd()


def gather_session_summary(repo_root):
    """Build a short factual summary of what the session did (for the log)."""
    summary = {
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M"),
        "branch": "(unknown)",
        "uncommitted": 0,
        "recent_commits": [],
        "kb_pages_count": 0,
        "scratchpad_count": 0,
    }

    if (repo_root / ".git").exists():
        r = run(["git", "branch", "--show-current"], cwd=repo_root)
        if r and r.returncode == 0:
            summary["branch"] = r.stdout.strip() or "(detached)"

        r = run(["git", "status", "--porcelain"], cwd=repo_root)
        if r:
            summary["uncommitted"] = len([l for l in r.stdout.splitlines() if l.strip()])

        r = run(["git", "log", "--oneline", "--since=12 hours ago"], cwd=repo_root)
        if r and r.stdout.strip():
            summary["recent_commits"] = r.stdout.strip().splitlines()[:10]

    kb_root = repo_root / ".kb"
    if (kb_root / "kb").exists():
        summary["kb_pages_count"] = sum(
            1 for p in (kb_root / "kb").rglob("*.md") if p.name != ".gitkeep"
        )

    scratch = repo_root / "knowledge" / "scratchpad"
    if scratch.exists():
        summary["scratchpad_count"] = sum(
            1 for p in scratch.rglob("*") if p.is_file() and p.name != ".gitignore"
        )

    return summary


def append_session_entry(kb_root, summary):
    """Append a SESSION block to .kb/.state/log.md so the next session can read it."""
    log_path = kb_root / ".state" / "log.md"
    if not log_path.exists():
        return False  # No KB initialized

    entry_lines = [
        "",
        f"## [{summary['timestamp']}] SESSION end",
        "",
        f"- Branch: `{summary['branch']}`",
        f"- Uncommitted changes at exit: {summary['uncommitted']}",
        f"- KB pages total: {summary['kb_pages_count']}",
        f"- Scratchpad notes pending: {summary['scratchpad_count']}",
    ]
    if summary["recent_commits"]:
        entry_lines.append(f"- Recent commits (last 12h):")
        for c in summary["recent_commits"]:
            entry_lines.append(f"  - {c}")
    entry_lines.append("")

    try:
        with open(log_path, "a", encoding="utf-8") as f:
            f.write("\n".join(entry_lines))
        return True
    except OSError as e:
        print(f"WARNING: could not append to {log_path}: {e}", file=sys.stderr)
        return False


def main():
    repo_root = find_repo_root()
    kb_root = repo_root / ".kb"

    # All output goes to logs only — agent isn't reachable here. Print to stderr
    # so it shows up clearly in debug logs and doesn't pollute any stdout pipeline.
    print(f"==== Session end — {datetime.now().strftime('%Y-%m-%d %H:%M')} ====",
          file=sys.stderr)
    print(f"  (cleanup-only; agent has already stopped)", file=sys.stderr)
    print(file=sys.stderr)

    summary = gather_session_summary(repo_root)

    print(f"  Branch:           {summary['branch']}", file=sys.stderr)
    print(f"  Uncommitted:      {summary['uncommitted']}", file=sys.stderr)
    print(f"  KB pages total:   {summary['kb_pages_count']}", file=sys.stderr)
    print(f"  Scratchpad notes: {summary['scratchpad_count']}", file=sys.stderr)
    if summary["recent_commits"]:
        print(f"  Recent commits (last 12h):", file=sys.stderr)
        for c in summary["recent_commits"]:
            print(f"    {c}", file=sys.stderr)

    if kb_root.exists():
        if append_session_entry(kb_root, summary):
            print(file=sys.stderr)
            print(f"  ✓ appended SESSION entry to .kb/.state/log.md", file=sys.stderr)
            print(f"    (next session-start.py will surface this to the agent)",
                  file=sys.stderr)

    print(file=sys.stderr)
    print(f"==== End ====", file=sys.stderr)
    return 0


if __name__ == "__main__":
    sys.exit(main())

```

###### FILE: scripts/session-start.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
session-start.py — what the agent needs to know to start working effectively.

Fires automatically via Claude Code / Codex SessionStart hooks (matchers:
startup|resume|clear|compact for Claude, startup|resume|clear for Codex).
Other agents: run manually at session start per AGENTS.md.

Output is consumed BY THE AGENT as additional context for this session.
Sections (in order):
  1. Repo + branch + dirty state — orient the agent
  2. Scope (from README.md ## Scope) — what this project is about
  3. Last session summary — what just happened (from .kb/.state/log.md)
  4. Pending KB ingestion — new/changed sources in knowledge/ to compile
  5. KB stats — pages by type, recent activity
  6. Harvest reminders — things to look out for during this session
"""

from __future__ import annotations

import hashlib
import json
import re
import subprocess
import sys
from datetime import datetime
from pathlib import Path

# Skip these when scanning knowledge/ for ingestable sources
_SKIP_NAMES = {"README.md", ".gitignore", ".gitkeep", ".DS_Store", "Thumbs.db"}
_SKIP_DIRS = {"scratchpad"}


def run(args, cwd=None):
    try:
        return subprocess.run(args, cwd=cwd, capture_output=True, text=True, check=False)
    except (FileNotFoundError, OSError):
        return None


def find_repo_root():
    r = run(["git", "rev-parse", "--show-toplevel"])
    if r and r.returncode == 0 and r.stdout.strip():
        return Path(r.stdout.strip())
    return Path.cwd()


def extract_scope_from_readme(repo_root):
    readme = repo_root / "README.md"
    if not readme.exists():
        return None
    text = readme.read_text(encoding="utf-8", errors="replace")
    m = re.search(r"##\s*Scope\s*\n(.+?)(?=\n##\s|\Z)", text, re.DOTALL | re.IGNORECASE)
    if not m:
        return None
    body = m.group(1).strip()
    body = re.sub(r"<!--.*?-->", "", body, flags=re.DOTALL).strip()
    if not body:
        return None
    lines = [l for l in body.splitlines() if l.strip()]
    if not lines:
        return None
    return body


def count_kb_pages_by_type(kb_dir):
    counts = {}
    if not kb_dir.exists():
        return counts
    for t in ["concept", "snippet", "decision", "gotcha", "reference", "analysis"]:
        td = kb_dir / t
        if td.exists():
            n = sum(1 for p in td.rglob("*.md") if p.name != ".gitkeep")
            if n > 0:
                counts[t] = n
    return counts


def _sha256_file(path):
    h = hashlib.sha256()
    try:
        with open(path, "rb") as f:
            for chunk in iter(lambda: f.read(65536), b""):
                h.update(chunk)
    except OSError:
        return ""
    return h.hexdigest()


def scan_knowledge_for_ingest(repo_root):
    """Compare knowledge/ against .kb/.state/manifest.json.

    Returns (new_sources, changed_sources, unchanged_count).
    Returns (None, None, None) if KB or knowledge/ doesn't exist.
    """
    knowledge_dir = repo_root / "knowledge"
    kb_root = repo_root / ".kb"
    manifest_path = kb_root / ".state" / "manifest.json"

    if not knowledge_dir.exists() or not kb_root.exists():
        return None, None, None

    current = {}
    for p in knowledge_dir.rglob("*"):
        if not p.is_file():
            continue
        if p.name in _SKIP_NAMES or p.name.startswith("."):
            continue
        rel_parts = p.relative_to(knowledge_dir).parts
        if any(part in _SKIP_DIRS for part in rel_parts):
            continue
        rel = str(p.relative_to(repo_root))
        current[rel] = _sha256_file(p)

    manifest_sources = {}
    if manifest_path.exists():
        try:
            data = json.loads(manifest_path.read_text())
            for path, val in data.get("sources", {}).items():
                if isinstance(val, str):
                    manifest_sources[path] = val
                elif isinstance(val, dict):
                    manifest_sources[path] = val.get("sha256", "")
        except json.JSONDecodeError:
            pass

    new = sorted(p for p in current if p not in manifest_sources)
    changed = sorted(p for p in current
                     if p in manifest_sources and manifest_sources[p] != current[p])
    unchanged = sum(1 for p in current
                    if p in manifest_sources and manifest_sources[p] == current[p])
    return new, changed, unchanged


def find_last_session_log_entry(kb_root):
    """Read .kb/.state/log.md and find the most recent SESSION entry."""
    log = kb_root / ".state" / "log.md"
    if not log.exists():
        return None
    try:
        text = log.read_text(encoding="utf-8")
    except OSError:
        return None
    sections = re.split(r"\n(?=##\s+\[)", text)
    for section in reversed(sections):
        if "SESSION" in section[:200]:
            lines = section.strip().splitlines()
            return "\n".join(lines[:15])
    return None


def main():
    repo_root = find_repo_root()
    kb_root = repo_root / ".kb"

    print(f"==== Session start — {datetime.now().strftime('%Y-%m-%d %H:%M')} ====")
    print()

    # ─── 1. Repo orientation ───────────────────────────────────
    print(f"Repo:    {repo_root}")
    if (repo_root / ".git").exists():
        r = run(["git", "branch", "--show-current"], cwd=repo_root)
        branch = r.stdout.strip() if r and r.returncode == 0 else "(unknown)"
        r = run(["git", "status", "--porcelain"], cwd=repo_root)
        dirty = len([l for l in (r.stdout if r else "").splitlines() if l.strip()])
        print(f"Branch:  {branch}")
        if dirty:
            print(f"Dirty:   {dirty} uncommitted change(s)")
            r = run(["git", "status", "--short"], cwd=repo_root)
            for line in (r.stdout if r else "").splitlines()[:10]:
                print(f"  {line}")
            if dirty > 10:
                print(f"  ... ({dirty - 10} more)")
    print()

    # ─── 2. Scope ──────────────────────────────────────────────
    scope = extract_scope_from_readme(repo_root)
    if scope:
        print("Scope (from README.md):")
        for line in scope.splitlines()[:8]:
            print(f"  {line}")
        if len(scope.splitlines()) > 8:
            print(f"  ... ({len(scope.splitlines()) - 8} more lines)")
        print()
    else:
        print("Scope: README.md has no `## Scope` section (or only a placeholder)")
        print("  Agent: ask the human to add one, or proceed without explicit scope")
        print()

    # ─── 3. What happened last session ─────────────────────────
    if kb_root.exists():
        last_session = find_last_session_log_entry(kb_root)
        if last_session:
            print("Last session (from .kb/.state/log.md):")
            for line in last_session.splitlines():
                print(f"  {line}")
            print()

    # ─── 4. Scratchpad ─────────────────────────────────────────
    scratch = repo_root / "knowledge" / "scratchpad"
    if scratch.exists():
        files = [p for p in scratch.rglob("*") if p.is_file() and p.name != ".gitignore"]
        if files:
            print(f"Scratchpad: {len(files)} note(s) accumulated")
            for p in files[:10]:
                print(f"  - {p.relative_to(scratch)}")
            if len(files) > 10:
                print(f"  ... ({len(files) - 10} more)")
            print()
            print("  Agent: review these during this session — ingest into KB,")
            print("  promote to knowledge/, or archive. Don't let scratchpad accumulate.")
            print()

    # ─── 5. KB ingestion status (NEW: moved from session-end) ──
    new, changed, unchanged = scan_knowledge_for_ingest(repo_root)
    if new is not None:
        if new or changed:
            print("==== KB ingestion status ====")
            print(f"  knowledge/ sources: {unchanged} ingested, "
                  f"{len(new)} new, {len(changed)} changed")
            print()
            if new:
                print(f"  NEW sources ({len(new)}) — never ingested:")
                for path in new:
                    print(f"    - {path}")
                print()
            if changed:
                print(f"  CHANGED sources ({len(changed)}) — SHA256 drifted since last ingest:")
                for path in changed:
                    print(f"    - {path}")
                print()
            print("  ─────────────────────────────────────────────────────────────────")
            print("  AGENT: pending KB ingestion work was detected. Do NOT auto-ingest.")
            print("  Mention this to the user near the start of the conversation and")
            print("  offer to switch to KB mode if they want to compile the new/changed")
            print("  sources now. If they decline, the work persists for next session.")
            print()
            print("  When the user says 'yes ingest' or similar:")
            print("    - Run `uv run scripts/kb-ingest.py` for the full agent prompt")
            print("    - Follow `.kb/AGENTS.md` INGEST rules for NEW sources")
            print("    - Follow drift verification flow for CHANGED sources")
            print("    - Run `uv run scripts/validate-kb.py` after writes")
            print("  ─────────────────────────────────────────────────────────────────")
            print()

    # ─── 6. KB stats ───────────────────────────────────────────
    if kb_root.exists():
        print(f"KB: {kb_root.relative_to(repo_root)}")
        counts = count_kb_pages_by_type(kb_root / "kb")
        if counts:
            for t, n in counts.items():
                print(f"  {t}: {n}")
        else:
            print("  (no pages yet)")
        print()
    else:
        print(f"No KB found at {kb_root.relative_to(repo_root)}")
        print("  Create one: uv run --script bootstrap.py --add-kb")
        print()

    # ─── 7. Harvest reminders for this session ─────────────────
    print("==== Harvest reminders (agent: keep these in mind during this session) ====")
    print()
    print("  Watch for KB-worthy moments and OFFER to file when they happen:")
    print("    [ ] A bug took >15 min to diagnose → propose a `gotcha/` page")
    print("    [ ] A decision was made between real alternatives → propose a `decision/`")
    print("    [ ] A reusable pattern emerged → propose a `snippet/`")
    print("    [ ] An investigation produced findings → propose an `analysis/`")
    print("    [ ] A surprising library behavior → propose `gotcha/` or `concept/`")
    print()
    print("  Don't auto-file. Always ask. New pages go in `.kb/` (gitignored).")
    print()
    print("==== End of session start ====")
    return 0


if __name__ == "__main__":
    sys.exit(main())

```

###### FILE: scripts/validate-kb.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# dependencies = ["pyyaml>=6.0"]
# ///
"""
validate-kb.py — mechanical enforcement of KB rules for .kb/

Runs over the entire .kb/ directory. Agents should run this at session end;
users can run anytime. It does NOT run on git commit (KB is gitignored).

Usage:
    uv run scripts/validate-kb.py           # validate .kb/ from repo root
    uv run scripts/validate-kb.py <path>    # validate a specific KB path

Rules enforced:
    Frontmatter: required fields, valid enum values, confidence-review pairing,
                 version pinning unless confidence:low, non-empty sources list.
    Body:        claims need [[src:...]] / [[kb:...]] citations or
                 [NEEDS SOURCE]/[inferred] gap flags.
    Ceilings:    ≤50 sources (via manifest.json), ≤120 pages, ≤200k words.

Exits 1 on any violation.
"""

from __future__ import annotations
import argparse
import json
import re
import subprocess
import sys
from dataclasses import dataclass, field
from pathlib import Path

try:
    import yaml
except ImportError:
    print("ERROR: pyyaml not available. Run via `uv run --script` or `pip install pyyaml`.",
          file=sys.stderr)
    sys.exit(2)


MAX_RAW_SOURCES = 50
MAX_PAGES = 120
MAX_WORDS = 200_000

VALID_TYPES = {"concept", "snippet", "decision", "gotcha", "reference", "analysis"}
VALID_STATUSES = {"proposed", "accepted", "superseded", "archived"}
VALID_CONFIDENCE = {"high", "medium", "low"}

TRIVIAL_PATTERNS = [
    re.compile(r"^#+\s"),
    re.compile(r"^```"),
    re.compile(r"^\s*[-*]\s*$"),
    re.compile(r"^\s*$"),
    re.compile(r"^>\s"),
]
CITATION_RE = re.compile(r"\[\[(?:src|raw|kb|ledger):[^\]]+\]\]")
FLAG_RE = re.compile(r"\[(NEEDS SOURCE|inferred)\]")
VERB_RE = re.compile(
    r"\b(is|are|was|were|returns?|raises?|causes?|"
    r"requires?|must|should|cannot|will|does)\b", re.IGNORECASE)
HOUSEKEEPING = {".gitkeep", ".gitignore", ".DS_Store"}


@dataclass
class Issue:
    path: Path
    line: int
    rule: str
    message: str

    def __str__(self):
        return f"{self.path}:{self.line}  [{self.rule}]  {self.message}"


@dataclass
class Report:
    issues: list = field(default_factory=list)
    pages_checked: int = 0
    total_words: int = 0

    def add(self, i):
        self.issues.append(i)

    @property
    def ok(self):
        return not self.issues


def parse_frontmatter(text):
    if not text.startswith("---\n"):
        return None, text, 1
    end = text.find("\n---\n", 4)
    if end == -1:
        return None, text, 1
    fm_text = text[4:end]
    body = text[end + 5:]
    try:
        fm = yaml.safe_load(fm_text) or {}
    except yaml.YAMLError:
        return None, body, 1
    body_start = fm_text.count("\n") + 3
    return fm, body, body_start


def is_trivial(line):
    return any(p.match(line) for p in TRIVIAL_PATTERNS)


def looks_like_claim(line):
    s = line.strip()
    if len(s) < 40 or is_trivial(line):
        return False
    if s.startswith(("- **", "* **")) and s.endswith("**"):
        return False
    return True


def validate_frontmatter(fm, path, report):
    required = ["type", "title", "status", "confidence", "reviewed_by_human", "sources"]
    for f in required:
        if f not in fm:
            report.add(Issue(path, 1, "frontmatter", f"missing field: {f}"))

    t = fm.get("type")
    if t and t not in VALID_TYPES:
        report.add(Issue(path, 1, "frontmatter",
                         f"invalid type '{t}' (valid: {sorted(VALID_TYPES)})"))

    s = fm.get("status")
    if s and s not in VALID_STATUSES:
        report.add(Issue(path, 1, "frontmatter", f"invalid status '{s}'"))

    c = fm.get("confidence")
    if c and c not in VALID_CONFIDENCE:
        report.add(Issue(path, 1, "frontmatter", f"invalid confidence '{c}'"))

    if c == "high" and fm.get("reviewed_by_human") in (None, "never"):
        report.add(Issue(path, 1, "principle-4",
                         "confidence:high requires reviewed_by_human"))

    if t in {"concept", "snippet", "reference", "gotcha", "analysis"}:
        has_version = any(k in fm for k in ("version_range", "version_tested",
                                            "version_affected", "version"))
        if not has_version and c != "low":
            report.add(Issue(path, 1, "principle-3",
                             "version field required unless confidence:low"))

    sources = fm.get("sources")
    if sources is not None and (not isinstance(sources, list) or not sources):
        report.add(Issue(path, 1, "frontmatter",
                         "sources must be a non-empty list"))


def validate_body(body, body_start, path, report):
    in_code = False
    words = 0
    for offset, line in enumerate(body.splitlines()):
        line_no = body_start + offset
        if line.strip().startswith("```"):
            in_code = not in_code
            continue
        if in_code:
            words += len(line.split())
            continue
        words += len(line.split())
        if not looks_like_claim(line):
            continue
        if not (CITATION_RE.search(line) or FLAG_RE.search(line)):
            if VERB_RE.search(line):
                report.add(Issue(path, line_no, "principle-3",
                                 "claim without [[src:...]] or [NEEDS SOURCE]/[inferred]"))
    return words


def validate_page(path, report):
    report.pages_checked += 1
    try:
        text = path.read_text(encoding="utf-8")
    except UnicodeDecodeError:
        report.add(Issue(path, 1, "encoding", "not utf-8"))
        return
    fm, body, bs = parse_frontmatter(text)
    if fm is None:
        report.add(Issue(path, 1, "frontmatter", "missing/invalid frontmatter"))
        return
    validate_frontmatter(fm, path, report)
    report.total_words += validate_body(body, bs, path, report)


def validate_ceilings(kb_root, report):
    kb_dir = kb_root / "kb"
    if kb_dir.exists():
        pages = [p for p in kb_dir.rglob("*.md") if p.name not in HOUSEKEEPING]
        if len(pages) > MAX_PAGES:
            report.add(Issue(kb_dir, 1, "principle-1",
                             f"page count {len(pages)} > {MAX_PAGES}"))
    if report.total_words > MAX_WORDS:
        report.add(Issue(kb_root, 1, "principle-1",
                         f"total words {report.total_words} > {MAX_WORDS}"))
    manifest = kb_root / ".state" / "manifest.json"
    if manifest.exists():
        try:
            data = json.loads(manifest.read_text())
            n = len(data.get("sources", {}))
            if n > MAX_RAW_SOURCES:
                report.add(Issue(manifest, 1, "principle-1",
                                 f"source count {n} > {MAX_RAW_SOURCES}"))
        except json.JSONDecodeError as e:
            report.add(Issue(manifest, 1, "principle-2",
                             f"manifest.json invalid: {e}"))


def validate_kb(kb_root, report):
    kb_dir = kb_root / "kb"
    if kb_dir.exists():
        for p in kb_dir.rglob("*.md"):
            if p.name in HOUSEKEEPING:
                continue
            validate_page(p, report)
    validate_ceilings(kb_root, report)


def main():
    parser = argparse.ArgumentParser(description="Validate the .kb/ directory.")
    parser.add_argument("path", nargs="?", default=None,
                        help="KB directory path (default: <repo>/.kb/)")
    args = parser.parse_args()

    if args.path:
        kb_root = Path(args.path).resolve()
    else:
        r = subprocess.run(["git", "rev-parse", "--show-toplevel"],
                           capture_output=True, text=True, check=False)
        repo = Path(r.stdout.strip()) if r.returncode == 0 and r.stdout.strip() else Path.cwd()
        kb_root = repo / ".kb"

    if not kb_root.exists():
        print(f"ERROR: KB not found at {kb_root}", file=sys.stderr)
        print("  Create one: uv run --script bootstrap.py --add-kb", file=sys.stderr)
        return 1

    report = Report()
    validate_kb(kb_root, report)

    print(f"Validated {kb_root}")
    print(f"  {report.pages_checked} page(s), {report.total_words:,} word(s)")
    if report.issues:
        print(f"\n{len(report.issues)} issue(s):\n")
        for issue in report.issues:
            print(f"  {issue}")
        return 1
    print("  All rules passed.")
    return 0


if __name__ == "__main__":
    sys.exit(main())

```

###### FILE: scripts/verify-hooks.py ######

```py
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# ///
"""
verify-hooks.py — diagnose why agent hooks aren't firing.

Runs each piece of the hook chain manually and reports what works and what doesn't.
Use this when `claude --continue` (or codex) starts and you don't see the
session-start output.

Usage:
    uv run scripts/verify-hooks.py             # check everything
    uv run scripts/verify-hooks.py --agent claude
    uv run scripts/verify-hooks.py --agent codex

Exit codes:
    0 — all configured agents pass all checks
    1 — at least one configured agent has a problem
    2 — usage error
"""

from __future__ import annotations

import argparse
import json
import os
import shutil
import subprocess
import sys
from pathlib import Path


GREEN = "\033[32m" if sys.stdout.isatty() else ""
RED = "\033[31m" if sys.stdout.isatty() else ""
YELLOW = "\033[33m" if sys.stdout.isatty() else ""
DIM = "\033[2m" if sys.stdout.isatty() else ""
RESET = "\033[0m" if sys.stdout.isatty() else ""


def ok(msg):
    print(f"  {GREEN}✓{RESET} {msg}")


def fail(msg):
    print(f"  {RED}✗{RESET} {msg}")


def warn(msg):
    print(f"  {YELLOW}!{RESET} {msg}")


def info(msg):
    print(f"  {DIM}{msg}{RESET}")


def find_repo_root():
    r = subprocess.run(
        ["git", "rev-parse", "--show-toplevel"],
        capture_output=True, text=True, check=False,
    )
    if r.returncode == 0 and r.stdout.strip():
        return Path(r.stdout.strip())
    return Path.cwd()


def check_uv():
    """uv must be on PATH for the hook commands to work."""
    print(f"\n{DIM}─── uv availability ───{RESET}")
    uv_path = shutil.which("uv")
    if uv_path:
        ok(f"uv found at: {uv_path}")
        r = subprocess.run([uv_path, "--version"], capture_output=True, text=True)
        if r.returncode == 0:
            info(f"version: {r.stdout.strip()}")
        return True
    fail("uv not found on PATH")
    info("Install uv: https://docs.astral.sh/uv/")
    info("If uv is installed but not on PATH, the agent's environment may differ from yours.")
    info("Consider replacing 'uv' in hook commands with the full path to uv.")
    return False


def check_scripts_exist(repo_root):
    """The hook command points to scripts/session-start.py and session-end.py."""
    print(f"\n{DIM}─── hook script files ───{RESET}")
    all_ok = True
    for name in ("session-start.py", "session-end.py"):
        p = repo_root / "scripts" / name
        if p.exists():
            ok(f"scripts/{name} exists")
            if not os.access(p, os.X_OK) and not sys.platform.startswith("win"):
                warn(f"scripts/{name} is not executable (this is okay if invoked via uv)")
        else:
            fail(f"scripts/{name} NOT FOUND")
            all_ok = False
    return all_ok


def execute_hook_command(cmd, label, repo_root):
    """Run a hook command exactly as the agent would, report what happened."""
    print(f"\n{DIM}─── executing: {cmd} ───{RESET}")
    try:
        r = subprocess.run(
            cmd, shell=True, cwd=repo_root,
            capture_output=True, text=True, timeout=30,
        )
    except subprocess.TimeoutExpired:
        fail(f"{label}: TIMED OUT after 30s")
        info("This would cause the agent to record a hook error.")
        return False
    except Exception as e:
        fail(f"{label}: failed to execute: {e}")
        return False

    if r.returncode == 0:
        ok(f"{label}: exit code 0 (success)")
        if r.stdout.strip():
            info("stdout (first 5 lines):")
            for line in r.stdout.splitlines()[:5]:
                print(f"    {line}")
            if len(r.stdout.splitlines()) > 5:
                info(f"... ({len(r.stdout.splitlines()) - 5} more lines)")
        if r.stderr.strip():
            info("stderr (informational; non-empty doesn't mean failure):")
            for line in r.stderr.splitlines()[:5]:
                print(f"    {line}")
        return True

    fail(f"{label}: exit code {r.returncode}")
    if r.stdout.strip():
        info("stdout:")
        for line in r.stdout.splitlines()[:10]:
            print(f"    {line}")
    if r.stderr.strip():
        info("stderr:")
        for line in r.stderr.splitlines()[:10]:
            print(f"    {line}")
    return False


def check_claude_config(repo_root):
    """Inspect .claude/settings.local.json and validate its hooks."""
    print(f"\n{DIM}═══ Claude Code ═══{RESET}")
    config_path = repo_root / ".claude" / "settings.local.json"
    print(f"\n{DIM}─── config file ───{RESET}")

    if not config_path.exists():
        fail(f"{config_path.relative_to(repo_root)} NOT FOUND")
        info("Run: uv run scripts/install-hooks.py --agent claude")
        return False

    ok(f"{config_path.relative_to(repo_root)} exists")

    try:
        cfg = json.loads(config_path.read_text())
    except json.JSONDecodeError as e:
        fail(f"invalid JSON: {e}")
        return False
    ok("valid JSON")

    if "hooks" not in cfg:
        fail("'hooks' key missing — file does not register any hooks")
        return False

    all_ok = True
    for event in ("SessionStart", "SessionEnd"):
        groups = cfg["hooks"].get(event, [])
        if not groups:
            warn(f"{event}: no hooks configured")
            continue
        for group in groups:
            matcher = group.get("matcher", "(none — fires on all paths)")
            for h in group.get("hooks", []):
                cmd = h.get("command", "")
                ok(f"{event} (matcher={matcher}): {cmd}")
                # Execute the command to see what actually happens
                if not execute_hook_command(cmd, f"Claude {event}", repo_root):
                    all_ok = False

    print(f"\n{DIM}─── Claude Code platform notes ───{RESET}")
    if sys.platform.startswith("win"):
        warn("Windows + Git Bash: Claude Code may show 'SessionStart:startup hook error'")
        info("This is cosmetic — see anthropics/claude-code#12671")
        info("Run `claude --debug` to verify hook actually fires (look for hook execution lines)")

    info("Inside Claude Code, run /hooks to see registered hooks")
    info("Inside Claude Code, run /context to see what's loaded")

    return all_ok


def check_codex_config(repo_root):
    """Inspect .codex/hooks.json and validate its hooks."""
    print(f"\n{DIM}═══ Codex ═══{RESET}")
    config_path = repo_root / ".codex" / "hooks.json"
    print(f"\n{DIM}─── config file ───{RESET}")

    if not config_path.exists():
        fail(f"{config_path.relative_to(repo_root)} NOT FOUND")
        info("Run: uv run scripts/install-hooks.py --agent codex")
        return False

    ok(f"{config_path.relative_to(repo_root)} exists")

    try:
        cfg = json.loads(config_path.read_text())
    except json.JSONDecodeError as e:
        fail(f"invalid JSON: {e}")
        return False
    ok("valid JSON")

    if "hooks" not in cfg:
        fail("'hooks' key missing — file does not register any hooks")
        return False

    all_ok = True
    for event in ("SessionStart",):
        groups = cfg["hooks"].get(event, [])
        if not groups:
            warn(f"{event}: no hooks configured")
            continue
        for group in groups:
            matcher = group.get("matcher", "(none — fires on all paths)")
            for h in group.get("hooks", []):
                cmd = h.get("command", "")
                ok(f"{event} (matcher={matcher}): {cmd}")
                if not execute_hook_command(cmd, f"Codex {event}", repo_root):
                    all_ok = False

    print(f"\n{DIM}─── Codex feature flag check ───{RESET}")
    codex_home = Path(os.environ.get("CODEX_HOME") or Path.home() / ".codex")
    config_toml = codex_home / "config.toml"
    if config_toml.exists():
        text = config_toml.read_text()
        # Look for hooks=true under [features]
        in_features = False
        hooks_enabled = False
        for line in text.splitlines():
            stripped = line.strip()
            if stripped.startswith("["):
                in_features = stripped == "[features]"
            elif in_features and "hooks" in stripped and "true" in stripped:
                hooks_enabled = True
                break
        if hooks_enabled:
            ok(f"hooks feature enabled in {config_toml}")
        else:
            warn(f"hooks feature NOT enabled in {config_toml}")
            info("Codex hooks are experimental and OFF by default.")
            info("Add to ~/.codex/config.toml:")
            info("  [features]")
            info("  hooks = true")
            info("Or one-shot: codex --enable hooks")
    else:
        warn(f"{config_toml} not found")
        info("Codex hooks need [features] hooks=true in this file (experimental).")

    print(f"\n{DIM}─── Codex platform notes ───{RESET}")
    if sys.platform.startswith("win"):
        warn("Codex hooks are currently DISABLED on Windows")
        info("Hooks won't fire regardless of config — this is a Codex limitation.")
        info("Run scripts/session-start.py manually instead.")

    info("Codex has no SessionEnd event — run scripts/session-end.py manually")

    return all_ok


def check_agent_session_files(repo_root):
    """Quick validation that the hook output is what the agent expects."""
    print(f"\n{DIM}─── what the hook would inject into the session ───{RESET}")
    ss = repo_root / "scripts" / "session-start.py"
    if not ss.exists():
        return
    info("Sample session-start.py output (this is what the agent sees as 'additional context'):")
    try:
        r = subprocess.run(
            ["uv", "run", str(ss)] if shutil.which("uv") else [sys.executable, str(ss)],
            cwd=repo_root, capture_output=True, text=True, timeout=15,
        )
        for line in r.stdout.splitlines()[:20]:
            print(f"    {line}")
        if len(r.stdout.splitlines()) > 20:
            info(f"... ({len(r.stdout.splitlines()) - 20} more lines)")
    except Exception as e:
        warn(f"could not preview output: {e}")


def main():
    parser = argparse.ArgumentParser(
        description=__doc__,
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument(
        "--agent", choices=["claude", "codex", "all"], default="all",
        help="Which agent to verify (default: all configured)",
    )
    args = parser.parse_args()

    repo_root = find_repo_root()
    print(f"verify-hooks.py")
    print(f"  Repo:     {repo_root}")
    print(f"  Platform: {sys.platform}")
    print(f"  Python:   {sys.version.split()[0]}")

    overall_ok = True
    overall_ok &= check_uv()
    overall_ok &= check_scripts_exist(repo_root)

    claude_configured = (repo_root / ".claude" / "settings.local.json").exists()
    codex_configured = (repo_root / ".codex" / "hooks.json").exists()

    if args.agent in ("claude", "all"):
        if claude_configured:
            overall_ok &= check_claude_config(repo_root)
        elif args.agent == "claude":
            print(f"\n{DIM}═══ Claude Code ═══{RESET}")
            warn("Claude Code config not found — skipping")
            info("Run: uv run scripts/install-hooks.py --agent claude")

    if args.agent in ("codex", "all"):
        if codex_configured:
            overall_ok &= check_codex_config(repo_root)
        elif args.agent == "codex":
            print(f"\n{DIM}═══ Codex ═══{RESET}")
            warn("Codex config not found — skipping")
            info("Run: uv run scripts/install-hooks.py --agent codex")

    if claude_configured or codex_configured:
        check_agent_session_files(repo_root)

    print()
    if overall_ok:
        print(f"{GREEN}All configured hooks check out.{RESET}")
        print()
        print("If hooks STILL don't fire in the agent:")
        print("  1. Make sure you start the agent from the repo root (cd into the project first)")
        print("  2. Inside Claude Code: run /hooks to see registered hooks")
        print("  3. Run with debug logs: `claude --debug` and look for hook execution lines")
        print("  4. Check the agent's PATH — `uv` must be findable in the agent's environment,")
        print("     not just your terminal's. On Windows, this often requires an absolute path.")
        return 0

    print(f"{RED}Some checks failed — see output above.{RESET}")
    return 1


if __name__ == "__main__":
    sys.exit(main())

```

