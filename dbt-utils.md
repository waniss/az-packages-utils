# azfr-dbt-utils

## Overview

This document was reconstructed from the provided repository dump.

### Purpose

`azfr-dbt-utils` is a Python utility library extending **dbt-trino** to support
custom Azure authentication mechanisms used internally.

## Main Features

- Custom `TrinoCredentials` implementation
- Azure token authentication
- Runtime patching of `dbt-trino`
- CLI utilities:
  - `patch_dbt_trino`
  - `get_token`

## Repository Structure

```text
.github/workflows/
src/
 └── azfr_dbt_utils/
      ├── adapter.py
      ├── patch.py
      ├── token.py
README.md
setup.py
Makefile
```

## Components

### adapter.py

Provides `TrinoAzfrDefaultCredentials`, extending the native dbt-trino
credentials to authenticate using Azure credentials.

### patch.py

Adds support for a custom credentials factory inside dbt-trino by patching the
installed package.

### token.py

CLI helper returning Azure access tokens using either:

- Azure CLI
- Workload Identity

## Build

The project uses:

- GitHub Actions
- Conda virtual environment
- Makefile
- Twine for publication
- Internal PyPI repository

## Packaging

Python package:

- name: azfr-dbt-utils
- Python >= 3.10

Dependencies include:

- azfr-azure-utils[trino]
- click
- dbt-trino (development)

## Configuration

The README documents how to configure `profiles.yml` with:

- method: custom
- credentials_class:
  azfr_dbt_utils.adapter.TrinoAzfrDefaultCredentials
- scope
- optional SSL certificate configuration.

---
Generated from the uploaded repository dump.
