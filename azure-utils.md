# Code dump: azfr-azure-utils

Generated from `/c/Users/LARBANI/repos_git/azfr-azure-utils`.

## File tree

```
.github/workflows/build.yaml
.gitignore
.pre-commit-config.yaml
.python-version
docs/_static/.gitkeep
docs/_templates/conf.py.jinja
docs/_templates/root_doc.rst.jinja
docs/azfr_azure_utils.identity.aio.rst
docs/azfr_azure_utils.identity.rst
docs/azfr_azure_utils.postgresql.asyncpg.rst
docs/azfr_azure_utils.postgresql.asyncpg.sqlalchemy_async.rst
docs/azfr_azure_utils.postgresql.pg8000.rst
docs/azfr_azure_utils.postgresql.psycopg.pool.rst
docs/azfr_azure_utils.postgresql.psycopg.rst
docs/azfr_azure_utils.postgresql.psycopg.sqlalchemy_async.rst
docs/azfr_azure_utils.postgresql.psycopg2.pool.rst
docs/azfr_azure_utils.postgresql.psycopg2.rst
docs/azfr_azure_utils.postgresql.rst
docs/azfr_azure_utils.rst
docs/azfr_azure_utils.secrets.rst
docs/azfr_azure_utils.trino.rst
docs/conf.py
docs/index.rst
docs/make.bat
docs/Makefile
Makefile
pyproject.toml
README.md
src/azfr_azure_utils/__init__.py
src/azfr_azure_utils/databricks/__init__.py
src/azfr_azure_utils/databricks/sdk.py
src/azfr_azure_utils/databricks/sql.py
src/azfr_azure_utils/identity/__init__.py
src/azfr_azure_utils/identity/aio.py
src/azfr_azure_utils/postgresql/__init__.py
src/azfr_azure_utils/postgresql/asyncpg/__init__.py
src/azfr_azure_utils/postgresql/asyncpg/sqlalchemy_async.py
src/azfr_azure_utils/postgresql/pg8000.py
src/azfr_azure_utils/postgresql/psycopg/__init__.py
src/azfr_azure_utils/postgresql/psycopg/pool.py
src/azfr_azure_utils/postgresql/psycopg/sqlalchemy_async.py
src/azfr_azure_utils/postgresql/psycopg2/__init__.py
src/azfr_azure_utils/postgresql/psycopg2/pool.py
src/azfr_azure_utils/secrets/__init__.py
src/azfr_azure_utils/trino.py
tests/dummy_test.py
tox.ini
```

## File contents

###### FILE: .github/workflows/build.yaml

```yaml
name: Build Python library

env:
  # Run tests by default
  RUN_TESTS: "true"
  # Build and publish Python library by default
  PUBLISH_ARTIFACTS: "true"
  # USe cache data by default
  USE_CACHE_DATA: "true"
  # Credentials - don't touch
  AZFR_CI_USERNAME: ${{ secrets.AZFR_CI_USERNAME }}
  AZFR_CI_PASSWORD: ${{ secrets.AZFR_CI_PASSWORD }}
  AZFR_PYPI_INTERNAL_REPO: ${{ secrets.AZFR_PYPI_INTERNAL_REPO }}
  AZFR_RAW_DOC_ROOT: ${{ secrets.AZFR_RAW_DOC_ROOT }}


on:
  workflow_dispatch:
    inputs:
      run_tests:
        description: "Run tests"
        required: true
        type: choice
        options: [ "true", "false" ]
        default: "true"
      publish_artifacts:
        description: "Publish Python module"
        required: true
        type: choice
        options: [ "true", "false" ]
        default: "true"
      use_cache_data:
        description: "Use cache data"
        required: true
        type: choice
        options: [ "true", "false" ]
        default: "true"
  push: { }

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:

  build:
    name: Test, build and publish the library
    runs-on: [ self-hosted ]
    container:
      image: prodazfrz6sh.azurecr.io/cicd-job:py313
      volumes:
        - /var/cache/gha:/var/cache/gha

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: azf-h1-datascience/use-cache@release
        with:
          uv: true
          sonar: true
          tox: true
          disable_cache_read: ${{ inputs.use_cache_data == 'false' || (inputs.use_cache_data == '' && env.USE_CACHE_DATA == 'false') }}

      - name: Test
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        run: |
          make update reports

      - name: Run SonarScanner
        uses: azf-h1-datascience/sonar-scanner/run@release
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        with:
          host_url: ${{ secrets.SONAR_HOST_URL }}
          token: ${{ secrets.SONAR_TOKEN }}
          sources: src
          params: |
            sonar.python.version: 3
            sonar.python.coverage.reportPaths: target/report/coverage.xml

      - name: Cross-version test
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        run: |
          make update tox

      - name: Publish Python module
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        run: |
          make deploy PYPI_REPO_URL=$AZFR_PYPI_INTERNAL_REPO \
                      PYPI_REPO_USER=$AZFR_CI_USERNAME \
                      PYPI_REPO_PASS=$AZFR_CI_PASSWORD

      - name: Publish documentation
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        shell: bash
        run: |
          if [[ -f "docs/index.rst" ]]; then
            make deploy-docs DOC_REPO_ROOT=$AZFR_RAW_DOC_ROOT \
                             DOC_REPO_USER=$AZFR_CI_USERNAME \
                             DOC_REPO_PASS=$AZFR_CI_PASSWORD
          fi
```

###### FILE: .gitignore

```gitignore
.DS_Store
.env
*.pyc
*.pyo
/.venv/
/target/
/build/
dist/
*.egg
*.egg-info/
.eggs/
.cache/
.pytest_cache/
.coverage
.tox/
/docs/_build/
.idea/
*.iml
uv.lock
/.claude
```

###### FILE: .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-docstring-first
      - id: check-json
      - id: check-yaml
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.12.0
    hooks:
      - id: ruff
        args: [ --fix, "--show-fixes"]
      - id: ruff-format
        types_or: [python]
```

###### FILE: .python-version

```python-version
3.14
```

###### FILE: docs/_templates/conf.py.jinja ######

```jinja
# Configuration file for the Sphinx documentation builder.
#
# For the full list of built-in configuration values, see the documentation:
# https://www.sphinx-doc.org/en/master/usage/configuration.html

{% if append_syspath -%}
# -- Path setup --------------------------------------------------------------

import os
import sys
sys.path.insert(0, {{ module_path | repr }})

{% endif -%}
# -- Project information -----------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#project-information

project = {{ project | repr }}
copyright = {{ copyright | repr }}
author = {{ author | repr }}

{%- if version %}

version = {{ version | repr }}
{%- endif %}
{%- if release %}
release = {{ release | repr }}
{%- endif %}

# -- General configuration ---------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#general-configuration

extensions = [{% if extensions %}
              {%- for ext in extensions %}
              '{{ ext }}',
              {%- endfor %}
              {% endif -%}
              'myst_parser',
]

templates_path = ['{{ dot }}templates']
exclude_patterns = [{{ exclude_patterns }}]

{% if suffix != '.rst' -%}
source_suffix = {{ suffix | repr }}
{% endif -%}

{% if root_doc != 'index' -%}
root_doc = {{ root_doc | repr }}
{% endif -%}

{% if language -%}
language = {{ language | repr }}
{%- endif %}

# -- Options for HTML output -------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#options-for-html-output

html_theme = 'alabaster'
html_static_path = ['{{ dot }}static']
{% if 'sphinx.ext.intersphinx' in extensions %}
# -- Options for intersphinx extension ---------------------------------------
# https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html#configuration

intersphinx_mapping = {
    'python': ('https://docs.python.org/3', None),
}
{% endif -%}
{% if 'sphinx.ext.todo' in extensions %}
# -- Options for todo extension ----------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/extensions/todo.html#configuration

todo_include_todos = True
{% endif %}
```

###### FILE: docs/_templates/root_doc.rst.jinja ######

```jinja
.. include:: ../README.md
   :parser: markdown

.. toctree::
   :maxdepth: 4
   :hidden:

{{ mastertoctree }}
```

###### FILE: docs/azfr_azure_utils.identity.aio.rst

```rst
azfr\_azure\_utils.identity.aio module
======================================

.. automodule:: azfr_azure_utils.identity.aio
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.identity.rst

```rst
azfr\_azure\_utils.identity package
===================================

.. automodule:: azfr_azure_utils.identity
   :members:
   :show-inheritance:
   :undoc-members:

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.identity.aio
```

###### FILE: docs/azfr_azure_utils.postgresql.asyncpg.rst

```rst
azfr\_azure\_utils.postgresql.asyncpg package
=============================================

.. automodule:: azfr_azure_utils.postgresql.asyncpg
   :members:
   :show-inheritance:
   :undoc-members:

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.postgresql.asyncpg.sqlalchemy_async
```

###### FILE: docs/azfr_azure_utils.postgresql.asyncpg.sqlalchemy_async.rst

```rst
azfr\_azure\_utils.postgresql.asyncpg.sqlalchemy\_async module
==============================================================

.. automodule:: azfr_azure_utils.postgresql.asyncpg.sqlalchemy_async
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.postgresql.pg8000.rst

```rst
azfr\_azure\_utils.postgresql.pg8000 module
===========================================

.. automodule:: azfr_azure_utils.postgresql.pg8000
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.postgresql.psycopg.pool.rst

```rst
azfr\_azure\_utils.postgresql.psycopg.pool module
=================================================

.. automodule:: azfr_azure_utils.postgresql.psycopg.pool
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.postgresql.psycopg.rst

```rst
azfr\_azure\_utils.postgresql.psycopg package
=============================================

.. automodule:: azfr_azure_utils.postgresql.psycopg
   :members:
   :show-inheritance:
   :undoc-members:

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.postgresql.psycopg.pool
   azfr_azure_utils.postgresql.psycopg.sqlalchemy_async
```

###### FILE: docs/azfr_azure_utils.postgresql.psycopg.sqlalchemy_async.rst

```rst
azfr\_azure\_utils.postgresql.psycopg.sqlalchemy\_async module
==============================================================

.. automodule:: azfr_azure_utils.postgresql.psycopg.sqlalchemy_async
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.postgresql.psycopg2.pool.rst

```rst
azfr\_azure\_utils.postgresql.psycopg2.pool module
==================================================

.. automodule:: azfr_azure_utils.postgresql.psycopg2.pool
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.postgresql.psycopg2.rst

```rst
azfr\_azure\_utils.postgresql.psycopg2 package
==============================================

.. automodule:: azfr_azure_utils.postgresql.psycopg2
   :members:
   :show-inheritance:
   :undoc-members:

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.postgresql.psycopg2.pool
```

###### FILE: docs/azfr_azure_utils.postgresql.rst

```rst
azfr\_azure\_utils.postgresql package
=====================================

.. automodule:: azfr_azure_utils.postgresql
   :members:
   :show-inheritance:
   :undoc-members:

Subpackages
-----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.postgresql.asyncpg
   azfr_azure_utils.postgresql.psycopg
   azfr_azure_utils.postgresql.psycopg2

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.postgresql.pg8000
```

###### FILE: docs/azfr_azure_utils.rst

```rst
azfr\_azure\_utils package
==========================

.. automodule:: azfr_azure_utils
   :members:
   :show-inheritance:
   :undoc-members:

Subpackages
-----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.identity
   azfr_azure_utils.postgresql
   azfr_azure_utils.secrets

Submodules
----------

.. toctree::
   :maxdepth: 4

   azfr_azure_utils.trino
```

###### FILE: docs/azfr_azure_utils.secrets.rst

```rst
azfr\_azure\_utils.secrets package
==================================

.. automodule:: azfr_azure_utils.secrets
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/azfr_azure_utils.trino.rst

```rst
azfr\_azure\_utils.trino module
===============================

.. automodule:: azfr_azure_utils.trino
   :members:
   :show-inheritance:
   :undoc-members:
```

###### FILE: docs/conf.py

```py
# Configuration file for the Sphinx documentation builder.
#
# For the full list of built-in configuration values, see the documentation:
# https://www.sphinx-doc.org/en/master/usage/configuration.html

# -- Project information -----------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#project-information

project = "azfr-azure-utils"
copyright = "2025, Allianz Technology"
author = "Allianz Technology"

# -- General configuration ---------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#general-configuration

extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.viewcode",
    "sphinx.ext.todo",
    "myst_parser",
]

templates_path = ["_templates"]
exclude_patterns = ["_build", "Thumbs.db", ".DS_Store"]

language = "en"

# -- Options for HTML output -------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#options-for-html-output

html_theme = "alabaster"
html_static_path = ["_static"]

# -- Options for todo extension ----------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/extensions/todo.html#configuration

todo_include_todos = True
```

###### FILE: docs/index.rst

```rst
.. include:: ../README.md
   :parser: markdown

.. toctree::
   :maxdepth: 4
   :hidden:

   azfr_azure_utils
```

###### FILE: docs/make.bat

```bat
@ECHO OFF

pushd %~dp0

REM Command file for Sphinx documentation

if "%SPHINXBUILD%" == "" (
    set SPHINXBUILD=sphinx-build
)
set SOURCEDIR=.
set BUILDDIR=_build

%SPHINXBUILD% >NUL 2>NUL
if errorlevel 9009 (
    echo.
    echo.The 'sphinx-build' command was not found. Make sure you have Sphinx
    echo.installed, then set the SPHINXBUILD environment variable to point
    echo.to the full path of the 'sphinx-build' executable. Alternatively you
    echo.may add the Sphinx directory to PATH.
    echo.
    echo.If you don't have Sphinx installed, grab it from
    echo.https://www.sphinx-doc.org/
    exit /b 1
)

if "%1" == "" goto help

%SPHINXBUILD% -M %1 %SOURCEDIR% %BUILDDIR% %SPHINXOPTS% %O%
goto end

:help
%SPHINXBUILD% -M help %SOURCEDIR% %BUILDDIR% %SPHINXOPTS% %O%

:end
popd
```

###### FILE: docs/Makefile

```docs/makefile
# Minimal makefile for Sphinx documentation
#

# You can set these variables from the command line, and also
# from the environment for the first two.
SPHINXOPTS    ?=
SPHINXBUILD   ?= sphinx-build
SOURCEDIR     = .
BUILDDIR      = _build

# Put it first so that "make" without argument is like "make help".
help:
    @$(SPHINXBUILD) -M help "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)

.PHONY: help Makefile

# Catch-all target: route all unknown targets to Sphinx using the new
# "make mode" option.  $(O) is meant as a shortcut for $(SPHINXOPTS).
%: Makefile
    @$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
```

###### FILE: Makefile

```
SHELL = /bin/bash
.PHONY: all venv update upgrade-dependencies test coverage cov tox reports clean-reports init-docs docs clean-docs clean distclean

all: info

info:  ## Show this infomation
    @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

REPORT_OUTPUT_DIR = ./target/report
DOCUMENT_OUTPUT_DIR = ./doc/_build

.venv: ## Python Python virtual environment
    uv sync --all-extras

venv: .venv ## Create virtual env and sync dependencies

update: venv ## Alias of venv
    uv sync --all-extras

upgrade-dependencies: ## Upgrade dependencies
    uv sync --all-extras --upgrade

test: venv  ## Run tests
    uv run pytest $(PYTEST_OPTIONS)

coverage: venv  ## Run tests and compute test coverage
    uv run coverage run -m pytest --durations=0 $(PYTEST_OPTIONS) && \
    uv run coverage report

cov: coverage

tox:
    uv tool run --with tox-uv tox

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
        uv run --group docs sphinx-apidoc \
            -F -T -M --separate \
            -H "`uv run --group docs azfr-pyproject-name`" \
            -A "`uv run --group docs azfr-pyproject-author`" \
            -t docs/_templates \
            -o docs \
            src && \
            git add docs; \
    fi

docs: venv init-docs clean-docs  ## Generate API documents
    uv run --group docs sphinx-apidoc -T -M --separate -t docs/_templates -o docs src && \
    uv run --group docs $(MAKE) -C docs html; \

clean-docs:  ## Clean API documents
    rm -rf $(DOCUMENT_OUTPUT_DIR)

deploy: venv  ## Deploy library packages
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

clean: clean-reports clean-docs  ## Clean artifacts and temporary files
    find src tests -name '*.pyc' -exec rm -f {} +
    find src tests -name '*.pyo' -exec rm -f {} +
    find src tests -name '*~' -exec rm -f {} +
    rm -rf .coverage

distclean: clean  ## Clean all files including virtualenv
    rm -rf .pytest_cache .tox .eggs ./src/*.egg-info build dist
    rm -rf .venv
```

###### FILE: pyproject.toml

```toml
[project]
name = "azfr-azure-utils"
description = "Utility libraries for using Azure SDK for Python in AZFR GDP environment"
url = "https://github.developer.allianz.io/azf-h1-datascience/azfr-azure-utils"
authors = [
    { name = "Allianz Technology" },
]

dynamic = ["version"]
readme = "README.md"

requires-python = ">=3.10"

dependencies = [

]

[dependency-groups]
dev = [
    "psycopg[pool]",
    "psycopg2",
    "pg8000",
    "asyncpg",
    "SQLAlchemy",
    "pytest",
    "pytest-html",
    "coverage",
    "ruff",
]

docs = [
    "azfr-release-utils>=3.0.8",
    "sphinx",
    "myst-parser",
]

[project.optional-dependencies]

identity = [
    "azure-identity",
]

secrets = [
    "azure-identity",
    "azure-keyvault-secrets"
]

trino = [
    "azure-identity",
    "trino"
]

databricks = [
    "databricks-sdk",
    "databricks-sql-connector[pyarrow]",
]

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
```

###### FILE: README.md

````md
# AZF Azure Utility Library

Utility library for using Azure SDK for Python in AZFR GDP environment.

## Azure token credential classes

### Prerequisite

Install `azfr-azure-utils` with `identity` extension.

```bash
pip install azfr-azure-utils[identity]
````

### Default token credential

* `azfr_azure_utils.identity.AzfrDefaultCredential`
* `azfr_azure_utils.identity.aio.AzfrDefaultCredential` (async)

The custom `ChainedTokenCredential` optimized for AZFR GDP environment,
trying different authentication methods in the following order.

1. Client Secret (`ClientSecretCredential`)

   * If env variables `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, and
     `AZURE_TENANT_ID` are set.
2. Managed Identity (`ManagedIdentityCredential`)

   * If env variable `AZURE_CLIENT_ID` is set.
   * Try `WorkloadIdentityCredential` if all required env variables
     for AAD workload identity are set
3. Azure CLI (`AzureCliCredential`)

* `azfr_azure_utils.identity.ProcessLocalTokenCredential`
* `azfr_azure_utils.identity.aio.ProcessLocalTokenCredential` (async)

### Wrapper for multiprocessing

A wrapper of `TokenCredential` for using `TokenCredential` in a
multiprocessing settings.

## Accessing secrets in Azure Key Vaults

### Prerequisite

Install `azfr-azure-utils` with `secrets` extension.

```bash
pip install azfr-azure-utils[secrets]
```

### Read secrets in Azure Key Vaults

`azfr_azure_utils.secrets.read_kv_secret`

```python
from azfr_azure_utils.secrets import read_kv_secret

my_secret = read_kv_secret(vault_name="myvault", secret_name="my-secret")
```

## Azure PostgreSQL support

### Prerequisite

Install `azfr-azure-utils` and PostgreSQL drivers to use.

Example:

```bash
pip install azfr-azure-utils psycopg2
```

If necessary, install `SQLAlchemy` as well.

### Authentication with Azure token credentials

You can use the following idiom to fetch access tokens from Azure token
credential `credential` you can use as PostgreSQL passwords.

```python
credential.get_token("https://ossrdbms-aad.database.windows.net/.default").token
```

Example:

```python
import psycopg2

from azfr_azure_utils.identity import AzfrDefaultCredential

credential = AzfrDefaultCredential()

conn = psycopg2.connect(
    dsn="postgresql://my-user-id@my-psql-server.postgres.database.azure.com/my_db",
    password=credential.get_token("https://ossrdbms-aad.database.windows.net/.default").token
)
```

### SQLAlchemy support

You can use the following idiom to dynamically retrieve access tokens
from `credential` when SQLAlchemy `Engine` connects Azure PostgreSQL.

```python
from sqlalchemy import event


@event.listens_for(engine, "do_connect")
def set_access_token_as_password(dialect, conn_rec, cargs, cparams):
    cparams["password"] = credential.get_token("https://ossrdbms-aad.database.windows.net/.default").token
```

Example:

```python
from azfr_azure_utils.identity import AzfrDefaultCredential
from sqlalchemy import create_engine, event

engine = create_engine(
    "postgresql://my-user-id@my-psql-server.postgres.database.azure.com/my_db",
)

credential = AzfrDefaultCredential()


@event.listens_for(engine, "do_connect")
def set_access_token_as_password(dialect, conn_rec, cargs, cparams):
    cparams["password"] = credential.get_token("https://ossrdbms-aad.database.windows.net/.default").token
```

### Psycopg (2 and 3) connection pool

This library provides custom connection pool classes for `psycopg2` and
`psycopg3` to use Azure token credentials.

* `ConnectionPool` in `azfr_azure_utils.postgresql.psycopg.pool`
* `SimpleConnectionPool` and `ThreadedConnectionPool` in
  `azfr_azure_utils.postgresql.psycopg2.pool`

Example:

```python
from azfr_azure_utils.identity import AzfrDefaultCredential
from azfr_azure_utils.postgresql.psycopg.pool import ConnectionPool

kwargs = {
    "host": "my-psql-server.postgres.database.azure.com",
    "port": 5432,
    "dbname": "my_db",
    "user": "my-user-id"
}

with ConnectionPool(kwargs=kwargs, credential=AzfrDefaultCredential()) as pool:
    with pool.connection() as conn:
        ...
```

### Patch DBAPI for PostgreSQL

**We recommend programmatically setting access tokens as shown above**.
However, if you cannot set the access token programmatically, for
example, a framework internally uses SQLAlchemy but does not expose APIs
to customize `Engine` objects, you can directly patch DBAPI for
PostgreSQL (`psycopg` etc.) to use Azure token credentials.

`azfr_azure_utils.postgresql` module provides the following utility
functions for patching DBAPIs for PostgreSQL.

* `patch_psycopg`: `psycogp(3)`
* `patch_psycopg_pool`: `psycogp_pool`
* `patch_psycopg2`: `psycogp2`
* `patch_psycopg2_pool`: `psycogp2.pool`
* `patch_pg8000`: `pg8000`
* `patch_asyncpg`: `asyncpg`
* `patch_dbapi`: Patch all supported DBAPIs

These utility functions take Azure token credentials to use as
arguments.

Example:

```python
from sqlalchemy import create_engine

from azfr_azure_utils.identity import AzfrDefaultCredential
from azfr_azure_utils.postgresql import patch_psycopg

credential = AzfrDefaultCredential()

patch_psycopg(credential=credential)

engine = create_engine(
    "postgresql://my-user-id@my-psql-server.postgres.database.azure.com/my_db"
)
```

## Trino support

### Prerequisite

Install `azfr-azure-utils` with `trino` extension.

```bash
pip install azfr-azure-utils[trino]
```

### Authentication with Azure token credentials

* `azfr_azure_utils.trino.AzureCredentialTokenAuthentication`
* `azfr_azure_utils.trino.AzfrAzureCredentialTokenAuthentication`

```python
from trino.dbapi import connect
from azfr_azure_utils.trino import AzfrAzureCredentialTokenAuthentication

conn = connect(
    host="azfr-trino.datascience-dev.azfr.allianz", port="443",
    auth=AzfrAzureCredentialTokenAuthentication("api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/.default"),
    http_scheme="https", verify="/etc/ssl/certs/ca-certificates.crt")
...
```

## Databricks support

### Prerequisite

Install `azfr-azure-utils` with `databricks` extension.

```bash
pip install azfr-azure-utils[databricks]
```

### Authentication with Azure token credentials

#### Databricks SDK

Use `azfr_azure_utils.databricks.sdk.AzfrAzureCredentialsStrategy` as a
credential strategy.

```python
from databricks.sdk import WorkspaceClient
from azfr_azure_utils.databricks.sdk import AzfrAzureCredentialsStrategy
client = WorkspaceClient(
   credentials_strategy=AzfrAzureCredentialsStrategy()
)
```

#### Databricks SQL Connector

Use `azfr_azure_utils.databricks.sql.AzfrAzureCredentialsProvider` as a
credential provider.

```python
import databricks.sql
from azfr_azure_utils.databricks.sql import AzfrAzureCredentialsProvider

conn = databricks.sql.connect(
   server_hostname=...,
   http_path=...,
   credentials_provider=AzfrAzureCredentialsProvider()
)
```

````
###### FILE: src/azfr_azure_utils/databricks/sdk.py ######

```py
from databricks.sdk.config import Config
from databricks.sdk.credentials_provider import CredentialsStrategy, CredentialsProvider

from azfr_azure_utils.identity import AzfrDefaultCredential


class AzfrAzureCredentialsStrategy(CredentialsStrategy):
    """Credentials strategy for Databricks SDK that creates a credentials provider for authenticating with
    Azure using the AzfrDefaultCredential chain."""

    def auth_type(self) -> str:
        return "azure-azfr"

    def __call__(self, cfg: Config) -> CredentialsProvider:
        credential = AzfrDefaultCredential()

        def authenticate():
            token = credential.get_token(
                "2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default"
            )
            return {"Authorization": f"Bearer {token.token}"}

        return authenticate


__all__ = [
    "AzfrAzureCredentialsStrategy",
]
````

###### FILE: src/azfr_azure_utils/databricks/sql.py

```py
from azfr_azure_utils.identity import AzfrDefaultCredential

from databricks.sql.auth.authenticators import CredentialsProvider, HeaderFactory


class AzfrAzureCredentialsProvider(CredentialsProvider):
    """Credentials provider for Databricks SQL Python Connector that authenticates with
    Azure using the AzfrDefaultCredential chain."""

    def auth_type(self) -> str:
        return "azure-azfr"

    def __call__(self, *args, **kwargs) -> HeaderFactory:
        credential = AzfrDefaultCredential()

        def authenticate():
            token = credential.get_token(
                "2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default"
            )
            return {"Authorization": f"Bearer {token.token}"}

        return authenticate


__all__ = [
    "AzfrAzureCredentialsProvider",
]
```

###### FILE: src/azfr_azure_utils/identity/**init**.py

```py
import os
import socket
from azure.core.credentials import AccessToken
from azure.identity import (
    AzureCliCredential,
    ChainedTokenCredential,
    ManagedIdentityCredential,
    ClientSecretCredential,
    InteractiveBrowserCredential,
)
from azure.identity._constants import EnvironmentVariables
from azure.identity._internal.get_token_mixin import GetTokenMixin
from typing import Any, Callable


class ProcessLocalTokenCredential:
    """
    Wrapper of Azure AsyncTokenCredential for using TokenCredential with multiprocessing.
    This wrapper does not serialize a wrapped token credential object when it is pickled. Instead, it recreates
    a new instance of the token credential object when deserialized in a different process.

    Parameters
    ----------
    factory: Callable[[], AsyncTokenCredential]
        A callable object returning a new AsyncTokenCredential instance
    """

    def __init__(self, factory: Callable):
        self.factory = factory
        self.credential = self.factory()

    def __getstate__(self):
        state = self.__dict__.copy()
        del state["credential"]
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self.credential = self.factory()

    def get_token(self, *scopes: str, **kwargs: "Any"):
        return self.credential.get_token(*scopes, **kwargs)

    def _get_token_info(self, *scopes: str, options=None):
        return self.credential.get_token_info(*scopes, options=options)

    def __getattribute__(self, name):
        if name == "get_token_info" and hasattr(self.credential, "get_token_info"):
            return self._get_token_info
        return super().__getattribute__(name)

    def close(self):
        return self.credential.close()

    def __enter__(self):
        return self.credential.__enter__()

    def __exit__(self, *args):
        return self.credential.__exit__(*args)


class CachedCredential(GetTokenMixin):
    """Wrapper of Azure AsyncTokenCredential caching retrieved tokens"""

    def __init__(self, credential) -> None:
        super().__init__()
        self.credential = credential
        self.__cache = {}

    @staticmethod
    def __key(*args):
        return str(args)

    def _acquire_token_silently(self, *scopes: str, **kwargs):
        return self.__cache.get(self.__key(scopes, kwargs), None)

    def _request_token(self, *scopes: str, **kwargs):
        if hasattr(self.credential, "get_token_info"):
            t = self.credential.get_token_info(*scopes, options=kwargs)
        else:
            t = self.credential.get_token(*scopes, **kwargs)
        self.__cache[self.__key(scopes, kwargs)] = t
        return t

    def close(self):
        return self.credential.close()

    def __enter__(self):
        return self.credential.__enter__()

    def __exit__(self, *args):
        return self.credential.__exit__(*args)


class AzfrDefaultCredential(ChainedTokenCredential):
    """
    Default token credential chain resolving access token in the following order:
        * Client secret (read from env variables `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`)
        * Managed identity (read client ID from env variable `AZURE_CLIENT_ID`)
        * Azure CLI
    """

    def __init__(self, use_env_settings: bool = False, **kwargs) -> None:
        credentials = []

        tenant_id = kwargs.get(
            "tenant_id", os.environ.get(EnvironmentVariables.AZURE_TENANT_ID)
        )
        client_id = kwargs.get(
            "client_id", os.environ.get(EnvironmentVariables.AZURE_CLIENT_ID)
        )
        client_secret = kwargs.get(
            "client_secret", os.environ.get(EnvironmentVariables.AZURE_CLIENT_SECRET)
        )

        if tenant_id and client_id and client_secret:
            credentials.append(
                ClientSecretCredential(
                    tenant_id=tenant_id,
                    client_id=client_id,
                    client_secret=client_secret,
                    use_env_settings=use_env_settings,
                )
            )

        if client_id:
            credentials.append(
                ManagedIdentityCredential(client_id=client_id, use_env_settings=use_env_settings)
            )

        credentials.append(CachedCredential(AzureCliCredential()))

        super().__init__(*credentials)

    def get_token(self, *scopes: str, **kwargs) -> AccessToken:
        if self._successful_credential:
            return self._successful_credential.get_token(*scopes, **kwargs)

        return super().get_token(*scopes, **kwargs)


def create_interactive_browser_credential(use_env_settings: bool = False, **kwargs):
    """Create an InteractiveBrowserCredential with proxy settings.

    Parameters
    ----------
    use_env_settings: bool
        Whether to use environment variable settings for the credential. Defaults to False.
    kwargs
        Additional keyword arguments passed to the `InteractiveBrowserCredential` constructor

    Returns
    -------
    InteractiveBrowserCredential
        An instance of `InteractiveBrowserCredential`
    """
    return InteractiveBrowserCredential(
        use_env_settings=use_env_settings,
        proxies={"https": _available_proxy_for_interactive_auth()},
        **kwargs,
    )


def _available_proxy_for_interactive_auth():
    """Return available proxy server

    Returns
    -------
    str
        Proxy server URL
    """

    def is_port_listening(port):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        try:
            s.bind(("0.0.0.0", port))
        except socket.error:
            return True
        finally:
            s.close()

    if is_port_listening(9000):
        return "http://localhost:9000"
    elif is_port_listening(3128):
        return "http://localhost:3128"
    else:
        return "http://fr000-svr.zone2.proxy.allianz:8090"
```

###### FILE: src/azfr_azure_utils/identity/aio.py

```py
import os
from azure.core.credentials import AccessToken
from azure.identity._constants import EnvironmentVariables
from azure.identity.aio import (
    AzureCliCredential,
    ChainedTokenCredential,
    ManagedIdentityCredential,
    ClientSecretCredential,
)
from azure.identity.aio._internal.get_token_mixin import GetTokenMixin
from typing import Any, Callable


class ProcessLocalTokenCredential:
    """
    Wrapper of Azure AsyncTokenCredential for using TokenCredential with multiprocessing.
    This wrapper does not serialize a wrapped token credential object when it is pickled. Instead, it recreates
    a new instance of the token credential object when deserialized in a different process.

    Parameters
    ----------
    factory: Callable[[], AsyncTokenCredential]
        A callable object returning a new AsyncTokenCredential instance
    """

    def __init__(self, factory: Callable):
        self.factory = factory
        self.credential = self.factory()

    def __getstate__(self):
        state = self.__dict__.copy()
        del state["credential"]
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self.credential = self.factory()

    async def get_token(self, *scopes: str, **kwargs: "Any"):
        return await self.credential.get_token(*scopes, **kwargs)

    async def _get_token_info(self, *scopes: str, options=None):
        return await self.credential.get_token_info(*scopes, options=options)

    def __getattribute__(self, name):
        if name == "get_token_info" and hasattr(self.credential, "get_token_info"):
            return self._get_token_info
        return super().__getattribute__(name)

    async def close(self):
        return await self.credential.close()

    async def __aenter__(self):
        return await self.credential.__aenter__()

    async def __aexit__(self, *args):
        return await self.credential.__aexit__(*args)


class CachedCredential(GetTokenMixin):
    """Wrapper of Azure AsyncTokenCredential caching retrieved tokens"""

    def __init__(self, credential) -> None:
        self.credential = credential
        self.__cache = {}

    @staticmethod
    def __key(*args):
        return str(args)

    async def _acquire_token_silently(self, *scopes: str, **kwargs):
        return self.__cache.get(self.__key(scopes, kwargs), None)

    async def _request_token(self, *scopes: str, **kwargs):
        if hasattr(self.credential, "get_token_info"):
            t = await self.credential.get_token_info(*scopes, options=kwargs)
        else:
            t = await self.credential.get_token(*scopes, **kwargs)
        self.__cache[self.__key(scopes, kwargs)] = t
        return t

    async def close(self):
        return await self.credential.close()

    async def __aenter__(self):
        return await self.credential.__aenter__()

    async def __aexit__(self, *args):
        return await self.credential.__aexit__(*args)


class AzfrDefaultCredential(ChainedTokenCredential):
    """
    Default token credential chain resolving access token in the following order:
        * Client secret (read from env variables `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`)
        * Managed identity (read client ID from env variable `AZURE_CLIENT_ID`)
        * Azure CLI
    """

    def __init__(self, use_env_settings: bool = False, **kwargs) -> None:
        credentials = []

        tenant_id = kwargs.get(
            "tenant_id", os.environ.get(EnvironmentVariables.AZURE_TENANT_ID)
        )
        client_id = kwargs.get(
            "client_id", os.environ.get(EnvironmentVariables.AZURE_CLIENT_ID)
        )
        client_secret = kwargs.get(
            "client_secret", os.environ.get(EnvironmentVariables.AZURE_CLIENT_SECRET)
        )

        if tenant_id and client_id and client_secret:
            credentials.append(
                ClientSecretCredential(
                    tenant_id=tenant_id,
                    client_id=client_id,
                    client_secret=client_secret,
                    use_env_settings=use_env_settings,
                )
            )

        if client_id:
            credentials.append(
                ManagedIdentityCredential(client_id=client_id, use_env_settings=use_env_settings)
            )

        credentials.append(CachedCredential(AzureCliCredential()))

        super().__init__(*credentials)

    async def get_token(self, *scopes: str, **kwargs) -> AccessToken:
        if self._successful_credential:
            return await self._successful_credential.get_token(*scopes, **kwargs)

        return await super().get_token(*scopes, **kwargs)
```

###### FILE: src/azfr_azure_utils/postgresql/**init**.py

```py
from functools import wraps
from typing import Type, cast, Optional, Dict, Any


def patch_psycopg(credential=None, async_credential=None):
    """Patch `psycopg` DBAPI to support Entra ID-based authentication.

    If Azure token credential objects are provided as `credential` and `async_credential`, the connection will
    automatically use the provided credentials.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    async_credential
        Async Azure token credential to use by default.
    """
    import psycopg as psycopg_orig
    from .psycopg import connect, connect_orig, AsyncConnection, AsyncConnection_orig

    if credential:

        @wraps(connect_orig)
        def connect_with_credential(*args, **kwargs):
            kwargs["credential"] = credential
            return connect(*args, **kwargs)

        psycopg_orig.connect = connect_with_credential
    else:
        psycopg_orig.connect = connect

    if async_credential:

        class AsyncConnectionWithCredential(AsyncConnection):
            @classmethod
            @wraps(AsyncConnection_orig.connect)
            async def connect(cls, *args, **kwargs):
                kwargs["credential"] = async_credential
                return await AsyncConnection.connect(*args, **kwargs)

        psycopg_orig.AsyncConnection = AsyncConnectionWithCredential
    else:
        psycopg_orig.AsyncConnection = AsyncConnection


def patch_psycopg_pool(credential=None):
    """Patch `psycopg_pool` API to support Entra ID-based authentication.

    If Azure token credential object is provided as `credential`, the connection will automatically use the provided
    credential.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    """
    import psycopg_pool as psycopg_pool_orig
    from psycopg_pool.abc import CT, ConnectionCB, ConnectFailedCB
    from .psycopg import Connection
    from .psycopg.pool import ConnectionPool, ConnectionPool_org

    if credential:

        class ConnectionPoolWithCredential(ConnectionPool):
            @wraps(ConnectionPool_org.__init__)
            def __init__(
                self,
                conninfo: str = "",
                *,
                connection_class: Type[CT] = cast(Type[CT], Connection),
                kwargs: Optional[Dict[str, Any]] = None,
                min_size: int = 4,
                max_size: Optional[int] = None,
                open: Optional[bool] = None,
                configure: Optional[ConnectionCB[CT]] = None,
                check: Optional[ConnectionCB[CT]] = None,
                reset: Optional[ConnectionCB[CT]] = None,
                name: Optional[str] = None,
                timeout: float = 30.0,
                max_waiting: int = 0,
                max_lifetime: float = 60 * 60.0,
                max_idle: float = 10 * 60.0,
                reconnect_timeout: float = 5 * 60.0,
                reconnect_failed: Optional[ConnectFailedCB] = None,
                num_workers: int = 3,
            ):
                super().__init__(
                    conninfo,
                    connection_class=connection_class,
                    kwargs=kwargs,
                    min_size=min_size,
                    max_size=max_size,
                    open=open,
                    configure=configure,
                    check=check,
                    reset=reset,
                    name=name,
                    timeout=timeout,
                    max_waiting=max_waiting,
                    max_lifetime=max_lifetime,
                    max_idle=max_idle,
                    reconnect_timeout=reconnect_timeout,
                    reconnect_failed=reconnect_failed,
                    num_workers=num_workers,
                    credential=credential,
                )

        psycopg_pool_orig.ConnectionPool = ConnectionPoolWithCredential
    else:
        psycopg_pool_orig.ConnectionPool = ConnectionPool


def patch_psycopg2(credential=None):
    """Patch `psycopg2` DBAPI to support Entra ID-based authentication.

    If Azure token credential object is provided as `credential`, the connection will automatically use the provided
    credential.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    """
    import psycopg2 as psycopg2_orig
    from .psycopg2 import connect, connect_orig

    if credential:

        @wraps(connect_orig)
        def connect_with_credential(*args, **kwargs):
            kwargs["credential"] = credential
            return connect(*args, **kwargs)

        psycopg2_orig.connect = connect_with_credential
    else:
        psycopg2_orig.connect = connect


def patch_psycopg2_pool(credential=None):
    """Patch `psycopg2.pool` API to support Entra ID-based authentication.

    If Azure token credential object is provided as `credential`, the connection will automatically use the provided
    credential.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    """
    import psycopg2.pool as psycopg2_pool_orig
    from .psycopg2.pool import SimpleConnectionPool, ThreadedConnectionPool

    if credential:

        class SimpleConnectionPoolWithCredential(SimpleConnectionPool):
            def __init__(self, minconn, maxconn, *args, **kwargs):
                super().__init__(
                    minconn, maxconn, *args, credential=credential, **kwargs
                )

        class ThreadedConnectionPoolWithCredential(ThreadedConnectionPool):
            def __init__(self, minconn, maxconn, *args, **kwargs):
                super().__init__(
                    minconn, maxconn, *args, credential=credential, **kwargs
                )

        psycopg2_pool_orig.SimpleConnectionPool = SimpleConnectionPoolWithCredential
        psycopg2_pool_orig.ThreadedConnectionPool = ThreadedConnectionPoolWithCredential
    else:
        psycopg2_pool_orig.SimpleConnectionPool = SimpleConnectionPool
        psycopg2_pool_orig.ThreadedConnectionPool = ThreadedConnectionPool


def patch_pg8000(credential=None):
    """Patch `pg8000` DBAPI to support Entra ID-based authentication.

    If Azure token credential object is provided as `credential`, the connection will automatically use the provided
    credential.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    """
    import pg8000 as pg8000_orig
    from .pg8000 import connect, connect_orig

    if credential:

        @wraps(connect_orig)
        def connect_with_credential(*args, **kwargs):
            kwargs["credential"] = credential
            return connect(*args, **kwargs)

        pg8000_orig.connect = connect_with_credential
    else:
        pg8000_orig.connect = connect


def patch_asyncpg(credential=None):
    """Patch `asyncpg` DBAPI to support Entra ID-based authentication.

    If Azure token credential object is provided as `credential`, the connection will automatically use the provided
    credential.

    Parameters
    ----------
    credential
        Async Azure token credential to use by default.
    """
    import asyncpg as asyncpg_orig
    from .asyncpg import connect, connect_orig

    if credential:

        @wraps(connect_orig)
        async def connect_with_credential(*args, **kwargs):
            kwargs["credential"] = credential
            return await connect(*args, **kwargs)

        asyncpg_orig.connect = connect_with_credential
    else:
        asyncpg_orig.connect = connect


def patch_dbapi(credential=None, async_credential=None):
    """Patch DBAPI (`psycopg`, `psycopg2`, `pg8000` and `asyncpg`) to support Entra ID-based authentication.

    If Azure token credential objects are provided as `credential` and `async_credential`, the connection will
    automatically use the provided access tokens.

    This function ignores `ImportError` if the corresponding DBAPI is not installed.

    Parameters
    ----------
    credential
        Azure token credential to use by default.
    async_credential
        Async Azure token credential to use by default.
    """
    try:
        patch_psycopg(credential, async_credential)
    except ImportError:
        pass

    try:
        patch_psycopg_pool(credential)
    except ImportError:
        pass

    try:
        patch_psycopg2(credential)
    except ImportError:
        pass

    try:
        patch_psycopg2_pool(credential)
    except ImportError:
        pass

    try:
        patch_pg8000(credential)
    except ImportError:
        pass

    try:
        patch_asyncpg(async_credential)
    except ImportError:
        pass
```

###### FILE: src/azfr_azure_utils/postgresql/asyncpg/**init**.py

```py
"""Wrapper of `asyncpg` DBAPI supporting Entra ID-based authentication"""

import asyncpg as __asyncpg
from asyncpg import *  # noqa

connect_orig = __asyncpg.connect


async def connect(
    dsn=None,
    *,
    host=None,
    port=None,
    user=None,
    password=None,
    passfile=None,
    database=None,
    loop=None,
    timeout=60,
    statement_cache_size=100,
    max_cached_statement_lifetime=300,
    max_cacheable_statement_size=1024 * 15,
    command_timeout=None,
    ssl=None,
    direct_tls=False,
    connection_class=Connection,  # noqa
    record_class=Record,  # noqa
    server_settings=None,
    target_session_attrs=None,
    credential=None,
):
    """Create a new database connection optionally with Entra ID-based authentication.

    If an Azure token credential object is provided as `credential`, the connection will be authenticated using
    an access token provided by the credential object.

    For more information about other parameter, see the official `asyncpg` documentation.

    https://magicstack.github.io/asyncpg/current/api/index.html

    Parameters
    ----------
    dsn
        Connection arguments specified using as a single string in the libpq connection URI format: `postgres://user:password@host:port/database?option=value`.
    host
        Database host address.
    port
        Port number to connect to at the server host.
    user
        The name of the database role used for authentication.
    password
        Password to be used for authentication, if the server requires one.
    passfile
        The name of the file used to store passwords.
    database
        The name of the database to connect to.
    loop
        An asyncio event loop instance.
    timeout
        Connection timeout in seconds.
    statement_cache_size
        The size of prepared statement LRU cache.
    max_cached_statement_lifetime
        The maximum time in seconds a prepared statement will stay in the cache.
    max_cacheable_statement_size
        The maximum size of a statement that can be cached (15KiB by default).
    command_timeout
        The default timeout for operations on this connection (the default is `None`: no timeout).
    ssl
        See the official documentation for more details.
    direct_tls
        Pass True to skip PostgreSQL STARTTLS mode and perform a direct SSL connection.
    connection_class
        Class of the returned connection object.
    record_class
        If specified, the class to use for records returned by queries on this connection object.
    server_settings
        An optional dict of server runtime parameters.
    target_session_attrs
        If specified, check that the host has the correct attribute. See the official documentation for more details.
    credential
        Async Azure token credential. If set, use an access token provided by this object as a password.
    """
    if credential is not None and not password:
        token = (
            await credential.get_token(
                "https://ossrdbms-aad.database.windows.net/.default"
            )
        ).token
        password = token

    return await connect_orig(
        dsn=dsn,
        host=host,
        port=port,
        user=user,
        password=password,
        passfile=passfile,
        database=database,
        loop=loop,
        timeout=timeout,
        statement_cache_size=statement_cache_size,
        max_cached_statement_lifetime=max_cached_statement_lifetime,
        max_cacheable_statement_size=max_cacheable_statement_size,
        command_timeout=command_timeout,
        ssl=ssl,
        direct_tls=direct_tls,
        connection_class=connection_class,
        record_class=record_class,
        server_settings=server_settings,
        target_session_attrs=target_session_attrs,
    )


__version__ = __asyncpg.__version__

__all__ = __asyncpg.__all__
```

###### FILE: src/azfr_azure_utils/postgresql/asyncpg/sqlalchemy_async.py

```py
"""Wrapper of SqlAlchemy's `AsyncAdapt_asyncpg_dbapi` to support Entra ID-based authentication"""


def _wrap_dbapi():
    from sqlalchemy.dialects.postgresql.asyncpg import AsyncAdapt_asyncpg_dbapi

    import azfr_azure_utils.postgresql.asyncpg

    return AsyncAdapt_asyncpg_dbapi(azfr_azure_utils.postgresql.asyncpg)


__dbapi = _wrap_dbapi()


def __getattr__(name):
    return __dbapi.__getattribute__(name)
```

###### FILE: src/azfr_azure_utils/postgresql/pg8000.py

```py
"""Wrapper of `pg8000` DBAPI supporting Entra ID-based authentication"""

import pg8000 as __pg8000
from pg8000 import *  # noqa
from psycopg import apilevel, threadsafety, paramstyle  # noqa

connect_orig = __pg8000.connect


def connect(
    user,
    host="localhost",
    database=None,
    port=5432,
    password=None,
    source_address=None,
    unix_sock=None,
    ssl_context=None,
    timeout=None,
    tcp_keepalive=True,
    application_name=None,
    replication=None,
    credential=None,
):
    """Create a new database connection optionally with Entra ID-based authentication.

    If an Azure token credential object is provided as `credential`, the connection will be authenticated using
    an access token provided by the credential object.

    For more information about other parameter, see the official `pg8000` documentation.

    Azure token credential. If set, use an access token provided by this object as a password.

    Parameters
    ----------
    user
        The username to connect to the PostgreSQL server with.
    host
        The hostname of the PostgreSQL server to connect with.
    database
        The name of the database instance to connect with.
    port
        The TCP/IP port of the PostgreSQL server instance.
    password
        The user password to connect to the server with.
    source_address
        The source IP address which initiates the connection to the PostgreSQL server.
    unix_sock
        The path to the UNIX socket to access the database through, for example, '/tmp/.s.PGSQL.5432'.
    ssl_context
        This governs SSL encryption for TCP/IP sockets. See the official documentation for more details.
    timeout
        This is the time in seconds before the connection to the server will time out.
    tcp_keepalive
        If `True` then use TCP keepalive. The default is `True`.
    application_name
        Sets the `application_name`.
    replication
        Used to run in streaming replication mode. See the official documentation for more details.
    credential
        Azure token credential. If set, use an access token provided by this object as a password.
    """
    if credential is not None and not password:
        token = credential.get_token(
            "https://ossrdbms-aad.database.windows.net/.default"
        ).token
        password = token

    return connect_orig(
        user=user,
        host=host,
        database=database,
        port=port,
        password=password,
        source_address=source_address,
        unix_sock=unix_sock,
        ssl_context=ssl_context,
        timeout=timeout,
        tcp_keepalive=tcp_keepalive,
        application_name=application_name,
        replication=replication,
    )


__version__ = __pg8000.__version__

__all__ = __pg8000.__all__
```

###### FILE: src/azfr_azure_utils/postgresql/psycopg/**init**.py

```py
"""Wrapper of `psycopg` (`psycopg3`) DBAPI supporting Entra ID-based authentication"""

from typing import Any
from typing import Optional, Type

import psycopg as __psycopg
from azure.core.credentials import TokenCredential
from psycopg import *  # noqa
from psycopg import adapters  # noqa
from psycopg.abc import AdaptContext
from psycopg.rows import RowFactory, Row, AsyncRowFactory
from typing_extensions import Self

Connection_orig = __psycopg.Connection


class Connection(Connection_orig):
    @classmethod
    def connect(
        cls,
        conninfo: str = "",
        *,
        autocommit: bool = False,
        prepare_threshold: Optional[int] = 5,
        row_factory: Optional[RowFactory[Row]] = None,  # noqa
        cursor_factory: Optional[Type[Cursor[Row]]] = None,  # noqa
        context: Optional[AdaptContext] = None,
        credential=None,
        **kwargs: Any,
    ) -> Self:
        """
        Connect to a database server and return a new `Connection` instance.
        """
        if credential is not None and not kwargs.get("password"):
            token = credential.get_token(
                "https://ossrdbms-aad.database.windows.net/.default"
            ).token
            kwargs["password"] = token

        return super().connect(
            conninfo,
            autocommit=autocommit,
            prepare_threshold=prepare_threshold,
            row_factory=row_factory,
            cursor_factory=cursor_factory,
            context=context,
            **kwargs,
        )


AsyncConnection_orig = __psycopg.AsyncConnection


class AsyncConnection(AsyncConnection_orig):
    @classmethod
    async def connect(
        cls,
        conninfo: str = "",
        *,
        autocommit: bool = False,
        prepare_threshold: Optional[int] = 5,
        context: Optional[AdaptContext] = None,
        row_factory: Optional[AsyncRowFactory[Row]] = None,  # noqa
        cursor_factory: Optional[Type[AsyncCursor[Row]]] = None,  # noqa
        credential=None,
        **kwargs: Any,
    ) -> Self:
        if credential is not None and not kwargs.get("password"):
            token = (
                await credential.get_token(
                    "https://ossrdbms-aad.database.windows.net/.default"
                )
            ).token
            kwargs["password"] = token

        return await super().connect(
            conninfo,
            autocommit=autocommit,
            prepare_threshold=prepare_threshold,
            context=context,
            row_factory=row_factory,
            cursor_factory=cursor_factory,
            **kwargs,
        )


connect_orig = __psycopg.connect


def connect(
    conninfo: str = "",
    *,
    autocommit: bool = False,
    prepare_threshold: Optional[int] = 5,
    row_factory: Optional[RowFactory[Row]] = None,  # noqa
    cursor_factory: Optional[Type[Cursor[Row]]] = None,  # noqa
    context: Optional[AdaptContext] = None,
    credential: Optional[TokenCredential] = None,
    **kwargs: Any,
):
    """Create a new database connection optionally with Entra ID-based authentication.

    If an Azure token credential object is provided as `credential`, the connection will be authenticated using
    an access token provided by the credential object.

    For more information about other parameter, see the official `psycopg` (`psycopg3`) documentation.

    https://www.psycopg.org/psycopg3/docs/api/connections.html#psycopg.Connection.connect

    Parameters
    ----------
    conninfo
        The connection string (a `postgresql:// url` or a list of `key=value` pairs) to specify where and how to connect.
    autocommit
        If `True` don’t start transactions automatically.
    prepare_threshold
        Initial value for the `prepare_threshold` attribute of the connection (new in Psycopg 3.1).
    row_factory
        The row factory specifying what type of records to create fetching data (default: `tuple_row()`).
    cursor_factory
        Initial value for the `cursor_factory` attribute of the connection (new in Psycopg 3.1).
    context
        A context to copy the initial adapters configuration from.
    credential
        Azure token credential. If set, use an access token provided by this object as a password.
    kwargs
        Further parameters specifying the connection string. They override the ones specified in `conninfo`.
    """
    return Connection.connect(
        conninfo=conninfo,
        autocommit=autocommit,
        prepare_threshold=prepare_threshold,
        row_factory=row_factory,
        cursor_factory=cursor_factory,
        context=context,
        credential=credential,
        **kwargs,
    )


__version__ = __psycopg.__version__

__all__ = __psycopg.__all__
```

###### FILE: src/azfr_azure_utils/postgresql/psycopg/pool.py

```py
from typing import Type, cast, Optional, Dict, Any

import psycopg_pool as __psycopg_pool
from psycopg_pool.abc import CT, ConnectionCB, ConnectFailedCB

from . import Connection

ConnectionPool_org = __psycopg_pool.ConnectionPool


class ConnectionPool(ConnectionPool_org):
    def __init__(
        self,
        conninfo: str = "",
        *,
        connection_class: Type[CT] = cast(Type[CT], Connection),
        kwargs: Optional[Dict[str, Any]] = None,
        min_size: int = 4,
        max_size: Optional[int] = None,
        open: Optional[bool] = None,
        configure: Optional[ConnectionCB[CT]] = None,
        check: Optional[ConnectionCB[CT]] = None,
        reset: Optional[ConnectionCB[CT]] = None,
        name: Optional[str] = None,
        timeout: float = 30.0,
        max_waiting: int = 0,
        max_lifetime: float = 60 * 60.0,
        max_idle: float = 10 * 60.0,
        reconnect_timeout: float = 5 * 60.0,
        reconnect_failed: Optional[ConnectFailedCB] = None,
        num_workers: int = 3,
        credential=None,
    ):
        if credential and not kwargs.get("password"):
            kwargs["credential"] = credential

        super().__init__(
            conninfo,
            connection_class=connection_class,
            kwargs=kwargs,
            min_size=min_size,
            max_size=max_size,
            open=open,
            configure=configure,
            check=check,
            reset=reset,
            name=name,
            timeout=timeout,
            max_waiting=max_waiting,
            max_lifetime=max_lifetime,
            max_idle=max_idle,
            reconnect_timeout=reconnect_timeout,
            reconnect_failed=reconnect_failed,
            num_workers=num_workers,
        )
```

###### FILE: src/azfr_azure_utils/postgresql/psycopg/sqlalchemy_async.py

```py
"""Wrapper of SqlAlchemy's `PsycopgAdaptDBAPI` to support Entra ID-based authentication"""


def _wrap_dbapi():
    from psycopg.pq import ExecStatus
    from sqlalchemy.dialects.postgresql.psycopg import (
        AsyncAdapt_psycopg_cursor,
        PsycopgAdaptDBAPI,
    )

    AsyncAdapt_psycopg_cursor._psycopg_ExecStatus = ExecStatus

    import azfr_azure_utils.postgresql.psycopg

    return PsycopgAdaptDBAPI(azfr_azure_utils.postgresql.psycopg)


__dbapi = _wrap_dbapi()


def __getattr__(name):
    return __dbapi.__getattribute__(name)
```

###### FILE: src/azfr_azure_utils/postgresql/psycopg2/**init**.py

```py
"""Wrapper of `psycopg2` DBAPI supporting Entra ID-based authentication"""

import psycopg2 as __psycopg2
from psycopg2 import *  # noqa

connect_orig = __psycopg2.connect


def connect(
    dsn=None, connection_factory=None, cursor_factory=None, credential=None, **kwargs
):
    """Create a new database connection optionally with Entra ID-based authentication.

    If an Azure token credential object is provided as `credential`, the connection will be authenticated using
    an access token provided by the credential object.

    For more information about other parameter, see the official `psycopg2` documentation.

    https://www.psycopg.org/docs/module.html

    Parameters
    ----------
    dsn
        Connection string.
    connection_factory
        Connection factory.
    cursor_factory
        Cursor factory.
    credential
        Azure token credential. If set, use an access token provided by this object as a password.
    kwargs
        Further parameters specifying the connection string. They override the ones specified in dsn.
    """
    if credential is not None and not kwargs.get("password"):
        token = credential.get_token(
            "https://ossrdbms-aad.database.windows.net/.default"
        ).token
        kwargs["password"] = token

    return connect_orig(
        dsn=dsn,
        connection_factory=connection_factory,
        cursor_factory=cursor_factory,
        **kwargs,
    )


__version__ = __psycopg2.__version__
```

###### FILE: src/azfr_azure_utils/postgresql/psycopg2/pool.py

```py
import psycopg2.pool as __psycopg2_pool

from . import connect as connect

SimpleConnectionPool_org = __psycopg2_pool.SimpleConnectionPool
ThreadedConnectionPool_org = __psycopg2_pool.ThreadedConnectionPool


class SimpleConnectionPool(SimpleConnectionPool_org):
    def _connect(self, key=None):
        conn = connect(*self._args, **self._kwargs)
        if key is not None:
            self._used[key] = conn
            self._rused[id(conn)] = key
        else:
            self._pool.append(conn)
        return conn


class ThreadedConnectionPool(ThreadedConnectionPool_org):
    def _connect(self, key=None):
        conn = connect(*self._args, **self._kwargs)
        if key is not None:
            self._used[key] = conn
            self._rused[id(conn)] = key
        else:
            self._pool.append(conn)
        return conn
```

###### FILE: src/azfr_azure_utils/secrets/**init**.py

```py
from azure.keyvault.secrets import SecretClient

from ..identity import AzfrDefaultCredential


def read_kv_secret(vault_name: str, secret_name: str) -> str:
    """Read a secret from Azure Key Vault.

    Parameters
    ----------
    vault_name
        Name of the Azure Key Vault.
    secret_name
        Name of the secret to read.

    Returns
    -------
    str
        Value of the secret in the KEy Vault.
    """
    with AzfrDefaultCredential() as credential:
        with SecretClient(
            vault_url=f"https://{vault_name}.vault.azure.net/", credential=credential
        ) as client:
            return client.get_secret(secret_name).value
```

###### FILE: src/azfr_azure_utils/trino.py

```py
from typing import Any, Tuple, Callable

from azure.core.credentials import TokenCredential
from requests import Session, PreparedRequest
from requests.auth import AuthBase
from trino.auth import Authentication

from .identity import AzfrDefaultCredential


class _JwtBearerAuth(AuthBase):
    """
    Custom implementation of Authentication class for bearer token
    """

    def __init__(self, token_provider: Callable[[], str]):
        self.token_provider = token_provider

    def __call__(self, r: PreparedRequest) -> PreparedRequest:
        r.headers["Authorization"] = "Bearer " + self.token_provider()
        return r


class AzureCredentialTokenAuthentication(Authentication):
    """Authentication with Azure token credential"""

    def __init__(self, credential: TokenCredential, scope: str):
        """Create new `AzureCredentialTokenAuthentication`.

        Parameters
        ----------
        credential
            Azure token credential to use.
        scope
            Application ID URI of the Trino to connect.
        """
        self.credential = credential
        self.scope = scope

    def set_http_session(self, http_session: Session) -> Session:
        http_session.auth = _JwtBearerAuth(
            lambda: self.credential.get_token(self.scope).token
        )
        return http_session

    def get_exceptions(self) -> Tuple[Any, ...]:
        return ()

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, AzureCredentialTokenAuthentication):
            return False
        return self.scope == other.scope


class AzfrAzureCredentialTokenAuthentication(AzureCredentialTokenAuthentication):
    """Authentication with `AzfrDefaultCredential`"""

    def __init__(self, scope: str):
        """Create new `AzfrAzureCredentialTokenAuthentication`.

        Parameters
        ----------
        scope
            Application ID URI of the Trino to connect
        """
        super().__init__(AzfrDefaultCredential(), scope)

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, AzfrAzureCredentialTokenAuthentication):
            return False
        return self.scope == other.scope
```

###### FILE: tests/dummy_test.py

```py
def test():
    assert 15 == 1 + 2 + 3 + 4 + 5
```

###### FILE: tox.ini

```ini
[tox]
envlist =
    py310
    py311
    py312
    py313

[testenv]
runner = uv-venv-lock-runner
passenv =
    JAVA_HOME
    USER
    USERNAME
commands =
    pytest
```
