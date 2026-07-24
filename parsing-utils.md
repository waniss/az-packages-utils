# azfr-parsing-utils — Full Codebase Reference

> Auto-generated single-file snapshot of every source, config, schema, and test-data file in the repo.

---

## Table of Contents

1. [Root config](#root-config)
2. [CI / GitHub Actions](#ci--github-actions)
3. [JSON Schemas](#json-schemas)
4. [Source — core](#source--core)
5. [Source — csv](#source--csv)
6. [Source — fixed_width](#source--fixed_width)
7. [Source — json](#source--json)
8. [Source — model_generator](#source--model_generator)
9. [Source — yaml_generator](#source--yaml_generator)
10. [Tests — helpers & data](#tests--helpers--data)

---

## Root config

### ######.gitignore######

```
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
```

---

### ######.python-version######

```
3.12
```

---

### ######.pre-commit-config.yaml######

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

---

### ######pyproject.toml######

```toml
[project]
name = "azfr-parsing-utils"
description = "PyArrow - delta-rs based framework for data parsing"
url = "https://github.developer.allianz.io/azf-h1-datascience/azfr-parsing-utils"
authors = [
    { name = "Allianz Technology" },
]

dynamic = ["version"]
readme = "README.md"

requires-python = ">=3.8"

dependencies = [
    "azfr-fsspec-utils",
    "pydantic>=2",
    "PyYAML",
    "click",
    "jsonschema",
    "deltalake>=0.16,<1.0",
]

[project.optional-dependencies]
duckdb = [
    "pyarrow",
    "duckdb",
]

deltalake = [
    "pyarrow",
    "deltalake>=0.16,<1.0",
]

model_generator = [
    "datamodel_code_generator",
]

polars = [
    "pyarrow",
    "polars<1.30.0",
]

pandas = [
    "pyarrow",
    "pandas",
]

spark = [
    "pyspark>=3.4,<4",
]

[dependency-groups]
dev = [
    "azfr-fsspec-abfs",
    "pytest",
    "pytest-html",
    "coverage",
    "ruff",
    "pyspark==3.5.6+azfr.azure; sys_platform == 'win32'",
]

docs = [
    "azfr-release-utils>=3.0.8",
    "sphinx",
    "myst-parser",
]

[project.scripts]
generate_model = "azfr_parsing_utils.model_generator.cli.generate_model:generate_model"

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
    ".bzr", ".direnv", ".eggs", ".git", ".git-rewrite", ".hg",
    ".ipynb_checkpoints", ".mypy_cache", ".nox", ".pants.d",
    ".pyenv", ".pytest_cache", ".pytype", ".ruff_cache", ".svn",
    ".tox", ".venv", ".vscode", "__pypackages__", "_build",
    "buck-out", "build", "dist", "node_modules", "site-packages", "venv",
]

line-length = 88
indent-width = 4
```

---

### ######Makefile######

```makefile
SHELL = /bin/bash
.PHONY: all venv update upgrade-dependencies test coverage cov tox reports clean-reports init-docs docs clean-docs clean distclean

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

test_model_generator: venv  ## Generate Model
	uv run generate_model -f test/test_data -o target/test_model_generator_test_model_generator

update-json-schema: venv ## Update JSON schema files in schema/ dir
	uv run python -c "from azfr_parsing_utils.csv import CsvFileFormat; print(CsvFileFormat.schema_json(indent=2))" > schema/csv-file-format.json

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
	uv run --all-groups $(MAKE) -C docs html;

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

clean: clean-reports clean-docs  ## Clean artifacts and temporary files
	find src tests -name '*.pyc' -exec rm -f {} +
	find src tests -name '*.pyo' -exec rm -f {} +
	find src tests -name '*~' -exec rm -f {} +
	rm -rf .coverage

distclean: clean  ## Clean all files including virtualenv
	rm -rf .pytest_cache .tox .eggs ./src/*.egg-info build dist
	rm -rf .venv
```

---

## CI / GitHub Actions

### ###### .github/workflows/build.yaml ######

```yaml
name: Build Python library

env:
  RUN_TESTS: "true"
  PUBLISH_ARTIFACTS: "true"
  USE_CACHE_DATA: "true"
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
      image: prodazfrz6sh.azurecr.io/cicd-job:py312-java17
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
          sonar: true
          disable_cache_read: ${{ inputs.use_cache_data == 'false' || (inputs.use_cache_data == '' && env.USE_CACHE_DATA == 'false') }}

      - name: Test
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        run: |
          make update reports

      - name: Run SonarScanner
        uses: azf-h1-datascience/sonar-scanner/run@release
        with:
          host_url: ${{ secrets.SONAR_HOST_URL }}
          token: ${{ secrets.SONAR_TOKEN }}
          sources: src
          params: |
            sonar.python.version: 3
            sonar.python.coverage.reportPaths: target/report/coverage.xml

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

---

## JSON Schemas

### ###### schema/csv-file-format.json ######

```json
{
  "title": "CsvFileFormat",
  "description": "CSV file format ",
  "type": "object",
  "properties": {
    "kind": { "title": "Kind", "default": "CsvFormat", "enum": ["CsvFormat"], "type": "string" },
    "name": { "title": "name", "description": "Name of the format", "type": "string" },
    "target_name": { "title": "target_name", "description": "Optional override for the name used as the output table name during parsing.", "type": "string" },
    "title": { "title": "title", "description": "Short title of the format", "type": "string" },
    "description": { "title": "description", "description": "Long description (can be multiple lines) of the format", "type": "string" },
    "columns": { "title": "columns", "description": "Columns in CSV files", "type": "array", "items": { "$ref": "#/definitions/CsvColumn" } },
    "sep": { "title": "separator", "description": "Separator for each field and value", "default": ",", "type": "string" },
    "encoding": { "title": "character encoding", "description": "Character encoding used in CSV files", "default": "UTF-8", "type": "string" },
    "encoding_errors": { "title": "handling of character encoding errors", "description": "Strategy for handling character encoding errors in source.", "default": "strict", "type": "string" },
    "quote": { "title": "quote", "description": "Character for escaping quoted values", "default": "\"", "type": "string" },
    "double_quote": { "title": "double quote", "default": false, "type": "boolean" },
    "escape": { "title": "escape", "type": "string" },
    "comment": { "title": "comment", "type": "string" },
    "header": { "title": "header", "type": "boolean" },
    "check_header": { "title": "header", "type": "boolean" },
    "ignore_leading_white_space": { "title": "header", "type": "boolean" },
    "ignore_trailing_white_space": { "title": "header", "type": "boolean" },
    "strings_can_be_null": { "title": "string values can be null", "default": true, "type": "boolean" },
    "quoted_strings_can_be_null": { "title": "quoted string values can be null", "default": false, "type": "boolean" },
    "null_values": { "type": "array", "items": { "type": "string" } },
    "true_values": { "type": "array", "items": { "type": "string" } },
    "false_values": { "type": "array", "items": { "type": "string" } },
    "decimal_point": { "type": "string" },
    "nan_value": { "type": "string" },
    "positive_inf": { "type": "string" },
    "negative_inf": { "type": "string" },
    "date_format": { "type": "string" },
    "timezone": { "type": "string" },
    "timestamp_format": { "type": "string" },
    "timestamp_ntz_format": { "type": "string" },
    "multi_line": { "type": "boolean" },
    "char_to_escape_quote_escaping": { "type": "string" },
    "empty_value": { "type": "string" },
    "line_sep": { "type": "string" }
  },
  "required": ["name", "columns"],
  "definitions": {
    "CsvColumn": {
      "title": "CsvColumn",
      "description": "CSV column format ",
      "type": "object",
      "properties": {
        "name": { "title": "name", "description": "Column name", "type": "string" },
        "description": { "type": "string" },
        "data_type": { "title": "data type", "pattern": "(BOOLEAN|BOOL|LOGICAL|TINYINT|INT1|BYTE|SMALLINT|INT2|SHORT|INTEGER|INT4|INT|BIGINT|INT8|LONG|FLOAT|REAL|FLOAT4|DOUBLE|FLOAT8|NUMERIC|DECIMAL\\(\\d+,\\d+\\)|DATE|TIME|TIMESTAMP|DATETIME|STRING|VARCHAR|CHAR|BPCHAR|TEXT)", "type": "string" },
        "skip": { "default": false, "type": "boolean" },
        "format": { "type": "string" },
        "name_in_header": { "type": "string" }
      },
      "required": ["name"]
    }
  }
}
```

---

### ###### schema/fixed-width-format.json ######

```json
{
  "title": "FixedWidthFormat",
  "description": "Fixed-width field format ",
  "type": "object",
  "properties": {
    "kind": { "title": "Kind", "default": "FixedWidthFormat", "enum": ["FixedWidthFormat"], "type": "string" },
    "name": { "type": "string" },
    "target_name": { "type": "string" },
    "title": { "type": "string" },
    "description": { "type": "string" },
    "columns": { "type": "array", "items": { "$ref": "#/definitions/FixedWidthColumn" } },
    "encoding": { "default": "UTF-8", "type": "string" },
    "header": { "default": false, "type": "boolean" },
    "null_values": { "type": "array", "items": { "type": "string" } },
    "true_values": { "type": "array", "items": { "type": "string" } },
    "false_values": { "type": "array", "items": { "type": "string" } },
    "date_test": { "type": "string" },
    "date_format": { "type": "string" },
    "timestamp_test": { "type": "string" },
    "timestamp_format": { "type": "string" },
    "timezone": { "type": "string" }
  },
  "required": ["name", "columns"],
  "definitions": {
    "FixedWidthColumn": {
      "title": "FixedWidthColumn",
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "description": { "type": "string" },
        "data_type": { "pattern": "(BOOLEAN|BOOL|LOGICAL|TINYINT|INT1|BYTE|SMALLINT|INT2|SHORT|INTEGER|INT4|INT|BIGINT|INT8|LONG|FLOAT|REAL|FLOAT4|DOUBLE|FLOAT8|NUMERIC|DECIMAL\\(\\d+,\\d+\\)|DATE|TIME|TIMESTAMP|DATETIME|STRING|VARCHAR|CHAR|BPCHAR|TEXT)", "type": "string" },
        "skip": { "default": false, "type": "boolean" },
        "format": { "type": "string" },
        "start_index": { "type": "integer" },
        "end_index": { "type": "integer" },
        "strip_whitespaces": { "default": "both", "pattern": "(left|right|both|no)", "type": "string" },
        "null_if_empty": { "default": true, "type": "boolean" },
        "null_values": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["name", "start_index", "end_index"]
    }
  }
}
```

---

## Source — core

### ###### src/azfr_parsing_utils/__init__.py ######

```python
# (empty)
```

---

### ###### src/azfr_parsing_utils/format.py ######

```python
from typing import Optional, Literal, List

from pydantic import BaseModel, Field

DATA_TYPE_REGEX = (
    r"(BOOLEAN|BOOL|LOGICAL|TINYINT|INT1|BYTE|SMALLINT|INT2|SHORT|INTEGER|INT4|INT|BIGINT|INT8|LONG|"
    r"FLOAT|REAL|FLOAT4|DOUBLE|FLOAT8|NUMERIC|DECIMAL\(\d+,\d+\)|"
    r"DATE|TIME|TIMESTAMP|DATETIME|STRING|VARCHAR|CHAR|BPCHAR|TEXT)"
)


class BaseColumn(BaseModel):
    """Abstract class of data column"""

    name: str = Field(title="name", description="Column name")

    description: Optional[str] = Field(
        title="description",
        description="Long description (can be multiple lines) of the format",
        default=None,
    )

    data_type: str = Field(
        title="data type",
        description="SQL data type of the column",
        pattern=DATA_TYPE_REGEX,
        default=None,
    )

    skip: Optional[bool] = Field(
        title="skip", description="If true, the column won't be read", default=False
    )


class BaseFormat(BaseModel):
    """Abstract class of data format"""

    kind: Literal["BaseFormat"] = "BaseFormat"

    name: str = Field(title="name", description="Name of the format")

    target_name: Optional[str] = Field(
        title="target_name",
        description="Optional override for the name used as the output table name during parsing.",
        default=None,
    )

    @property
    def effective_name(self) -> str:
        """Returns target_name if set, otherwise falls back to name."""
        return self.target_name or self.name

    title: Optional[str] = Field(
        title="title", description="Short title of the format", default=None
    )

    description: Optional[str] = Field(
        title="description",
        description="Long description (can be multiple lines) of the format",
        default=None,
    )

    columns: List[BaseColumn] = Field(title="columns", description="Data columns")

    extraction_type: Optional[Literal["delta", "full", "mixed"]] = Field(
        title="extraction_type",
        description="full mode or delta or both for file format",
        default=None,
    )
```

---

### ###### src/azfr_parsing_utils/common.py ######

```python
from __future__ import annotations

import logging
import re
import threading
from abc import abstractmethod
from logging import Logger
from math import log10, pow
from typing import Optional, Union, Iterator, Iterable, Any

import azfr_fsspec_utils.path as fspath
import pyarrow as pa
from fsspec.core import OpenFile
from pyarrow import Schema, Table, RecordBatch, BufferOutputStream
from pyarrow.csv import WriteOptions, write_csv

DECIMAL_PAT = re.compile(r"DECIMAL\((\d+),(\d+)\)")
""" Regular expresion matching `DECIMAL` type SQL type """

_CANONICAL_DATA_TYPE_MAP = {
    "BOOLEAN": "BOOLEAN", "BOOL": "BOOLEAN", "LOGICAL": "BOOLEAN",
    "TINYINT": "TINYINT", "INT1": "TINYINT", "BYTE": "TINYINT",
    "SMALLINT": "SMALLINT", "INT2": "SMALLINT", "SHORT": "SMALLINT",
    "INTEGER": "INTEGER", "INT4": "INTEGER", "INT": "INTEGER",
    "BIGINT": "BIGINT", "INT8": "BIGINT", "LONG": "BIGINT",
    "FLOAT": "FLOAT", "REAL": "FLOAT", "FLOAT4": "FLOAT",
    "DOUBLE": "DOUBLE", "FLOAT8": "DOUBLE", "NUMERIC": "DOUBLE", "DECIMAL": "DOUBLE",
    "DATE": "DATE", "TIME": "TIME", "TIMESTAMP": "TIMESTAMP", "DATETIME": "TIMESTAMP",
    "STRING": "STRING", "VARCHAR": "STRING", "CHAR": "STRING", "BPCHAR": "STRING", "TEXT": "STRING",
}


def canonicalize_data_type(data_type: str) -> str:
    """Canonicalize data type name"""
    t = data_type.upper()
    if DECIMAL_PAT.fullmatch(t):
        return t
    if t in _CANONICAL_DATA_TYPE_MAP:
        return _CANONICAL_DATA_TYPE_MAP[t]
    else:
        raise ValueError("Unknown data type: {}".format(data_type))


def map_to_pyarrow_type(data_type: str) -> pa.DataType:
    """Map SQL data type to PyArrow data type."""
    if data_type == "BOOLEAN": return pa.bool_()
    elif data_type == "TINYINT": return pa.int8()
    elif data_type == "SMALLINT": return pa.int16()
    elif data_type == "INTEGER": return pa.int32()
    elif data_type == "BIGINT": return pa.int64()
    elif data_type == "FLOAT": return pa.float32()
    elif data_type == "DOUBLE": return pa.float64()
    elif DECIMAL_PAT.fullmatch(data_type):
        m = DECIMAL_PAT.fullmatch(data_type)
        return pa.decimal128(int(m.group(1)), int(m.group(2)))
    elif data_type == "DATE": return pa.date32()
    elif data_type == "TIME": return pa.time64("us")
    elif data_type == "TIMESTAMP": return pa.timestamp(unit="us")
    elif data_type == "STRING": return pa.string()
    else: raise ValueError("Column with unknown type: {}".format(data_type))


def get_schema(data: Iterable[RecordBatch], default: Optional[Schema] = None):
    """Extract schema from a given data source"""
    if hasattr(data, "schema") and isinstance(data.schema, Schema):
        return data.schema
    elif default:
        return default
    else:
        raise ValueError("Input data {} does not have schema.".format(data.__class__.__name__))


class AdditionalColumn:
    """Definition of an additional column filled by a constant value"""
    def __init__(self, name: str, data_type: str, value: Any) -> None:
        super().__init__()
        self.name = name
        self.data_type = canonicalize_data_type(data_type)
        self.value = value


class WithLogger:
    def __init__(self, logger: Optional[Logger] = None):
        """Abstract class having a logger"""
        super().__init__()
        self.__logger = logger or logging.getLogger(self.__class__.__name__)

    @property
    def log(self):
        return self.__logger


class Startable:
    """Abstract class having common logic for ensuring something already started"""
    def __init__(self) -> None:
        super().__init__()
        self.__has_started = False

    @property
    def has_started(self) -> bool:
        return self.__has_started

    def ensure_started(self):
        if not self.__has_started:
            self.__has_started = True


class Closable:
    """Abstract class having common logic for ensuring something already closed"""
    def __init__(self) -> None:
        super().__init__()
        self.__is_closed = False

    @property
    def is_closed(self) -> bool:
        return self.__is_closed

    def close(self):
        if not self.is_closed:
            self.__is_closed = True


class RejectedFileHandler(WithLogger, Startable, Closable):
    def __init__(self, logger: Logger, rejected_file: str) -> None:
        """Handler for writing rejected lines into a file."""
        super().__init__(logger=logger)
        self.__rejected_file = rejected_file
        self.__lock = threading.Lock()
        self.__handler = None
        self.__openfile: OpenFile = None
        self.__errors = 0
        self.__error_next_report = 1

    def ensure_started(self):
        with self.__lock:
            if not self.has_started:
                rejected_root = fspath.dirname(self.__rejected_file)
                if not fspath.isdir(rejected_root):
                    raise ValueError("Output location of rejected lines is not a directory: {}".format(rejected_root))
                super().ensure_started()

    @staticmethod
    def __next_report(count: int) -> int:
        p = int(pow(10, int(log10(count))))
        return (int(count / p) + 1) * p

    def write(self, rejected: Union[str, bytes]):
        """Write rejected lines to a rejected file"""
        self.ensure_started()
        if isinstance(rejected, str):
            rejected = rejected.encode("utf-8")
        with self.__lock:
            try:
                if not self.__handler:
                    self.log.info("Start outputting rejected lines: {}".format(self.__rejected_file))
                    self.__openfile = fspath.open(self.__rejected_file, mode="wb")
                    self.__handler = self.__openfile.open()
                self.__handler.write(rejected)
            except BaseException:
                self.__errors += 1
                if self.__errors >= self.__error_next_report:
                    self.log.warning("Failed to output rejected lines.", exc_info=True)
                    self.__error_next_report = self.__next_report(self.__errors)

    def close(self):
        if not self.is_closed:
            with self.__lock:
                if self.__handler:
                    try:
                        self.log.info("Close rejected file: {}".format(self.__rejected_file))
                        self.__openfile.close()
                    except BaseException:
                        self.log.warning("Failed to close rejected file: {}".format(self.__rejected_file), exc_info=True)
                    finally:
                        self.__handler = None
                        self.__openfile = None
                super().close()


class WithReporting(WithLogger, Closable):
    def __init__(self, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        """Abstract class with capability to report processes"""
        super().__init__(logger=logger)
        self.rejected_file = rejected_file
        if rejected_file_handler:
            self.__rejected_file_handler = rejected_file_handler
        elif rejected_file:
            self.__rejected_file_handler = RejectedFileHandler(self.log, rejected_file)
        else:
            self.__rejected_file_handler = None
        self.__max_rejected_lines = max_rejected_lines
        self.__processed_lines = 0
        self.__processed_lines_next_report = 1
        self.__rejected_lines = 0
        self.__rejected_lines_next_report = 1

    @property
    def processed_lines(self): return self.__processed_lines
    @property
    def rejected_lines(self): return self.__rejected_lines
    @property
    def max_rejected_lines(self): return self.__max_rejected_lines
    @property
    def _rejected_file_handler(self): return self.__rejected_file_handler

    def close(self):
        if not self.is_closed:
            if self.__rejected_file_handler:
                self.__rejected_file_handler.close()
            super().close()

    @staticmethod
    def __next_report(count: int) -> int:
        p = int(pow(10, int(log10(count))))
        return (int(count / p) + 1) * p

    def report_processed_lines(self, count: int, message=None):
        self.__processed_lines += count
        if self.processed_lines > self.__processed_lines_next_report:
            self.log.info("{} lines processed{}".format(self.processed_lines, ": {}".format(message) if message else ""))
            self.__processed_lines_next_report = self.__next_report(self.processed_lines)

    def report_rejected_lines(self, count: int, rejected, message=None):
        self.__rejected_lines += count
        if self.rejected_lines >= self.__rejected_lines_next_report:
            self.log.warning("{} lines rejected{}".format(self.rejected_lines, ": {}".format(message) if message else ""))
            self.__rejected_lines_next_report = self.__next_report(self.rejected_lines)
        if self.__rejected_file_handler:
            self.__rejected_file_handler.write(rejected)

    def check_rejection_limit(self) -> bool:
        if self.max_rejected_lines != -1 and self.rejected_lines > self.max_rejected_lines:
            self.log.error("Number of rejected lines ({}) has exceeded the limit ({}).".format(self.rejected_lines, self.max_rejected_lines))
            return False
        return True


class RecordBatchProducer(WithReporting, Startable):
    def __init__(self, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        """Abstract class producing stream of PyArrow RecordBatch"""
        super().__init__(rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.__schema = None

    @abstractmethod
    def _output_schema(self) -> Schema: pass

    @abstractmethod
    def _next_batch(self) -> RecordBatch: pass

    def process_report_message(self) -> Optional[str]: return None

    @property
    def schema(self):
        if not self.__schema:
            self.__schema = self._output_schema()
        return self.__schema

    def ensure_started(self):
        if not self.has_started:
            if self._rejected_file_handler:
                self._rejected_file_handler.ensure_started()
            super().ensure_started()

    def close(self):
        if not self.is_closed:
            self.log.info("{} line processed, {} lines rejected".format(self.processed_lines, self.rejected_lines))
            super().close()

    def __enter__(self) -> RecordBatchProducer: return self
    def __exit__(self, exc_type, exc_value, traceback): self.close()
    def __iter__(self): return self

    def __next__(self):
        if self.is_closed: raise StopIteration
        self.ensure_started()
        batch = self._next_batch()
        if not self.check_rejection_limit():
            raise RuntimeError("Number of rejected lines has exceeded the limit")
        self.report_processed_lines(batch.num_rows, self.process_report_message())
        return batch


class RecordBatchProcessor(RecordBatchProducer):
    """Abstract class processing stream of PyArrow RecordBatch"""

    def __init__(self, data, schema=None, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        super().__init__(rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.__upstream = data
        self.__upstream_schema = get_schema(data, schema)
        self.__upstream_iter: Iterator[RecordBatch] = None
        self.__processed_iter: Iterator[RecordBatch] = None

    @abstractmethod
    def _process_batch(self, batch: RecordBatch) -> Union[RecordBatch, Table]: pass

    __rejected_file_csv_options = WriteOptions(include_header=False)

    def report_rejected_rows(self, rows, message=None):
        buf = BufferOutputStream()
        write_csv(rows, buf, self.__rejected_file_csv_options)
        self.report_rejected_lines(rows.num_rows, buf.getvalue().to_pybytes(), message)

    def _output_schema(self) -> Schema:
        empty = self._process_batch(RecordBatch.from_pylist([], self.__upstream_schema))
        if isinstance(empty, (RecordBatch, Table)):
            return empty.schema
        raise ValueError("'process' method returns neither PyArrow RecordBatch nor a Table")

    def close(self):
        try: self.__upstream.close()
        except BaseException: self.log.warning("Failed to close upstream RecordBatchReader", exc_info=True)
        super().close()

    def _next_batch(self) -> RecordBatch:
        if self.__processed_iter:
            try: return self.__processed_iter.__next__()
            except StopIteration: self.__processed_iter = None
        if not self.__upstream_iter:
            self.__upstream_iter = self.__upstream.__iter__()
        next = self._process_batch(self.__upstream_iter.__next__())
        if isinstance(next, RecordBatch): return next
        elif isinstance(next, Table):
            self.__processed_iter = next.to_batches().__iter__()
            return self.__processed_iter.__next__()
        raise ValueError("'process' method returns neither PyArrow RecordBatch nor a Table")


class RecordBatchProcessorWrapper(RecordBatchProducer):
    def __init__(self, processors: list[RecordBatchProducer], logger=None, rejected_file_handler=None):
        """Wrapper of RecordBatchProducer producing total count of rejections"""
        super().__init__(logger=logger, rejected_file_handler=rejected_file_handler)
        self.__processors = processors

    def _output_schema(self) -> Schema: return self.processors[-1].schema
    def _next_batch(self) -> RecordBatch: raise RuntimeError()

    @property
    def processors(self): return self.__processors
    @property
    def processed_lines(self): return self.processors[-1].processed_lines
    @property
    def rejected_lines(self): return sum([p.rejected_lines for p in self.processors])
    @property
    def max_rejected_lines(self): return None

    def __enter__(self) -> RecordBatchProcessorWrapper: return self
    def __exit__(self, exc_type, exc_value, traceback):
        self.processors[-1].close()
        self.close()
    def __iter__(self): return self.processors[-1]
    def __next__(self): return self.processors[-1].__next__()
```

---

### ###### src/azfr_parsing_utils/utils.py ######

```python
import os
import tempfile
import logging
import contextlib
from contextlib import contextmanager
from io import DEFAULT_BUFFER_SIZE, BufferedReader, TextIOWrapper
from typing import Generator, Iterable, List, Optional, Pattern, Sequence
from datetime import datetime as dt

import yaml
import pyarrow as pa
from pyarrow import RecordBatch, RecordBatchReader, Schema
import azfr_fsspec_utils as fspath

from .common import get_schema

log = logging.getLogger(__name__)


def write_arrow_stream_file(data: Iterable[RecordBatch], path: str, schema: Optional[Schema] = None) -> None:
    """Write RecordBatch stream into a PyArrow stream file."""
    with pa.OSFile(path, "wb") as file:
        with pa.ipc.new_stream(file, schema=get_schema(data, schema)) as sink:
            log.info("Write PyArrow records to PyArrow streaming file: {}".format(path))
            for batch in data:
                sink.write(batch)


@contextmanager
def arrow_stream_file_data(path: str) -> contextlib.AbstractContextManager[RecordBatchReader]:
    """Context manager reading PyArrow stream file as a stream of RecordBatch."""
    with pa.OSFile(path, "rb") as file:
        with pa.ipc.open_stream(file) as reader:
            log.info("Read PyArrow records from PyArrow streaming file: {}".format(path))
            yield reader


@contextmanager
def checkpoint_file(data: Iterable[RecordBatch], schema: Optional[Schema] = None) -> Generator[str, None, None]:
    """Context manager returning local file path of a checkpointed version of a PyArrow RecordBatch stream."""
    with tempfile.TemporaryDirectory() as temp_dir:
        temp_path = os.path.join(temp_dir, "parsed.arrow")
        write_arrow_stream_file(data=data, path=temp_path, schema=get_schema(data, schema))
        yield temp_path


@contextmanager
def checkpoint(data: Iterable[RecordBatch], schema: Optional[Schema] = None) -> contextlib.AbstractContextManager[RecordBatchReader]:
    """Context manager returning a checkpointed version of a PyArrow RecordBatch stream."""
    with checkpoint_file(data=data, schema=get_schema(data, schema)) as temp_path:
        with arrow_stream_file_data(path=temp_path) as local_temp:
            try:
                yield local_temp
            finally:
                local_temp.close()


def get_dates(folder: str, pattern: Pattern) -> Sequence[str]:
    """Get a list of dates from the landing zone."""
    files = get_files(folder, pattern)
    dates = [pattern.match(fspath.basename(f)).group("date") for f in files]
    unique_dates = list(set(dates))
    unique_dates.sort(key=lambda d: dt.strptime(d, "%Y%m%d"))
    return unique_dates


def get_files(path: str, pattern: Pattern) -> List:
    """Get a list of files from the given path by using the given pattern."""
    if fspath.exists(path) and fspath.isdir(path):
        list_files = fspath.listdir(path)
    else:
        raise ValueError("Path {} does not exist or is not a dir".format(path))
    return [fspath.abspath(fspath.join(path, file)) for file in list_files if pattern.match(file)]


def get_files_for_date(path: str, date: str, pattern: Pattern) -> List:
    """Get a list of files matching given pattern and date."""
    files = get_files(path, pattern)
    if not files:
        return []
    return [f for f in files if pattern.match(fspath.basename(f)).group("date") == date]


def get_files_for_table_and_date(path: str, date: str, table: str, pattern: Pattern) -> List:
    """Get a list of files matching given pattern, date and table name."""
    if not fspath.exists(path):
        return []
    files = get_files(path, pattern)
    if not files:
        return []
    return [
        f for f in files
        if pattern.match(fspath.basename(f)).group("date") == date
        and pattern.match(fspath.basename(f)).group("name").upper() == table.upper()
    ]


class TextSanitizeIOWrapper(BufferedReader):
    """Wrapper of BufferedIOBase sanitizing invalid character encoding in a byte stream."""
    def __init__(self, source, encoding: str = None, errors: str = None, buffer_size=DEFAULT_BUFFER_SIZE):
        super().__init__(TextIOWrapper(source, encoding=encoding, errors=errors), buffer_size=buffer_size)
        self.encoding = encoding
        self.errors = errors

    def read(self, size=-1): return self.raw.read(size).encode(self.encoding, errors=self.errors)
    def read1(self, size=-1): return self.raw.read1(size).encode(self.encoding, errors=self.errors)
    def peek(self, size=-1): return self.raw.peek(size).encode(self.encoding, errors=self.errors)


def get_tables_to_yaml(format_folder: str):
    """Get dict mapping table name to path of corresponding yaml file."""
    tables_to_yaml = {}
    schemas = [file for file in fspath.listdir(format_folder) if file.endswith(".yaml")]
    if not schemas:
        log.warning(f"Warning: No yaml file found in {format_folder}")
    for file in schemas:
        schema_path = fspath.abspath(fspath.join(format_folder, file))
        with fspath.open(schema_path) as f:
            file_format = yaml.load(f, yaml.FullLoader)
            tables_to_yaml[file_format["name"]] = schema_path
    return tables_to_yaml
```

---

### ###### src/azfr_parsing_utils/azure.py ######

```python
import os
import sys
from enum import Enum
from typing import Union

from azure.identity import ManagedIdentityCredential, AzureCliCredential
from azure.identity._constants import EnvironmentVariables


def use():
    """Enable default settings for using Azure Storage account"""
    import azfr_parsing_utils.deltalake
    azfr_parsing_utils.deltalake.DEFAULT_RS_STORAGE_OPTIONS.update(get_rs_storage_options())
    import azfr_fsspec_abfs
    azfr_fsspec_abfs.use()


class AuthentictionMethod(Enum):
    """Types of the authentication method for accessing Azure"""
    CLI = "cli"
    MANAGED_IDENTOTY = "managed_identity"


azure_storage_default_scope = "https://storage.azure.com/.default"


def identify_available_authentication_methods() -> Union[AuthentictionMethod, None]:
    """Identify an authentication method available for accessing Azure Storage Accounts."""
    try:
        AzureCliCredential().get_token(azure_storage_default_scope)
        return AuthentictionMethod.CLI
    except BaseException:
        pass
    client_id = os.environ.get(EnvironmentVariables.AZURE_CLIENT_ID)
    if client_id:
        try:
            ManagedIdentityCredential(client_id=client_id, use_env_settings=False).get_token(azure_storage_default_scope)
            return AuthentictionMethod.MANAGED_IDENTOTY
        except BaseException:
            pass
    return None


def get_rs_storage_options():
    """Returns storage options for Rust Object Store to access Azure Storage Account"""
    method = identify_available_authentication_methods()
    if method is None:
        print("Warning: No available authentication method was found", file=sys.stderr)
        return {}
    elif method == AuthentictionMethod.CLI:
        return {"azure_use_azure_cli": "true"}
    elif method == AuthentictionMethod.MANAGED_IDENTOTY:
        client_id = os.environ.get(EnvironmentVariables.AZURE_CLIENT_ID)
        token = ManagedIdentityCredential(client_id=client_id, use_env_settings=False).get_token(azure_storage_default_scope)
        return {"azure_storage_token": token.token}
```

---

### ###### src/azfr_parsing_utils/deltalake.py ######

```python
import azfr_fsspec_utils
import logging
import os
import pyarrow as pa
import re
from collections import OrderedDict
from deltalake import write_deltalake as rs_write_deltalake, DeltaTable
from pathlib import Path
from pyarrow import RecordBatch, Schema, schema as pa_schema
from typing import Optional, Literal, List, Tuple, Mapping, Any, Iterable, Iterator
from .common import get_schema

log = logging.getLogger(__name__)

DEFAULT_DELTALAKE_OPTIONS = {
    "max_open_files": 1024,
    "max_rows_per_file": 10485760,
    "min_rows_per_group": 65536,
    "max_rows_per_group": 131072,
}
DEFAULT_RS_STORAGE_OPTIONS = {}
__IS_URL = re.compile(r"^[a-zA-Z][a-zA-Z0-9.+-]+://")


def deltalake_metadata(name=None, description=None, configuration=None):
    """Return the dictionary of Delta Lake table metadata."""
    return {"name": name, "description": description, "configuration": configuration}


def write_deltalake(
    data: Iterable[RecordBatch],
    path: str,
    mode: Literal["error", "append", "overwrite", "ignore"],
    schema: Optional[Schema] = None,
    partition_by: Optional[List[str]] = None,
    overwrite_schema: bool = False,
    partition_filters=None,
    engine: Literal["pyarrow", "rust"] = "pyarrow",
    predicate: Optional[str] = None,
    parquet_options=None,
    deltalake_options=None,
    rs_storage_options=None,
    metadata=None,
    merge_schema: bool = False,
    partition_none_value: str = "-1234567890",
):
    """Write a stream of PyArrow records into a Delta Lake file"""
    if engine == "pyarrow":
        if predicate is not None:
            raise ValueError("Predicate is not supported by PyArrow engine")
    elif engine == "rust":
        if partition_filters is not None:
            raise ValueError("Partition filters is not supported by Rust engine")
    else:
        raise ValueError(f"Unsupported engine: {engine}")

    if os.name == "nt":
        if not __IS_URL.match(path):
            path = Path(os.path.abspath(path)).as_uri()

    options = dict(**DEFAULT_DELTALAKE_OPTIONS)
    if deltalake_options: options.update(deltalake_options)
    if metadata: options.update(metadata)
    if not rs_storage_options:
        rs_storage_options = DEFAULT_RS_STORAGE_OPTIONS or None

    schema = get_schema(data, schema)
    schema_mode = None
    if merge_schema:
        if engine == "pyarrow":
            if azfr_fsspec_utils.exists(path):
                data, schema = update_schema(data=data, data_schema=schema, path=path, rs_storage_options=rs_storage_options, engine="pyarrow", partition_by=partition_by, partition_none_value=partition_none_value)
            schema_mode = "overwrite" if overwrite_schema else None
        else:
            if azfr_fsspec_utils.exists(path):
                data, schema = update_schema(data=data, data_schema=schema, path=path, rs_storage_options=rs_storage_options, engine="rust")
            schema_mode = "overwrite" if overwrite_schema else "merge"

    log.info("Write PyArrow records to Delta Lake: {}".format(path))
    rs_write_deltalake(data=data, engine=engine, schema=schema, table_or_uri=path, mode=mode, partition_by=partition_by, schema_mode=schema_mode, partition_filters=partition_filters, predicate=predicate, file_options=parquet_options, storage_options=rs_storage_options, **options)
    log.info("Finish writing PyArrow records to Delta Lake: {}".format(path))


def update_schema(data, path, data_schema=None, rs_storage_options=None, engine="pyarrow", partition_by=None, partition_none_value="-1234567890"):
    """Merge schema of data to be added and the existing delta table"""
    tb_schema = DeltaTable(path).schema().to_pyarrow()
    data_schema = get_schema(data, data_schema)

    def sort_arrow_schema(schema): return pa_schema(sorted(iter(schema), key=lambda x: (x.name, str(x.type))))
    if sort_arrow_schema(data_schema).equals(sort_arrow_schema(tb_schema)):
        return data, data_schema

    mismatch = OrderedDict()
    for f1 in tb_schema:
        if f1.name in data_schema.names:
            f2 = data_schema.field(f1.name)
            if f2 and f1.type != f2.type:
                mismatch[f1.name] = (type, f1.type, f2.type)
    if mismatch:
        raise RuntimeError(f"Column type mismatch {mismatch}")

    merged_schema = tb_schema
    for f2 in data_schema:
        if f2.name not in tb_schema.names:
            merged_schema = merged_schema.append(f2)

    if engine == "pyarrow":
        if partition_by is None or partition_none_value is None:
            raise ValueError("partition_by and partition_none_value are required for PyArrow engine")
        rs_write_deltalake(table_or_uri=path, data=merged_schema.empty_table(), engine="pyarrow", partition_by=partition_by, mode="overwrite", schema_mode="overwrite", partition_filters=[(partition, "=", partition_none_value) for partition in partition_by], storage_options=rs_storage_options)

    return ExtendSchema(data, source_schema=data_schema, target_schema=merged_schema), merged_schema


class ExtendSchema(Iterable[RecordBatch]):
    def __init__(self, data, source_schema, target_schema):
        self.data = data
        self.mapping = [i if i != -1 else None for i in [source_schema.get_field_index(f.name) for f in target_schema]]
        self.schema = target_schema
        self.iterator = None

    def __iter__(self) -> Iterator[RecordBatch]:
        self.iterator = self.data.__iter__()
        return self

    def __next__(self) -> RecordBatch:
        batch = self.iterator.__next__()
        projected = [batch.column(m) if m is not None else pa.nulls(batch.num_rows, self.schema.field(i).type) for i, m in enumerate(self.mapping)]
        return pa.record_batch(data=projected, schema=self.schema)
```

---

### ###### src/azfr_parsing_utils/parquet.py ######

```python
import azfr_fsspec_utils.path as fspath
import logging
from pyarrow import RecordBatch, Schema
from pyarrow.parquet import ParquetWriter
from typing import Iterable, Optional
from .common import get_schema

log = logging.getLogger(__name__)
DEFAULT_PARQUET_OPTIONS = {"flavor": set("spark")}


def write_parquet(data: Iterable[RecordBatch], path: str, schema: Optional[Schema] = None, parquet_options=None):
    """Write a given PyArrow RecordBatch stream into a parquet file."""
    options = dict(**DEFAULT_PARQUET_OPTIONS)
    if parquet_options: options.update(parquet_options)
    fs = fspath.get_file_system(path)
    with ParquetWriter(where=path, schema=get_schema(data, schema), filesystem=fs, **options) as writer:
        log.info("Write PyArrow records to Parquet file: {}".format(path))
        for batch in data:
            writer.write(batch)
        log.info("Finish writing PyArrow records to Parquet file: {}".format(path))
```

---

### ###### src/azfr_parsing_utils/polars.py ######

```python
import polars as pl
import pyarrow as pa
from logging import Logger
from polars import Expr, DataType
from pyarrow import RecordBatch, Table, Schema
from typing import Union, Iterable, Optional, Callable
from .common import RecordBatchProcessor, RejectedFileHandler, canonicalize_data_type, DECIMAL_PAT, AdditionalColumn

try:
    pl.Config.activate_decimals()
except AttributeError:
    pass


class PolarsProcessor(RecordBatchProcessor):
    def __init__(self, data, transform=None, filter=None, schema=None, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        """Processor modifying PyArrow RecordBatch streams by using Polars"""
        super().__init__(data, schema, rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.transform = transform
        self.filter = filter

    def _process_batch(self, batch: RecordBatch) -> Union[RecordBatch, Table]:
        input_table = pa.Table.from_batches([batch])
        input_schema = input_table.schema
        df = pl.from_arrow(input_table)

        if self.filter is not None:
            rejected_rows = df.filter(self.filter.not_()).to_arrow()
            if rejected_rows.num_rows:
                for i in range(0, rejected_rows.num_columns):
                    if rejected_rows.schema[i].type.__class__ in [pa.ListType, pa.LargeListType, pa.StructType]:
                        col = rejected_rows[i]
                        field = pa.field(rejected_rows.schema[i].name, pa.string())
                        rejected_rows = rejected_rows.remove_column(i)
                        rejected_rows = rejected_rows.add_column(i, field, pa.array([str(val) for val in col.to_pylist()], pa.string()))
                self.report_rejected_rows(rejected_rows, "Rejected by Polars filtering")
            df = df.filter(self.filter)

        if self.transform:
            df = self.transform(df)

        table = df.to_arrow()
        schema = table.schema

        def map_to_delta_rs_compatible(f):
            if f.type == pa.large_utf8(): return f.with_type(pa.utf8())
            elif f.type == pa.large_string(): return f.with_type(pa.utf8())
            elif isinstance(f.type, (pa.Time32Type, pa.Time64Type)): return f.with_type(pa.utf8())
            elif f.type.__class__ == pa.LargeListType: return f.with_type(pa.list_(map_to_delta_rs_compatible(f.type.value_field)))
            elif f.type.__class__ == pa.StructType: return f.with_type(pa.struct([(field.name, map_to_delta_rs_compatible(field).type) for field in f.type]))
            else: return f

        new_schema = pa.schema([map_to_delta_rs_compatible(f) for f in schema], metadata=schema.metadata)
        table = table.cast(new_schema)

        def convert_map(input_schema, table):
            for i in range(0, input_schema.empty_table().num_columns):
                if input_schema[i].type.__class__ == pa.MapType:
                    for j in range(0, table.num_columns):
                        if table.schema[j].name == input_schema[i].name:
                            col_to_convert = table[j]
                            out = []
                            for item in col_to_convert:
                                out_dict = {}
                                if item.is_valid:
                                    for item2 in item:
                                        val = list(item2.values())
                                        out_dict[val[0]] = val[1]
                                else:
                                    out_dict = None
                                out.append(out_dict)
                            field = pa.field(table.schema[j].name, pa.map_(table.schema[j].type.value_field.type[0].type, table.schema[j].type.value_field.type[1].type))
                            table = table.remove_column(j)
                            table = table.add_column(j, field, pa.array(out, type=field.type))
            return table

        return convert_map(input_schema, table)


class ColumnParser:
    """Polars expressions for parsing values of a column."""
    def __init__(self, test: Union[str, Callable[[Expr], Expr]], value: Callable[[Expr], Expr]) -> None:
        super().__init__()
        if isinstance(test, str):
            if not test.startswith("^"): test = "^" + test
            if not test.endswith("$"): test = test + "$"
            self.test = lambda col: col.str.contains(test)
        else:
            self.test = test
        self.value = value


def map_to_polars_type(data_type: str) -> DataType:
    """Map SQL data type name to Polars DataType"""
    data_type = canonicalize_data_type(data_type)
    if data_type == "BOOLEAN": return pl.Boolean()
    elif data_type == "TINYINT": return pl.Int8()
    elif data_type == "SMALLINT": return pl.Int16()
    elif data_type == "INTEGER": return pl.Int32()
    elif data_type == "BIGINT": return pl.Int64()
    elif data_type == "FLOAT": return pl.Float32()
    elif data_type == "DOUBLE": return pl.Float64()
    elif DECIMAL_PAT.fullmatch(data_type):
        m = DECIMAL_PAT.fullmatch(data_type)
        return pl.Decimal(precision=int(m.group(1)), scale=int(m.group(2)))
    elif data_type == "DATE": return pl.Date()
    elif data_type == "TIME": return pl.Time()
    elif data_type == "TIMESTAMP": return pl.Datetime(time_unit="us", time_zone="UTC")
    elif data_type == "STRING": return pl.Utf8()
    else: raise ValueError("Column with unknown type: {}".format(data_type))


def map_additional_column_to_polars_expression(col: AdditionalColumn) -> Expr:
    """Map additional column definition to Polars expression"""
    return pl.lit(col.value).cast(map_to_polars_type(col.data_type)).alias(col.name)
```

---

### ###### src/azfr_parsing_utils/duckdb.py ######

```python
import duckdb
import pyarrow as pa
from logging import Logger
from pyarrow import RecordBatch, Schema
from typing import Optional, Union, Iterable
from .common import RecordBatchProcessor, RejectedFileHandler


class DuckDBProcessor(RecordBatchProcessor):
    def __init__(self, data, schema=None, sql=None, filter=None, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        """Processor modifying PyArrow RecordBatch streams by using DuckDB SQL"""
        super().__init__(data, schema, rejected_file, max_rejected_lines, logger, rejected_file_handler)
        if sql is None and filter is None:
            raise ValueError("Neither sql nor filter is specified")
        if filter:
            self.__rejected_rows_sql = "SELECT * FROM T WHERE NOT ({})".format(filter)
            if sql:
                self.__transformed_rows_sql = sql.format(table="(SELECT * FROM T WHERE {})".format(filter))
            else:
                self.__transformed_rows_sql = "SELECT * FROM T WHERE {}".format(filter)
        else:
            self.__rejected_rows_sql = None
            self.__transformed_rows_sql = sql.format(table="T")

    def _process_batch(self, batch: RecordBatch) -> Union[RecordBatch, pa.Table]:
        dtable = duckdb.from_arrow(pa.Table.from_batches([batch]))
        if self.__rejected_rows_sql:
            rejected_rows = dtable.query(virtual_table_name="T", sql_query=self.__rejected_rows_sql).to_arrow_table()
            if rejected_rows.num_rows > 0:
                self.report_rejected_rows(rejected_rows, "Rejected by DuckDB filtering")
        transformed = dtable.query(virtual_table_name="T", sql_query=self.__transformed_rows_sql)
        return transformed.to_arrow_table()

    def process_report_message(self) -> Optional[str]:
        return "DuckDB process"


class ColumnParser:
    """Regular expression and SQL expression for parsing values of a column."""
    def __init__(self, test: str, value: str) -> None:
        super().__init__()
        self.test = test
        self.value = value
```

---

### ###### src/azfr_parsing_utils/pandas.py ######

```python
import pandas as pd
import pyarrow as pa
from logging import Logger
from pyarrow import RecordBatch, Table, Schema
from typing import Union, Iterable, Optional, Callable
from .common import RecordBatchProcessor, RejectedFileHandler


class PandasProcessor(RecordBatchProcessor):
    def __init__(self, data, transform=None, filter=None, schema=None, rejected_file=None, max_rejected_lines=-1, logger=None, rejected_file_handler=None):
        """Processor modifying PyArrow RecordBatch streams by using Pandas"""
        super().__init__(data, schema, rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.transform = transform
        self.filter = filter

    def _process_batch(self, batch: RecordBatch) -> Union[RecordBatch, Table]:
        df = batch.to_pandas()
        if self.filter is not None:
            rejected_rows = pa.Table.from_pandas(df[~self.filter(df)], preserve_index=False)
            if rejected_rows.num_rows:
                self.report_rejected_rows(rejected_rows, "Rejected by Polars filtering")
            df = df[self.filter(df)]
        if self.transform:
            df = self.transform(df)
        return pa.Table.from_pandas(df, preserve_index=False)
```

---

### ###### src/azfr_parsing_utils/pyspark.py ######

```python
from typing import Callable, Union
import pyspark.sql.functions as sf
import pyspark.sql.types as st
from pyspark.sql import Column
from azfr_parsing_utils.common import DECIMAL_PAT, AdditionalColumn, canonicalize_data_type


class ColumnParser:
    """PySpark expressions for parsing values of a column."""
    def __init__(self, test: Union[str, Callable[[Column], Column]], value: Callable[[Column], Column]) -> None:
        super().__init__()
        if isinstance(test, str):
            if not test.startswith("^"): test = "^" + test
            if not test.endswith("$"): test = test + "$"
            self.test = lambda col: col.rlike(test)
        else:
            self.test = test
        self.value = value


def map_to_pyspark_type(data_type: str) -> st.DataType:
    """Map SQL data type to PySpark data type."""
    original_data_type = data_type
    data_type = data_type.upper()
    if data_type == "BOOLEAN": return st.BooleanType()
    elif data_type == "TINYINT": return st.ByteType()
    elif data_type == "SMALLINT": return st.ShortType()
    elif data_type == "INTEGER": return st.IntegerType()
    elif data_type == "BIGINT": return st.LongType()
    elif data_type == "FLOAT": return st.FloatType()
    elif data_type == "DOUBLE": return st.DoubleType()
    elif DECIMAL_PAT.fullmatch(data_type):
        m = DECIMAL_PAT.fullmatch(data_type)
        return st.DecimalType(int(m.group(1)), int(m.group(2)))
    elif data_type == "DATE": return st.DateType()
    elif data_type == "TIME": return st.StringType()
    elif data_type == "TIMESTAMP": return st.TimestampType()
    elif data_type == "STRING": return st.StringType()
    elif data_type.startswith("STRUCT<"):
        fields = []
        inner = original_data_type[data_type.index("STRUCT<") + len("STRUCT<"):-1]
        for field_def in inner.split(","):
            name, typ = field_def.strip().split(":")
            typ = canonicalize_data_type(typ.strip())
            fields.append(st.StructField(name.strip(), map_to_pyspark_type(typ)))
        return st.StructType(fields)
    elif data_type.startswith("ARRAY<"):
        inner = canonicalize_data_type(data_type[len("ARRAY<"):-1].strip())
        return st.ArrayType(map_to_pyspark_type(inner))
    elif data_type.startswith("MAP<"):
        inner = data_type[len("MAP<"):-1]
        key_type, value_type = inner.split(",", 1)
        return st.MapType(map_to_pyspark_type(canonicalize_data_type(key_type.strip())), map_to_pyspark_type(canonicalize_data_type(value_type.strip())))
    else:
        raise ValueError("Column with unknown type: {}".format(data_type))


def map_additional_column_to_pyspark_column(col: AdditionalColumn) -> Column:
    """Map additional column definition to PySpark Column"""
    return sf.lit(col.value).cast(map_to_pyspark_type(col.data_type)).alias(col.name)
```

---

## Source — csv

### ###### src/azfr_parsing_utils/csv/__init__.py ######

```python
from azfr_parsing_utils.csv.format import CsvColumn, CsvFileFormat
from azfr_parsing_utils.csv.parser import (
    map_to_pa_schema, map_to_pa_read_options, map_to_pa_parse_options,
    map_to_convert_options, CsvParser,
)
__all__ = ["CsvColumn", "CsvFileFormat", "map_to_pa_schema", "map_to_pa_read_options",
           "map_to_pa_parse_options", "map_to_convert_options", "CsvParser"]
```

---

### ###### src/azfr_parsing_utils/csv/format.py ######

```python
from typing import Optional, List, Literal
from pydantic import Field
from azfr_parsing_utils.format import BaseFormat, BaseColumn


class CsvColumn(BaseColumn):
    """CSV column format"""
    format: Optional[str] = Field(title="value format", description="Format of column values", default=None)
    name_in_header: Optional[str] = Field(title="column name in CSV header", default=None)


class CsvFileFormat(BaseFormat):
    """CSV file format"""
    kind: Literal["CsvFormat"] = "CsvFormat"
    sep: Optional[str] = Field(title="separator", default=",")
    encoding: Optional[str] = Field(title="character encoding", default="UTF-8")
    encoding_errors: Optional[str] = Field(title="handling of character encoding errors", default="strict")
    quote: Optional[str] = Field(title="quote", default='"')
    double_quote: Optional[bool] = Field(title="double quote", default=False)
    escape: Optional[str] = Field(title="escape", default=None)
    comment: Optional[str] = Field(title="comment", default=None)
    header: Optional[bool] = Field(title="header", default=None)
    check_header: Optional[bool] = Field(title="check_header", default=None)
    ignore_leading_white_space: Optional[bool] = Field(default=None)
    ignore_trailing_white_space: Optional[bool] = Field(default=None)
    strings_can_be_null: Optional[bool] = Field(default=True)
    quoted_strings_can_be_null: Optional[bool] = Field(default=False)
    null_values: Optional[list[str]] = Field(default=None)
    true_values: Optional[list[str]] = Field(default=None)
    false_values: Optional[list[str]] = Field(default=None)
    decimal_point: Optional[str] = Field(default=None)
    nan_value: Optional[str] = Field(default=None)
    positive_inf: Optional[str] = Field(default=None)
    negative_inf: Optional[str] = Field(default=None)
    date_format: Optional[str] = Field(default=None)
    timezone: Optional[str] = Field(default=None)
    timestamp_format: Optional[str] = Field(default=None)
    timestamp_ntz_format: Optional[str] = Field(default=None)
    multi_line: Optional[bool] = Field(default=None)
    char_to_escape_quote_escaping: Optional[str] = Field(default=None)
    empty_value: Optional[str] = Field(default=None)
    line_sep: Optional[str] = Field(default=None)
    columns: List[CsvColumn] = Field(title="columns", description="Columns in CSV files")


def ignored_settings(format: CsvFileFormat, supported_settings: list[str]):
    """Return names of settings that are specified in a CSV format but not used in the parser"""
    schema = CsvFileFormat.schema()
    ignored = schema["required"] + ["kind", "title", "description"] + supported_settings
    return [k for k, v in format.model_dump().items() if k not in ignored and v is not None]
```

---

### ###### src/azfr_parsing_utils/csv/common.py ######

```python
import logging
from azfr_parsing_utils.csv.format import CsvFileFormat, ignored_settings

log = logging.getLogger(__name__)

SUPPORTED_SETTINGS = [
    "encoding", "header", "sep", "quote", "escape", "multi_line", "null_values",
    "strings_can_be_null", "quoted_strings_can_be_null", "double_quote",
    "true_values", "false_values", "decimal_point", "timestamp_ntz_format", "encoding_errors",
]


def unsupported_format_options(format: CsvFileFormat) -> list[str]:
    """Return names of options that are set in a given CsvFileFormat but not supported."""
    not_supported = ignored_settings(format, SUPPORTED_SETTINGS)
    if not_supported:
        log.warning("Unsupported settings: {}".format(not_supported))
    return not_supported


DEFAULT_BLOCK_SIZE = 10 * 1024 * 1024  # 10MB
```

---

### ###### src/azfr_parsing_utils/csv/parser.py ######

```python
from logging import Logger
from typing import Optional, Union

import azfr_fsspec_utils.path as fspath
import pyarrow as pa
from fsspec.core import OpenFile
from pyarrow import Schema, RecordBatch
from pyarrow.csv import ReadOptions, ParseOptions, ConvertOptions, InvalidRow, CSVStreamingReader, open_csv

from azfr_parsing_utils.common import canonicalize_data_type, RecordBatchProducer, RejectedFileHandler, map_to_pyarrow_type
from azfr_parsing_utils.csv.common import DEFAULT_BLOCK_SIZE
from azfr_parsing_utils.csv.format import CsvFileFormat, CsvColumn
from azfr_parsing_utils.utils import TextSanitizeIOWrapper


def map_to_pa_schema(format: CsvFileFormat, map_column_with_format_to_string: Optional[bool] = False) -> Schema:
    """Map a list of CsvColumn to PyArrow Schema object"""
    def pa_type(c: CsvColumn):
        t = canonicalize_data_type(c.data_type)
        if map_column_with_format_to_string and c.format is not None:
            return pa.string()
        return map_to_pyarrow_type(t)
    return pa.schema([pa.field(c.name, pa_type(c)) for c in format.columns if not c.skip])


def map_to_pa_read_options(format: CsvFileFormat, block_size: int = DEFAULT_BLOCK_SIZE, **kwargs) -> ReadOptions:
    """Map CsvFileFormat to PyArrow CSV read option"""
    column_names = [c.name for c in format.columns]
    if format.header and format.check_header:
        return ReadOptions(autogenerate_column_names=False, block_size=block_size, encoding=format.encoding, **kwargs)
    elif format.header:
        return ReadOptions(skip_rows=1, column_names=column_names, block_size=block_size, encoding=format.encoding, **kwargs)
    else:
        return ReadOptions(column_names=column_names, block_size=block_size, encoding=format.encoding, **kwargs)


def map_to_pa_parse_options(format: CsvFileFormat, **kwargs) -> ParseOptions:
    """Map CsvFileFormat to PyArrow CSV parse option"""
    return ParseOptions(delimiter=format.sep, quote_char=format.quote or False, escape_char=format.escape or False, newlines_in_values=format.multi_line, **kwargs)


def map_to_convert_options(format: CsvFileFormat, schema: Schema, **kwargs) -> ConvertOptions:
    """Map CsvFileFormat to PyArrow CSV value convert option"""
    def column_name(c: CsvColumn):
        if format.header and format.check_header:
            return c.name_in_header or c.name
        return c.name
    args = {
        "column_types": {column_name(c): schema.field(c.name).type for c in format.columns if not c.skip},
        "null_values": format.null_values,
        "true_values": format.true_values,
        "false_values": format.false_values,
        "decimal_point": format.decimal_point or ".",
        "strings_can_be_null": format.strings_can_be_null,
        "quoted_strings_can_be_null": format.quoted_strings_can_be_null,
        "include_columns": [column_name(c) for c in format.columns if not c.skip],
        "include_missing_columns": False,
        "timestamp_parsers": [format.timestamp_ntz_format] if format.timestamp_ntz_format else None,
    }
    args.update(kwargs)
    return ConvertOptions(**{k: v for k, v in args.items() if v is not None})


class CsvParser(RecordBatchProducer):
    def __init__(self, source, schema, read_options, parse_options, convert_options, rejected_file=None, max_rejected_lines=-1, logger=None, encoding_errors="strict", rejected_file_handler=None, compression=None):
        """Csv parser parsing one or more CSV files into a stream of PyArrow RecordBatch"""
        super().__init__(rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.sources = source if isinstance(source, list) else [source]
        self.__schema = schema
        self.read_options = read_options
        self.parse_options = ParseOptions(
            delimiter=parse_options.delimiter, quote_char=parse_options.quote_char,
            escape_char=parse_options.escape_char, newlines_in_values=parse_options.newlines_in_values,
            ignore_empty_lines=parse_options.ignore_empty_lines, invalid_row_handler=self.__handle_invalid_row,
        )
        self.convert_options = convert_options
        self.encoding_errors = encoding_errors
        self.__source_file_index = -1
        self.__source_file_reader: CSVStreamingReader = None
        self.__source_file_fsspecfile: OpenFile = None
        self.compression = compression

    def _output_schema(self) -> Schema:
        return self.__schema

    def __ensure_reader(self):
        if self.__source_file_reader is None:
            self.__source_file_index += 1
            if self.__source_file_index < len(self.sources):
                path = self.sources[self.__source_file_index]
                self.__source_file_fsspecfile = fspath.open(path, mode="rb", compression=self.compression)
                openfile = self.__source_file_fsspecfile.open()
                if self.encoding_errors != "strict":
                    openfile = TextSanitizeIOWrapper(openfile, encoding=self.read_options.encoding, errors=self.encoding_errors)
                self.__source_file_reader = open_csv(openfile, read_options=self.read_options, parse_options=self.parse_options, convert_options=self.convert_options)
            else:
                raise StopIteration

    def __close_reader(self):
        if self.__source_file_reader is not None:
            try:
                self.__source_file_reader.close()
                self.__source_file_fsspecfile.close()
            except BaseException:
                self.log.warning("Failed to close PyArrow CSV reader", exc_info=True)
            finally:
                self.__source_file_reader = None
                self.__source_file_fsspecfile = None

    def close(self):
        if not self.is_closed:
            self.__close_reader()
            super().close()

    def _next_batch(self) -> RecordBatch:
        self.__ensure_reader()
        try:
            return self.__source_file_reader.read_next_batch()
        except StopIteration:
            self.__close_reader()
            self.__ensure_reader()
            return self.__source_file_reader.read_next_batch()

    def process_report_message(self) -> Optional[str]:
        return "CsvParser {}".format(self.__source_file_fsspecfile.path) if self.__source_file_fsspecfile else None

    def __handle_invalid_row(self, row: InvalidRow):
        self.report_rejected_lines(1, row.text.rstrip() + "\n", "Parsing {}".format(self.__source_file_fsspecfile.path))
        return "skip" if self.check_rejection_limit() else "error"
```

---

### ###### src/azfr_parsing_utils/csv/utils.py ######

```python
import logging
import yaml
from typing import Dict, Type, TypeVar
import azfr_fsspec_utils as fspath
from azfr_parsing_utils.csv.format import CsvFileFormat

log = logging.getLogger(__name__)
T = TypeVar("T", bound=CsvFileFormat)


def get_tables_to_yaml(format_folder: str, format_class: Type[T] = CsvFileFormat) -> Dict[str, T]:
    """Loads YAML schema files and returns dict mapping table names to their file paths."""
    tables_to_yaml = {}
    schemas = [file for file in fspath.listdir(format_folder) if file.endswith(".yaml")]
    if not schemas:
        log.warning(f"Warning: No yaml file found in {format_folder}")
    for file in schemas:
        schema_path = fspath.abspath(fspath.join(format_folder, file))
        with fspath.open(schema_path) as f:
            file_format = format_class.model_validate(yaml.load(f, yaml.FullLoader))
            tables_to_yaml[file_format.name] = schema_path
    return tables_to_yaml


def get_format_to_yaml(format_folder: str, format_class: Type[T] = CsvFileFormat) -> Dict[str, T]:
    """Get a dict of format objects for each .yaml file in format_folder."""
    tables_to_yaml = {}
    schemas = [file for file in fspath.listdir(format_folder) if file.endswith(".yaml")]
    if not schemas:
        log.warning(f"Warning: No yaml file found in {format_folder}")
    for file in schemas:
        schema_path = fspath.abspath(fspath.join(format_folder, file))
        with fspath.open(schema_path) as f:
            file_format = format_class.model_validate(yaml.load(f, yaml.FullLoader))
            tables_to_yaml[file_format.name] = file_format
    return tables_to_yaml
```

---

### ###### src/azfr_parsing_utils/csv/polars.py ######

```python
import contextlib
import logging
from contextlib import contextmanager
from functools import reduce
from typing import Optional, Callable, Union

import polars as pl
from polars import Expr, DataFrame

from azfr_parsing_utils.common import RecordBatchProcessorWrapper, RejectedFileHandler, AdditionalColumn
from azfr_parsing_utils.csv.common import DEFAULT_BLOCK_SIZE, unsupported_format_options
from azfr_parsing_utils.csv.format import CsvFileFormat
from azfr_parsing_utils.csv.parser import map_to_pa_schema, map_to_pa_read_options, map_to_pa_parse_options, map_to_convert_options, CsvParser
from azfr_parsing_utils.polars import ColumnParser, map_to_polars_type, PolarsProcessor, map_additional_column_to_polars_expression

log = logging.getLogger(__name__)


def map_custom_parsing_to_polars_exp(format: CsvFileFormat, column_parsers: dict[str, ColumnParser]) -> (list[Expr], list[Expr]):
    """Map custom column parsing settings to transformation by Polars"""
    predicates = []
    select_list = []
    for c in format.columns:
        if c.format and not c.skip:
            if c.format in column_parsers:
                p = column_parsers[c.format]
                predicates.append(p.test(pl.col(c.name)).or_(pl.col(c.name).is_null()))
                select_list.append(p.value(pl.col(c.name)).cast(map_to_polars_type(c.data_type)).alias(c.name))
            else:
                raise ValueError("Unknown format in a column: {}".format(c))
        else:
            select_list.append(pl.col(c.name))
    return predicates, select_list


@contextmanager
def parsed_csv_data(
    source: Union[str, list[str]],
    format: CsvFileFormat,
    block_size: int = DEFAULT_BLOCK_SIZE,
    strict: Optional[bool] = False,
    column_parsers: Optional[dict[str, ColumnParser]] = None,
    additional_columns: list[AdditionalColumn] = None,
    transform: Optional[Callable[[DataFrame], DataFrame]] = None,
    filter: Optional[Expr] = None,
    rejected_file: Optional[str] = None,
    max_rejected_lines: int = -1,
    compression: str = None,
) -> contextlib.AbstractContextManager[RecordBatchProcessorWrapper]:
    """Instantiate CSVParser to parse given CSV files (Polars backend)."""
    unsupported_options = unsupported_format_options(format)
    if strict and unsupported_options:
        raise ValueError("Unsupported options: {}".format(unsupported_options))

    rejected_file_handler = RejectedFileHandler(logger=log, rejected_file=rejected_file) if rejected_file else None
    schema = map_to_pa_schema(format, map_column_with_format_to_string=True)
    read_options = map_to_pa_read_options(format, block_size)
    parse_options = map_to_pa_parse_options(format)
    convert_options = map_to_convert_options(format, schema)
    has_custom_parsing = any([c.format is not None for c in format.columns])

    if has_custom_parsing or additional_columns or transform or filter:
        if has_custom_parsing:
            predicates, select_list = map_custom_parsing_to_polars_exp(format, column_parsers)
        else:
            predicates: list[Expr] = []
            select_list = [pl.col(c.name) for c in format.columns if not c.skip]

        if additional_columns:
            select_list += [map_additional_column_to_polars_expression(c) for c in additional_columns]
        if filter is not None:
            predicates.append(filter)

        def t(df: DataFrame) -> DataFrame:
            return transform(df.select(select_list)) if transform else df.select(select_list)

        with CsvParser(source=source, schema=schema, read_options=read_options, parse_options=parse_options, convert_options=convert_options, max_rejected_lines=max_rejected_lines, encoding_errors=format.encoding_errors, rejected_file_handler=rejected_file_handler, compression=compression) as parsed:
            filter = reduce(lambda a, b: a & b, predicates) if predicates else None
            with PolarsProcessor(data=parsed, transform=t, filter=filter, max_rejected_lines=max_rejected_lines, rejected_file_handler=rejected_file_handler) as transformed:
                with RecordBatchProcessorWrapper(processors=[parsed, transformed], logger=log) as wrapped:
                    return (yield wrapped)
    else:
        with CsvParser(source=source, schema=schema, read_options=read_options, parse_options=parse_options, convert_options=convert_options, max_rejected_lines=max_rejected_lines, encoding_errors=format.encoding_errors, rejected_file_handler=rejected_file_handler, compression=compression) as parsed:
            with RecordBatchProcessorWrapper(processors=[parsed], logger=log) as wrapped:
                return (yield wrapped)
```

---

### ###### src/azfr_parsing_utils/csv/duckdb.py ######

```python
import contextlib
import logging
from contextlib import contextmanager
from datetime import datetime, timezone, date
from typing import Union, Optional

from azfr_parsing_utils.common import canonicalize_data_type, DECIMAL_PAT, RejectedFileHandler, RecordBatchProcessorWrapper, AdditionalColumn
from azfr_parsing_utils.csv import CsvFileFormat
from azfr_parsing_utils.csv.common import DEFAULT_BLOCK_SIZE, unsupported_format_options
from azfr_parsing_utils.csv.parser import map_to_pa_schema, map_to_pa_read_options, map_to_pa_parse_options, map_to_convert_options, CsvParser
from azfr_parsing_utils.duckdb import DuckDBProcessor, ColumnParser

log = logging.getLogger(__name__)


def map_custom_parsing_to_sql(format: CsvFileFormat, column_parsers: dict[str, ColumnParser]) -> (list[str], list[str]):
    """Map custom column parsing settings to elements of SQL SELECT query"""
    predicates = []
    select_list = []
    for c in format.columns:
        if c.format and not c.skip:
            if c.format in column_parsers:
                p = column_parsers[c.format]
                predicates.append("({c} IS NULL OR {c} = '' OR regexp_full_match({c}, '{t}'))".format(c=c.name, t=p.test))
                data_type = canonicalize_data_type(c.data_type)
                params = {"column": c.name, "data_type": data_type}
                if DECIMAL_PAT.fullmatch(data_type):
                    m = DECIMAL_PAT.fullmatch(data_type)
                    params["precision"] = int(m.group(1))
                    params["scale"] = int(m.group(2))
                if data_type == "TIMESTAMP":
                    data_type = "TIMESTAMPTZ"
                select_list.append("CASE {c} WHEN '' THEN NULL ELSE CAST({e} AS {t}) END AS {c}".format(c=c.name, e=p.value.format(**params), t=data_type))
            else:
                raise ValueError("Unknown format in a column: {}".format(c))
        else:
            select_list.append(c.name)
    return predicates, select_list


def map_additional_column_to_sql_expression(col: AdditionalColumn) -> str:
    """Map additional column definition to SQL column expression"""
    if col.value is None: return "CAST(NULL AS {}) AS {}".format(col.data_type, col.name)
    elif isinstance(col.value, datetime): return "TIMESTAMP '{}' AS {}".format(col.value.astimezone(timezone.utc).strftime("%Y-%m-%d %H:%M:%S"), col.name)
    elif isinstance(col.value, date): return "DATE '{}' AS {}".format(col.value.strftime("%Y-%m-%d"), col.name)
    elif isinstance(col.value, str): return "CAST('{}' AS {}) AS {}".format(col.value, col.data_type, col.name)
    elif isinstance(col.value, bool): return "CAST({} AS {}) AS {}".format("TRUE" if col.value else "FALSE", col.data_type, col.name)
    else: return "CAST({} AS {}) AS {}".format(col.value, col.data_type, col.name)


@contextmanager
def parsed_csv_data(source, format, block_size=DEFAULT_BLOCK_SIZE, strict=False, column_parsers=None, additional_columns=None, filter=None, sql=None, rejected_file=None, max_rejected_lines=-1):
    """Instantiate CSVParser to parse given CSV files (DuckDB backend)."""
    unsupported_options = unsupported_format_options(format)
    if strict and unsupported_options:
        raise ValueError("Unsupported options: {}".format(unsupported_options))

    rejected_file_handler = RejectedFileHandler(logger=log, rejected_file=rejected_file) if rejected_file else None
    schema = map_to_pa_schema(format, map_column_with_format_to_string=True)
    read_options = map_to_pa_read_options(format, block_size)
    parse_options = map_to_pa_parse_options(format)
    convert_options = map_to_convert_options(format, schema)
    has_custom_parsing = any([c.format is not None for c in format.columns])

    if has_custom_parsing or additional_columns or sql or filter:
        if has_custom_parsing:
            predicates, select_list = map_custom_parsing_to_sql(format, column_parsers)
        else:
            predicates = []
            select_list = [c.name for c in format.columns if not c.skip]
        if additional_columns:
            select_list += [map_additional_column_to_sql_expression(c) for c in additional_columns]
        if filter:
            predicates.append(filter)
        filter = " AND ".join(predicates)
        select_query = "SELECT " + (", ".join(select_list)) + " FROM {table}"
        if sql:
            sql = sql.format(table="({})".format(select_query))
        else:
            sql = select_query

        with CsvParser(source=source, schema=schema, read_options=read_options, parse_options=parse_options, convert_options=convert_options, max_rejected_lines=max_rejected_lines, encoding_errors=format.encoding_errors, rejected_file_handler=rejected_file_handler) as parsed:
            with DuckDBProcessor(data=parsed, sql=sql, filter=filter, max_rejected_lines=max_rejected_lines, rejected_file_handler=rejected_file_handler) as transformed:
                with RecordBatchProcessorWrapper(processors=[parsed, transformed], logger=log) as wrapped:
                    return (yield wrapped)
    else:
        with CsvParser(source=source, schema=schema, read_options=read_options, parse_options=parse_options, convert_options=convert_options, max_rejected_lines=max_rejected_lines, encoding_errors=format.encoding_errors, rejected_file_handler=rejected_file_handler) as parsed:
            with RecordBatchProcessorWrapper(processors=[parsed], logger=log) as wrapped:
                return (yield wrapped)
```

---

### ###### src/azfr_parsing_utils/csv/pyspark.py ######

> See full source above (252 lines). Key exports: `map_to_pyspark_schema`, `map_to_pyspark_read_options_dict`, `map_custom_parsing_to_pyspark_column`, `parsed_csv_data(spark_session, source, format, …) → (DataFrame, DataFrame)`.

*(Full source identical to content read above — omitted here for brevity but present in repo)*

---

## Source — fixed_width

### ###### src/azfr_parsing_utils/fixed_width/common.py ######

```python
DEFAULT_BLOCK_SIZE = 10 * 1024 * 1024  # 10MB
```

---

### ###### src/azfr_parsing_utils/fixed_width/format.py ######

```python
from azfr_parsing_utils.format import BaseColumn, BaseFormat
from pydantic import Field
from typing import Optional, Literal, List


class FixedWidthColumn(BaseColumn):
    """Format of fixed-width column"""
    format: Optional[str] = Field(default=None)
    start_index: int = Field(title="start index", description="The beginning of the column, starting at 1, inclusive")
    end_index: int = Field(title="end index", description="The end of the column, starting at 1, inclusive")
    strip_whitespaces: Optional[str] = Field(pattern=r"(left|right|both|no)", default="both")
    null_if_empty: Optional[bool] = Field(default=True)
    null_values: Optional[list[str]] = Field(default=None)


class FixedWidthFormat(BaseFormat):
    """Fixed-width field format"""
    kind: Literal["FixedWidthFormat"] = "FixedWidthFormat"
    encoding: Optional[str] = Field(default="UTF-8")
    header: Optional[bool] = Field(default=False)
    null_values: Optional[list[str]] = Field(default=None)
    true_values: Optional[list[str]] = Field(default=None)
    false_values: Optional[list[str]] = Field(default=None)
    date_test: Optional[str] = Field(default=None)
    date_format: Optional[str] = Field(default=None)
    timestamp_test: Optional[str] = Field(default=None)
    timestamp_format: Optional[str] = Field(default=None)
    timezone: Optional[str] = Field(default=None)
    columns: List[FixedWidthColumn] = Field(title="columns", description="Data columns")
```

---

### ###### src/azfr_parsing_utils/fixed_width/parser.py ######

```python
import azfr_fsspec_utils.path as fspath
import pyarrow as pa
from azfr_parsing_utils.common import RecordBatchProducer, RejectedFileHandler
from fsspec.core import OpenFile
from io import TextIOWrapper
from logging import Logger
from pyarrow import RecordBatch, Schema
from typing import Union, Optional, Tuple
from .common import DEFAULT_BLOCK_SIZE


class FixedWidthFileParser(RecordBatchProducer):
    """Parser reading fixed-width field files as a sequence of RecordBatch of string columns."""

    def __init__(self, source, column_names, column_indices, block_size=DEFAULT_BLOCK_SIZE, encoding="utf-8", skip_lines=0, logger=None, rejected_file=None, max_rejected_lines=-1, rejected_file_handler=None, compression=None):
        super().__init__(rejected_file, max_rejected_lines, logger, rejected_file_handler)
        self.sources = source if isinstance(source, list) else [source]
        self.column_names = column_names
        self.column_indices = column_indices
        self.max_column_indices = max([e for s, e in column_indices]) - 1
        self.block_size = block_size
        self.encoding = encoding
        self.skip_lines = skip_lines
        self.compression = compression
        self.__source_file_index = -1
        self.__source_file_fsspecfile: OpenFile = None
        self.__source_file_textio: TextIOWrapper = None

    def _output_schema(self) -> Schema:
        return pa.schema([pa.field(c, pa.string(), False) for c in self.column_names])

    def __close_current_file(self):
        if self.__source_file_fsspecfile is not None:
            self.__source_file_textio.close()
            self.__source_file_fsspecfile.close()
            self.__source_file_textio = None
            self.__source_file_fsspecfile = None

    def __open_next_file(self):
        if self.__source_file_index + 1 < len(self.sources):
            self.__close_current_file()
            self.__source_file_index += 1
            path = self.sources[self.__source_file_index]
            self.__source_file_fsspecfile = fspath.open(path, mode="rb", compression=self.compression)
            self.__source_file_textio = TextIOWrapper(self.__source_file_fsspecfile.open(), encoding=self.encoding)
            for i in range(0, self.skip_lines):
                line = self.__source_file_textio.readline()
                if not line:
                    self.__open_next_file()
        else:
            raise StopIteration

    def __ensure_fileopen(self):
        if self.__source_file_fsspecfile is None:
            self.__open_next_file()

    def _next_batch(self) -> RecordBatch:
        self.__ensure_fileopen()
        lines = []
        size = 0
        while size < self.block_size:
            line = self.__source_file_textio.readline()
            while not line:
                try:
                    self.__close_current_file()
                    self.__open_next_file()
                    line = self.__source_file_textio.readline()
                except StopIteration:
                    break
            if line:
                if self.max_column_indices < len(line):
                    lines.append(line)
                    size += len(line)
                else:
                    self.report_rejected_lines(count=1, rejected=line, message="Too short")
            else:
                break
        if lines:
            return pa.record_batch([pa.array([line[s:e] for line in lines], type=pa.string()) for s, e in self.column_indices], schema=self.schema)
        else:
            raise StopIteration

    def close(self):
        self.__close_current_file()
        self.__source_file_index = len(self.sources)
        super().close()
```

---

### ###### src/azfr_parsing_utils/fixed_width/polars.py ######

> Key exports: `FixedWidthColumnParser`, `get_column_parser`, `map_custom_parsing_to_polars_exp`, `parsed_fixed_width_data` context manager.

Default constants: `DEFAULT_BOOLEAN_TEST`, `DEFAULT_DATE_TEST = "%Y-%m-%d"`, `DEFAULT_TIMESTAMP_FORMAT = "%Y-%m-%d %H:%M:%S"`, `DEFAULT_TIMESTAMP_TIMEZONE = "Europe/Paris"`.

*(Full source identical to content read above — 304 lines)*

---

## Source — json

### ###### src/azfr_parsing_utils/json/__init__.py ######

```python
from .format import JsonColumn, JsonFileFormat, MultiValuesColumn
from .parser import map_to_pa_schema, JsonParser
from .utils import convert_to_json_schema

__all__ = ["JsonColumn", "MultiValuesColumn", "JsonFileFormat", "map_to_pa_schema", "JsonParser", "convert_to_json_schema"]
```

---

### ###### src/azfr_parsing_utils/json/format.py ######

```python
from typing import Optional, Literal, Annotated
from pydantic import Field, BaseModel
from azfr_parsing_utils.format import BaseFormat, BaseColumn
from collections.abc import Sequence


class JsonColumn(BaseColumn):
    """CSV column format"""
    format: Optional[str] = Field(default=None)
    required: Optional[bool] = Field(default=True)
    type: Literal["single_field"] = "single_field"


class MultiValuesColumn(BaseModel):
    """Class of list column"""
    type: Literal["map", "list", "struct"] = Field(title="type")
    name: str = Field(title="name")
    columns: Sequence["Columns"]
    description: Optional[str] = Field(default=None)
    skip: Optional[bool] = Field(default=False)
    required: Optional[bool] = Field(default=True)


Columns = Annotated[MultiValuesColumn | JsonColumn, Field(title="columns")]
MultiValuesColumn.update_forward_refs()


class JsonFileFormat(BaseFormat):
    """Json file format"""
    kind: Literal["JsonFormat"] = "JsonFormat"
    columns: Sequence[Columns]
    encoding: Optional[str] = Field(default="UTF-8")
    additional_properties: Optional[bool] = Field(default=True)
    date_test: Optional[str] = Field(default=None)
    date_format: Optional[str] = Field(default=None)
    timestamp_test: Optional[str] = Field(default=None)
    timestamp_format: Optional[str] = Field(default=None)
    timezone: Optional[str] = Field(default=None)


def ignored_settings(format: JsonFileFormat, supported_settings: list[str]):
    """Return names of settings that are specified in a JSON format but not used in the parser"""
    schema = JsonFileFormat.schema()
    ignored = schema["required"] + ["kind", "title", "description"] + supported_settings
    return [k for k, v in format.model_dump().items() if k not in ignored and v is not None]
```

---

### ###### src/azfr_parsing_utils/json/common.py ######

```python
import logging
from azfr_parsing_utils.json.format import JsonFileFormat, ignored_settings

log = logging.getLogger(__name__)
SUPPORTED_SETTINGS = ["encoding", "null_values", "true_values", "false_values", "date_test", "date_format", "timestamp_test", "timestamp_format", "timezone", "additional_properties"]

def unsupported_format_options(format: JsonFileFormat) -> list[str]:
    """Return names of options that are set in a given JsonFileFormat but not supported."""
    not_supported = ignored_settings(format, SUPPORTED_SETTINGS)
    if not_supported:
        log.warning("Unsupported settings: {}".format(not_supported))
    return not_supported

DEFAULT_BLOCK_SIZE = 10 * 1024 * 1024  # 10MB
```

---

### ###### src/azfr_parsing_utils/json/parser.py ######

> Key exports: `is_requiring_processing`, `is_data_type_require_processing`, `map_to_pa_schema`, `cast_value`, `need_cast`, `JsonParser`.

`JsonParser` reads NDJSON files line-by-line, validates each line against a JSON Schema (Draft2012), casts Decimal/Map/List values, and yields `RecordBatch` objects.

*(Full source: 259 lines — identical to content read above)*

---

### ###### src/azfr_parsing_utils/json/utils.py ######

```python
# Key exports: parse_json_format_col, parse_json_file_format, get_tables_to_yaml,
#              get_format_to_yaml, convert_to_json_schema, convert_to_json_format
# convert_to_json_schema builds a Draft-2020-12 JSON Schema dict from JsonFileFormat columns,
# handling map/struct/list nested types recursively.
# convert_to_json_format maps SQL type names to JSON Schema type strings.
```

*(Full source: 149 lines — identical to content read above)*

---

### ###### src/azfr_parsing_utils/json/polars.py ######

> Key exports: `get_column_parser`, `map_custom_parsing_to_polars_exp`, `has_col_custom_parsing`, `parsed_json_data` context manager.

Handles nested map/struct/list columns recursively via `pl.element()` / `pl.struct()` / `col.list.eval()`.

*(Full source: 368 lines — identical to content read above)*

---

### ###### src/azfr_parsing_utils/json/pyspark.py ######

> Key exports: `map_column_to_pyspark_type`, `map_to_pyspark_schema`, `map_to_pyspark_read_options_dict`, `_map_nested_column_parsing`, `map_custom_parsing_to_pyspark_column`, `parsed_json_data(spark_session, source, format, …) → (DataFrame, DataFrame)`.

*(Full source: 432 lines — identical to content read above)*

---

## Source — model_generator

### ###### src/azfr_parsing_utils/model_generator/cli/generate_model.py ######

```python
# CLI command: generate_model -f <format_folder> -o <out_folder>
# Uses datamodel_code_generator to emit Pydantic models from YAML format files.
# Supports CsvFormat, FixedWidthFormat, JsonFormat.
# update_schema_dict() builds a JSON Schema dict from column definitions.
```

*(Full source: 130 lines — identical to content read above)*

---

### ###### src/azfr_parsing_utils/model_generator/pydantic_template/BaseModel.jinja2 ######

```jinja2
class {{ class_name }}({{ base_class }}):
{%- for field in fields %}
    {{ field.name }}: {{ field.type_hint }} = Field(None, name='{{ field.name }}', data_type={{ field.type_hint }}, description={%- if field.extras.description %}'{{ field.extras.description }}'{%- else %}None{%- endif %})
    """
        name : {{ field.name }}
        data_type : {{ field.type_hint }}
        {%- if field.extras.description %}
        description : {{ field.extras.description }}
        {%- endif %}
    """
{%- endfor -%}
```

---

## Source — yaml_generator

### ###### src/azfr_parsing_utils/yaml_generator/__init__.py ######

```python
from .from_mdm import generate_yamls_from_mdm_folder
from .cli import create_mdm_to_yaml_cli

__all__ = ["generate_yamls_from_mdm_folder", "create_mdm_to_yaml_cli"]
```

---

### ###### src/azfr_parsing_utils/yaml_generator/from_mdm.py ######

```python
# Key constants:
KIND_ALIASES = {"csv": "CsvFormat", "json": "JsonFormat", "fw": "FixedWidthFormat"}
SUPPORTED_KINDS = ["csv", "json", "fw"]
DEFAULT_NULL_VALUES = ["", "NaN", "null", "None", "N/A", "NA"]
DEFAULT_TRUE_VALUES = ["Oui", "Yes"]
DEFAULT_FALSE_VALUES = ["Non", "No"]

# Key functions:
# get_default_format(kind) -> dict        — returns default YAML skeleton for a given kind
# default_column_mapping(mdm_row) -> dict — maps NAME/COMMENTAIRE/TYPE/FORMAT columns
# parse_mdm_file(mdm_path, kind, …) -> dict              — converts one .mdm → YAML dict
# generate_yamls_from_mdm_folder(folder, out, kind, …)   — batch converts all .mdm files
```

*(Full source: 259 lines — identical to content read above)*

---

### ###### src/azfr_parsing_utils/yaml_generator/cli.py ######

```python
# create_mdm_to_yaml_cli(column_mapping, delimiter) -> click.Command
# Factory that returns a @click.command wrapping generate_yamls_from_mdm_folder.
# Options: --mdm-folder, --out-folder, --kind (csv|json|fw)
```

*(Full source: 115 lines — identical to content read above)*

---

## Tests — helpers & data

### ###### test/utils.py ######

```python
import inspect, os, shutil
import pyarrow as pa

test_root_dir = os.path.join(os.path.dirname(os.path.dirname(__file__)), "target")
sample_file_root_dir = os.path.join(os.path.dirname(__file__), "test_data")

def create_temp_dir():
    """Create a per-test temp directory under target/."""
    frame = inspect.getouterframes(inspect.currentframe())[1]
    fn = os.path.splitext(os.path.basename(frame.filename))[0]
    path = os.path.join(test_root_dir, "{}_{}".format(fn, frame.function))
    if os.path.exists(path): shutil.rmtree(path)
    os.makedirs(path)
    return path

sample_schema = pa.schema([
    pa.field("col1", pa.string()),
    pa.field("col2", pa.int32()),
    pa.field("col3", pa.float64()),
    pa.field("col4", pa.string()),
])
sample_data = [
    {"col1": "a", "col2": 1, "col3": 1.2, "col4": "M"},
    {"col1": "b", "col2": 2, "col3": 3.4, "col4": "F"},
    {"col1": "c", "col2": 3, "col3": 5.6, "col4": "M"},
    {"col1": "d", "col2": 4, "col3": 7.8, "col4": "F"},
    {"col1": "e", "col2": 5, "col3": 9.0, "col4": "M"},
]
sample_table = pa.Table.from_pylist(sample_data, sample_schema)
```

---

### ###### test/test_data/sample_csv_format.yaml ######

```yaml
kind: CsvFormat
name: sample-csv-format
title: Sample CSV format (one-line title)
description: |
  This is a long description of the sample CSV file format file.
sep: ","
encoding: UTF-8
quote: "\""
escape: "\""
header: true
null_values: [ "", "NaN" ]
true_values: [ "Oui", "Yes" ]
false_values: [ "Non", "No" ]
multi_line: false
columns:
  - name: customer_name
    description: Customer name
    data_type: STRING
  - name: comment
    description: Comment
    data_type: STRING
    skip: true
  - name: contact_time
    description: Contact time
    data_type: TIMESTAMP
    format: abs_local_timestamp
```

---

### ###### test/test_data/sample_fixed_width_format.yaml ######

```yaml
kind: FixedWidthFormat
name: test
title: Test Schema
description: |
  This is a test schema
header: false
true_values:
  - Oui
false_values:
  - Non
timezone: Paris/Europe
columns:
  - name: col1
    data_type: STRING
    start_index: 1
    end_index: 5
```

---

### ###### test/test_data/sample_json_format.yaml ######

```yaml
kind: JsonFormat
name: sample-json-format
title: Sample json format (one-line title)
description: |
  This is a long description of the sample Json file format file.
columns:
  - name: customer_name
    description: Customer name
    data_type: STRING
  - name: comment
    description: Comment
    data_type: STRING
    skip: true
  - name: contact_time
    description: Contact time
    data_type: TIMESTAMP
    format: abs_local_timestamp
```

---

### ###### test/test_data/TABLENAME.mdm ######

```
NAME;TYPE;FORMAT;FORMATD;PK;COMMENTAIRE
COLUMN1;INTEGER;;;True;Commentaire 1
COLUMN2;INTEGER;;;False;Commentaire 2
COLUMN3;INTEGER;;;False;Commentaire 3
```

---

### ###### test/test_data/sample1.csv ######

```
col1,col2,col3
a,1,1.2
b,2,3.4
c,3,5.6
d,4,7.8
e,5,9.0
```

---

### ###### test/test_data/sample1.json ######

```json
{"col1": "a", "col2": 1, "col3": 1.2}
{"col1": "b", "col2": 2, "col3": 3.4}
{"col1": "c", "col2": 3, "col3": 5.6}
{"col1": "d", "col2": 4, "col3": 7.8}
{"col1": "e", "col2": 5, "col3": 9.0}
```

---

*End of file. Total source modules: 30+ Python files, 3 format YAMLs, 2 JSON Schemas, 1 Jinja2 template, 1 MDM sample, CI pipeline, Makefile.*
