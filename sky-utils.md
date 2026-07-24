# Code dump: azfr-skywalker-utils

Generated from `/c/Users/LARBANI/repos_git/azfr-skywalker-utils`.

## File tree

```
.github/workflows/build.yaml
.gitignore
activate_venv
constraints.txt
dbt-macros/dbt_project.yml
dbt-macros/macros/metadata/get_common_vars.sql
dbt-macros/macros/metadata/get_create_table_str.sql
dbt-macros/macros/metadata/helpers.sql
dbt-macros/macros/metadata/parse_dbt_results.sql
dbt-macros/macros/metadata/save_dbt_invocation.sql
dbt-macros/macros/metadata/save_dbt_invocation_details.sql
dbt-macros/macros/metadata/save_dbt_invocation_start_event.sql
ensure_conda
Makefile
pip.conf
README.md
setup.cfg
setup.py
src/azfr_skywalker_utils/__init__.py
src/azfr_skywalker_utils/dbt/__init__.py
src/azfr_skywalker_utils/dbt/common.py
src/azfr_skywalker_utils/ibis/__init__.py
src/azfr_skywalker_utils/ibis/trino.py
src/azfr_skywalker_utils/metadata/__init__.py
src/azfr_skywalker_utils/metadata/parsing/__init__.py
src/azfr_skywalker_utils/metadata/parsing/analyzer.py
src/azfr_skywalker_utils/metadata/parsing/calendar.py
src/azfr_skywalker_utils/metadata/parsing/exception.py
src/azfr_skywalker_utils/metadata/parsing/flow.py
src/azfr_skywalker_utils/metadata/parsing/metadata.py
src/azfr_skywalker_utils/metadata/parsing/workflowconfig.py
src/azfr_skywalker_utils/metadata/workflow_dependency/__init__.py
src/azfr_skywalker_utils/metadata/workflow_dependency/config/__init__.py
src/azfr_skywalker_utils/metadata/workflow_dependency/config/workflow_registry.yml
src/azfr_skywalker_utils/metadata/workflow_dependency/version_provider.py
src/azfr_skywalker_utils/metadata/workflow_dependency/workflow_registry.py
src/azfr_skywalker_utils/metadata/workflow_run/__init__.py
src/azfr_skywalker_utils/metadata/workflow_run/abstract.py
src/azfr_skywalker_utils/metadata/workflow_run/datavault.py
src/azfr_skywalker_utils/metadata/workflow_run/extraction.py
src/azfr_skywalker_utils/trino/__init__.py
src/azfr_skywalker_utils/trino/common.py
src/azfr_skywalker_utils/trino/dbapi.py
src/azfr_skywalker_utils/trino/sql_alchemy.py
src/azfr_skywalker_utils/utils/__init__.py
src/azfr_skywalker_utils/utils/archive.py
src/azfr_skywalker_utils/utils/date.py
src/azfr_skywalker_utils/utils/file.py
src/azfr_skywalker_utils/utils/format_extension.py
src/azfr_skywalker_utils/utils/helpers.py
src/azfr_skywalker_utils/utils/mail.py
src/azfr_skywalker_utils/utils/sql.py
test/__init__.py
test/metadata/__init__.py
test/metadata/parsing/__init__.py
test/metadata/parsing/analyzer/__init__.py
test/metadata/parsing/analyzer/test_analyze_landing_files.py
test/metadata/parsing/analyzer/test_analyze_parse_table.py
test/metadata/parsing/analyzer/test_analyzer_validation.py
test/metadata/parsing/analyzer/test_raise_error_if_delta_and_empty.py
test/metadata/parsing/analyzer/test_raise_error_if_full_and_empty.py
test/metadata/parsing/analyzer/test_register_missing_files.py
test/metadata/parsing/archive_analyzers/__init__.py
test/metadata/parsing/archive_analyzers/test_tar_analyzer.py
test/metadata/parsing/archive_analyzers/test_zip_analyzer.py
test/metadata/parsing/calendar/__init__.py
test/metadata/parsing/calendar/test_calendar.py
test/metadata/parsing/conftest.py
test/metadata/parsing/helpers.py
test/metadata/parsing/workflow_metadata/__init__.py
test/metadata/parsing/workflow_metadata/test_workflow_metadata.py
test/metadata/workflow_dependency/scenarios/end_to_end_scenarios.md
test/metadata/workflow_dependency/version_provider/__init__.py
test/metadata/workflow_dependency/version_provider/conftest.py
test/metadata/workflow_dependency/version_provider/get_versions_auto/__init__.py
test/metadata/workflow_dependency/version_provider/get_versions_auto/test_get_versions_auto.py
test/metadata/workflow_dependency/version_provider/providers/__init__.py
test/metadata/workflow_dependency/version_provider/providers/test_create_provider.py
test/metadata/workflow_dependency/version_provider/providers/test_datavault_version_provider.py
test/metadata/workflow_dependency/version_provider/providers/test_extraction_version_provider.py
test/metadata/workflow_dependency/version_provider/providers/test_parsing_version_provider.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/__init__.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_collect_dependencies_versions.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_find_valid_versions_across_dependencies.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_get_all_successful_versions.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_optional_dependencies.py
test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_unique_versions.py
```

## File contents

###### FILE: .github/workflows/build.yaml ######

```yaml
name: Build Python library

env:
  # Run tests
  RUN_TESTS: "true"
  # Build and publish Python library
  PUBLISH_ARTIFACTS: "true"
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
        required: false
        type: choice
        options: [ "true", "false" ]
      publish_artifacts:
        description: "Publish Python module"
        required: false
        type: choice
        options: [ "true", "false" ]
      disable_cache_read:
        description: "Not use cache data"
        required: true
        type: choice
        options: [ "true", "false" ]
        default: "false"
  push: { }

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:

  build:
    name: Test, build and publish the workflow
    runs-on: [ self-hosted ]
    container:
      image: prodazfrz6sh.azurecr.io/cicd-job:py312
      volumes:
        - /var/cache/gha:/var/cache/gha

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: azf-h1-datascience/use-cache@release
        with:
          pip: true
          venv: true
          sonar: true
          disable_cache_read: ${{ inputs.disable_cache_read == 'true' }}

      - name: Test
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        run: |
          make update reports

      - name: Run SonarScanner for dbt-macros
        uses: azf-h1-datascience/sonar-scanner/configure@release
        with:
          host_url: ${{ secrets.SONAR_HOST_URL }}
          token: ${{ secrets.SONAR_TOKEN }}
          module: dbt-macros
          sources: dbt-macros/macros
          params: |
            sonar.python.version: 3
            sonar.python.coverage.reportPaths: target/report/coverage.xml
            sonar.exclusions: "**/*.yaml,**/*.yml"

      - name: Run SonarScanner for python code
        uses: azf-h1-datascience/sonar-scanner/run@release
        with:
          host_url: ${{ secrets.SONAR_HOST_URL }}
          token: ${{ secrets.SONAR_TOKEN }}
          module: python
          sources: src/azfr_skywalker_utils
          tests: test
          params: |
            sonar.python.version: 3
            sonar.python.coverage.reportPaths: target/report/coverage.xml
            sonar.exclusions: "**/*.yaml,**/*.yml"

      - name: Publish Python module
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        run: |
          make deploy PYPI_REPO_URL=$AZFR_PYPI_INTERNAL_REPO \
                      PYPI_REPO_USER=$AZFR_CI_USERNAME \
                      PYPI_REPO_PASS=$AZFR_CI_PASSWORD

#      - name: Publish documentation
#        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
#        run: |
#          make deploy-docs DOC_REPO_ROOT=$AZFR_RAW_DOC_ROOT \
#                           DOC_REPO_USER=$AZFR_CI_USERNAME \
#                           DOC_REPO_PASS=$AZFR_CI_PASSWORD

```

###### FILE: .gitignore ######

```gitignore
### JetBrains template
# Covers JetBrains IDEs: IntelliJ, RubyMine, PhpStorm, AppCode, PyCharm, CLion, Android Studio, WebStorm and Rider
# Reference: https://intellij-support.jetbrains.com/hc/en-us/articles/206544839

# User-specific stuff
.idea/**/workspace.xml
.idea/**/tasks.xml
.idea/**/usage.statistics.xml
.idea/**/dictionaries
.idea/**/shelf

# AWS User-specific
.idea/**/aws.xml

# Generated files
.idea/**/contentModel.xml

# Sensitive or high-churn files
.idea/**/dataSources/
.idea/**/dataSources.ids
.idea/**/dataSources.local.xml
.idea/**/sqlDataSources.xml
.idea/**/dynamic.xml
.idea/**/uiDesigner.xml
.idea/**/dbnavigator.xml

# Gradle
.idea/**/gradle.xml
.idea/**/libraries

.idea/

# Gradle and Maven with auto-import
# When using Gradle or Maven with auto-import, you should exclude module files,
# since they will be recreated, and may cause churn.  Uncomment if using
# auto-import.
.idea/artifacts
.idea/compiler.xml
.idea/jarRepositories.xml
.idea/modules.xml
.idea/*.iml
.idea/modules
*.iml
*.ipr

# CMake
cmake-build-*/

# Mongo Explorer plugin
.idea/**/mongoSettings.xml

# File-based project format
*.iws

# IntelliJ
out/

# mpeltonen/sbt-idea plugin
.idea_modules/

# JIRA plugin
atlassian-ide-plugin.xml

# Cursive Clojure plugin
.idea/replstate.xml

# SonarLint plugin
.idea/sonarlint/

# Crashlytics plugin (for Android Studio and IntelliJ)
com_crashlytics_export_strings.xml
crashlytics.properties
crashlytics-build.properties
fabric.properties

# Editor-based Rest Client
.idea/httpRequests

# Android studio 3.1+ serialized cache file
.idea/caches/build_file_checksums.ser

### Python template
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
#   https://pdm.fming.dev/#use-with-ide
.pdm.toml

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


```

###### FILE: activate_venv ######

```activate_venv
# Check whether the current python is one in a venv under the current directory
python_is_in_venv() {
  if command -v python >/dev/null; then
    d="$(command -v python)"
    while [ "$d" != "$(pwd)" ]; do
      nd=$(dirname "$d")
      if [ "$d" = "$nd" ]; then
        break
      fi
      d="$nd"
    done
    test "$d" = "$(pwd)"
  else
    false
  fi
}

find_activate() {
  if command -v conda >/dev/null; then
    conda_base=$(conda info --base)
    if [ -f "${conda_base}/bin/activate" ]; then
      echo "${conda_base}/bin/activate"
    elif [ -f "${conda_base}/Scripts/activate" ]; then
      echo "${conda_base}/Scripts/activate"
    fi
  else
    command -v activate
  fi
}

if ! python_is_in_venv; then
  activate=$(find_activate)

  if [ "$activate" != "" ]; then
    . "$activate" ./venv

    if ! python_is_in_venv; then
      echo "Conda environment was not correctly activated."
      false
    fi
  else
    echo "Cannot find conda installation"
    false
  fi
fi

```

###### FILE: constraints.txt ######

```txt
sphinx<7
pydantic>=2
azfr-parsing-utils>=0.2.0
deltalake>=0.16,<1.0
polars<1.30.0

```

###### FILE: dbt-macros/dbt_project.yml ######

```yml
name: 'azfr_skywalker_utils_dbt_macros'
version: '0.1.0'
config-version: 2
require-dbt-version: ["<2.0.0"]
macro-paths: ["macros"]

```

###### FILE: dbt-macros/macros/metadata/get_common_vars.sql ######

```sql
{% macro get_common_vars() %}
    {# Config #}
    {% set create_metadata_tables = var('create_metadata_tables', False) %}

    {# Insert values #}
    {% set workflow_name = var('workflow_name', 'local-run') %}
    {% set run_id = var('run_id', invocation_id) %}
    {% set dataset = var('dataset', 'ADMIN') %}
    {% set version = var('version', none) %}
    {% set run_mode = var('run_mode', 'MANUAL') %}
    {% set task = var('task', 'ADMIN') %}
    {% set start_ts = "TIMESTAMP '" ~ run_started_at ~ "'" %}
    {% set end_ts = "now()" %}
    {% set is_full_refresh = flags.FULL_REFRESH %}
    {% set select_arg = invocation_args_dict.get('select') | join(', ') %}
    {% set invocation_command = invocation_args_dict.get('invocation_command').replace("'", "''") %}
    {% set run_message = var('run_message', '') %}
    {% set github_repository = env_var('GITHUB_REPOSITORY', 'azf-h1-datascience/azfr-skywalker-dv') %}
    {% set github_sha = env_var('GITHUB_SHA', 'local-run') %}


    {% do return({
        'create_metadata_tables': create_metadata_tables,
        'workflow_name': workflow_name,
        'run_id': run_id,
        'invocation_id': invocation_id,
        'dataset': dataset,
        'version': version,
        'run_mode': run_mode,
        'task': task,
        'start_ts': start_ts,
        'end_ts': end_ts,
        'is_full_refresh': is_full_refresh,
        'select_arg': select_arg,
        'invocation_command': invocation_command,
        'run_message': run_message,
        'github_repository': github_repository,
        'github_sha': github_sha,
        'run_message': run_message,
    }) %}
{% endmacro %}

```

###### FILE: dbt-macros/macros/metadata/get_create_table_str.sql ######

```sql
{% macro get_create_table_str(create_metadata_tables, database, schema, table_name, columns) %}

{% if create_metadata_tables %}
  {% set create_table_str %}
  CREATE TABLE IF NOT EXISTS {{ database }}.{{ schema }}.{{ table_name }} (
      {%- for col in columns %}
      {{ col.name }} {{ col.type }}{{ "," if not loop.last else "" }}
      {%- endfor %}
  );
  {% endset %}
{% else %}
  {% set create_table_str = '' %}
{% endif %}

    {% do return(create_table_str) %}

{% endmacro %}
```

###### FILE: dbt-macros/macros/metadata/helpers.sql ######

```sql
{% macro to_trino_ts(ts) %}
    {% if ts %}
        TIMESTAMP '{{ ts.replace("Z","").replace("T"," ") }}'
    {% else %}
        cast(null as timestamp)
    {% endif %}
{% endmacro %}

```

###### FILE: dbt-macros/macros/metadata/parse_dbt_results.sql ######

```sql
{% macro parse_dbt_results(results) %}
	-- Create a list of parsed results
	{%- set parsed_results = [] %}
	-- Flatten results and add to list
	{% for run_result in results %}
		-- Convert the run result object to a simple dictionary
		{% set run_result_dict = run_result.to_dict() %}

		-- Get the underlying dbt graph node that was executed
		{% set node = run_result_dict.get('node') %}
		{% set rows_affected = run_result_dict.get('adapter_response', {}).get('rows_affected', 0) %}
		{% set query_id = run_result_dict.get('adapter_response', {}).get('query_id', none) %}
		{%- if not rows_affected -%}
			{% set rows_affected = 0 %}
		{%- endif -%}

		{#-- Default to empty string in case timing info is missing --#}
		{% set timing_ns = namespace(
			compile_start_ts="cast(null as timestamp)",
			compile_end_ts="cast(null as timestamp)",
			execute_start_ts="cast(null as timestamp)",
			execute_end_ts="cast(null as timestamp)"
		) %}

		{% set timing = run_result_dict.get('timing', []) %}
        {% for timing_phase in timing %}
            {% if timing_phase.get('name', '') == 'compile' %}
                {% set timing_ns.compile_start_ts = azfr_skywalker_utils_dbt_macros.to_trino_ts(timing_phase.get('started_at')) %}
				{% set timing_ns.compile_end_ts = azfr_skywalker_utils_dbt_macros.to_trino_ts(timing_phase.get('completed_at')) %}
            {% elif timing_phase.get('name', '') == 'execute' %}
				{% set timing_ns.execute_start_ts = azfr_skywalker_utils_dbt_macros.to_trino_ts(timing_phase.get('started_at')) %}
				{% set timing_ns.execute_end_ts = azfr_skywalker_utils_dbt_macros.to_trino_ts(timing_phase.get('completed_at')) %}
            {% endif %}
        {% endfor %}

		{% set parsed_result_dict = {
				'unique_id': node.get('unique_id'),
				'database_name': node.get('database'),
				'schema_name': node.get('schema'),
				'name': node.get('name'),
				'resource_type': node.get('resource_type'),
				'status': run_result_dict.get('status'),
				'message': run_result_dict.get('message'),
				'execution_time': run_result_dict.get('execution_time'),
				'rows_affected': rows_affected,
				'query_id': query_id,
				'thread_id': node.get('thread_id'),
				'compile_start_ts': timing_ns.compile_start_ts,
				'compile_end_ts': timing_ns.compile_end_ts,
				'execute_start_ts': timing_ns.execute_start_ts,
				'execute_end_ts': timing_ns.execute_end_ts,
				'tags': node.get('tags', []),
				}%}
		{% do parsed_results.append(parsed_result_dict) %}
	{% endfor %}
	{{ return(parsed_results) }}
{% endmacro %}


```

###### FILE: dbt-macros/macros/metadata/save_dbt_invocation.sql ######

```sql
{% macro save_dbt_invocation(results) %}

{# Precompute values to avoid complex expressions in YAML #}
{% set common_vars = azfr_skywalker_utils_dbt_macros.get_common_vars() %}
{% set all_succeeded = (results | selectattr('status', 'in', ['success', 'pass']) | list | length == results | length) %}
{% set nb_failure = results | selectattr('status', 'in', ['error', 'fail']) | list | length %}
{% set nb_success = results | selectattr('status', 'in', ['success', 'pass']) | list | length %}
{% set total_nb_objects = results | length %}
{% set nb_threads = target.threads %}
{% set dbt_version_str = dbt_version %}

{# YAML block #}
{% set yaml_metadata %}
table_name: _dbt_invocation
columns:
  - name: invocation_id
    type: varchar
    value: "{{ common_vars.invocation_id }}"
  - name: workflow_name
    type: varchar
    value: "{{ common_vars.workflow_name }}"
  - name: run_id
    type: varchar
    value: "{{ common_vars.run_id }}"
  - name: dataset
    type: varchar
    value: "{{ common_vars.dataset }}"
  - name: version
    type: varchar
    value: "{{ common_vars.version }}"
  - name: task
    type: varchar
    value: "{{ common_vars.task }}"
  - name: all_succeeded
    type: boolean
    value: "{{ all_succeeded }}"
  - name: nb_failure
    type: int
    value: "{{ nb_failure }}"
  - name: nb_success
    type: int
    value: "{{ nb_success }}"
  - name: total_nb_objects
    type: int
    value: "{{ total_nb_objects }}"
  - name: start_ts
    type: timestamp(3) with time zone
    value: "{{ common_vars.start_ts }}"
  - name: end_ts
    type: timestamp(3) with time zone
    value: "{{ common_vars.end_ts }}"
  - name: is_full_refresh
    type: boolean
    value: "{{ common_vars.is_full_refresh }}"
  - name: run_mode
    type: varchar
    value: "{{ common_vars.run_mode }}"
  - name: select_arg
    type: varchar
    value: "{{ common_vars.select_arg }}"
  - name: invocation_command
    type: varchar
    value: '{{ common_vars.invocation_command }}' {# enclosed by single-quote because value has double-quotes #}
  - name: nb_threads
    type: int
    value: "{{ nb_threads }}"
  - name: dbt_version
    type: varchar
    value: "{{ dbt_version_str }}"
  - name: run_message
    type: varchar
    value: "{{ common_vars.run_message }}"
{% endset %}

{# Parse YAML #}
{% set metadata = fromyaml(yaml_metadata) %}
{% set table_name = metadata['table_name'] %}
{% set columns = metadata['columns'] %}

{# CREATE TABLE #}
{% set create_table_str = azfr_skywalker_utils_dbt_macros.get_create_table_str(common_vars.create_metadata_tables, target.database, target.schema, table_name, columns) %}

{# INSERT #}
{% set insert_columns = columns | map(attribute='name') | join(', ') %}

{% set formatted_values = [] %}
{% for col in columns %}
    {% if col.type.lower() in ['varchar', 'string', 'text'] %}
        {% do formatted_values.append("'" ~ col.value ~ "'") %}
    {% elif 'timestamp' in col.type.lower() %}
        {% do formatted_values.append(col.value) %}
    {% else %}
        {% do formatted_values.append(col.value) %}
    {% endif %}
{% endfor %}

{% set insert_values = formatted_values | join(',\n    ') %}

{% set insert_statement_str %}
INSERT INTO {{ target.database }}.{{ target.schema }}.{{ table_name }} (
    {{ insert_columns }}
) VALUES (
    {{ insert_values }}
);
{% endset %}

{{ return(create_table_str + insert_statement_str) }}

{% endmacro %}

```

###### FILE: dbt-macros/macros/metadata/save_dbt_invocation_details.sql ######

```sql
{% macro save_dbt_invocation_details(results) %}

{% set yaml_metadata %}
table_name: _dbt_invocation_details
columns:
  - name: id
    type: varchar
  - name: invocation_id
    type: varchar
  - name: node_id
    type: varchar
  - name: workflow_name
    type: varchar
  - name: run_id
    type: varchar
  - name: dataset
    type: varchar
  - name: version
    type: varchar
  - name: task
    type: varchar
  - name: is_full_refresh
    type: boolean
  - name: resource_type
    type: varchar
  - name: node_name
    type: varchar
  - name: resource_tags
    type: varchar
  - name: status
    type: varchar
  - name: message
    type: varchar
  - name: rows_affected
    type: int
  - name: execution_time_seconds
    type: int
  - name: compile_start_ts
    type: timestamp(3) with time zone
  - name: compile_end_ts
    type: timestamp(3) with time zone
  - name: execute_start_ts
    type: timestamp(3) with time zone
  - name: execute_end_ts
    type: timestamp(3) with time zone
  - name: trino_query_id
    type: varchar

{% endset %}

{% set metadata = fromyaml(yaml_metadata) %}
{% set table_name = metadata['table_name'] %}
{% set columns = metadata['columns'] %}
{% set common_vars = azfr_skywalker_utils_dbt_macros.get_common_vars() %}

{# CREATE TABLE #}
{% set create_table_str = azfr_skywalker_utils_dbt_macros.get_create_table_str(common_vars.create_metadata_tables, target.database, target.schema, table_name, columns) %}

{# INSERT VALUES LOOP #}
{% set parsed_results = azfr_skywalker_utils_dbt_macros.parse_dbt_results(results) %}
{% if parsed_results | length > 0 %}
  {% set insert_rows = [] %}
  {% for parsed_result in parsed_results %}

    {% set id = local_md5(common_vars.invocation_id ~ '-' ~ parsed_result.get('unique_id') ~ '-' ~ common_vars.workflow_name) %}
    {% set values = [
      id,
      common_vars.invocation_id,
      parsed_result.get('unique_id'),
      common_vars.workflow_name,
      common_vars.run_id,
      common_vars.dataset,
      common_vars.version,
      common_vars.task,
      common_vars.is_full_refresh,
      parsed_result.get('resource_type'),
      parsed_result.get('name'),
      parsed_result.get('tags') | join(', '),
      parsed_result.get('status'),
      parsed_result.get('message') | default('') | replace("'", "''"),
      parsed_result.get('rows_affected'),
      parsed_result.get('execution_time'),
      parsed_result.get('compile_start_ts'),
      parsed_result.get('compile_end_ts'),
      parsed_result.get('execute_start_ts'),
      parsed_result.get('execute_end_ts'),
      parsed_result.get('query_id'),
    ] %}

    {% set formatted_values = [] %}
    {% for i in range(columns | length) %}
      {% set col = columns[i] %}
      {% set val = values[i] %}
      {% if col.type.lower() in ['varchar'] %}
        {% do formatted_values.append("'" ~ val ~ "'") %}
      {% elif 'timestamp' in col.type.lower() %}
        {% do formatted_values.append(val) %}
      {% else %}
        {% do formatted_values.append(val) %}
      {% endif %}
    {% endfor %}
    {% do insert_rows.append("(" ~ formatted_values | join(', ') ~ ")") %}
  {% endfor %}

  {# FINAL INSERT QUERY #}
  {% set insert_statement_str %}
  INSERT INTO {{ target.database }}.{{ target.schema }}.{{ table_name }} (
    {{ columns | map(attribute='name') | join(', ') }}
  ) VALUES
  {{ insert_rows | join(',\n  ') }};
  {% endset %}

{% else %}
  {% set insert_statement_str = '' %}
{% endif %}

  {{ return(create_table_str + insert_statement_str) }}

{% endmacro %}

```

###### FILE: dbt-macros/macros/metadata/save_dbt_invocation_start_event.sql ######

```sql
{% macro save_dbt_invocation_start_event() %}

{# Precompute values to avoid complex expressions in YAML #}
{% set common_vars = azfr_skywalker_utils_dbt_macros.get_common_vars() %}

{# YAML block #}
{% set yaml_metadata -%}
table_name: _dbt_invocation_start_event
columns:
  - name: invocation_id
    type: varchar
    value: "{{ common_vars.invocation_id }}"
  - name: workflow_name
    type: varchar
    value: "{{ common_vars.workflow_name }}"
  - name: run_id
    type: varchar
    value: "{{ common_vars.run_id }}"
  - name: dataset
    type: varchar
    value: "{{ common_vars.dataset }}"
  - name: version
    type: varchar
    value: "{{ common_vars.version }}"
  - name: task
    type: varchar
    value: "{{ common_vars.task }}"
  - name: start_ts
    type: timestamp(3) with time zone
    value: "{{ common_vars.start_ts }}"
  - name: is_full_refresh
    type: boolean
    value: "{{ common_vars.is_full_refresh }}"
  - name: run_mode
    type: varchar
    value: "{{ common_vars.run_mode }}"
  - name: select_arg
    type: varchar
    value: "{{ common_vars.select_arg }}"
  - name: invocation_command
    type: varchar
    value: '{{ common_vars.invocation_command }}' {# enclosed by single-quote because value has double-quotes #}
  - name: run_message
    type: varchar
    value: "{{ common_vars.run_message }}"
  - name: creator_github_repository
    type: varchar
    value: "{{ common_vars.github_repository }}"
  - name: creator_commit_id
    type: varchar
    value: "{{ common_vars.github_sha }}"
{% endset %}

{% set metadata = fromyaml(yaml_metadata) %}
{% set table_name = metadata['table_name'] %}
{% set columns = metadata['columns'] %}

{# CREATE TABLE #}
{% set create_table_str = azfr_skywalker_utils_dbt_macros.get_create_table_str(common_vars.create_metadata_tables, target.database, target.schema, table_name, columns) %}

{# INSERT #}
{% set insert_columns = columns | map(attribute='name') | join(', ') %}

{# Format values with quotes for strings #}
{% set formatted_values = [] %}
{% for col in columns %}
    {% if col.type.lower() in ['varchar'] %}
        {% do formatted_values.append("'" ~ col.value ~ "'") %}
    {% elif 'timestamp' in col.type.lower() %}
        {% do formatted_values.append(col.value) %}
    {% else %}
        {% do formatted_values.append(col.value) %}
    {% endif %}
{% endfor %}

{% set insert_values = formatted_values | join(',\n    ') %}

{% set insert_statement_str %}
INSERT INTO {{ target.database }}.{{ target.schema }}.{{ table_name }} (
    {{ insert_columns }}
) VALUES (
    {{ insert_values }}
);
{% endset %}

{{ return(create_table_str + insert_statement_str) }}

{% endmacro %}

```

###### FILE: ensure_conda ######

```ensure_conda
if ! command -v conda > /dev/null; then
  if ! command -v activate > /dev/null; then
    echo "Neither 'conda' nor 'activate' was found in PATH."
    false
  else
    . activate
    if ! command -v conda > /dev/null; then
      echo "Conda environment was not correctly activated."
      false
    fi
  fi
fi

```

###### FILE: Makefile ######

```
SHELL = /bin/bash
.PHONY: all update test coverage cov tox reports clean-reports docs clean distclean

all: info

info:  ## Show this infomation
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

PYTHON_VERSION=3.12

REPORT_OUTPUT_DIR = ./target/report
DOCUMENT_OUTPUT_DIR = ./doc/_build

venv:  ## Create virtualenv by Conda
	. ./ensure_conda && conda create -yq -p venv python=$(PYTHON_VERSION)
	cp -f pip.conf venv/pip.conf
	cp -f pip.conf venv/pip.ini
	. ./activate_venv && \
		python -m pip install -U -q -c constraints.txt pip && \
		pip install -U -c constraints.txt -e '.[dev]'

update: venv  ## Update dependencies
	cp -f pip.conf venv/pip.conf
	cp -f pip.conf venv/pip.ini
	. ./activate_venv && pip install -U -c constraints.txt -e '.[dev]'

test: venv  ## Run tests
	. ./activate_venv && pytest $(PYTEST_OPTIONS)

coverage: venv  ## Run tests and compute test coverage
	. ./activate_venv && \
		coverage run -m pytest --durations=0 $(PYTEST_OPTIONS) && \
		coverage report

cov: coverage

reports: clean-reports  ## Create coverage report
	. ./activate_venv && \
		coverage run -m pytest \
			--durations=0 \
			--html=$(REPORT_OUTPUT_DIR)/test/pytests.html --self-contained-html \
			--junitxml=$(REPORT_OUTPUT_DIR)/test/junit.xml $(PYTEST_OPTIONS) && \
		coverage html -d $(REPORT_OUTPUT_DIR)/coverage_html && \
		coverage xml -o $(REPORT_OUTPUT_DIR)/coverage.xml

clean-reports:  ## Clean reports
	rm -rf $(REPORT_OUTPUT_DIR)

docs: venv clean-docs  ## Generate API documents
	. ./activate_venv && \
		sphinx-apidoc -T -M --separate -o docs src && \
		$(MAKE) -C docs html

clean-docs:  ## Clean API documents
	rm -rf $(DOCUMENT_OUTPUT_DIR)

deploy: venv  ## Deploy library packages
	. ./activate_venv && \
		pip install -q twine wheel && \
		python setup.py sdist bdist_wheel && \
		twine upload \
			--repository-url $(PYPI_REPO_URL) \
			--username $(PYPI_REPO_USER) \
			--password $(PYPI_REPO_PASS) \
			dist/*

deploy-docs: docs  ## Deploy API documents
	. ./activate_venv && \
		azfr-document-deploy \
			-i docs/_build/html \
			-o $(DOC_REPO_ROOT) \
			-n `python setup.py --name` \
			-u $(DOC_REPO_USER) \
			-p $(DOC_REPO_PASS) \
			-c /etc/ssl/certs/ca-certificates.crt

clean: clean-reports clean-docs  ## Clean artifacts and temporary files
	find . -name '*.pyc' -exec rm -f {} +
	find . -name '*.pyo' -exec rm -f {} +
	find . -name '*~' -exec rm -f {} +
	rm -rf .coverage

distclean: clean  ## Clean all files including virtualenv
	rm -rf .pytest_cache .tox .eggs ./src/*.egg-info build dist
	rm -rf venv
```

###### FILE: pip.conf ######

```conf
[global]
index = https://nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz/repository/pypi-public/pypi
index-url = https://nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz/repository/pypi-public/simple
trusted-host = nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz
```

###### FILE: README.md ######

```md
# azfr-skywalker-utils

Comprehensive utility library for monitoring, metadata management, and workflow orchestration in data pipeline projects. Designed for the Allianz France Skywalker data platform ecosystem.

## Overview

This package provides a robust set of tools for:
- **Metadata Management**: Track workflow executions, file processing, and table transformations
- **Workflow Orchestration**: Manage complex dependencies between data pipelines with version tracking
- **File Analysis**: Monitor data ingestion with customizable analyzers
- **Database Integration**: Trino/Ibis utilities for data warehouse operations
- **DBT Utilities**: Macros and helpers for DBT (data build tool) projects

## Key Features

### 📊 Metadata Management
- **Workflow Tracking**: Record and query workflow runs with comprehensive status tracking
- **File Status Monitoring**: Track file ingestion lifecycle from landing to processing
- **Table Status Management**: Monitor data transformations and quality metrics
- **Event Logging**: Capture workflow events for debugging and auditing
- **Lineage Tracking**: Trace data flow across parsing, datavault, and extraction layers

### 🔗 Workflow Dependencies
- **YAML-Based Configuration**: Define dependencies in human-readable YAML format
- **Version Resolution**: Automatically determine valid versions across dependencies
- **Optional Dependencies**: Flexible dependency handling:
  - Workflows proceed if optional dependencies are delayed
  - Workflows block if optional dependencies fail
  - At least one optional dependency must be present
- **Version Strategies**:
  - `ENSURE_ORDER`: Sequential processing, stops at first failure
  - `ALL_AVAILABLE`: Process all successful versions, skip failures
- **Dependency Types**: Support for required and optional dependencies

### 📁 File Analysis
- **Multiple Archive Formats**: Built-in support for tar, zip, and custom formats
- **Extensible Analyzers**: Create custom analyzers by inheriting from base `Analyzer` class
- **Pattern Matching**: Flexible file pattern recognition with regex support
- **Missing File Detection**: Automatically identify delayed or missing files
- **Overdue Tracking**: Configure time-based alerting for expected files

### 🗄️ Database Utilities

#### Trino Integration
- **Connection Management**: Simplified Trino connection handling with context managers
- **DBAPI Support**: Full Python DB-API 2.0 compatibility
- **Authentication**: Azure Active Directory token-based authentication
- **Configuration**: Pydantic-based configuration with environment variable support

#### Ibis Integration
- **High-Level API**: Use Ibis for expressive data transformations
- **Trino Backend**: Optimized Ibis backend for Trino
- **Type Safety**: Strong typing with Ibis expression system

### 🛠️ DBT Macros
- **Custom Macros**: Reusable Jinja2 macros for DBT projects
- **Trino Optimizations**: Macros optimized for Trino SQL dialect
- **Metadata Integration**: Seamless integration with metadata tracking

## Installation

### Basic Installation

```shell
pip install azfr-skywalker-utils
```

### Development Installation

For development with testing and documentation tools:

```shell
pip install azfr-skywalker-utils
```

### Requirements

- **Python**: >= 3.8
- **Key Dependencies**:
  - `pydantic >= 2.0` - Data validation
  - `polars` - Fast dataframe operations
  - `pyarrow` - Apache Arrow support
  - `deltalake` - Delta Lake table format
  - `trino[sqlalchemy]` - Trino database connectivity
  - `ibis-framework[trino]` - Ibis with Trino backend
  - `workalendar` - Working days calculation

## Quick Start

### 1. Basic Metadata Tracking

Track workflow execution and file processing:

```python
from azfr_skywalker_utils.metadata import (
    WorkflowMetadata, 
    FileStatus, 
    FileDetailedStatus,
    TableStatus,
    TableDetailedStatus
)
from azfr_skywalker_utils.utils import get_now_UTC

# Initialize metadata tracker
metadata = WorkflowMetadata()

# Start workflow with configuration
config = {
    "metadata_dir": "path/to/metadata",
    "workflow_name": "MY_WORKFLOW",
}
metadata.start(config)

# Log events
metadata.write_event("PROCESSING_STARTED", '{"file": "data.csv"}')

# Track file status
metadata.write_file_status(
    file_identifier="data_file",
    date="20241025",
    status=FileStatus.SUCCESS,
    detailed_status=FileDetailedStatus.FILE_SUCCESS,
    error_message=None,
    tables=["table1", "table2"],
    timestamp=metadata.run_start_ts
)

# Track table status
metadata.write_table_status(
    file_identifier="data_file",
    date="20241025",
    table_name="table1",
    status=TableStatus.SUCCESS,
    detailed_status=TableDetailedStatus.TABLE_SUCCESS,
    rows_inserted=1000,
    rows_rejected=0,
    message="Processing completed",
    file_path="path/to/data.csv",
    timestamp=get_now_UTC()
)
```

### 2. Workflow Dependencies

Define and manage workflow dependencies:

**workflow_registry.yml**:
```yaml
workflows:
  datavault.customers:
    layer: datavault
    depends_on:
      - parsing.customer_data
  
  datavault.orders:
    layer: datavault
    depends_on:
      - parsing.order_data
      - datavault.customers  # Required dependency
  
  extraction.customer_report:
    layer: extraction
    depends_on:
      - datavault.customers
      - { datavault.orders: { optional: true } }  # Optional dependency
```

**Python code**:
```python
from azfr_skywalker_utils.metadata.workflow_dependency import (
    WorkflowRegistry,
    WorkflowVersionProvider
)

# Load workflow configuration
registry = WorkflowRegistry.from_yaml("workflow_registry.yml")

# Get valid versions for a workflow
version_provider = WorkflowVersionProvider(
    metadata_path="path/to/metadata",
    workflow_registry=registry
)

versions = version_provider.get_valid_versions("extraction.customer_report")
# Returns: ["20241025", "20241024"] if dependencies are satisfied
```

### 3. File Analysis with Built-in Analyzers

Monitor and analyze incoming files:


### 3. File Analysis with Built-in Analyzers

Monitor and analyze incoming files:

```python
import re
import logging
from azfr_skywalker_utils.metadata import WorkflowBaseConfig, WorkflowMetadata, FileConfig
from azfr_skywalker_utils.analyzer import TarAnalyzer, ZipAnalyzer

# Define workflow configuration
class WorkflowConfig(WorkflowBaseConfig):
    config_A: str
    config_B: str

logger = logging.getLogger(__name__)
workflow_metadata = WorkflowMetadata()

config = WorkflowConfig(**{
    "metadata_dir": "path/to/metadata",
    "workflow_name": "FILE_PROCESSOR",
    "file_pattern": re.compile(
        r"(?P<file_identifier>[a-zA-Z_]+)-(?P<date>\d{8})-(?P<timestamp>\d{14})\.txt"
    ),
    "files_configs": {
        "file_A": FileConfig(
            file_identifier="file_A",
            overdue_time="18:00",
            mode="daily",
            min_expected_date="20241010",
            period_checked=30,
            working_days_only=False,
            tables=["table_1", "table_2", "table_3"]
        ),
        "file_B": FileConfig(
            file_identifier="file_B",
            overdue_time="12:00",
            mode="monthly",
            min_expected_date="20241010",
            period_checked=30,
            working_days_only=False,
            expected_days=[10],  # Expected on 10th of each month
            tables=["table_1", "table_2", "table_3"]
        )
    },
    "config_A": "value_A",
    "config_B": "value_B"
})

# Setup workflow metadata
workflow_metadata.start(config)

# Create analyzer (TarAnalyzer for .tar files, ZipAnalyzer for .zip files)
analyzer = TarAnalyzer(
    workflow_metadata=workflow_metadata,
    config=config,
    logger=logger
)

# Analyze landing files
landing_files = [
    "path/to/file_A-20241010-20241010012325.txt",
    "path/to/file_B-20241010-20241010012326.txt",
    "path/to/file_A-20241012-20241012012325.txt"
]
analyzer.analyze_landing_files(landing_files)

# Process files and track table-level results
file_start_ts = get_now_UTC()
file_path = "full/path/to/file_A-20241025.csv"
tables = ["table_1", "table_2"]
file_identifier = "file_A"
date = "20241025"

# Process each table in the file
for table in tables:
    table_start_ts = get_now_UTC()
    
    # Your parsing logic here (e.g., with Prefect task)
    state = parse(table, return_state=True)
    
    # Analyze and record table parsing results
    analyzer.analyze_parse_table(
        state, file_identifier, table, date, file_path, table_start_ts
    )

# Record file-level results after processing all tables
analyzer.analyze_file_after_parse(file_path, file_start_ts)
```

### 4. Custom File Analyzer

Create a custom analyzer for your specific file format:

```python
from azfr_skywalker_utils.analyzer import Analyzer

class CustomAnalyzer(Analyzer):
    """Custom analyzer for pipe-delimited files."""
    
    def get_tables_from_file(self, file):
        """Extract table names from file content."""
        with open(file) as f:
            content = f.read()
            # Custom logic: tables separated by pipes
            return content.split("|")

# Use your custom analyzer
analyzer = CustomAnalyzer(
    workflow_metadata=workflow_metadata,
    config=config,
    logger=logger
)

analyzer.analyze_landing_files(["path/to/custom_file.dat"])
```

### 5. Trino Database Connection

Connect to Trino for data operations:

```python
from azfr_skywalker_utils.trino.dbapi import trino_connection, TrinoConfig

# Configure Trino connection
config = TrinoConfig(
    host="trino.example.com",
    port=443,
    user="data_engineer",
    catalog="data_lake",
    schema_name="bronze",
    http_scheme="https",
    ssl_verify=True,
    access_token_scope="api://your-app-id/.default"
)

# Use connection context manager
with trino_connection(config) as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM my_table LIMIT 10")
    results = cursor.fetchall()
    
    for row in results:
        print(row)
```

### 6. Ibis for Data Transformations

Use Ibis for high-level, type-safe data operations:

```python
from azfr_skywalker_utils.ibis.trino import ibis_connection
from azfr_skywalker_utils.trino.dbapi import TrinoConfig

config = TrinoConfig(
    host="trino.example.com",
    catalog="data_lake",
    schema_name="silver"
)

with ibis_connection(config) as conn:
    # Get table reference
    customers = conn.table("customers")
    
    # Build query with Ibis
    result = (
        customers
        .filter(customers.country == "FR")
        .select(["customer_id", "name", "email"])
        .limit(100)
    )
    
    # Execute and get pandas DataFrame
    df = result.execute()
    print(df.head())
```

## Advanced Usage

### Optional Dependencies

Optional dependencies allow workflows to be more resilient:

```yaml
workflows:
  extraction.daily_report:
    layer: extraction
    depends_on:
      - datavault.main_data  # Required: blocks if missing or failed
      - { datavault.supplementary: { optional: true } }  # Optional: blocks only if failed
```

**Behavior**:
- If `datavault.supplementary` is **delayed** (not yet run): report proceeds without it
- If `datavault.supplementary` **failed**: report is blocked
- If `datavault.main_data` is **missing/failed**: report is blocked
- At least one optional dependency must be present to proceed

### Version Strategies

Choose the appropriate version strategy for your workflow:

#### ENSURE_ORDER (Default)
```yaml
workflows:
  datavault.customers:
    version_strategy: ENSURE_ORDER
```
- Processes versions sequentially in chronological order
- Stops at the first failure
- Ensures data consistency by maintaining order
- **Use when**: Order matters and you need all previous data

#### ALL_AVAILABLE
```yaml
workflows:
  extraction.report:
    version_strategy: ALL_AVAILABLE
```
- Processes all successful versions independently
- Skips failed versions
- Allows partial processing
- **Use when**: Independent daily loads that don't depend on previous days

### Working with Delta Lake

The package includes Delta Lake support for ACID transactions:

```python
from deltalake import DeltaTable
import polars as pl

# Read Delta table
dt = DeltaTable("path/to/delta/table")
df = dt.to_polars()

# Process with metadata tracking
workflow_metadata.write_table_status(
    file_identifier="delta_load",
    date="20241025",
    table_name="customers_delta",
    status=TableStatus.SUCCESS,
    detailed_status=TableDetailedStatus.TABLE_SUCCESS,
    rows_inserted=len(df),
    rows_rejected=0,
    message="Delta table loaded",
    file_path="path/to/delta/table",
    timestamp=get_now_UTC()
)
```

## Project Structure

```
azfr-skywalker-utils/
├── src/azfr_skywalker_utils/
│   ├── metadata/              # Metadata tracking and workflow management
│   │   ├── workflow_dependency/  # Dependency resolution system
│   │   │   ├── config/          # YAML workflow configurations
│   │   │   ├── workflow_registry.py
│   │   │   ├── version_provider.py
│   │   │   └── version_status.py
│   │   ├── workflow_metadata.py
│   │   └── ...
│   ├── analyzer/              # File analysis utilities
│   │   ├── analyzer.py        # Base analyzer class
│   │   ├── tar_analyzer.py    # TAR archive analyzer
│   │   └── zip_analyzer.py    # ZIP archive analyzer
│   ├── trino/                 # Trino database integration
│   │   ├── dbapi.py           # DB-API 2.0 connection
│   │   └── config.py          # Trino configuration
│   ├── ibis/                  # Ibis high-level API
│   │   └── trino.py           # Ibis Trino backend
│   ├── dbt/                   # DBT macros and utilities
│   └── utils/                 # Common utilities
│       ├── dates.py           # Date handling
│       └── ...
├── test/                      # Comprehensive test suite
│   ├── metadata/
│   ├── analyzer/
│   └── ...
├── dbt-macros/               # DBT Jinja2 macros
├── setup.py                  # Package configuration
└── README.md
```

## Configuration

### Environment Variables

Configure Trino connection using environment variables:

```bash
# Trino Connection
TRINO_HOST=trino.example.com
TRINO_PORT=443
TRINO_CATALOG=data_lake
TRINO_SCHEMA=bronze
TRINO_USER=data_engineer

# Security
TRINO_HTTP_SCHEME=https
TRINO_SSL_VERIFY=true
TRINO_ACCESS_TOKEN_SCOPE=api://app-id/.default

# Metadata
METADATA_DIR=/path/to/metadata
```

### Metadata Directory Structure

The metadata system automatically creates this structure:

```
metadata/
├── workflow_runs/        # Workflow execution records (Parquet)
│   └── workflow_name=MY_WORKFLOW/
│       └── run_date=20241025/
├── file_results/         # File processing status (Parquet)
│   └── workflow_name=MY_WORKFLOW/
│       └── date=20241025/
├── table_results/        # Table transformation status (Parquet)
│   └── workflow_name=MY_WORKFLOW/
│       └── date=20241025/
└── events/              # Event logs (Parquet)
    └── workflow_name=MY_WORKFLOW/
        └── run_date=20241025/
```

## Testing

Run the comprehensive test suite:

```shell
# Run all tests
pytest

# Run with coverage report
pytest --cov=azfr_skywalker_utils --cov-report=html

# Run specific test module
pytest test/metadata/test_workflow_metadata.py

# Run with verbose output
pytest -v

# Run tests matching a pattern
pytest -k "test_optional_dependencies"
```

## Development

### Setup Development Environment

```shell
# Clone repository
git clone https://github.developer.allianz.io/azf-h1-datascience/azfr-skywalker-utils.git
cd azfr-skywalker-utils

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in development mode
pip install -e .[dev]

# Run tests
pytest
```

### Code Quality

The project uses:
- **pytest**: Testing framework
- **coverage**: Code coverage analysis
- **SonarQube**: Static code analysis
- **Type hints**: For better IDE support and type checking

## Documentation

### Available Documentation

- **[Optional Dependencies Guide](OPTIONAL_DEPENDENCIES.md)**: Detailed guide on optional dependency configuration
- **API Documentation**: Auto-generated with Sphinx (in `docs/` directory)
- **Code Examples**: See `test/` directory for comprehensive examples

### Generate API Documentation

```shell
# Install Sphinx
pip install sphinx sphinx-rtd-theme

# Generate docs
cd docs
make html

# View documentation
open _build/html/index.html
```

## Common Use Cases

### Use Case 1: Daily Data Pipeline

```python
# 1. Track file arrival
analyzer.analyze_landing_files(daily_files)

# 2. Check dependencies
versions = version_provider.get_valid_versions("datavault.daily_load")

# 3. Process each version
for version in versions:
    process_daily_load(version)
    
# 4. Record results
metadata.write_file_status(...)
```

### Use Case 2: Monthly Reporting

```yaml
# workflow_registry.yml
workflows:
  extraction.monthly_report:
    layer: extraction
    version_strategy: ALL_AVAILABLE
    depends_on:
      - datavault.transactions
      - { datavault.corrections: { optional: true } }
```

### Use Case 3: Data Quality Checks

```python
# Run quality checks
with trino_connection(config) as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM my_table WHERE date = '20241025'")
    count = cursor.fetchone()[0]

# Record results
metadata.write_table_status(
    file_identifier="quality_check",
    table_name="my_table",
    status=TableStatus.SUCCESS if count > 0 else TableStatus.FAILED,
    rows_inserted=count,
    ...
)
```

## Troubleshooting

### Common Issues

**Issue**: `Import "azfr_skywalker_utils" could not be resolved`
- **Solution**: Ensure package is installed: `pip install azfr-skywalker-utils`

**Issue**: Workflow shows "DELAYED" status
- **Check**: Required dependencies might be missing or not yet executed
- **Solution**: Run dependent workflows first or check `workflow_registry.yml`

**Issue**: Trino connection fails
- **Check**: Environment variables are set correctly
- **Check**: Network connectivity to Trino server
- **Check**: Access token scope is correct

**Issue**: Metadata files not found
- **Check**: `metadata_dir` path is correct and accessible
- **Check**: Write permissions on the directory

## Performance Tips

1. **Batch Operations**: Process multiple files/tables together when possible
2. **Delta Lake**: Use for large datasets requiring ACID transactions
3. **Polars**: Faster than pandas for large dataframes
4. **Connection Pooling**: Reuse Trino connections within a workflow
5. **Metadata Partitioning**: Metadata is automatically partitioned by date for efficient queries

## Security Considerations

- **Credentials**: Never hardcode credentials; use environment variables or Azure Key Vault
- **Token Management**: Access tokens are automatically refreshed
- **SSL/TLS**: Always use HTTPS for Trino connections in production
- **Metadata Access**: Control access to metadata directories with proper permissions

## License

**UNLICENSED** - Allianz France Internal Use Only

This software is proprietary and confidential. Unauthorized copying, distribution, or use is strictly prohibited.

## Authors

**Allianz France AI/Big Data Team**

## Support

For issues, questions, or contributions:

- **Issues**: Create an issue in the GitHub repository
- **Questions**: Contact the data engineering team
- **Documentation**: See [OPTIONAL_DEPENDENCIES.md](OPTIONAL_DEPENDENCIES.md) for specific topics

---

**Version**: Check `setup.py` for current version  
**Last Updated**: February 2026  
**Maintained by**: Allianz France Data Platform Team

## Related Projects

- `azfr-parsing-utils`: Utilities for parsing various file formats
- `azfr-fsspec-utils`: Filesystem abstraction utilities
- `azfr-data-sidp`: SIDP data processing workflows
- `azfr-skywalker-dv`: Data Vault implementation

---

⚠️ **Note**: This package is designed for internal use within the Allianz France data platform ecosystem and requires access to internal Allianz infrastructure.



```

###### FILE: setup.cfg ######

```cfg
[bdist_wheel]
universal=1

[easy_install]
index_url = https://nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz/repository/pypi-public/simple

[tool:pytest]
minversion = 3.0
testpaths = test

[coverage:run]
branch = True
source = src/
```

###### FILE: setup.py ######

```py
from setuptools import setup, find_packages

setup(
    name='azfr-skywalker-utils',
    description='Tools for metadata and monitoring, designed for skywalker projects',
    url='https://github.developer.allianz.io/azf-h1-datascience/azfr-skywalker-utils.git',

    long_description='file: README.md',
    long_description_content_type='text/markdown',

    author='Allianz France AI/Big Data Team',
    license='UNLICENSED',

    package_dir={'': 'src'},
    packages=find_packages(where='src'),
    package_data={
        'azfr_skywalker_utils.metadata.workflow_dependency': ['config/*.yml', 'config/*.yaml'],
    },
    include_package_data=True,

    zip_safe=False,

    python_requires='>=3.8',
    classifiers=[
        'Programming Language :: Python :: 3.8',
        'Programming Language :: Python :: 3.9',
        'Programming Language :: Python :: 3.10',
    ],

    install_requires=[
        'azfr-fsspec-utils',
        'azfr_parsing_utils',
        'workalendar',
        'pyarrow',
        'polars',
        'deltalake',
        'uuid7',
        'pydantic>=2',
        'trino[sqlalchemy]',
        'ibis-framework[trino]'
    ],

    extras_require={
        'dev': [
            'azfr_release_utils',
            'azfr-fsspec-abfs',
            'pytest',
            'pytest-html',
            'coverage',
            'sphinx',
            'myst-parser',
            'prefect[shell]'
        ],
    },

    use_azfr_versioning=True
)

```

###### FILE: src/azfr_skywalker_utils/__init__.py ######

```py
import logging

# Due to difficulties to setup the log level with Prefect, we ensure that the logger is set up here.
# Using a logging.yml file has the drawback that we need to copy/past Prefect base logging configuration from Github
# https://github.com/PrefectHQ/prefect/blob/main/src/prefect/logging/logging.yml
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

```

###### FILE: src/azfr_skywalker_utils/dbt/common.py ######

```py
import json
from enum import Enum
from typing import Optional, Union

from prefect import get_run_logger, task
from prefect_shell import ShellOperation

class RunMode(Enum):
    MANUAL = 'MANUAL'
    AUTO = 'AUTO'

class DbtCommand(Enum):
    RUN = "run"
    TEST = "test"
    SEED = "seed"
    SNAPSHOT = "snapshot"
    SOURCE_FRESHNESS = "source-freshness"
    DOCS_GENERATE = "docs generate"
    CLEAN = "clean"
    DEBUG = "debug"
    DEPS = "deps"

def parse_versions(value: Union[str, list[str], int]) -> list[str]:
    """Parse the input value into a list of version strings.

    :param value: A comma-separated string or a list of strings representing versions. A single integer value is also accepted and converted to string.
    :raises TypeError: If the input is neither a string nor a list of strings nor an int.
    :return: A list of version strings.
    """
    if isinstance(value, int):
        value = str(value)

    if isinstance(value, str):
        value = [item.strip() for item in value.split(',') if item.strip()]
    elif not isinstance(value, list):
        raise TypeError("versions must be a list of strings or a comma-separated string or an int.")
    return value

def prepare_dbt_command(dbt_command: str, dbt_vars: Optional[dict], additional_args: str = "", project_dir: str = "dbt_project") -> str:
    """
    Constructs a DBT command string with the specified parameters.

    Args:
        dbt_command (str): The DBT command to execute (e.g., "run", "test").
        dbt_vars (dict): A dictionary of DBT variables to pass as `--vars`.
        additional_args (str, optional): Additional command-line arguments to include. Defaults to an empty string.

    Returns:
        str: The constructed DBT command string ready for execution.
    """
    if dbt_vars is None:
        dbt_vars = {}

    threads = dbt_vars.get("threads")
    thread_arg = f"--threads {threads} " if threads else ""
    dbt_vars_arg = f"--vars '{json.dumps(dbt_vars)}' " if dbt_vars else ""
    additional_args = additional_args if additional_args else ""
    cmd = (
        f"dbt {dbt_command} --project-dir {project_dir} "
        f"{thread_arg}"
        f"{dbt_vars_arg}"
        f"{additional_args}"
    )
    return cmd


@task(task_run_name="run_shell_command")
def run_shell_command(command: str):
    logger = get_run_logger()
    logger.info(f"Running command:\n{command}")
    cmd_block = ShellOperation(commands=[command])
    cmd_output = cmd_block.run()
    return cmd_output


```

###### FILE: src/azfr_skywalker_utils/ibis/trino.py ######

```py
from contextlib import contextmanager

import ibis
from trino.auth import JWTAuthentication

from azfr_skywalker_utils.trino.common import TrinoConfig
from azfr_azure_utils.identity import AzfrDefaultCredential


@contextmanager
def ibis_connection(config: TrinoConfig):
    """
    Context manager for Ibis Trino connection using TrinoConfig.
    """
    jwt_token = AzfrDefaultCredential().get_token(config.access_token_scope)
    conn = ibis.trino.connect(
        host=config.host,
        port=config.port,
        http_scheme=config.http_scheme,
        verify=config.ssl_verify,
        auth=JWTAuthentication(jwt_token.token),
        user=config.user,
        database=config.catalog,
        schema=config.schema_name,
        source=config.source,
    )
    try:
        yield conn
    finally:
        conn.disconnect()

```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/__init__.py ######

```py
from azfr_skywalker_utils.metadata.parsing.analyzer import *
from azfr_skywalker_utils.metadata.parsing.calendar import *
from azfr_skywalker_utils.metadata.parsing.exception import *
from azfr_skywalker_utils.metadata.parsing.flow import *
from azfr_skywalker_utils.metadata.parsing.metadata import *
from azfr_skywalker_utils.metadata.parsing.workflowconfig import *

```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/analyzer.py ######

```py
import re
import logging
from datetime import datetime
from itertools import groupby, repeat
from operator import itemgetter
from typing import Protocol, Tuple
from abc import ABC, abstractmethod

from azfr_fsspec_utils.zipfile import FsspecZipFile
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.metadata import FileStatus, FileDetailedStatus, TableStatus, TableDetailedStatus, WorkflowMetadata
from azfr_skywalker_utils.metadata.parsing.workflowconfig import TableConfig, WorkflowBaseConfig
from azfr_skywalker_utils.metadata.parsing.calendar import is_future_date, is_expected_date


logger = logging.getLogger(__name__)


class StateProtocol(Protocol):
    """
    This protocol represents a Prefect-like state object, which tracks the 
    progress of a task, including whether it is completed, the result of the task, 
    and a message related to the task's status. 
    """
    def is_completed(self) -> bool:
        """Returns whether the state is completed."""
        ...
    
    def result(self, raise_on_failure: bool = ..., retry_result_failure: bool = ...) -> Tuple[int, int]:
        """Fetches and returns a tuple of two integers representing parsed and rejected lines.
        Function signature adapted from: https://github.com/PrefectHQ/prefect/blob/3.3.5/src/prefect/client/schemas/objects.py#L254-L279
        """
        ...

    @property
    def message(self) -> str:
        """Returns the message associated with the state."""
        ...

class ParseResults:
    def __init__(self, file_identifier: str, date: str):
        self.file_status = {} # see metadata.WORKFLOW_FILE_STATUS_SCHEMA
        self.total_parsed_lines: int = 0
        self.total_rejected_lines: int = 0
        self.fullmode_empty_tables: list[str] = []
        self.deltamode_empty_tables: list[str] = []
        self.rejected_tables: list[str] = []
        self.file_identifier = file_identifier
        self.date = date


class AbstractAnalyzer(ABC):
    def __init__(self, workflow_metadata: WorkflowMetadata, config: WorkflowBaseConfig):
        self.workflow_metadata = workflow_metadata
        self.config = config
        self.landing_files_info = {}
        self.errors: list[str] = []
        self.valid_landing_files: list[str] = []
        self.parse_results: dict[str, ParseResults] = {}


    @staticmethod
    def extract_file_info(file_pattern: re.Pattern, file: str) -> dict:
        match = re.match(file_pattern, fspath.basename(file))
        file_info = {
            "file_path": file,
            "date": match.group("date"),
            "file_identifier": match.group("file_identifier")}
        return file_info

    def extract_landing_info(self, landing_files: list[str]):
        self.landing_files_info = {}
        extracted_file_info = map(self.extract_file_info, repeat(self.config.file_pattern), landing_files)
        data = sorted(extracted_file_info, key=itemgetter('date', 'file_identifier'))
        for key, group in groupby(data, key=itemgetter('date', 'file_identifier')):
            self.landing_files_info[key] = [file_info['file_path'] for file_info in group]

    def reset_landing_analysis_state(self):
        """Reset the internal state variables used during landing file analysis"""
        self.landing_files_info = {}
        self.errors = []
        self.valid_landing_files = []

    def _write_file_failure(self, file_identifier: str, date: str, detailed_status: FileDetailedStatus,
                            message: str, file_paths: list[str]):
        self.errors.append(message)
        logger.error(message)
        self.workflow_metadata.write_file_status(file_identifier, date, FileStatus.FAILED,
                                                 detailed_status, message, file_paths,
                                                 self.workflow_metadata.run_start_ts)

    def _write_file_failure_for_each_path(self, file_identifier: str, date: str,
                                          detailed_status: FileDetailedStatus,
                                          message: str, file_paths: list[str]):
        for file_path in file_paths:
            self._write_file_failure(file_identifier, date, detailed_status, message, [file_path])

    def _handle_future_date(self, date: str, file_identifier: str, file_paths: list[str]) -> bool:
        if not self.is_future_date(date, file_identifier):
            return False
        message = f"Fichier {file_identifier}-{date}: Date fonctionnelle {date} dans le futur"
        self._write_file_failure_for_each_path(file_identifier, date,
                                                FileDetailedStatus.FILE_FUNC_DATE_FUTURE,
                                                message, file_paths)
        return True

    def _handle_unexpected_date(self, date: str, file_identifier: str, file_paths: list[str]) -> bool:
        if not self.is_unexpected_date(date, file_identifier):
            return False
        message = f"Fichier {file_identifier}-{date}: Date fonctionnelle {date} non attendue"
        self._write_file_failure_for_each_path(file_identifier, date,
                                                FileDetailedStatus.FILE_FUNC_DATE_UNEXPECTED,
                                                message, file_paths)
        return True

    def _handle_duplicate_files(self, date: str, file_identifier: str, file_paths: list[str]) -> bool:
        if not self.is_duplicate(file_paths):
            return False
        message = (
            f"Plusieurs fichiers avec la même date fonctionnelle {date} et le même identifiant {file_identifier} "
            "sont présents dans la landing zone"
        )
        self._write_file_failure(file_identifier, date, FileDetailedStatus.FILE_FUNC_DATE_DUPLICATED,
                                 message, file_paths)
        return True

    def _handle_already_ingested(self, date: str, file_identifier: str, file_path: str) -> bool:
        if not self.is_already_ingested(date, file_identifier):
            return False
        message = f"Fichier {file_identifier}-{date}: Date fonctionnelle {date} déjà ingérée"
        self._write_file_failure(file_identifier, date, FileDetailedStatus.FILE_FUNC_DATE_ALREADY_INGESTED,
                                 message, [file_path])
        return True

    def _handle_table_checks(self, date: str, file_identifier: str, file_path: str,
                             check_mismatch_tables: bool) -> bool:
        file_config = self.config.files_configs[file_identifier]
        should_check_tables = check_mismatch_tables or file_config.raise_error_if_empty_file
        if not should_check_tables:
            return False

        file_tables = self.get_tables_from_file(file_path)
        if file_config.raise_error_if_empty_file and self.is_file_empty(file_tables):
            message = f"Fichier {file_identifier}-{date}: fichier vide"
            self._write_file_failure(file_identifier, date, FileDetailedStatus.FILE_EMPTY,
                                     message, [file_path])
            return True

        if check_mismatch_tables and self.is_mismatch_tables(file_path, date, file_identifier, file_tables):
            message = f"Fichier {file_identifier}-{date}: table(s) manquante(s) et/ou table(s) non reconnue(s)"
            self._write_file_failure(file_identifier, date, FileDetailedStatus.FILE_MISMATCH_TABLES,
                                     message, [file_path])
            return True

        return False

    def analyze_landing_files(self, landing_files: list[str], check_mismatch_tables: bool=True) -> list[str]:
        """
        Analyze provided landing_files. It is focused on file level checks
        If any check fails to be validated, corresponding row is written to metadata.
        Results can be obtained via class parameters landing_files_info, duplicates, errors and valid_landing_files.
        Mismatch tables check can be disabled, in case it is irrelevant or if requires to be done somewhere else
        for optimization, for example in case of tar files that are decompressed later to avoid multiple decompressions.
        your_analyzer_instance.is_mismatch_tables(...) can be called by itself at fitting time.
        
        Parameters
        ----------
        landing_files: list[str]: List of files to analyze
        check_mismatch_tables: boolean: Whether mismatch tables check needs to be done in this function or not.

        Returns
        -------
        valid_landing_files: list[str]: List of valid files
        """
        self.reset_landing_analysis_state()
        self.extract_landing_info(landing_files)
        for (date, file_identifier), file_paths in self.landing_files_info.items():
            if self._handle_future_date(date, file_identifier, file_paths):
                continue

            if self._handle_unexpected_date(date, file_identifier, file_paths):
                continue

            if self._handle_duplicate_files(date, file_identifier, file_paths):
                continue

            # Past is_duplicate(), we are guaranteed to have a single file per Key
            # We no longer need to loop over file_paths, and can suffice ourself to check file_paths[0]
            file_path = file_paths[0]
            if self._handle_already_ingested(date, file_identifier, file_path):
                continue

            if self._handle_table_checks(date, file_identifier, file_path, check_mismatch_tables):
                continue
            
            self.valid_landing_files.extend(file_paths)

        return self.valid_landing_files

    def is_future_date(self, date: str, file_identifier: str) -> bool:
        return is_future_date(date, offset=self.config.files_configs[file_identifier].functional_date_offset,
                              date_format=self.config.date_format or "%Y%m%d")

    def is_unexpected_date(self, date: str, file_identifier: str) -> bool:
        return not is_expected_date(date, self.config.files_configs[file_identifier].mode,
                                    self.config.files_configs[file_identifier].working_days_offset or 0,
                                    self.config.files_configs[file_identifier].expected_days,
                                    self.config.files_configs[file_identifier].working_days_only,
                                    self.config.date_format or "%Y%m%d")

    @staticmethod
    def is_duplicate(landing_files: list[str]) -> bool:
        return len(landing_files) > 1
    
    def is_already_ingested(self, date: str, file_identifier: str) -> bool:
        archive_dir = fspath.join(self.config.archive_dir, date)
        if fspath.exists(archive_dir):
            for archived_file in fspath.listdir(archive_dir):
                match = re.match(self.config.file_pattern, fspath.basename(archived_file))
                if (match.group("date") == date) and (match.group("file_identifier") == file_identifier):
                    return True
        return False

    @abstractmethod
    def get_tables_from_file(self, file: str) -> list[str]:
        """Extract table names from the file based on its format"""
        pass

    def is_file_empty(self, file_tables: list[str]) -> bool:
        """Check if file is empty by looking at tables it contains

        Parameters
        ----------
        file_tables: list[str]
            List of tables found in the file

        Returns
        -------
        bool
            True if the file contains no tables, False otherwise
        """
        return len(file_tables) == 0
    
    def is_mismatch_tables(self, landing_file: str, date: str, file_identifier: str, file_tables: list[str]) -> bool:
        """Check for missing or unknown tables using provided file_tables list"""
        expected_tables = self.config.files_configs[file_identifier].tables # list of tables configured for the parsing WF (as a collection of model.yml)

        has_missing_tables = self.has_missing_tables(landing_file, date, file_identifier, file_tables, expected_tables)
        has_unknown_tables = self.has_unknown_tables(landing_file, date, file_identifier, file_tables, expected_tables)

        return has_missing_tables or has_unknown_tables

    def has_missing_tables(self, landing_file: str, date, file_identifier: str,
                           file_tables: list[str], expected_tables: list[str]) -> bool:
        missing_tables = [table for table in expected_tables if table not in file_tables]

        for missing_table in missing_tables:
            message = f"Table {missing_table}-{date} attendue manquante"
            logger.info(message)
            self.workflow_metadata.write_table_status(file_identifier, date, missing_table, TableStatus.FAILED.value,
                                                      TableDetailedStatus.TABLE_MISSING.value,
                                                      None, None, message, landing_file,
                                                      self.workflow_metadata.run_start_ts)
        if missing_tables:
            return True
        return False

    def has_unknown_tables(self, landing_file: str, date: str, file_identifier: str,
                           file_tables: list[str], expected_tables: list[str]) -> bool:
        unknown_tables = [table for table in file_tables if table not in expected_tables]

        for unknown_table in unknown_tables:
            message = f"Table {unknown_table}-{date} non reconnue"
            logger.info(message)
            self.workflow_metadata.write_table_status(file_identifier, date, unknown_table, TableStatus.FAILED.value,
                                                      TableDetailedStatus.TABLE_UNKNOWN.value,
                                                      None, None, message, landing_file,
                                                      self.workflow_metadata.run_start_ts)
        if unknown_tables:
            return True
        return False

    def _raise_error_if_empty(self, table: str, nb_parsed_lines: int, nb_rejected_lines: int, date_str: str,
                                  table_config: TableConfig, file_path: str, mode: str) -> bool:
        """ Check if a table is empty for a specific extraction type and raise error if configured to do so. """
        if self.config.tables_to_format[table].extraction_type != mode:
            return False

        is_empty = (nb_parsed_lines == 0 and nb_rejected_lines == 0)
        raise_error_flag = {
            "full": table_config.raise_error_if_empty_full_mode,
            "delta": table_config.raise_error_if_empty_delta_mode
        }.get(mode, False)
        
        if is_empty and raise_error_flag:
            if table_config.raise_error_if_empty_after:
                if datetime.strptime(date_str, "%Y%m%d").date() < table_config.raise_error_if_empty_after:
                    log_message = f"Warning: Table {table} in {mode} mode but empty detected. File: {file_path}"
                    logger.warning(log_message)
                    return False
            return True
        return False

    def raise_error_if_full_and_empty(self, table: str, nb_parsed_lines: int, nb_rejected_lines: int, date_str: str,
                                      table_config: TableConfig, file_path: str) -> bool:
        return self._raise_error_if_empty(table, nb_parsed_lines, nb_rejected_lines, date_str, table_config, file_path, mode="full")

    def raise_error_if_delta_and_empty(self, table: str, nb_parsed_lines: int, nb_rejected_lines: int, date_str: str,
                                       table_config: TableConfig, file_path: str) -> bool:
        return self._raise_error_if_empty(table, nb_parsed_lines, nb_rejected_lines, date_str, table_config, file_path, mode="delta")

    def analyze_parse_table(self, state: StateProtocol, file_identifier: str, table: str, date: str, file_path: str, start_ts: datetime):
        """
        Analyze provided table according to a state object. It is focused on table level checks.
        If any check fails to be validated, corresponding row is written to metadata
        and file status in parse_results is updated accordingly.
        If there are no fails, success row is written to metadata
        Results can be obtained via class attribute parse_results.
        This function needs to be called for each table parsed.

        Parameters
        ----------
        state: statStateProtocole: Contains state of result of parsing of this data table, for example prefect state.
        file_identifier: str: File identifier of the file containing this data table
        table: str: Name of the table
        date: str: Date of the data at format YYYYMMDD
        file_path: str: Path to file that contains this data table
        start_ts: datetime: Timestamp of the start of table processing

        Returns None
        -------

        """
        table_config = self.config.files_configs[file_identifier].table_configs.get(table.lower(), TableConfig()) # fichiers json dans l'archive
        file = fspath.basename(file_path)

        if not self.parse_results.get(file):
            self.parse_results[file] = ParseResults(file_identifier=file_identifier, date=date)

        if not state.is_completed():
            status, detailed_status, message = self._handle_table_parse_failure(file_identifier, date, file, state.message)
            self._log_table_result(status, message)
            self.workflow_metadata.write_table_status(file_identifier, date, table, status, detailed_status,
                                                      None, None, message, file_path,
                                                      start_ts)
            return

        nb_parsed_lines, nb_rejected_lines = state.result()
        status, detailed_status, message = self._analyze_completed_table_parse(
            file_identifier=file_identifier,
            table=table,
            date=date,
            file_path=file_path,
            table_config=table_config,
            file=file,
            nb_parsed_lines=nb_parsed_lines,
            nb_rejected_lines=nb_rejected_lines
        )

        self._log_table_result(status, message)
        self.workflow_metadata.write_table_status(file_identifier, date, table, status, detailed_status,
                                                  nb_parsed_lines, nb_rejected_lines, message, file_path,
                                                  start_ts)

    def _ensure_file_error_status(self, file_identifier: str, date: str, file: str):
        file_status = self.parse_results[file].file_status
        if (not file_status) or (file_status.get("detailed_status") == FileDetailedStatus.FILE_LINES_REJECTED.value):
            file_status["status"] = FileStatus.FAILED.value
            file_status["detailed_status"] = FileDetailedStatus.FILE_ERROR_TABLES.value
            file_status["message"] = f"Fichier {file_identifier}-{date} : erreur table(s)"

    def _handle_table_parse_failure(self, file_identifier: str, date: str, file: str, state_message: str):
        self._ensure_file_error_status(file_identifier, date, file)
        return TableStatus.FAILED, TableDetailedStatus.TABLE_PARSING_ERROR, state_message

    def _analyze_completed_table_parse(self, *, file_identifier: str, table: str, date: str, file_path: str,
                                       table_config: TableConfig, file: str, nb_parsed_lines: int,
                                       nb_rejected_lines: int):
        if self.raise_error_if_full_and_empty(table, nb_parsed_lines, nb_rejected_lines, date, table_config, file_path):
            self.parse_results[file].fullmode_empty_tables.append(table)
            self._ensure_file_error_status(file_identifier, date, file)
            return TableStatus.FAILED, TableDetailedStatus.TABLE_FULL_EMPTY, f"Table {table}-{date} au format full est vide"

        if self.raise_error_if_delta_and_empty(table, nb_parsed_lines, nb_rejected_lines, date, table_config, file_path):
            self.parse_results[file].deltamode_empty_tables.append(table)
            self._ensure_file_error_status(file_identifier, date, file)
            return TableStatus.FAILED, TableDetailedStatus.TABLE_DELTA_EMPTY, f"Table {table}-{date} au format delta est vide"

        if nb_rejected_lines > 0:
            self.parse_results[file].total_rejected_lines += nb_rejected_lines
            self.parse_results[file].total_parsed_lines += nb_parsed_lines
            self.parse_results[file].rejected_tables.append(table)
            if not self.parse_results[file].file_status:
                self.parse_results[file].file_status["status"] = FileStatus.FAILED.value
                self.parse_results[file].file_status["detailed_status"] = FileDetailedStatus.FILE_LINES_REJECTED.value
            return TableStatus.FAILED, TableDetailedStatus.TABLE_LINES_REJECTED, f"Table {table}-{date} ingérée avec rejets"

        self.parse_results[file].total_parsed_lines += nb_parsed_lines
        return TableStatus.SUCCESS, TableDetailedStatus.TABLE_SUCCESS, f"Table {table}-{date} ingérée avec succès"

    @staticmethod
    def _log_table_result(status: TableStatus, message: str):
        if status == TableStatus.FAILED:
            logger.error(message)

    def analyze_file_after_parse(self, file_path, start_ts):
        file = fspath.basename(file_path)
        if not self.parse_results.get(file):
            raise KeyError(f"No parsed results available for {file}.")
        if not self.parse_results[file].file_status:
            total_lines = self.parse_results[file].total_parsed_lines + self.parse_results[file].total_rejected_lines
            if total_lines == 0:
                self.parse_results[file].file_status["status"] = FileStatus.SUCCESS.value
                self.parse_results[file].file_status["detailed_status"] = FileDetailedStatus.FILE_NO_ROWS.value
                self.parse_results[file].file_status["message"] = f"Fichier {self.parse_results[file].file_identifier}-{self.parse_results[file].date} : Toutes les tables sont vides"
            else:
                self.parse_results[file].file_status["status"] = FileStatus.SUCCESS.value
                self.parse_results[file].file_status["detailed_status"] = FileDetailedStatus.FILE_SUCCESS.value
                self.parse_results[file].file_status["message"] = f"Fichier {self.parse_results[file].file_identifier}-{self.parse_results[file].date} ingéré avec succès"
        elif self.parse_results[file].file_status["detailed_status"] == FileDetailedStatus.FILE_LINES_REJECTED.value:
            self.parse_results[file].file_status[
                "message"] = f"Fichier {self.parse_results[file].file_identifier}-{self.parse_results[file].date} : {self.parse_results[file].total_rejected_lines} lignes rejetées sur {len(self.parse_results[file].rejected_tables)} tables"
        if self.parse_results[file].file_status["message"]:
            if self.parse_results[file].file_status["status"] == FileStatus.FAILED.value:
                logger.error(self.parse_results[file].file_status["message"])
            else:
                logger.info(self.parse_results[file].file_status["message"])
        self.workflow_metadata.write_file_status(self.parse_results[file].file_identifier,
                                                 self.parse_results[file].date,
                                                 self.parse_results[file].file_status["status"],
                                                 self.parse_results[file].file_status["detailed_status"],
                                                 self.parse_results[file].file_status["message"], [file_path],
                                                 start_ts)

    def set_file_empty(self, file_path: str, file_identifier: str, date: str):
        """Set file status to empty"""
        # No need to check raise_error_if_empty_file here, this case was already handled in analyze_landing_files().
        file = fspath.basename(file_path)
        self.parse_results[file] = ParseResults(file_identifier, date)
        self.parse_results[file].file_status["status"] = FileStatus.SUCCESS.value
        self.parse_results[file].file_status["detailed_status"] = FileDetailedStatus.FILE_EMPTY.value
        self.parse_results[file].file_status["message"] = f"Fichier {file_identifier}-{date} : Archive vide (aucune table)"


class TarAnalyzer(AbstractAnalyzer):
    def get_tables_from_file(self, file: str) -> list[str]:
        with fspath.open(file, compression="tar", mode="rb") as untar_file:
            file_names = [item.name for item in untar_file.getmembers()]
        uncompressed_file_regex = re.compile(self.config.uncompressed_file_pattern)
        return [re.match(uncompressed_file_regex, file_name).group("name") for file_name in file_names]


class ZipAnalyzer(AbstractAnalyzer):
    def get_tables_from_file(self, file: str) -> list[str]:
        with FsspecZipFile(file) as f:
            files_name = f.namelist()
        uncompressed_file_regex = re.compile(self.config.uncompressed_file_pattern)
        return [re.match(uncompressed_file_regex, file_name).group("name") for file_name in files_name]


class PlainFileAnalyser(AbstractAnalyzer):
    """Analyzer for plain text files"""
    def get_tables_from_file(self, file: str) -> list[str]:
        """For plain text files, assume filename is the table name"""
        file_regex = re.compile(self.config.file_pattern)
        match = re.match(file_regex, fspath.basename(file))
        return [match.group("name")] if match else []

```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/calendar.py ######

```py
from datetime import datetime, timedelta, date
from workalendar.europe import France

calendar = France()

SUPPORTED_MODES = {"daily", "weekly", "monthly"}


def _normalize_expected_days(expected_days: int | list[int] | None) -> list[int] | None:
    if isinstance(expected_days, int):
        return [expected_days]
    return expected_days


def _matches_expected_day(date_to_check: datetime, mode: str, expected_days: int | list[int] | None) -> bool:
    expected_days = _normalize_expected_days(expected_days)
    if mode == "weekly":
        return date_to_check.weekday() + 1 in expected_days
    if mode == "monthly":
        return date_to_check.day in expected_days
    return False


def _is_working_day(date_to_check: datetime, offset: int) -> bool:
    return calendar.is_working_day((date_to_check - timedelta(days=offset)).date())


def _is_first_working_day_after_expected(date_to_check: datetime, mode: str, offset: int, expected_days: int | list[int] | None) -> bool:
    td = 1
    while not calendar.is_working_day((date_to_check - timedelta(days=offset + td)).date()):
        if _matches_expected_day(date_to_check - timedelta(days=td), mode, expected_days):
            return True
        td += 1
    return False


def is_expected_date(date_str: str, mode: str, offset: int = 0, expected_days: int | list[int] = None, working_days_only: bool = False, date_format: str = "%Y%m%d"):
    """
    date: Date to check must match date_format
    mode: Mode for the valid dates
    offset: If there is an offset to apply to date, for example set offset = 1 if date is expected the next day of working day
    expected_days: In case of monthly, valid day(s) of month. In case of weekly, valid day(s) of week (between 1 and 7).
    working_days_only: If date matches only working day, set to True.
    date_format: Date format
    Return True if date is valid according to mode, else return False
    Add new mode if new behaviour. Weekly and monthly with working_days_only set to True returns True for the first workay day after expected_days.
    """
    date = datetime.strptime(date_str, date_format)
    if mode not in SUPPORTED_MODES:
        raise ValueError(f"Mode {mode} not supported")
    if working_days_only and not _is_working_day(date, offset):
        return False
    if mode == "daily":
        return True
    if working_days_only and _is_first_working_day_after_expected(date, mode, offset, expected_days):
        return True
    return _matches_expected_day(date, mode, expected_days)


def get_expected_dates(min_date: str, mode: str, offset: int = 0, expected_days: int | list[int] = None, working_days_only: bool = False, max_date: str = None, date_format: str = "%Y%m%d"):
    """
    min_date: Minimal date expected, must match date_format
    mode: Name of mode to validate dates
    offset: If there is an offset to apply to date, for example set offset = 1 if date is expected the next day of working day
    expected_days: In case of monthly, day(s) of month valid. In case of weekly, day(s) of week valid.
    working_days_only: If date matches only working day, set to True.
    max_date: Max date expected, must match date_format. If not set, use today date
    date_format: Date format
    Return a list of dates in str format between min_expected_date and max_date, based on mode
    """
    sdate = datetime.strptime(min_date, date_format).date()
    if max_date:
        edate = datetime.strptime(max_date, date_format).date()
    else:
        edate = date.today()
    if edate < sdate:
        return []
    dates_in_range = [(sdate + timedelta(days=x)).strftime(date_format) for x in range((edate - sdate).days + 1)]
    return [date for date in dates_in_range if is_expected_date(date, mode=mode, offset=offset, expected_days=expected_days, working_days_only=working_days_only, date_format=date_format)]


def is_future_date(date_to_check: str, offset: int=0, max_date: str = None, date_format = "%Y%m%d"):
    """
    date_to_check: Date to check, must match date_format
    offset: If there is an offset to apply to date, for example set offset = 1 if data is expected one day later
    max_date: Max date expected, must match date_format. If not set, use today date
    date_format: Date format
    Return true if date_to_check + offset is later than limit_date
    """
    if max_date:
        limit_date = datetime.strptime(max_date, date_format).date()
    else:
        limit_date = date.today()
    return datetime.strptime(date_to_check, date_format).date() + timedelta(days=offset) > limit_date

```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/exception.py ######

```py

class RejectedRowsError(Exception):
    """This Exception can be used when a table with rejected rows is detected."""
    def __init__(self, table_list: list[str]):
        count = len(table_list)
        table_word = "table" if count == 1 else "tables"
        super().__init__(f"Table with rejected rows detected. {count} {table_word}.")

class DeltaModeEmptyTableError(Exception):
    """This Exception can be used when a table in delta mode but empty is detected."""
    def __init__(self, table_list: list[str]):
        count = len(table_list)
        table_word = "table" if count == 1 else "tables"
        self.message = (f"Table in delta mode but empty detected. {count} {table_word}.")
        super().__init__(self.message)

class FullModeEmptyTableError(Exception):
    """This Exception can be used when a table in full mode but empty is detected."""
    def __init__(self, table_list: list[str]):
        count = len(table_list)
        table_word = "table" if count == 1 else "tables"
        self.message = (f"Table in full mode but empty detected. {count} {table_word}.")
        super().__init__(self.message)


class AnalyzerValidationError(Exception):
    """Raised when analyzer validation returns one or more errors for a flow run."""

    def __init__(self, errors: list[str]):
        self.errors = errors
        super().__init__(f"List of errors for this run: {errors}")

```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/flow.py ######

```py
from typing import Type
from datetime import datetime

from prefect import Flow, flow, get_run_logger
from azfr_parsing_utils.utils import get_files

from azfr_skywalker_utils.metadata.parsing.analyzer import AbstractAnalyzer
from azfr_skywalker_utils.metadata.parsing.exception import AnalyzerValidationError
from azfr_skywalker_utils.metadata.parsing.metadata import register_missing_files, WorkflowMetadata
from azfr_skywalker_utils.metadata.parsing.workflowconfig import WorkflowBaseConfig
from azfr_skywalker_utils.utils.mail import EmailConfig

@flow
def main_flow(config: WorkflowBaseConfig,
              email_config: EmailConfig,
              archive_and_parse: Flow,
              analyzer_class: Type[AbstractAnalyzer],
              write_start_event:bool = True,
              check_mismatch_tables:bool = True):
    """
    The flow will:
    - Start workflow metadata tracking
    - Identify files to process based on the `file_pattern` and `input_dir`
    - Optionally validate the landing files using the specified `analyzer_class`
    - Call the `archive_and_parse` subflow for each valid landing file
    - Register any missing files if validation is enabled
    - End workflow metadata tracking
    
    Parameters:
    - config: The configuration object for the workflow (WorkflowBaseConfig).
    - email_config: Configuration for email notifications (EmailConfig).
    - archive_and_parse: The subflow responsible for archiving and parsing the landing files.
    - analyzer_class: The class to use for analyzing the landing files (typically a subclass of Analyzer).
    - write_start_event: Flag to determine whether to log the start event of the workflow (default is True).
    - check_mismatch_tables: Flag to indicate whether to check for mismatch tables during analysis (default is True).
    """
    logger = get_run_logger()
    logger.info(f"Using config: {config.to_json()}")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=write_start_event)
    landing_files = get_files(config.input_dir, config.file_pattern)
    logger.info("Files detected from {} : {}".format(str(config.input_dir), str(landing_files)))

    if config.validate_files:
        # Keep only dates from landing that are valid
        analyzer = analyzer_class(workflow_metadata=workflow_metadata, config=config)
        valid_landing_files = analyzer.analyze_landing_files(landing_files, check_mismatch_tables=check_mismatch_tables)
        logger.info("Files that are parsed are validated")
    else:
        valid_landing_files = landing_files
        logger.info("File validation is disabled !")
    
    for landing_file in valid_landing_files:
        archive_and_parse(config, landing_file, analyzer, return_state=True)
    
    if config.validate_files:
        register_missing_files(config.files_configs, config.metadata_dir, workflow_metadata,
                               datetime.now(), config.date_format, config.time_format)
    
    workflow_metadata.end()
    
    if config.validate_files and analyzer.errors:
        raise AnalyzerValidationError(analyzer.errors)



```

###### FILE: src/azfr_skywalker_utils/metadata/parsing/metadata.py ######

```py
import os
import logging
from enum import Enum
from typing import Optional, Union
from datetime import datetime, timedelta

import polars as pl
import pyarrow as pa
from uuid_extensions import uuid7str

import azfr_fsspec_utils as fspath
from azfr_parsing_utils.common import AdditionalColumn
from azfr_parsing_utils.deltalake import write_deltalake
from deltalake.exceptions import TableNotFoundError

from azfr_skywalker_utils.metadata.parsing.calendar import get_expected_dates, is_expected_date
from azfr_skywalker_utils.metadata.parsing.workflowconfig import FileConfig, WorkflowBaseConfig
from azfr_skywalker_utils.utils.date import get_now_UTC

logger = logging.getLogger(__name__)

WORKFLOW_EVENTS = "workflow_events"
WORKFLOW_FILE_STATUS = "workflow_file_status"
WORKFLOW_TABLE_STATUS = "workflow_table_status"

WORKFLOW_EVENTS_SCHEMA = pa.schema([
    pa.field("id", pa.string()),
    pa.field("workflow_name", pa.string()),
    pa.field("run_id", pa.string()),
    pa.field("run_start_ts", pa.timestamp('us', tz='UTC')),
    pa.field("run_end_ts", pa.timestamp('us', tz='UTC')),
    pa.field("event_type", pa.string()),
    pa.field("event_ts", pa.timestamp('us', tz='UTC')),
    pa.field("event_info_json", pa.string()),
    pa.field("creator_github_repository", pa.string()),
    pa.field("creator_commit_id", pa.string()),
    pa.field("wf_config_json", pa.string()),
])
WORKFLOW_FILE_STATUS_SCHEMA = pa.schema([
    pa.field("id", pa.string()),
    pa.field("run_id", pa.string()),
    pa.field("workflow_name", pa.string()),
    pa.field("file_identifier", pa.string()),
    pa.field("version", pa.string()),
    pa.field("global_status", pa.string()),
    pa.field("detailed_status", pa.string()),
    pa.field("message", pa.string()),
    pa.field("file_paths", pa.list_(pa.string())),
    pa.field("start_ts", pa.timestamp('us', tz='UTC')),
    pa.field("end_ts", pa.timestamp('us', tz='UTC')),
    pa.field("additional_info_json", pa.string()),
])
WORKFLOW_TABLE_STATUS_SCHEMA = pa.schema([
    pa.field("id", pa.string()),
    pa.field("run_id", pa.string()),
    pa.field("workflow_name", pa.string()),
    pa.field("file_identifier", pa.string()),
    pa.field("version", pa.string()),
    pa.field("table", pa.string()),
    pa.field("global_status", pa.string()),
    pa.field("detailed_status", pa.string()),
    pa.field("nb_parsed_lines", pa.int64()),
    pa.field("nb_rejected_lines", pa.int64()),
    pa.field("message", pa.string()),
    pa.field("file_path", pa.string()),
    pa.field("start_ts", pa.timestamp('us', tz='UTC')),
    pa.field("end_ts", pa.timestamp('us', tz='UTC')),
    pa.field("additional_info_json", pa.string()),
])


class TableStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"


class TableDetailedStatus(Enum):
    TABLE_FULL_EMPTY = "TABLE_FULL_EMPTY"
    TABLE_DELTA_EMPTY = "TABLE_DELTA_EMPTY"
    TABLE_MISSING = "TABLE_MISSING"
    TABLE_PARSING_ERROR = "TABLE_PARSING_ERROR"
    TABLE_SUCCESS = "TABLE_SUCCESS"
    TABLE_LINES_REJECTED = "TABLE_LINES_REJECTED"
    TABLE_QUALITY_NULL_COLUMNS = "TABLE_QUALITY_NULL_COLUMNS"
    TABLE_UNKNOWN = "TABLE_UNKNOWN"


class FileStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"
    WAITING = "WAITING"


class FileDetailedStatus(Enum):
    FILE_SUCCESS = "FILE_SUCCESS"
    FILE_EMPTY = "FILE_EMPTY"
    FILE_NO_ROWS = "FILE_NO_ROWS"
    FILE_WAITING = "FILE_WAITING"
    FILE_NOT_RECEIVED = "FILE_NOT_RECEIVED"
    FILE_FUNC_DATE_ALREADY_INGESTED = "FILE_FUNC_DATE_ALREADY_INGESTED"
    FILE_FUNC_DATE_DUPLICATED = "FILE_FUNC_DATE_DUPLICATED"
    FILE_FUNC_DATE_UNEXPECTED = "FILE_FUNC_DATE_UNEXPECTED"
    FILE_LINES_REJECTED = "FILE_LINES_REJECTED"
    FILE_ERROR_TABLES = "FILE_ERROR_TABLES"
    FILE_FUNC_DATE_FUTURE = "FILE_FUNC_DATE_FUTURE"
    FILE_MISMATCH_TABLES = "FILE_MISMATCH_TABLES"
    FILE_TECHNICAL_ERROR = "FILE_TECHNICAL_ERROR"


class ParsingDomainStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"
    WAITING = "WAITING"


class ParsingDetailedStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"
    WAITING = "WAITING"    
    EMPTY = "EMPTY"


class WorkflowMetadata:
    def __init__(self):
        self.creator_github_repository: str = os.environ.get("GITHUB_REPOSITORY")
        self.creator_commit_id: str = os.environ.get("GITHUB_SHA")
        self.run_end_ts: Optional[datetime] = None
        self.run_start_ts: Optional[datetime] = None
        self.run_id: Optional[str] = None
        self.config: Optional[WorkflowBaseConfig] = None

    def start(self, config: WorkflowBaseConfig, run_start_ts: Optional[datetime] = None, write_start_event: bool = True,
              wf_config_json: Optional[str] = None):
        """
        Integrate config and set run_id and run_start_ts. And write WORKFLOW_START event
        Parameters
        ----------
        config: Workflow Config
        run_start_ts: Datetime: Run start ts
        write_start_event: bool: Write start event if True
        wf_config_json: str: Value to add to wf_config_json column in metadata event. Is only used if write_start_event is set to True

        Returns
        -------

        """
        self.config = config
        self.run_id = uuid7str()
        self.run_start_ts = run_start_ts or get_now_UTC()
        if write_start_event:
            self.write_event("WORKFLOW_START", wf_config_json)

    def end(self, wf_config_json: Optional[str] = None, run_end_ts: Optional[datetime] = None):
        """
        Set run_end_ts to current time and write WORKFLOW_END event
        Returns
        -------

        """
        self.run_end_ts = run_end_ts or get_now_UTC()
        self.write_event("WORKFLOW_END", wf_config_json)

    def write_table_status(self, file_identifier: str, version: str, table: str,
                           global_status: Union[TableStatus|str], detailed_status: Union[TableDetailedStatus|str],
                           nb_parsed_lines: int, nb_rejected_lines: int, message: str, file_path: str,
                           start_ts: datetime, end_ts: Optional[datetime] = None,
                           additional_info_json: Optional[str] = None):
        """
        Write a row with given parameters in WORKFLOW_TABLE_STATUS table
        Parameters
        ----------
        file_identifier: str : File identifier
        version: str: Version of the file containing the table
        table: str: Table name
        global_status: str: Global status
        detailed_status: str: Detailed status
        nb_parsed_lines: int: Number of parsed lines
        nb_rejected_lines: int: Number of rejected lines
        message: str: Message
        file_path: str: Path of the file
        start_ts: timestamp: Timestamp of start of processing of the table
        end_ts: timestamp: Timestamp of end of processing of the table
        additional_info_json: str: Additional info

        Returns
        -------

        """
        if end_ts is None:
            end_ts = get_now_UTC()
        if isinstance(detailed_status, Enum):
            detailed_status = detailed_status.value
        if isinstance(global_status, Enum):
            global_status = global_status.value
        result = {
            "id": uuid7str(),
            "run_id": self.run_id,
            "workflow_name": self.config.workflow_name,
            "file_identifier": file_identifier,
            "version": version,
            "table": table,
            "global_status": global_status,
            "detailed_status": detailed_status,
            "nb_parsed_lines": nb_parsed_lines,
            "nb_rejected_lines": nb_rejected_lines,
            "message": message,
            "file_path": file_path,
            "start_ts": start_ts,
            "end_ts": end_ts,
            "additional_info_json": additional_info_json
        }
        write_deltalake(data=pa.Table.from_pylist([result], schema=WORKFLOW_TABLE_STATUS_SCHEMA),
                        path=fspath.join(self.config.metadata_dir, WORKFLOW_TABLE_STATUS),
                        mode="append",
                        overwrite_schema=True
                        )

    def write_file_status(self, file_identifier: str, version: str,
                          global_status: Union[FileStatus|str], detailed_status: Union[FileDetailedStatus|str], message: str,
                          file_paths: Optional[list[str]], start_ts: datetime, end_ts: Optional[datetime] = None,
                          additional_info_json: Optional[str] = None):
        """
        Write a row with given parameters in WORKFLOW_FILE_STATUS table
        Parameters
        ----------
        file_identifier: str: File identifier
        version: str: Version of the file
        global_status: str: Global status
        detailed_status: str: Detailed status
        message: str: Message
        file_paths: list[str]: List of file paths
        start_ts: datetime: Timestamp of start of processing of the file
        end_ts: datetime: Timestamp of end of processing of the file
        additional_info_json: str: Additional info

        Returns
        -------

        """
        if end_ts is None:
            end_ts = get_now_UTC()
        if isinstance(detailed_status, Enum):
            detailed_status = detailed_status.value
        if isinstance(global_status, Enum):
            global_status = global_status.value
        result = {
            "id": uuid7str(),
            "run_id": self.run_id,
            "workflow_name": self.config.workflow_name,
            "file_identifier": file_identifier,
            "version": version,
            "global_status": global_status,
            "detailed_status": detailed_status,
            "message": message,
            "file_paths": file_paths,
            "start_ts": start_ts,
            "end_ts": end_ts,
            "additional_info_json": additional_info_json,
        }

        write_deltalake(data=pa.Table.from_pylist([result], schema=WORKFLOW_FILE_STATUS_SCHEMA),
                        path=fspath.join(self.config.metadata_dir, WORKFLOW_FILE_STATUS),
                        mode="append",
                        overwrite_schema=True
                        )

    def write_event(self, event_type: str, wf_config_json: Optional[str], event_info_json: Optional[str] = None,
                    run_end_ts: Optional[datetime] = None, event_ts: Optional[datetime] = None):
        """
        Write a row with given parameters in WORKFLOW_EVENTS table
        Parameters
        ----------
        event_type: str: Type of event
        wf_config_json: str: Information related to the config of the workflow for this run
        event_info_json: Optional[str]: Information related to the event
        run_end_ts: Optional[datetime]: Timestamp of end of run
        event_ts: Optional[datetime]: Timestamp of the event, current time by default

        Returns
        -------

        """
        if event_ts is None:
            event_ts = get_now_UTC()
        if run_end_ts is None:
            run_end_ts = self.run_end_ts
        result = {
            "id": uuid7str(),
            "workflow_name": self.config.workflow_name,
            "run_id": self.run_id,
            "run_start_ts": self.run_start_ts,
            "run_end_ts": run_end_ts,
            "event_type": event_type,
            "event_ts": event_ts,
            "event_info_json": event_info_json,
            "creator_github_repository": self.creator_github_repository,
            "creator_commit_id": self.creator_commit_id,
            "wf_config_json": wf_config_json,
        }
        write_deltalake(data=pa.Table.from_pylist([result], schema=WORKFLOW_EVENTS_SCHEMA),
                        path=fspath.join(self.config.metadata_dir, WORKFLOW_EVENTS),
                        mode="append",
                        overwrite_schema=True
                        )

    def create_additional_columns(self, version: str, file_name: str) -> tuple[list[AdditionalColumn], pl.Expr]:
        """
        Generates additional metadata columns for a table, including versioning information 
        and technical metadata related to the workflow.

        Args:
            version (str): The table version in 'YYYYMMDD' format.
            file_name (str): The name of the source file.

        Returns:
            tuple: A tuple containing:
                - list of AdditionalColumn: Extra columns with versioning details.
                - pl.Struct: A structured object containing technical metadata, 
                such as creation timestamp, run ID, and GitHub repository details.
        """

        version_dt = datetime.strptime(version, "%Y%m%d")
        additional_columns = [
            AdditionalColumn(name="__functional_date__", data_type="DATE", value=version_dt),
            AdditionalColumn(name="__version__", data_type="STRING", value=version)
            ]
        
        # pl.struct are not a canonical datatype, this field can not be included inside an AdditionalColumn object
        metadata = pl.struct(
            pl.lit(file_name).alias('source'),
            pl.lit(version).alias('version'),
            pl.lit(get_now_UTC()).alias('creation_ts'),
            pl.lit(self.run_id).alias('creation_run_id'),
            pl.lit(self.creator_github_repository).alias('creator_github_repository'),
            pl.lit(self.creator_commit_id).alias('creator_commit_id'),
            schema={"source": pl.String, "version": pl.String, "creation_ts": pl.Datetime(time_unit='us', time_zone="UTC"), "creation_run_id": pl.String, "creator_github_repository": pl.String, "creator_commit_id": pl.String}
        )
        
        return additional_columns, metadata


def get_registered_info(workflow_file_status_path: str, file_identifiers: list[str], min_date: Optional[str] = None):
    """
    workflow_file_status_path: str: Path to the delta table containing file metadata
    file_identifiers: list[str]: List of file_identifiers
    min_date: str: Minimum date to filter on
    Returns a dict with a field for each file identifier containing fields for each version with global and detailed
    status of the most recent row for this version.
    """
    results = {}
    try:
        for file_identifier in file_identifiers:
            df = pl.scan_delta(workflow_file_status_path)
            df = df.filter(pl.col("file_identifier") == file_identifier)
            if min_date:
                df = df.filter(pl.col("version") >= min_date)
            data = df.filter(
                (pl.int_range(pl.len()) == pl.col.start_ts.arg_max()).over("version")
            ).select(["version", "global_status", "detailed_status"]).collect().to_dicts()
            results[file_identifier] = {
                row["version"]: {"global_status": row["global_status"], "detailed_status": row["detailed_status"]} for
                row in data}
    except (TableNotFoundError, FileNotFoundError):
        for file_identifier in file_identifiers:
            results[file_identifier] = {}
    return results


def _get_finished_dates(registered_info: dict[str, dict[str, str]]) -> set[str]:
    ignored_statuses = {
        FileDetailedStatus.FILE_FUNC_DATE_FUTURE.value,
        FileDetailedStatus.FILE_WAITING.value,
    }
    return {
        date
        for date, info in registered_info.items()
        if info["detailed_status"] not in ignored_statuses
    }


def _is_effectively_finished(registered_info: dict[str, dict[str, str]], date: str) -> bool:
    ignored_statuses = {
        FileDetailedStatus.FILE_FUNC_DATE_FUTURE.value,
        FileDetailedStatus.FILE_WAITING.value,
    }
    date_info = registered_info.get(date)
    if not date_info:
        return False
    return date_info["detailed_status"] not in ignored_statuses


def _write_waiting_row(workflow_metadata: WorkflowMetadata, file_identifier: str, max_date_str: str, overdue_time: str):
    logger.info(
        f"Adding waiting for table {file_identifier} for date {max_date_str} in metadata. Expected before {overdue_time}")
    message = f"En attente du fichier {file_identifier}-{max_date_str}"
    workflow_metadata.write_file_status(file_identifier, max_date_str, FileStatus.WAITING,
                                        FileDetailedStatus.FILE_WAITING,
                                        message, None, workflow_metadata.run_start_ts)


def _write_overdue_row(workflow_metadata: WorkflowMetadata, file_identifier: str, date: str, overdue_time: str):
    logger.warning(f"Registering overdue date {date} for file {file_identifier} in metadata")
    message = f"Fichier {file_identifier}-{date} attendu au plus tard {overdue_time} non reçu "
    workflow_metadata.write_file_status(file_identifier, date, FileStatus.FAILED,
                                        FileDetailedStatus.FILE_NOT_RECEIVED,
                                        message, None, workflow_metadata.run_start_ts)


def _get_min_expected_date(file_config: FileConfig, max_date: datetime, date_format: str) -> str:
    return max([
        file_config.min_expected_date,
        (max_date - timedelta(days=file_config.period_checked - 1)).strftime(date_format),
    ])


def _process_missing_dates_for_file(file_identifier: str,
                                    file_config: FileConfig,
                                    registered_info: dict[str, dict[str, str]],
                                    workflow_metadata: WorkflowMetadata,
                                    current_time: datetime,
                                    date_format: str,
                                    time_format: str):
    max_date = current_time - timedelta(days=file_config.functional_date_offset)
    max_date_str = max_date.strftime(date_format)
    finished_dates = _get_finished_dates(registered_info)

    min_expected_date = _get_min_expected_date(file_config, max_date, date_format)
    if min_expected_date and (max_date_str < min_expected_date):
        return

    expected_dates = get_expected_dates(min_expected_date,
                                        file_config.mode,
                                        file_config.working_days_offset,
                                        file_config.expected_days,
                                        file_config.working_days_only,
                                        max_date_str)
    is_today_not_overdue_yet = (max_date.time() <= datetime.strptime(file_config.overdue_time,
                                                                      time_format).time())
    if is_today_not_overdue_yet:
        # If not overdue, remove today from expected dates to avoid adding error if not arrived yet
        expected_dates = list(set(expected_dates) - {max_date_str})
        # If today date is expected, not parsed yet and not overdue,
        # add a row in metadata (Add a new waiting row even if waiting already exists)
        if (is_expected_date(max_date_str, file_config.mode,
                             file_config.working_days_offset or 0,
                             file_config.expected_days,
                             file_config.working_days_only,
                             date_format or "%Y%m%d")) \
                and (max_date_str not in finished_dates):
            _write_waiting_row(workflow_metadata, file_identifier, max_date_str, file_config.overdue_time)

    # Add row with FAILED status for each table for each date expected but not registered yet
    for date in sorted(set(expected_dates) - finished_dates):
        if _is_effectively_finished(registered_info, date):
            continue
        _write_overdue_row(workflow_metadata, file_identifier, date, file_config.overdue_time)


def register_missing_files(files_configs: dict[str, FileConfig], metadata_dir: str, workflow_metadata: WorkflowMetadata,
                           current_time: datetime, date_format: str="%Y%m%d", time_format: str="%H:%M"):
    """
    Register missing files in metadata file table between current time and closest date depending on files_configs
    Parameters
    ----------
    files_configs: Dict of FileConfig containing file_identifier, mode, working_days_only, working_days_offset, expected_days, overdue_time parameters cf FileConfig
    metadata_dir: str: Directory of the metadata file table
    workflow_metadata: WorkflowMetadata instance
    current_time: datetime: Current datetime
    date_format: Optional str: Date format
    time_format: Optional str: Time format

    Returns
    -------

    """
    file_identifiers = [files_configs[file_config].file_identifier for file_config in files_configs]
    registered_infos = get_registered_info(fspath.join(metadata_dir, WORKFLOW_FILE_STATUS), file_identifiers)
    for file_identifier in file_identifiers:
        _process_missing_dates_for_file(file_identifier=file_identifier,
                                        file_config=files_configs[file_identifier],
                                        registered_info=registered_infos[file_identifier],
                                        workflow_metadata=workflow_metadata,
                                        current_time=current_time,
                                        date_format=date_format,
                                        time_format=time_format)





```


###### FILE: src/azfr_skywalker_utils/metadata/parsing/workflowconfig.py ######

```py
from typing import Optional
from collections import defaultdict
from datetime import datetime, date
import re

from pydantic import BaseModel, Field, field_validator

class TableConfig(BaseModel):
    raise_error_if_empty_full_mode: Optional[bool] = Field(
        default=True,
        description="If True, an error will be raised if the table is empty in full mode. Default is True for safety."
    )
    raise_error_if_empty_delta_mode: Optional[bool] = Field(
        default=False,
        description="If True, an error will be raised if the table is empty in delta mode. Default is False."
    )
    raise_error_if_empty_after: Optional[date] = Field(
        default=None,
        description="A date in YYYYMMDD format. If specified, an error will be raised if the table is empty after this date."
    )

    @field_validator("raise_error_if_empty_after", mode='before')
    def validate_date_format(cls, value: str) -> Optional[date]:
        if value is None:
            return value
        if not isinstance(value, str):
            value = str(value)
        
        try:
            return datetime.strptime(value, "%Y%m%d").date()
        except ValueError:
            raise ValueError("Date must be in the format YYYYMMDD")


class FileConfig(BaseModel):
    file_identifier: str = Field(description="Identifier for the file.")
    overdue_time: str = Field(description="A time value indicating when the file is considered overdue. Format: HH:MM")
    mode: str = Field(description="The mode of the file processing. Accepted values: daily, monthly.")
    min_expected_date: str = Field(description="The earliest date (in YYYYMMDD format) from which a file was expected to be processed, typically the first day we received files or the day the workflow was deployed.")
    period_checked: int = Field(description="The number of days to check backward for expected files.")
    expected_days: list[int] = Field(default=[], description="A list of expected days. In case of monthly, days of month. In case of weekly, days of week..")
    working_days_only: bool = Field(default=False, description="If True, processing will only consider working days.")
    working_days_offset: int = Field(default=0, description="The offset (in days) to be applied when determining working days.")
    functional_date_offset: int = Field(default=0,description="The offset (in days) applied to determine the functional date of the file.")
    raise_error_if_empty_file: Optional[bool] = Field(default=False, description="If True, an error will be raised if the file is empty")
    tables: list[str] = Field(default=[], description="A list of tables that are expected in this file. This attribute is automatically filled on the workflow side, based on the table models.")
    table_configs: dict[str, TableConfig] = Field(
        default_factory=lambda: defaultdict(TableConfig),
        description="A dictionary of table-specific configurations, keyed by table name, where each value is a `TableConfig` model."
    )

    @field_validator("table_configs", mode="after")
    @classmethod
    def set_table_configs_defaultdict(cls, v):
        return defaultdict(TableConfig, v)

    @field_validator("overdue_time", mode="after")
    def validate_overdue_time(cls, value: str) -> str:
        try:
            datetime.strptime(value, "%H:%M")
        except ValueError:
            raise ValueError("Invalid format for overdue_time. Expected format is HH:MM (eg., '14:30').")
        return value


class WorkflowBaseConfig(BaseModel):
    workflow_name: str = Field(description="The name of the workflow, used to identify the workflow in metadata and logs.")
    metadata_dir: str = Field(description="The directory where metadata tables exists.")
    file_pattern: re.Pattern = Field(description="A regex pattern used to match and validate filenames for this workflow.")
    uncompressed_file_pattern: Optional[re.Pattern] = Field(default=None, description="A regex pattern used to match and validate filenames contained inside archive file (tar, zip, ...).")
    files_configs: dict[str, FileConfig] = Field(description="A dictionary of file-specific configurations, keyed by file identifiers, where each value is a `FileConfig` model.")
    date_format: str = Field(description="The date format (e.g., YYYYMMDD) used throughout the workflow.")
    archive_dir: str = Field(description="The directory where processed files are archived (/row_data).")
    input_dir: str = Field(description="The directory where new files are landed for processing.")
    tables_to_format: dict = Field(description="A dictionary mapping table names to their associated format or extraction type.")
    validate_files: bool = Field(default=True, description="Determine if the Analyzer should validate landing files.")
    date_format: str = "%Y%m%d"
    time_format: str = "%H:%M"

    @field_validator("file_pattern", "uncompressed_file_pattern", mode='before')
    def validate_pattern(cls, value: Optional[str]) -> Optional[re.Pattern]:
        if isinstance(value, str):
            return re.compile(value) # Convert string to re.Pattern
        return value

    def to_json(self, exclude={"tables_to_format"}) -> str:
        return self.model_dump_json(indent=2, exclude=exclude)

```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_dependency/config/workflow_registry.yml ######

```yml
workflows:

#### Parsing workflows
  parsing.c1_oav_coll_sante_prev_devis:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: c1_oav_coll_sante_prev

  parsing.cetip_reporting:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: cetip

  parsing.cetip_migration_sante:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: cetip_migration_sante

  parsing.cetip_genesys:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: cetip_genesys

  parsing.sidp_oav_souscription:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: sidp_oav_souscription

  parsing.sidp_financier:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: sidp_financier

  parsing.sidp_rcd:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: sidp_rcd

  parsing.sidp_prdg_asn:
    layer: parsing
    metadata_catalog: azfrdatalake_delta
    metadata_schema: sidp_prdg_asn

#### Datavault workflows
  datavault.c1_oav_coll_sante_prev_devis:
    layer: datavault
    metadata_catalog: azfrdatalake_delta
    metadata_schema: bo_datavault
    depends_on:
      - parsing.c1_oav_coll_sante_prev_devis

  datavault.cetip_reporting:
    layer: datavault
    metadata_catalog: azfrdatalake_delta
    metadata_schema: bo_datavault
    depends_on:
      - parsing.cetip_reporting

  datavault.sidp_oav_souscription:
    layer: datavault
    metadata_catalog: azfrdatalake_delta
    metadata_schema: bo_datavault
    depends_on:
      - parsing.sidp_oav_souscription

  datavault.sidp_financier:
    layer: datavault
    metadata_catalog: azfrdatalake_delta
    metadata_schema: bo_datavault
    depends_on:
      - parsing.sidp_financier

  datavault.sidp_rcd:
    layer: datavault
    metadata_catalog: azfrdatalake_delta
    metadata_schema: bo_datavault
    depends_on:
      - parsing.sidp_rcd
  
#### Extraction workflows
  extraction.distribution_y1_dataprep_sante_prevoyance_remuneration:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: distribution_y1_dataprep_sante_prevoyance_remuneration
    depends_on:
      - datavault.cetip_reporting
      - datavault.sidp_oav_souscription
  
  extraction.distribution_y1_dataprep_telephonie:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: distribution_y1_dataprep_telephonie
    depends_on:
      - parsing.cetip_genesys
  
  extraction.distribution_y1_dataprep_pilotage_force_de_vente:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: distribution_y1_dataprep_pilotage_force_de_vente
    depends_on:
      - datavault.sidp_oav_souscription
      - datavault.c1_oav_coll_sante_prev_devis
      - datavault.cetip_reporting
  
  extraction.pilotage_operationnel_datamart_prestations:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: pilotage_operationnel_datamart_prestations
    depends_on:
      - datavault.sidp_financier:
          optional: true
      - datavault.sidp_rcd:
          optional: true

  extraction.pilotage_operationnel_datamart_cotisations:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: pilotage_operationnel_datamart_cotisations
    depends_on:
      - datavault.sidp_financier:
          optional: true
      - datavault.sidp_rcd:
          optional: true
  
  extraction.analytics_y1_dtm_sante_prevoyance.asn:
    layer: extraction
    metadata_catalog: azfrdatamart_dspc_pg
    metadata_schema: datamart
    depends_on:
      - parsing.sidp_prdg_asn

  extraction.analytics_y1_dtm_sante_prevoyance.cnt:
    layer: extraction
    metadata_catalog: azfrdatamart_dspc_pg
    metadata_schema: datamart
    depends_on:
      - datavault.cetip_reporting:
          optional: true
      - datavault.c1_oav_coll_sante_prev_devis:
          optional: true
      - datavault.sidp_oav_souscription:
          optional: true
      - datavault.sidp_rcd:
          optional: true

  extraction.analytics_y1_dtm_sante_prevoyance.flux:
    layer: extraction
    metadata_catalog: azfrdatamart_dspc_pg
    metadata_schema: datamart
    depends_on:
      - extraction.pilotage_operationnel_datamart_prestations
      - extraction.pilotage_operationnel_datamart_cotisations
  
  extraction.distribution_y1_dataprep_sante_prev_reconcil_comptable:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: distribution_y1_dataprep_sante_prev_reconcil_comptable
    depends_on:
      - datavault.sidp_financier
      - datavault.sidp_rcd
      - extraction.pilotage_operationnel_datamart_prestations
      - extraction.pilotage_operationnel_datamart_cotisations
  
  extraction.mkg_y1_dataprep_connaissance_clients:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: mkg_y1_dataprep_connaissance_clients
    depends_on:
      - datavault.cetip_reporting
      - datavault.sidp_oav_souscription

  extraction.datamart_interessement_des_collectives.annuel:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: dtm_interessement_des_collectives
    depends_on:
      - datavault.cetip_reporting
      - datavault.sidp_oav_souscription

  extraction.datamart_interessement_des_collectives.mensuel:
    layer: extraction 
    metadata_catalog: azfrdatalake_delta
    metadata_schema: dtm_interessement_des_collectives 
    depends_on:
      - datavault.cetip_reporting
      - datavault.sidp_oav_souscription

  extraction.datamart_interessement_des_collectives.mensuel_ant:
    layer: extraction 
    metadata_catalog: azfrdatalake_delta
    metadata_schema: dtm_interessement_des_collectives
    depends_on:
      - datavault.cetip_reporting
  
  extraction.distribution_y1_datamart_sante_prevoyance_adps_echosante:
    layer: extraction
    metadata_catalog: azfrdatalake_delta
    metadata_schema: adps_y1_dtm_sante_prevoyance_adps_echosante
    depends_on:
      - datavault.cetip_reporting
      - datavault.sidp_oav_souscription
```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_dependency/version_provider.py ######

```py
import logging

from abc import ABC, abstractmethod
from collections import defaultdict
from collections.abc import Sequence
from enum import Enum
from typing import Optional

from sqlalchemy import Engine, RowMapping, text

from azfr_skywalker_utils.metadata.workflow_run.abstract import WorkflowGlobalStatus 
from azfr_skywalker_utils.metadata.workflow_run.datavault import DatasetDetailedStatus
from azfr_skywalker_utils.metadata.workflow_run.extraction import ExtractionDetailedStatus
from azfr_skywalker_utils.metadata.parsing.metadata import FileStatus, FileDetailedStatus, ParsingDomainStatus, ParsingDetailedStatus
from azfr_skywalker_utils.utils.helpers import max_with_none
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import WORKFLOW_REGISTRY


logger = logging.getLogger(__name__)

ALL_VERSIONS_CONDITION = "1=1 /* version is None, retrive all rows. */"


class VersionStrategy(str, Enum):
    """Strategy for determining which versions to process.

    ENSURE_ORDER: Return successful versions in order, stopping at the first non successful version.
                  Ensures sequential processing order.

    ALL_AVAILABLE: Return all successful versions. As long as a version is successful across all dependencies,
                  allow processing of that version even if some anterior versions have failed.
    """
    ENSURE_ORDER = "ensure_order"
    ALL_AVAILABLE = "all_available"


class VersionStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"
    WAITING = "WAITING"    
    EMPTY = "EMPTY"

    
class WorkflowVersionProvider(ABC):
    """ Abstract base class for resolving workflow dependencies versions."""

    def __init__(self, workflow_name: str, engine: Engine):
        self.workflow_entry = WORKFLOW_REGISTRY.get_workflow_config(workflow_name)
        self.engine = engine

    @abstractmethod
    def get_last_successful_version(self) -> Optional[str]:
        """ Retrieve the max version for which the workflow execution was successful."""
        pass

    @abstractmethod
    def _collect_all_versions(self, min_version: Optional[str] = None) -> list[dict]:
        """ Retrieve all versions (successful or not) greater than given version. """
        pass

    @abstractmethod
    def _is_version_successful(self, version_data: RowMapping) -> bool:
        """ Check if a specific version is successful based on its attributes. """
        pass

    @abstractmethod
    def _is_empty(self, detailed_status: str | list[str]) -> bool:
        """ Check if a specific version is empty based on its detailed status.
        
        Args:
            detailed_status: Either a single status string or a list of status strings
        
        Returns:
            True if the version should be considered empty, False otherwise
        """
        pass

    @staticmethod
    def _build_version_condition(min_version: Optional[str]) -> str:
        if min_version is None:
            return ALL_VERSIONS_CONDITION
        return f"version > '{min_version}'"

    def _filter_successful_versions(self, versions_list: list[dict], strategy: VersionStrategy) -> list[dict]:
        """Filter a list of versions to keep only successful ones.
        
        Args:
            versions_list: List of version dicts from _collect_all_versions
            strategy: Strategy for handling version order
            
        Returns:
            List of successful versions with 'version' and 'dependency_status' keys
        """
        if not versions_list:
            return []

        sorted_versions = sorted(versions_list, key=lambda x: x['version'])
        successful_versions = []

        for version_data in sorted_versions:                            
            if self._is_version_successful(version_data):
                detailed_status = version_data.get('domain_detail_status') or version_data.get('detailed_status')
                successful_versions.append({
                    'version': version_data['version'],
                    'dependency_status': detailed_status})
            else:
                if strategy == VersionStrategy.ENSURE_ORDER: # log only global_status for subsequent versions
                    remaining_versions = [{
                        'version': v['version'], 
                        'dependency_status': v.get('domain_status') or v.get('global_status')
                    } for v in sorted_versions[sorted_versions.index(version_data):]]                    
                    logger.warning(f"Version '{version_data['version']}' is unsuccessful. Ignoring it and all subsequent versions.")
                    logger.warning(f"Versions ignored: {remaining_versions}")
                    break

        return successful_versions

    def get_all_successful_versions(self, min_version: Optional[str], strategy: VersionStrategy) -> list[dict]:
        """ Retrieve all successful versions since a given version. 
        
        Args:
            min_version: Minimum version to start from
            strategy: Strategy for handling version order
        """
        versions_list = self._collect_all_versions(min_version)
        return self._filter_successful_versions(versions_list, strategy)
    
    def _collect_dependencies_versions(self, min_version: Optional[str], strategy: VersionStrategy) -> dict[str, list[dict]]:
        """
        Collect successful versions from all dependencies.
        Returns a dictionary mapping dependency names to their version sets.
        Also stores all versions (including failed) for optional dependency checking.
        """
        logger.info(f"Processing dependencies for workflow {self.workflow_entry.name}. Dependencies: {self.workflow_entry.get_dependency_names()}")
        dependencies_versions = {}
        # Store all versions including failed ones for optional dependency validation
        self._all_dependency_versions = {}
        
        for dependency in self.workflow_entry.depends_on:
            logger.info(f"Dependency {dependency.workflow_name} - Seeking successful versions greater than {min_version}")
            try:
                dependency_provider = create_provider(dependency.workflow_name, self.engine)
                
                # Get all versions (including failed) once
                all_versions = dependency_provider._collect_all_versions(min_version)
                self._all_dependency_versions[dependency.workflow_name] = all_versions
                
                # Filter to successful versions using extracted method (avoids code duplication)
                dependency_versions = dependency_provider._filter_successful_versions(all_versions, strategy)
                
                dependencies_versions[dependency.workflow_name] = dependency_versions
                if not dependency_versions:
                    logger.warning(f"Dependency {dependency.workflow_name} - No successful versions found after {min_version}. strategy: {strategy.value}")
                else:
                    # Extract and log only version strings
                    version_strings = [v['version'] for v in dependency_versions]
                    logger.info(f"Dependency {dependency.workflow_name} - Versions found: {version_strings}")
            except Exception as e:
                raise RuntimeError(f"Dependency {dependency.workflow_name} - Error while trying to get versions. Error: {e}")

        return dependencies_versions

    def _unique_versions(self, dependencies_versions: dict[str, list[dict]]) -> set[str]:
        """ Returns a set containing the union of all versions across dependencies. """
        all_versions = set()
        for version_list in dependencies_versions.values():
            all_versions.update(v['version'] for v in version_list)
        return all_versions

    def _optional_dependencies(self) -> dict[str, bool]:
        return {dep.workflow_name: dep.optional for dep in self.workflow_entry.depends_on}

    def _is_optional_dependency_explicitly_failed(self, dependency_name: str, version: str) -> bool:
        all_versions_for_dep = getattr(self, '_all_dependency_versions', {}).get(dependency_name, [])
        failed_version = next((v for v in all_versions_for_dep if v['version'] == version), None)
        return bool(failed_version and not self._is_version_successful(failed_version))

    def _evaluate_version(
            self,
            version: str,
            dependencies_versions: dict[str, list[dict]],
            optional_dependencies: dict[str, bool],
            all_deps_optional: bool,
    ) -> dict:
        present_count = 0
        missing_optional_count = 0
        dependency_statuses: list[str] = []

        for dependency_name, version_list in dependencies_versions.items():
            version_data = next((v for v in version_list if v['version'] == version), None)
            is_optional = optional_dependencies.get(dependency_name, False)

            if version_data is None:
                if not is_optional:
                    return {'skip_reason': f"required dependency {dependency_name} is missing"}

                if self._is_optional_dependency_explicitly_failed(dependency_name, version):
                    logger.warning(f"Version {version} - Optional dependency {dependency_name} has explicitly FAILED")
                    return {'skip_reason': f"optional dependency {dependency_name} has FAILED"}

                logger.info(f"Version {version} - Optional dependency {dependency_name} is missing (delay)")
                missing_optional_count += 1
                continue

            dependency_status = version_data['dependency_status']
            present_count += 1
            if dependency_status == VersionStatus.FAILED.value:
                logger.warning(f"Version {version} - Dependency {dependency_name} has FAILED status")
                return {'skip_reason': "at least one dependency has FAILED"}

            dependency_statuses.append(dependency_status)

        if all_deps_optional and present_count == 0:
            return {'skip_reason': "all dependencies are optional and none are present"}

        if not dependency_statuses:
            return {'skip_reason': "no dependency status available"}

        aggregated_status = (
            VersionStatus.EMPTY.value
            if all(status == VersionStatus.EMPTY.value for status in dependency_statuses)
            else VersionStatus.SUCCESS.value
        )

        return {
            'version': version,
            'dependencies_status': aggregated_status,
            'missing_optional_count': missing_optional_count,
            'skip_reason': None,
        }

    @staticmethod
    def _must_break_for_strategy(strategy: VersionStrategy) -> bool:
        return strategy == VersionStrategy.ENSURE_ORDER

    def _find_valid_versions_across_dependencies(self, dependencies_versions: dict[str, list[dict]], strategy: VersionStrategy) -> list[dict]:
        """
        Args:
                dependencies_versions: Dict mapping dependency names to list of version dicts
                strategy: Strategy for handling missing versions
            
        Returns:
            List of dicts with:
            - 'version': str - The version identifier
            - 'dependencies_status': VersionStatus - Aggregated status
        """
        all_versions = self._unique_versions(dependencies_versions)
        if not all_versions:
            logger.warning("No versions found across all dependencies")
            return []

        valid_versions = []

        optional_dependencies = self._optional_dependencies()
        all_deps_optional = all(dep.optional for dep in self.workflow_entry.depends_on)

        for version in sorted(all_versions):
            version_eval = self._evaluate_version(
                version=version,
                dependencies_versions=dependencies_versions,
                optional_dependencies=optional_dependencies,
                all_deps_optional=all_deps_optional,
            )

            if version_eval['skip_reason']:
                logger.warning(f"Version {version} will be ignored because {version_eval['skip_reason']}")
                if self._must_break_for_strategy(strategy):
                    break
                continue

            valid_versions.append({
                'version': version_eval['version'],
                'dependencies_status': version_eval['dependencies_status'],
                'missing_optional_count': version_eval['missing_optional_count']
            })

        return valid_versions

    def get_valid_dependencies_versions(self, min_version: Optional[str], strategy: VersionStrategy) -> list[dict]:
        """ Return the list of versions that are successful across all workflow dependencies. """
        if not self.workflow_entry.depends_on:
            logger.info(f"Workflow {self.workflow_entry.name} has no dependencies (param: depends_on)")
            return []

        dependencies_versions = self._collect_dependencies_versions(min_version, strategy)
        valid_versions = self._find_valid_versions_across_dependencies(dependencies_versions, strategy)
        return valid_versions


class ParsingVersionProvider(WorkflowVersionProvider):
    """ Version provider for Parsing Workflows. """

    def get_last_successful_version(self) -> str:
        """Not implemented for ParsingVersionProvider."""
        raise NotImplementedError("""ParsingVersionProvider doesn't implement get_last_successful_version() because Parsing WF don't have dependencies.""")

    def _get_all_files_status(self, min_version: Optional[str] = None) -> Sequence[RowMapping]:
        """ Retrieve the most recent export date for which the workflow execution was successful. """
        version_condition = self._build_version_condition(min_version)

        query = f"""
        with status_rn as (
            select *, row_number() over (partition by version, file_identifier order by start_ts desc) as rn
            from {self.workflow_entry.metadata_catalog}.{self.workflow_entry.metadata_schema}._workflow_file_status
            where 1=1
                and {version_condition}
                and workflow_name = '{self.workflow_entry.name}'
            )
        select version, file_identifier, global_status, detailed_status
        from status_rn
        where rn = 1
        order by version
        """
        with self.engine.connect() as conn:
            result = conn.execute(text(query)).mappings().fetchall()
            return result

    def _compute_domain_status(self, file_status_list: Sequence[RowMapping]) -> list[dict]:
        """ Compute the domain status based on file statuses.
        This function takes a list of file statuses and computes the global status for each version.
        """
        date_results = []

        if not file_status_list:
            return date_results

        # Group files by version
        date_status_map = defaultdict(list)
        date_detailed_status_map = defaultdict(list)
        for file_status in file_status_list:
            date_status_map[file_status.version].append(file_status.global_status)
            date_detailed_status_map[file_status.version].append(file_status.detailed_status)

        for version, global_status_list in date_status_map.items():
            detailed_status_list = date_detailed_status_map[version]
            if FileStatus.FAILED.value in global_status_list:
                domain_status = ParsingDomainStatus.FAILED.value
                domain_detail_status = ParsingDetailedStatus.FAILED.value
            elif FileStatus.WAITING.value in global_status_list:
                domain_status = ParsingDomainStatus.WAITING.value
                domain_detail_status = ParsingDetailedStatus.WAITING.value
            elif all(status == FileStatus.SUCCESS.value for status in global_status_list):
                domain_status = ParsingDomainStatus.SUCCESS.value
                if self._is_empty(detailed_status_list):
                    domain_detail_status = ParsingDetailedStatus.EMPTY.value
                else:
                    domain_detail_status = ParsingDetailedStatus.SUCCESS.value
            else:
                unknown_status = set(global_status_list).difference([FileStatus.FAILED.value, FileStatus.WAITING.value, FileStatus.SUCCESS.value])
                raise ValueError(f"Dependency {self.workflow_entry.name} - Unexpected FileStatus for version {version}: {unknown_status}")

            date_results.append({
                'version': version,
                'domain_status': domain_status,
                'domain_detail_status': domain_detail_status
            })

        return date_results

    def _collect_all_versions(self, min_version: Optional[str] = None) -> list[dict]:
        file_status_list = self._get_all_files_status(min_version)
        domain_status_list = self._compute_domain_status(file_status_list)
        return domain_status_list

    def _is_version_successful(self, version_data: RowMapping) -> bool:
        """ Check if a version is successful for parsing workflow. """
        return version_data['domain_status'] == ParsingDomainStatus.SUCCESS.value

    def _is_empty(self, detailed_status: str | list[str]) -> bool:
        """ Check if a version is empty for parsing workflow. """
        detailed_status_list = [detailed_status] if isinstance(detailed_status, str) else detailed_status
        return all(detailed in (FileDetailedStatus.FILE_EMPTY.value, FileDetailedStatus.FILE_NO_ROWS.value)
                   for detailed in detailed_status_list)


class DatavaultVersionProvider(WorkflowVersionProvider):
    """ Version provider for Datavault Workflows. """

    def get_last_successful_version(self) -> Optional[str]:
        """ Retrieve the max version for which the workflow execution was successful."""
        query = f"""
            select max(version) as version
            from {self.workflow_entry.metadata_catalog}.{self.workflow_entry.metadata_schema}._workflow_run
            where 1=1
                and workflow_name = '{self.workflow_entry.name}'
                and global_status = '{WorkflowGlobalStatus .SUCCESS.value}'
                and detailed_status not in ('{DatasetDetailedStatus.INSERT_OR_POST_TEST_DISABLED.value}')
        """
        with self.engine.connect() as conn:
            result = conn.execute(text(query)).fetchone()
            if not result or result[0] is None:
                logger.warning(f"No successful runs found for workflow {self.workflow_entry.name}.")
                return None
            return result[0]

    def _collect_all_versions(self, min_version: Optional[str] = None) -> list[dict]:
        """
        Retrieve all successful dates since a given version.
        Returns a list of strings in the format 'YYYYMMDD'.
        """
        version_condition = self._build_version_condition(min_version)

        query = f"""
        with status_rn as (
            select *, row_number() over (partition by version order by start_ts desc) as rn
            from {self.workflow_entry.metadata_catalog}.{self.workflow_entry.metadata_schema}._workflow_run
            where 1=1
                and workflow_name = '{self.workflow_entry.name}'
            --  and global_status = '{WorkflowGlobalStatus .SUCCESS.value}'
            --  and detailed_status not in ('{DatasetDetailedStatus.INSERT_OR_POST_TEST_DISABLED.value}')
                and {version_condition}
            )
        select version, global_status, detailed_status
        from status_rn
        where rn = 1
        order by version
        """

        with self.engine.connect() as conn:
            result = conn.execute(text(query)).mappings().fetchall()
            if not result:
                logger.warning(f"No successful runs found for workflow {self.workflow_entry.name} since version {min_version if min_version is not None else 'None'}.")
            
            status_list = []
            for row in result:
                status_list.append({
                    'version': row['version'],
                    'global_status': row['global_status'],
                    'detailed_status': row['detailed_status']
                })
            return status_list

    def _is_version_successful(self, version_data: RowMapping) -> bool:
        """ Check if a version is successful for datavault workflow. """
        return (version_data['global_status'] == WorkflowGlobalStatus .SUCCESS.value
                and version_data['detailed_status'] != DatasetDetailedStatus.INSERT_OR_POST_TEST_DISABLED.value)

    def _is_empty(self, detailed_status: str | list[str]) -> bool:
        """Datavault workflows don't have a concept of empty versions."""
        return False


class ExtractionVersionProvider(WorkflowVersionProvider):
    """ Version provider for Extraction Workflows. """

    def get_last_successful_version(self) -> Optional[str]:
        """ Retrieve the max version for which the workflow execution was successful."""
        query = f"""
            select max(version) as version
            from {self.workflow_entry.metadata_catalog}.{self.workflow_entry.metadata_schema}._workflow_run
            where 1=1
                and workflow_name = '{self.workflow_entry.name}'
                and global_status = '{WorkflowGlobalStatus .SUCCESS.value}'
        """
        with self.engine.connect() as conn:
            result = conn.execute(text(query)).fetchone()
            if not result or result[0] is None:
                logging.warning(f"No successful runs found for workflow {self.workflow_entry.name}.")
                return None
            return result[0]

    def _collect_all_versions(self, min_version: Optional[str] = None) -> list[dict]:
        """
        Retrieve all successful dates since a given version.
        Returns a list of strings in the format 'YYYYMMDD'.
        """
        version_condition = self._build_version_condition(min_version)

        query = f"""
        with status_rn as (
            select *, row_number() over (partition by version order by start_ts desc) as rn
            from {self.workflow_entry.metadata_catalog}.{self.workflow_entry.metadata_schema}._workflow_run
            where 1=1
                and workflow_name = '{self.workflow_entry.name}'
            --  and global_status = '{WorkflowGlobalStatus .SUCCESS.value}'
                and {version_condition}
            )
        select version, global_status, detailed_status
        from status_rn
        where rn = 1
        order by version
        """

        with self.engine.connect() as conn:
            result = conn.execute(text(query)).mappings().fetchall()
            if not result:
                logging.warning(f"No successful runs found for workflow {self.workflow_entry.name} since version {min_version if min_version is not None else 'None'}.")
            status_list = []
            for row in result:
                detailed_status = VersionStatus.EMPTY.value if self._is_empty(row['detailed_status']) else row['detailed_status']
                status_list.append({
                    'version': row['version'],
                    'global_status': row['global_status'],
                    'detailed_status': detailed_status
                })
            return status_list

    def _is_version_successful(self, version_data: RowMapping) -> bool:
        """ Check if a version is successful for datavault workflow. """
        return (version_data['global_status'] == WorkflowGlobalStatus .SUCCESS.value)

    def _is_empty(self, detailed_status: str | list[str]) -> bool:
        """ Check if extraction version is empty. """
        empty_statuses = (ExtractionDetailedStatus.DEPENDENCIES_EMPTY.value, ExtractionDetailedStatus.NO_CHANGES.value)
        if isinstance(detailed_status, str):
            return detailed_status in empty_statuses
        return all(status in empty_statuses for status in detailed_status)


def create_provider(workflow_name: str, engine: Engine) -> WorkflowVersionProvider:
    """ Create the appropriate version provider class based on workflow name."""
    if workflow_name.startswith("parsing."):
        return ParsingVersionProvider(workflow_name, engine)
    elif workflow_name.startswith("datavault."):
        return DatavaultVersionProvider(workflow_name, engine)
    elif workflow_name.startswith("extraction."):
       return ExtractionVersionProvider(workflow_name, engine)
    else:
        raise ValueError(f"Unknown workflow type for {workflow_name}. Cannot create provider.")


def get_versions_auto(workflow_name: str, engine: Engine, strategy: VersionStrategy, min_version: Optional[str] = None) -> list[dict]:
    """ Get a sorted list of successful versions across all dependencies for a given workflow. 
    Returns:
        List of dicts with 'version' and 'dependencies_status' keys.    
    """
    provider = create_provider(workflow_name, engine)
    if strategy == VersionStrategy.ENSURE_ORDER:
        workflow_last_successful_version = provider.get_last_successful_version()
        effective_min_version = max_with_none(workflow_last_successful_version, min_version)
        valid_versions = provider.get_valid_dependencies_versions(effective_min_version, strategy)
    elif strategy == VersionStrategy.ALL_AVAILABLE:
        already_processed_versions = provider.get_all_successful_versions(min_version, strategy)
        # Extract version strings from dict list
        already_processed_version_strings = [v['version'] for v in already_processed_versions]
            
        valid_versions_with_status = provider.get_valid_dependencies_versions(min_version, strategy)
        
        # Filter out already processed versions
        valid_versions = [
            v for v in valid_versions_with_status 
            if v['version'] not in already_processed_version_strings
        ]
    else:
        raise ValueError(f"Unsupported version strategy: {strategy}")

    return sorted(valid_versions, key=lambda x: x['version'])

```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_dependency/workflow_registry.py ######

```py
import os 
from typing import Optional
from enum import Enum
from pydantic import BaseModel, Field, field_validator, model_validator


class LayerType(str, Enum):
    """Enum for different layer types in the data architecture."""
    PARSING = "parsing"
    DATAVAULT = "datavault"
    EXTRACTION = "extraction"


class Dependency(BaseModel):
    """Configuration for a workflow dependency with optional parameters."""
    workflow_name: str = Field(description="Name of the workflow this workflow depends on")
    ignore_empty_version: Optional[bool] = Field(default=False, description="Whether to ignore empty versions for this dependency")
    optional: Optional[bool] = Field(default=False, description="Whether this dependency is optional (workflow can proceed if dependency has delay but not if it failed)")


class WorkflowEntry(BaseModel):
    """Base model for a workflow and it's dependencies."""
    name: str = Field(description="Name of the workflow")
    layer: LayerType = Field(description="The layer this workflow operates on")
    metadata_catalog: str = Field(description="Metadata catalog where this workflow's data is stored")
    metadata_schema: str = Field(description="Metadata schema where this workflow's data is stored")
    depends_on: list[Dependency] = Field(default_factory=list, description="List of workflow dependencies")
    
    @field_validator('layer', mode="before")
    def validate_layer(cls, v):
        """Convert string layers to enum values."""
        return LayerType(v) if isinstance(v, str) else v
    
    @field_validator('depends_on', mode="before")
    def validate_depends_on(cls, value):
        """Transform input format to internal Dependency objects."""
        if not value:
            return value
        
        result = []
        for dep in value:
            if isinstance(dep, str):
                result.append(Dependency(workflow_name=dep))
            elif isinstance(dep, dict):
                if len(dep) != 1:
                    raise ValueError(f"Dependency dict must have exactly one workflow name key: {dep}")
                (workflow_name, params), = dep.items()
                result.append(Dependency(workflow_name=workflow_name, **params))
        return result
    
    def get_dependency_names(self) -> list[str]:
        """Get list of workflow names this workflow depends on."""
        return [dep.workflow_name for dep in self.depends_on]


class WorkflowRegistry(BaseModel):
    """Registry containing all workflow dependencies."""
    workflows: dict[str, WorkflowEntry] = Field(description="Dictionary mapping workflow names to their dependencies")
    
    @model_validator(mode='after')
    def validate_dependencies(self):
        """Validate that all dependencies reference known workflows."""
        known_workflows = set(self.workflows.keys())
        
        for workflow_name, workflow_config in self.workflows.items():
            if workflow_config.depends_on:
                for dependency in workflow_config.depends_on:
                    if dependency.workflow_name not in known_workflows:
                        raise ValueError(
                            f"Workflow '{workflow_name}' depends on unknown workflow '{dependency.workflow_name}'. "
                            f"Known workflows: {sorted(known_workflows)}"
                        )
        return self
    
    def get_workflow_config(self, workflow_name: str) -> WorkflowEntry:
        """Get configuration for a specific workflow."""
        if workflow_name not in self.workflows:
            raise ValueError(f"Workflow not found in WorkflowRegistry: {workflow_name}")
        return self.workflows[workflow_name]
    
    def get_workflows_by_layer(self, layer_type: LayerType) -> list[str]:
        """Get all workflow names that operate on a specific layer."""
        return [name for name, config in self.workflows.items() if config.layer == layer_type]
    
    def get_workflows_depending_on(self, source: str) -> list[str]:
        """Get all workflow names that depend on a specific source."""
        result = []
        for workflow_name, config in self.workflows.items():
            if config.depends_on:
                dependency_workflows = config.get_dependency_names()
                if source in dependency_workflows:
                    result.append(workflow_name)
        return result

    @classmethod
    def from_dict(cls, config_dict: dict[str, dict]) -> "WorkflowRegistry":
        """Load workflow dependencies from a dictionary. Handle both flat and nested structures:
        If the dictionary contains a "workflows" key, its value is used as the workflows mapping.
        Otherwise, the entire dictionary is treated as the workflows mapping."""
        if "workflows" in config_dict:
            workflows_data = config_dict["workflows"]
        else:
            workflows_data = config_dict
            
        workflows = {}
        for name, config in workflows_data.items():
            # Add the name field if not present
            if "name" not in config:
                config = dict(config)  # Make a copy to avoid modifying original
                config["name"] = name
            workflows[name] = WorkflowEntry(**config)
        return cls(workflows=workflows)

    @classmethod
    def from_yaml(cls, file_path: str) -> "WorkflowRegistry":
        """Load workflow dependencies from a YAML file."""
        try:
            import yaml
        except ImportError:
            raise ImportError("PyYAML is required to load from YAML files. Install it with: pip install pyyaml")
        
        with open(file_path, 'r') as f:
            config_dict = yaml.safe_load(f)
        return cls.from_dict(config_dict)


WORKFLOW_REGISTRY_PATH = os.path.abspath(os.path.join(os.path.dirname(__file__), "config/workflow_registry.yml"))
WORKFLOW_REGISTRY = WorkflowRegistry.from_yaml(WORKFLOW_REGISTRY_PATH)

```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_run/abstract.py ######

```py
import os
import uuid
import logging
from abc import ABC, ABCMeta, abstractmethod
from enum import Enum
from datetime import datetime
from typing import Any, Optional, Type

from prefect.client.schemas.objects import State
from sqlalchemy import String, DateTime, Enum as SAEnum
from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase
from sqlalchemy.engine import Engine

from azfr_skywalker_utils.dbt.common import RunMode
from azfr_skywalker_utils.trino.sql_alchemy import trino_session


logger = logging.getLogger(__name__)


class BaseDetailedStatus(str, Enum):
    pass


class WorkflowGlobalStatus(str, Enum):
    SUCCESS = "SUCCESS"
    FAILED = "FAILED"


class CombinedMeta(ABCMeta, type(DeclarativeBase)):
    """Custom metaclass that combines ABCMeta with SQLAlchemy's DeclarativeBase metaclass."""
    pass


class AbstractDeclarativeBase(ABC, DeclarativeBase, metaclass=CombinedMeta):
    """Base class that combines SQLAlchemy's DeclarativeBase with ABC functionality."""
    __abstract__ = True


class AbstractWorkflowRun(AbstractDeclarativeBase):
    __tablename__ = "_workflow_run"
    run_id: Mapped[str] = mapped_column(String, primary_key=True, nullable=True)
    workflow_name: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    version: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    global_status: Mapped[Optional[WorkflowGlobalStatus]] = mapped_column(SAEnum(WorkflowGlobalStatus), nullable=True)
    detailed_status: Mapped[Optional[BaseDetailedStatus]] = mapped_column(String, nullable=True)
    start_ts: Mapped[Optional[datetime]] = mapped_column(DateTime, nullable=True)
    end_ts: Mapped[Optional[datetime]] = mapped_column(DateTime, nullable=True)
    run_mode: Mapped[Optional[RunMode]] = mapped_column(SAEnum(RunMode), nullable=True)
    additional_info_json: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    creator_github_repository: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    creator_commit_id: Mapped[Optional[str]] = mapped_column(String, nullable=True)    

    @property
    @abstractmethod
    def detailed_status_enum(self) -> Type[Any]:
        """Return the detailed status enum class for this workflow type."""
        pass

    @abstractmethod
    def create_table(self, engine: Engine) -> None:
        """Create the underlying table in the target engine (DDL)."""
        pass


class AbstractWorkflowRunService:
    """Abstract service for managing run metadata lifecycle and persistence.
        Concrete services must implement `get_metadata_class()` and provide a concrete
        `AbstractWorkflowRun` subclass that declares its `detailed_status` column and
        `detailed_status_enum` property."""
    def __init__(self, metadata: AbstractWorkflowRun, engine: Engine, create_table: bool = False):
        self.metadata = metadata
        self.engine = engine

        logger.info(f"WorkflowRunService.run_id: {self.metadata.run_id}")

        if create_table:
            self.metadata.create_table(self.engine)
            logger.info(f"Created table: {self.metadata.__tablename__}")

    def __enter__(self):
        self.metadata.run_id = str(uuid.uuid4())
        self.metadata.start_ts = datetime.now()
        self.metadata.creator_github_repository = os.environ.get("GITHUB_REPOSITORY")
        self.metadata.creator_commit_id = os.environ.get("GITHUB_SHA")
        logger.info(f"WorkflowRunService initialized. run_id : {self.metadata.run_id}")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.metadata.end_ts = datetime.now()
        try:
            if exc_type is None:  # No exception occurred
                self.mark_success()
            else:
                logger.error(f"Workflow failed with exception: {exc_type.__name__}: {exc_val}", exc_info=(exc_type, exc_val, exc_tb))
                self.mark_failed(self.metadata.detailed_status_enum.TECHNICAL_ERROR)
            self.save()
        except Exception:
            logger.error(f"Error saving run metadata. RunMetadata values : {self.metadata}")
            raise

    @classmethod
    @abstractmethod
    def get_metadata_class(cls) -> Type[AbstractWorkflowRun]:
        """Return the concrete ORM model class for this service type."""
        raise NotImplementedError


    @classmethod
    def from_dict(cls, init_dict: dict, engine: Engine) -> "AbstractWorkflowRunService":
        """Create a service instance from provided initialization values.
        Filters out unexpected fields and honors `create_metadata_tables` flag.
        """
        metadata_cls = cls.get_metadata_class()
        model_fields = metadata_cls.__table__.columns.keys()
        extra_fields = set(init_dict) - set(model_fields) - {"create_metadata_tables"}
        if extra_fields:
            logger.warning(f"Ignoring unexpected fields in metadata: {extra_fields}")

        metadata = metadata_cls(**{k: v for k, v in init_dict.items() if k in model_fields})
        create_table = init_dict.get("create_metadata_tables", False)
        return cls(metadata, engine, create_table)

    def fail_if_not_completed(self, state: State, detailed_status: BaseDetailedStatus, raise_exception: bool = True):
        """Check if a Prefect state is completed, mark as failed with given status if not.
        
        Args:
            state: Prefect State object to check
            detailed_status: Any enum that inherits from BaseDetailedStatus (e.g., DatasetDetailedStatus, ExtractionDetailedStatus)
            raise_exception: Whether to raise an exception if the state is not completed
        """
        if not isinstance(state, State):
            raise TypeError(f"Expected Prefect `State`, got type {type(state)}")
        if not state.is_completed():
            self.mark_failed(detailed_status)
            if raise_exception:
                raise RuntimeError(f"Task failed with detailed status: {detailed_status.value}")

    def mark_success(self, overwrite: bool = False):
        """Mark the workflow as successful."""
        self.metadata.global_status = WorkflowGlobalStatus.SUCCESS
        self.set_detailed_status(self.metadata.detailed_status_enum.SUCCESS, overwrite)

    def mark_failed(self, detailed_status: BaseDetailedStatus, overwrite: bool = False):
        """Mark the workflow as failed with the given detailed status.
        
        Args:
            detailed_status: Any enum that inherits from BaseDetailedStatus (e.g., DatasetDetailedStatus, ExtractionDetailedStatus)
            overwrite: Whether to overwrite an existing detailed status
        """
        self.metadata.global_status = WorkflowGlobalStatus.FAILED
        self.set_detailed_status(detailed_status, overwrite)            

    def set_detailed_status(self, detailed_status: BaseDetailedStatus, overwrite: bool = False):
        """Set the detailed status of the workflow run.
        
        Args:
            detailed_status: Any enum that inherits from BaseDetailedStatus (e.g., DatasetDetailedStatus, ExtractionDetailedStatus)
            overwrite: If False and a detailed status is already set, it will not change the existing status.
                      If True, it will override the existing detailed status.
        """
        if not isinstance(detailed_status, self.metadata.detailed_status_enum):
            raise ValueError(f"Invalid detailed status '{detailed_status}'. Expected instance of {self.metadata.detailed_status_enum.__name__}")

        if self.metadata.detailed_status is None or overwrite:
            self.metadata.detailed_status = detailed_status
        else:
            logger.warning(
                f"New status '{detailed_status}' was not applied because an existing detailed status is already set ('{self.metadata.detailed_status}')."
                f"Call mark_failed with overwrite=True to override."
            )

    def save(self):
        """Save the workflow run metadata to the database."""
        with trino_session(self.engine) as session:
            session.add(self.metadata)
            session.commit()

```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_run/datavault.py ######

```py
from typing import Optional, Type
from sqlalchemy import String, text
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.engine import Engine

from azfr_skywalker_utils.metadata.workflow_run.abstract import AbstractWorkflowRun, AbstractWorkflowRunService, BaseDetailedStatus

class DatasetDetailedStatus(BaseDetailedStatus):
    SUCCESS = "SUCCESS"
    PRE_TEST_FAILED = "PRE_TEST_FAILED"
    INSERT_FAILED = "INSERT_FAILED"
    POST_TEST_FAILED = "POST_TEST_FAILED"
    INSERT_OR_POST_TEST_DISABLED = "INSERT_OR_POST_TEST_DISABLED"
    TECHNICAL_ERROR = "TECHNICAL_ERROR"
    ADMIN_FAILED = "ADMIN_FAILED"
    ADMIN_SUCCESS = "ADMIN_SUCCESS" 


class WorkflowRunDatavault(AbstractWorkflowRun):
    dataset: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    detailed_status: Mapped[Optional[DatasetDetailedStatus]] = mapped_column(use_existing_column=True)

    @property
    def detailed_status_enum(self) -> Type[DatasetDetailedStatus]:
        return DatasetDetailedStatus

    def create_table(self, engine: Engine):
        sql_str = f"""
        CREATE TABLE IF NOT EXISTS {self.__tablename__} (
            run_id varchar /*PRIMARY KEY -- Generated by SQL Alchemy ORM, but incompatible with Trino */,
            workflow_name varchar,
            dataset varchar,
            version varchar,
            global_status varchar,
            detailed_status varchar,
            start_ts timestamp(0),
            end_ts timestamp(0),
            run_mode varchar,
            additional_info_json varchar,
            creator_github_repository varchar,
            creator_commit_id varchar
        )
        """
        with engine.connect() as conn:
            conn.execute(text(sql_str))


class DatavaultWorkflowRunService(AbstractWorkflowRunService):

    @classmethod
    def get_metadata_class(cls) -> Type[WorkflowRunDatavault]:
        return WorkflowRunDatavault

```

###### FILE: src/azfr_skywalker_utils/metadata/workflow_run/extraction.py ######

```py
from typing import Optional, Type, List
from sqlalchemy import String, text, ARRAY
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.engine import Engine

from azfr_skywalker_utils.metadata.workflow_run.abstract import AbstractWorkflowRun, AbstractWorkflowRunService, BaseDetailedStatus


class ExtractionDetailedStatus(BaseDetailedStatus):
    SUCCESS = "SUCCESS"
    EXTRACTION_FAILED = "EXTRACTION_FAILED"
    POST_TEST_FAILED = "POST_TEST_FAILED"
    GENERATE_FILE_FAILED = "GENERATE_FAILED"
    TECHNICAL_ERROR = "TECHNICAL_ERROR"
    DEPENDENCIES_EMPTY = "DEPENDENCIES_EMPTY"
    DEPENDENCIES_INCOMPLETE = "DEPENDENCIES_INCOMPLETE"
    NO_CHANGES = "NO_CHANGES"


class WorkflowRunExtract(AbstractWorkflowRun):
    detailed_status: Mapped[Optional[ExtractionDetailedStatus]] = mapped_column(use_existing_column=True)
    sources_max_date: Mapped[Optional[str]] = mapped_column(String, nullable=True)
    generated_files: Mapped[Optional[List[str]]] = mapped_column(ARRAY(String), nullable=True)

    @property
    def detailed_status_enum(self) -> Type[ExtractionDetailedStatus]:
        return ExtractionDetailedStatus

    def create_table(self, engine: Engine):
        sql_str = f"""
        CREATE TABLE IF NOT EXISTS {self.__tablename__} (
            run_id varchar,
            workflow_name varchar,
            version varchar,
            global_status varchar,
            detailed_status varchar,
            sources_max_date varchar,
            generated_files array(varchar),
            start_ts timestamp(0),
            end_ts timestamp(0),
            run_mode varchar,
            additional_info_json varchar,
            creator_github_repository varchar,
            creator_commit_id varchar
        )
        """
        with engine.connect() as conn:
            conn.execute(text(sql_str))


class ExtractionWorkflowRunService(AbstractWorkflowRunService):

    @classmethod
    def get_metadata_class(cls) -> Type[WorkflowRunExtract]:
        return WorkflowRunExtract

```

###### FILE: src/azfr_skywalker_utils/trino/common.py ######

```py
from typing import Optional
from pydantic import BaseModel, Field
from trino.constants import DEFAULT_SOURCE

class TrinoConfig(BaseModel):
    """
    Configuration for Trino connection.
    """
    source: Optional[str] = Field(default=DEFAULT_SOURCE, description="Source for Trino connection")
    host: str = Field(description="Trino host")
    port: int = Field(default=443, description="Trino port")
    http_scheme: str = Field(default="https", description="HTTP scheme for Trino connection")
    ssl_verify: bool = Field(default=True,  description="Whether to verify SSL certificates for HTTPS connections.")
    access_token_scope : str = Field(description="Scope for Azure access token")
    user: str = Field(default="", description="Trino user name.")
    catalog: str = Field(description="Trino catalog name")
    schema_name: str = Field(description="Trino schema name")

```

###### FILE: src/azfr_skywalker_utils/trino/dbapi.py ######

```py
from contextlib import contextmanager
from typing import Generator
from trino.dbapi import connect, Connection, Cursor
from trino.auth import JWTAuthentication

from azfr_azure_utils.identity import AzfrDefaultCredential
from azfr_skywalker_utils.trino.common import TrinoConfig


@contextmanager
def trino_connection(config: TrinoConfig) -> Generator[Connection, None, None]:
    """
    Context manager for Trino DB connection.
    Usage:
        with trino_connection(config) as conn:
            # use conn
    """
    jwt_token = AzfrDefaultCredential().get_token(config.access_token_scope)
    with connect(
        host=config.host,
        port=config.port,
        http_scheme=config.http_scheme,
        verify=config.ssl_verify,
        auth=JWTAuthentication(jwt_token.token),
        user=config.user,
        catalog=config.catalog,
        schema=config.schema_name,
        source=config.source,
    ) as connection:
        yield connection

@contextmanager
def get_cursor(connection: Connection) -> Generator[Cursor, None, None]:
    """ Get a cursor object for trino db """
    cursor = connection.cursor()
    try:
        yield cursor
    finally:
        cursor.close()

```

###### FILE: src/azfr_skywalker_utils/trino/sql_alchemy.py ######

```py
from typing import Generator
from contextlib import contextmanager
from urllib.parse import quote_plus
from sqlalchemy import create_engine
from sqlalchemy.engine import Engine
from sqlalchemy.orm import sessionmaker, Session
from trino.auth import JWTAuthentication

from azfr_azure_utils.identity import AzfrDefaultCredential
from azfr_skywalker_utils.trino.common import TrinoConfig


def build_trino_sqlalchemy_url(config: TrinoConfig) -> str:
    # https://github.com/trinodb/trino-python-client#sqlalchemy-support
    user = quote_plus(config.user)
    return (
        f"trino://{user}@{config.host}:{config.port}/{config.catalog}/{config.schema_name}"
    )

@contextmanager
def trino_engine(config: TrinoConfig) -> Generator[Engine, None, None]:
    jwt_token = AzfrDefaultCredential().get_token(config.access_token_scope)
    url = build_trino_sqlalchemy_url(config)
    engine = create_engine(url,
                           connect_args={
                               "auth": JWTAuthentication(jwt_token.token),
                               "http_scheme": config.http_scheme,
                               "verify": config.ssl_verify
                               })
    try:
        yield engine
    finally:
        engine.dispose()

@contextmanager
def trino_session(engine: Engine) -> Generator[Session, None, None]:
    session_local = sessionmaker(bind=engine)
    session = session_local()
    try:
        yield session
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

```

###### FILE: src/azfr_skywalker_utils/utils/__init__.py ######

```py
from azfr_skywalker_utils.utils.archive import *
from azfr_skywalker_utils.utils.date import *
from azfr_skywalker_utils.utils.file import *
from azfr_skywalker_utils.utils.format_extension import *
from azfr_skywalker_utils.utils.helpers import *
from azfr_skywalker_utils.utils.mail import *
from azfr_skywalker_utils.utils.sql import *

```

###### FILE: src/azfr_skywalker_utils/utils/archive.py ######

```py
from prefect import task, get_run_logger
import azfr_fsspec_utils as fspath
from azfr_fsspec_utils.zipfile import FsspecZipFile

@task(task_run_name="clean({dir_to_clean})")
def clean(dir_to_clean: str) -> None:
    """ Remove recursively the directory that needs to be clean """
    fspath.remove(dir_to_clean, recursive=True)

@task(task_run_name="uncompress(input_file={input_file}, out_dir={out_dir}, compression={compression})")
def uncompress(input_file: str, out_dir: str, compression: str) -> list[str]:
    """
    Uncompresses a file based on the specified compression type.
    Supports 'tar' and 'zip' formats.
    
    Args:
        input_file (str): Path to the compressed file.
        out_dir (str): Directory where the extracted files will be stored.
        compression (str): Compression format ('tar' or 'zip').
    
    Returns:
        list[str]: List of extracted file paths.
    """
    if compression == "tar":
        return untar(input_file, out_dir)
    elif compression == "zip":
        return unzip(input_file, out_dir)
    else:
        raise ValueError(f"Unsupported compression format: {compression}")

def untar(input_file: str, out_dir: str) -> list[str]:
    """
    Extracts files from a tar archive.
    
    Args:
        input_file (str): Path to the tar file.
        out_dir (str): Directory where files will be extracted.
    
    Returns:
        list[str]: List of extracted file paths.
    """
    with fspath.open(input_file, compression="tar", mode="rb") as untar_file:
        filepaths = [fspath.join(out_dir, item.name) for item in untar_file.getmembers()]
        untar_file.extractall(out_dir)
    return filepaths

def unzip(input_file: str, out_dir: str) -> list[str]:
    """
    Extracts files from a zip archive.
    
    Args:
        input_file (str): Path to the zip file.
        out_dir (str): Directory where files will be extracted.
    
    Returns:
        list[str]: List of extracted file paths.
    """
    filepaths = []
    with FsspecZipFile(input_file) as f:
        csv_files = f.infolist()
        for csv_file in csv_files:
            extract_file = fspath.join(out_dir, csv_file.filename)
            filepaths.append(extract_file)
            fspath.makedirs(out_dir, exist_ok=True)
            with fspath.open(extract_file, 'wb') as extracted_file:
                extracted_file.write(f.read(csv_file))
    return filepaths

@task(task_run_name="archive_file(source_file={source_file}, archive_path={archive_path})")
def archive_file(source_file, archive_path):
    """
    Move file that matches the pattern from input_dir to archive_dir
    :param source_file: Path to the file to archive
    :param archive_path: Folder that will contain archived data
    :return: Path to files that have been archived
    """
    logger = get_run_logger()
    try:
        dest = _archive(source_file, archive_path)

        return dest
    except Exception as e:
        logger.error("Failed to archive : " + str(e))
        if source_file:
            _rollback(archive_path, fspath.dirname(source_file))
        raise e


def _rollback(source_dir, dest_dir):
    """
    Move files from source_dir to dest_dir
    :param source_dir: Source directory
    :param dest_dir: Dest directory
    """
    logger = get_run_logger()
    files = fspath.listdir(source_dir)
    if files:
        logger.info(f"Roll back. Move files from archive ({source_dir}) to landing ({dest_dir}).")
        for file in files:
            landing_file = fspath.join(dest_dir, fspath.basename(file))
            if not fspath.exists(landing_file):
                fspath.move(file, landing_file)
                logger.info(f"OK - File moved: {file}")


def _archive(file, archive_path) -> str:
    """
    Move file to archive_path. Raise FileExistsError if file already exists.
    :param file: Path to file
    :param archive_path: Archive directory
    :return: Path to files that have been archived
    """
    fspath.makedirs(archive_path, exist_ok=True)
    target_file = fspath.join(archive_path, fspath.basename(file))
    if fspath.exists(target_file):
        raise FileExistsError(f"File already exists: {target_file}")
    else:
        fspath.move(file, target_file)
        return target_file

```

###### FILE: src/azfr_skywalker_utils/utils/date.py ######

```py
import pytz
from datetime import datetime


def get_now_utc() -> datetime:
    return datetime.now(pytz.UTC)


# Backward compatibility alias
get_now_UTC = get_now_utc

```

###### FILE: src/azfr_skywalker_utils/utils/file.py ######

```py
from typing import Any, Generator
import azfr_fsspec_utils as fspath


def make_gen(reader) -> Generator[Any, Any, Any]:
    """
    Make a generator from reader
    """
    b = reader(1024 * 1024)
    while b:
        yield b
        b = reader(1024*1024)


def count_file_lines(file: str, offset: int = 0) -> int:
    """
    Count the number of new lines in file, add the offset to count (to handle header, empty row at the end etc)
    """
    with fspath.open(file, 'rb') as f:
        f_gen = make_gen(f.raw.read)
        return sum(buf.count(b'\n') for buf in f_gen) + offset



```

###### FILE: src/azfr_skywalker_utils/utils/format_extension.py ######

```py
from pydantic import BaseModel, Field
from typing import Optional
from azfr_parsing_utils.csv.format import CsvFileFormat
from azfr_parsing_utils.json.format import JsonFileFormat
from azfr_parsing_utils.fixed_width.format import FixedWidthFormat


class MetadataRequiredFields(BaseModel):
    file_identifier: str = Field(
        title="file_identifier",
        description="file_identifier of the file containing this table",
    )


class CsvFileFormatExtended(CsvFileFormat, MetadataRequiredFields):
    pass

class FixedWidthFormatExtended(FixedWidthFormat, MetadataRequiredFields):
    pass

class JsonFileFormatExtended(JsonFileFormat, MetadataRequiredFields):
    pass
```

###### FILE: src/azfr_skywalker_utils/utils/helpers.py ######

```py
from typing import Optional


def max_with_none(a: Optional[str], b: Optional[str]) -> Optional[str]:
    """Get the maximum of two version strings, handling None values.
    
    Python's built-in max() function doesn't accept None values. This function
    returns the non-None value when one input is None, or None when both inputs are None.
    
    Args:
        a: First version string or None
        b: Second version string or None
        
    Returns:
        Maximum version string, or None if both are None
        
    Examples:
        >>> max_with_none("1.0.0", "2.0.0")
        "2.0.0"
        >>> max_with_none("1.0.0", None)
        "1.0.0"
        >>> max_with_none(None, None)
        None
    """
    if a is None and b is None:
        return None
    if a is None:
        return b
    if b is None:
        return a
    return max(a, b)

```

###### FILE: src/azfr_skywalker_utils/utils/mail.py ######

```py
from pydantic import BaseModel

class EmailConfig(BaseModel):
    sender: str
    to: str
    subject: str
    host: str
    port: int
    ssl: bool
    color: str  # Use it if you need a new color
    error_receivers: str  # Use it if you need to receive only errors

```

###### FILE: src/azfr_skywalker_utils/utils/sql.py ######

```py
from typing import Optional


def qualified_table_name(catalog: Optional[str], schema: Optional[str], table: str) -> str:
    """Returns the fully qualified name, supporting optional catalog and schema."""
    if not table:
        raise ValueError("Table must be provided.")

    parts = []
    if catalog:
        parts.append(catalog)
    if schema:
        parts.append(schema)
    parts.append(table)

    return ".".join(parts)

```

###### FILE: test/metadata/parsing/__init__.py ######

```py
from pydantic import BaseModel
import re

import azfr_fsspec_utils as fspath
from azfr_skywalker_utils.metadata.parsing.analyzer import AbstractAnalyzer
from azfr_skywalker_utils.metadata.parsing.workflowconfig import WorkflowBaseConfig

class ConfigTest(WorkflowBaseConfig):
    file_pattern: re.Pattern = re.compile("")
    uncompressed_file_pattern: re.Pattern | None = None
    files_configs: dict = {}
    date_format: str | None = None
    archive_dir: str | None = None
    input_dir: str | None = None
    tables_to_format: dict = {}


class FileFormatTest(BaseModel):
    extraction_type: str | None = "delta"


class AnalyzerTest(AbstractAnalyzer):
    def get_tables_from_file(self, file):
        with fspath.open(file) as f:
            return f.read().split("|")

```

###### FILE: test/metadata/parsing/analyzer/test_analyze_landing_files.py ######

```py
import pytest
import polars as pl
from datetime import datetime, timedelta
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.metadata import WORKFLOW_FILE_STATUS, WORKFLOW_TABLE_STATUS
from test.metadata.parsing.conftest import dir_paths, test_config, analyzer


@pytest.fixture
def landing_files(dir_paths):
    file_configs = [
        {"file_identifier": "file_A", "date": "20240108", "timestamp": "20240108000000", "content": "table_1|table_2|table_3"},  # valid
        {"file_identifier": "file_A", "date": "20240109", "timestamp": "20240109000000", "content": "table_1|table_2|table_3"},  # duplicate date
        {"file_identifier": "file_A", "date": "20240109", "timestamp": "20240110000000", "content": "table_1|table_2|table_3"},  # duplicate date
        {"file_identifier": "file_A", "date": "20240110", "timestamp": "20240110000000", "content": "table_1|table_2|table_3"},  # already ingested (see setup_archive_file)
        {"file_identifier": "file_A", "date": "20240111", "timestamp": "20240111000000", "content": "table_1|table_2|table_3|table_4"},  # table mismatch (extra table)
        {"file_identifier": "file_A", "date": "20240112", "timestamp": "20240112000000", "content": "table_1|table_3"},  # table mismatch (missing table)
        {"file_identifier": "file_A", "date": "20240114", "timestamp": "20240114000000", "content": "table_1|table_2|table_3"},  # unexpected date
        {"file_identifier": "file_A", "date": "20240115", "timestamp": "20240115000000", "content": "table_1|table_2|table_4"},  # table mismatch (missing/extra table)
        {"file_identifier": "file_A", "date": (datetime.now()+timedelta(days=100)).strftime("%Y%m%d"), 
         "timestamp": (datetime.now()+timedelta(days=100)).strftime("%Y%m%d%H%M%S"), "content": "table_1|table_2|table_3"},  # future date
    ]
    landing_files = []
    for file_config in file_configs:
        landing_file = fspath.join(dir_paths['landing'], f"{file_config['file_identifier']}-{file_config['date']}-{file_config['timestamp']}.txt")
        with fspath.open(landing_file, mode="w") as f:
            f.write(file_config['content'])
        landing_files.append(landing_file)
    return landing_files


class TestAnalyzeLandingFiles:
    @staticmethod
    def setup_archive_file(dir_paths, landing_files):
        """ Creates an archive entry for 20240110, simulating that this file has already been processed """
        fspath.makedirs(fspath.join(dir_paths['archive'], "20240110"))
        fspath.touch(fspath.join(dir_paths['archive'], "20240110", fspath.basename(landing_files[3])))

    @staticmethod
    def run_analyzer_extract_and_analyze(analyzer, landing_files):
        analyzer.landing_files_info = {}
        analyzer.extract_landing_info(landing_files)
        analyzer.analyze_landing_files(landing_files)

    def test_landing_error_count(self, analyzer, dir_paths, test_config, landing_files):
        self.setup_archive_file(dir_paths, landing_files)
        self.run_analyzer_extract_and_analyze(analyzer, landing_files)
        assert len(analyzer.errors) == 7
        assert analyzer.valid_landing_files == [fspath.join(test_config.input_dir, "file_A-20240108-20240108000000.txt")]

    def test_metadata_table_results(self, analyzer, dir_paths, landing_files, test_config):
        self.setup_archive_file(dir_paths, landing_files)
        self.run_analyzer_extract_and_analyze(analyzer, landing_files)
        df_metadata_table = pl.scan_delta(fspath.join(test_config.metadata_dir, WORKFLOW_TABLE_STATUS))
        metadata_table_results = df_metadata_table.select([
            "workflow_name", "file_identifier", "version", "table",
            "global_status", "detailed_status", "nb_parsed_lines",
            "nb_rejected_lines", "message", "file_path"
        ]).collect().to_dicts()
        assert len(metadata_table_results) == 4

        landing_path = test_config.input_dir
        assert metadata_table_results == [
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A',
            'version': '20240115', 'table': 'table_4', 'global_status': 'FAILED',
            'detailed_status': 'TABLE_UNKNOWN', 'nb_parsed_lines': None,
            'nb_rejected_lines': None, 'message': 'Table table_4-20240115 non reconnue',
            'file_path': fspath.join(landing_path, 'file_A-20240115-20240115000000.txt')},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A',
            'version': '20240115', 'table': 'table_3', 'global_status': 'FAILED',
            'detailed_status': 'TABLE_MISSING', 'nb_parsed_lines': None,
            'nb_rejected_lines': None, 'message': 'Table table_3-20240115 attendue manquante',
            'file_path': fspath.join(landing_path, 'file_A-20240115-20240115000000.txt')},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A',
            'version': '20240112', 'table': 'table_2', 'global_status': 'FAILED',
            'detailed_status': 'TABLE_MISSING', 'nb_parsed_lines': None,
            'nb_rejected_lines': None, 'message': 'Table table_2-20240112 attendue manquante',
            'file_path': fspath.join(landing_path, 'file_A-20240112-20240112000000.txt')},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A',
            'version': '20240111', 'table': 'table_4', 'global_status': 'FAILED',
            'detailed_status': 'TABLE_UNKNOWN', 'nb_parsed_lines': None,
            'nb_rejected_lines': None, 'message': 'Table table_4-20240111 non reconnue',
            'file_path': fspath.join(landing_path, 'file_A-20240111-20240111000000.txt')}
        ]

    def test_metadata_file_results(self, analyzer, dir_paths, landing_files, test_config):
        self.setup_archive_file(dir_paths, landing_files)
        self.run_analyzer_extract_and_analyze(analyzer, landing_files)
        df_metadata_file = pl.scan_delta(fspath.join(test_config.metadata_dir, WORKFLOW_FILE_STATUS))
        metadata_file_results = df_metadata_file.select([
            "workflow_name", "file_identifier", "version", "global_status",
            "detailed_status", "message", "file_paths"
        ]).collect().to_dicts()
        assert len(metadata_file_results) == 7

        landing_path = test_config.input_dir
        assert metadata_file_results == [
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': (datetime.now()+timedelta(days=100)).strftime("%Y%m%d"),
            'global_status': 'FAILED', 'detailed_status': 'FILE_FUNC_DATE_FUTURE',
            'message': f'Fichier file_A-{(datetime.now()+timedelta(days=100)).strftime("%Y%m%d")}: Date fonctionnelle {(datetime.now()+timedelta(days=100)).strftime("%Y%m%d")} dans le futur',
            'file_paths': [fspath.join(landing_path, fspath.basename(landing_files[8]))]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240115',
            'global_status': 'FAILED', 'detailed_status': 'FILE_MISMATCH_TABLES',
            'message': 'Fichier file_A-20240115: table(s) manquante(s) et/ou table(s) non reconnue(s)',
            'file_paths': [fspath.join(landing_path, 'file_A-20240115-20240115000000.txt')]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240114',
            'global_status': 'FAILED', 'detailed_status': 'FILE_FUNC_DATE_UNEXPECTED',
            'message': 'Fichier file_A-20240114: Date fonctionnelle 20240114 non attendue',
            'file_paths': [fspath.join(landing_path, 'file_A-20240114-20240114000000.txt')]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240112',
            'global_status': 'FAILED', 'detailed_status': 'FILE_MISMATCH_TABLES',
            'message': 'Fichier file_A-20240112: table(s) manquante(s) et/ou table(s) non reconnue(s)',
            'file_paths': [fspath.join(landing_path, 'file_A-20240112-20240112000000.txt')]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240111',
            'global_status': 'FAILED', 'detailed_status': 'FILE_MISMATCH_TABLES',
            'message': 'Fichier file_A-20240111: table(s) manquante(s) et/ou table(s) non reconnue(s)',
            'file_paths': [fspath.join(landing_path, 'file_A-20240111-20240111000000.txt')]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240110',
            'global_status': 'FAILED', 'detailed_status': 'FILE_FUNC_DATE_ALREADY_INGESTED',
            'message': 'Fichier file_A-20240110: Date fonctionnelle 20240110 déjà ingérée',
            'file_paths': [fspath.join(landing_path, 'file_A-20240110-20240110000000.txt')]},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20240109',
            'global_status': 'FAILED', 'detailed_status': 'FILE_FUNC_DATE_DUPLICATED',
            'message': 'Plusieurs fichiers avec la même date fonctionnelle 20240109 et le même identifiant file_A sont présents dans la landing zone',
            'file_paths': [fspath.join(landing_path, 'file_A-20240109-20240109000000.txt'), fspath.join(landing_path, 'file_A-20240109-20240110000000.txt')]}
        ]

```

###### FILE: test/metadata/parsing/analyzer/test_analyze_parse_table.py ######

```py
import polars as pl
from prefect import task
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.metadata import WORKFLOW_FILE_STATUS, WORKFLOW_TABLE_STATUS, FileDetailedStatus, FileStatus, get_now_UTC
from test.metadata.parsing.conftest import dir_paths, test_config, analyzer
from test.metadata.parsing.helpers import dummy_parse


class TestAnalyzeParseTable:
    @staticmethod
    def setup_test(loader, dir_paths, test_config):
        file_basename = "file_A-20241025-20241025235959.txt"
        file_path = fspath.join(test_config.archive_dir, file_basename)
        return file_basename, file_path

    def _get_metadata_results(self, analyzer, test_config):
        """Helper to fetch metadata file and table results for the current file."""
        df_metadata_file = pl.scan_delta(fspath.join(analyzer.config.metadata_dir, WORKFLOW_FILE_STATUS))
        file_results = df_metadata_file.select(["workflow_name", "file_identifier", "version", "global_status", "detailed_status", "message", "file_paths"]).collect().to_dicts()
        
        df_metadata_table = pl.scan_delta(fspath.join(analyzer.config.metadata_dir, WORKFLOW_TABLE_STATUS))
        table_results = df_metadata_table.select(["workflow_name", "file_identifier", "version", "table", "global_status", "detailed_status", "nb_parsed_lines", "nb_rejected_lines", "message", "file_path"]).collect().to_dicts()
        
        return file_results, table_results

    def test_error_table(self, dir_paths, test_config, analyzer, loader):
        file_basename, file_path = self.setup_test(loader, dir_paths, test_config)
        
        try:
            analyzer.analyze_file_after_parse(file_path, get_now_UTC())
            assert False
        except Exception:
            pass
        
        tables = ["table_1", "table_2"]
        file_identifier = "file_A"
        date = "20241025"

        for table in tables:
            state = dummy_parse(0, 10, False, return_state=True) if table == "table_1" else dummy_parse(0, 0, True, return_state=True)
            analyzer.analyze_parse_table(state, file_identifier, table, date, file_path, get_now_UTC())
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
       
        # Assert analyzer state
        expected_parse_result = {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 10,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': ['table_1'],
            'file_identifier': 'file_A',
            'date': '20241025'
        }
        assert list(analyzer.parse_results.keys()) == [file_basename]
        assert analyzer.parse_results[file_basename].__dict__ == expected_parse_result

        # Metadata assertions
        file_results, table_results = self._get_metadata_results(analyzer, test_config)
        expected_file_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025',
            'global_status': 'FAILED', 'detailed_status': 'FILE_ERROR_TABLES',
            'message': 'Fichier file_A-20241025 : erreur table(s)',
            'file_paths': [file_path]
        }
        expected_table_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_2',
            'global_status': 'FAILED', 'detailed_status': 'TABLE_PARSING_ERROR', 'nb_parsed_lines': None, 'nb_rejected_lines': None,
            'message': 'Task run encountered an exception Exception: dummy parse ko',
            'file_path': file_path
        }
        assert expected_file_row in file_results
        assert expected_table_row in table_results

    def test_success(self, dir_paths, test_config, analyzer, loader):
        file_basename, file_path = self.setup_test(loader, dir_paths, test_config)
        tables = ["table_1", "table_2"]
        file_identifier = "file_A"
        date = "20241025"
        analyzer.parse_results = {}

        state = dummy_parse(10, 0, False, return_state=True)
        for table in tables:
            analyzer.analyze_parse_table(state, file_identifier, table, date, file_path, get_now_UTC())
        
        # Assert analyzer state before file status
        expected_parse_result = {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 20,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'
        }
        assert analyzer.parse_results[file_basename].__dict__ == expected_parse_result

        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        expected_final_result = {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_SUCCESS.value,
                            'message': 'Fichier file_A-20241025 ingéré avec succès'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 20,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'
        }
        assert list(analyzer.parse_results.keys()) == [file_basename]
        assert analyzer.parse_results[file_basename].__dict__ == expected_final_result

        # Metadata assertions
        file_results, table_results = self._get_metadata_results(analyzer, test_config)
        expected_file_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025',
            'global_status': 'SUCCESS', 'detailed_status': 'FILE_SUCCESS',
            'message': 'Fichier file_A-20241025 ingéré avec succès',
            'file_paths': [file_path]
        }
        expected_table_rows = [
            {
                'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_1',
                'global_status': 'SUCCESS', 'detailed_status': 'TABLE_SUCCESS', 'nb_parsed_lines': 10, 'nb_rejected_lines': 0,
                'message': 'Table table_1-20241025 ingérée avec succès',
                'file_path': file_path
            },
            {
                'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_2',
                'global_status': 'SUCCESS', 'detailed_status': 'TABLE_SUCCESS', 'nb_parsed_lines': 10, 'nb_rejected_lines': 0,
                'message': 'Table table_2-20241025 ingérée avec succès',
                'file_path': file_path
            }
        ]

        assert expected_file_row in file_results
        for expected_row in expected_table_rows:
            assert expected_row in table_results

    def test_lines_rejected(self, dir_paths, test_config, analyzer, loader):
        file_basename, file_path = self.setup_test(loader, dir_paths, test_config)
        tables = ["table_1", "table_2"]
        file_identifier = "file_A"
        date = "20241025"
        analyzer.parse_results = {}
        
        state = dummy_parse(10, 5, False, return_state=True)
        for table in tables:
            analyzer.analyze_parse_table(state, file_identifier, table, date, file_path, get_now_UTC())
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())

        # Assert analyzer state
        expected_parse_result = {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_LINES_REJECTED.value,
                            'message': 'Fichier file_A-20241025 : 10 lignes rejetées sur 2 tables'},
            'total_parsed_lines': 20,
            'total_rejected_lines': 10,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': ["table_1", "table_2"],
            'file_identifier': 'file_A',
            'date': '20241025'
        }
        assert list(analyzer.parse_results.keys()) == [file_basename]
        assert analyzer.parse_results[file_basename].__dict__ == expected_parse_result

        # Metadata assertions
        file_results, table_results = self._get_metadata_results(analyzer, test_config)
        expected_file_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025',
            'global_status': 'FAILED', 'detailed_status': 'FILE_LINES_REJECTED',
            'message': 'Fichier file_A-20241025 : 10 lignes rejetées sur 2 tables',
            'file_paths': [file_path]
        }
        expected_table_rows = [
            {
                'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_1',
                'global_status': 'FAILED', 'detailed_status': 'TABLE_LINES_REJECTED', 'nb_parsed_lines': 10, 'nb_rejected_lines': 5,
                'message': 'Table table_1-20241025 ingérée avec rejets',
                'file_path': file_path
            },
            {
                'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_2',
                'global_status': 'FAILED', 'detailed_status': 'TABLE_LINES_REJECTED', 'nb_parsed_lines': 10, 'nb_rejected_lines': 5,
                'message': 'Table table_2-20241025 ingérée avec rejets',
                'file_path': file_path
            }
        ]
        assert expected_file_row in file_results
        for expected_row in expected_table_rows:
            assert expected_row in table_results

    def test_empty_table(self, dir_paths, test_config, analyzer, loader):
        file_basename, file_path = self.setup_test(loader, dir_paths, test_config)
        tables = ["table_1", "table_2"]
        file_identifier = "file_A"
        date = "20241025"
        analyzer.parse_results = {}

        state = dummy_parse(0, 0, False, return_state=True)
        for table in tables:
            analyzer.analyze_parse_table(state, file_identifier, table, date, file_path, get_now_UTC())
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())

        # Assert analyzer state
        expected_parse_result = {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': ["table_1"],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'
        }
        assert list(analyzer.parse_results.keys()) == [file_basename]
        assert analyzer.parse_results[file_basename].__dict__ == expected_parse_result

        # Metadata assertions
        file_results, table_results = self._get_metadata_results(analyzer, test_config)
        expected_file_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025',
            'global_status': 'FAILED', 'detailed_status': 'FILE_ERROR_TABLES',
            'message': 'Fichier file_A-20241025 : erreur table(s)',
            'file_paths': [file_path]
        }
        expected_table_row = {
            'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_A', 'version': '20241025', 'table': 'table_1',
            'global_status': 'FAILED', 'detailed_status': 'TABLE_FULL_EMPTY', 'nb_parsed_lines': 0, 'nb_rejected_lines': 0,
            'message': 'Table table_1-20241025 au format full est vide',
            'file_path': file_path
        }
        assert expected_file_row in file_results
        assert expected_table_row in table_results

```

###### FILE: test/metadata/parsing/analyzer/test_analyzer_validation.py ######

```py
import pytest
import re
from datetime import datetime, timedelta
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.analyzer import PlainFileAnalyser
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata
from test.metadata.parsing.conftest import loader, dir_paths, analyzer


@pytest.fixture
def landing_files(dir_paths):
    """Create test landing file paths"""
    file_configs = [
        {"file_identifier": "file_A", "date": "20240101", "timestamp": "20240101000000"},
        {"file_identifier": "file_A", "date": "20240102", "timestamp": "20240102000000"},
        {"file_identifier": "file_A", "date": "20240102", "timestamp": "20240103000000"},
        {"file_identifier": "file_A", "date": "20240103", "timestamp": "20240103000000"}
    ]
    return [
        fspath.join(dir_paths["landing"], f"{config['file_identifier']}-{config['date']}-{config['timestamp']}.txt")
        for config in file_configs
    ]


class TestAnalyzerLandingValidation:
    def test_landing_info_extraction(self, analyzer, dir_paths, landing_files):
        """ Test that landing file info is correctly extracted and grouped by (date, file_identifier)."""
        analyzer.landing_files_info = {}
        analyzer.extract_landing_info(landing_files)
        expected_info = {
            ('20240101', 'file_A'): [fspath.join(dir_paths["landing"], 'file_A-20240101-20240101000000.txt')],
            ('20240102', 'file_A'): [fspath.join(dir_paths["landing"], 'file_A-20240102-20240102000000.txt'),
                                     fspath.join(dir_paths["landing"], 'file_A-20240102-20240103000000.txt')],
            ('20240103', 'file_A'): [fspath.join(dir_paths["landing"], 'file_A-20240103-20240103000000.txt')]
        }
        assert analyzer.landing_files_info == expected_info
    
    def test_duplicate_detection(self, analyzer, dir_paths, landing_files):
        """ Test that duplicate files are correctly detected in landing_files_info. """
        for landing_file in landing_files:
            fspath.touch(landing_file)
        analyzer.landing_files_info = {}
        analyzer.extract_landing_info(landing_files)
        for files in analyzer.landing_files_info:
            assert (len(files) > 1) == analyzer.is_duplicate(files)

    def test_ingestion_status(self, analyzer, dir_paths, landing_files):
        """ Test that ingestion status is correctly determined for files in the archive."""
        fspath.touch(fspath.join(dir_paths["archive"], "20240102", fspath.basename(landing_files[1])))
        assert not analyzer.is_already_ingested("20240101", "file_A")
        assert analyzer.is_already_ingested("20240102", "file_A")
    
    @pytest.mark.parametrize(
        "content,expected_mismatch",
        [
            ("table_1|table_2|table_3", False),  # correct
            ("table_1|table_2", True),           # missing table_3
            ("table_1|table_2|table_3|table_4", True),  # extra table_4
            ("table_1|table_2|table_4", True),  # missing table_3, extra table_4
        ]
    )
    def test_table_mismatch_detection(self, analyzer, dir_paths, landing_files, content, expected_mismatch):
        """ Test that table mismatches are detected for various file contents. """
        # Write content to the first landing file for each test case
        with fspath.open(landing_files[0], mode="w") as f:
            f.write(content)
        # Split content into tables list
        file_tables = analyzer.get_tables_from_file(landing_files[0])
        # Only test the first file for each param case
        result = analyzer.is_mismatch_tables(landing_files[0], '20240101', 'file_A', file_tables)
        assert result == expected_mismatch

class TestAnalyzerDateValidation:

    @pytest.mark.parametrize(
        "date,file_identifier,expected_future",
        [
            ((datetime.now()+timedelta(days=100)).strftime("%Y%m%d"), "file_A", True),
            (datetime.now().strftime("%Y%m%d"), "file_A", False),
        ]
    )
    def test_is_future_date(self, analyzer, date, file_identifier, expected_future):
        """ Test is_future_date method for various scenarios."""
        assert analyzer.is_future_date(date, file_identifier=file_identifier) == expected_future

    @pytest.mark.parametrize(
        "date,file_identifier,expected_unexpected",
        [
            ("20241006", "file_A", True),
            ("20241008", "file_A", False),
        ]
    )
    def test_is_unexpected_date(self, analyzer, date, file_identifier, expected_unexpected):
        """ Test is_unexpected_date method for various scenarios. """
        assert analyzer.is_unexpected_date(date, file_identifier=file_identifier) == expected_unexpected


def test_analyze_file_after_parse_raises_if_missing_result(analyzer):
    with pytest.raises(KeyError, match="No parsed results available"):
        analyzer.analyze_file_after_parse("missing-file.txt", datetime.now())


def test_set_file_empty_sets_expected_file_status(analyzer):
    file_path = "landing/file_A-20240101-20240101000000.txt"
    analyzer.set_file_empty(file_path=file_path, file_identifier="file_A", date="20240101")
    file_name = fspath.basename(file_path)
    assert analyzer.parse_results[file_name].file_status["status"] == "SUCCESS"
    assert analyzer.parse_results[file_name].file_status["detailed_status"] == "FILE_EMPTY"


def test_plain_file_analyser_returns_empty_when_pattern_does_not_match(test_config):
    config = test_config.model_copy(deep=True)
    config.file_pattern = re.compile(r"(?P<name>table_[0-9]+)\\.txt")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    plain_analyzer = PlainFileAnalyser(workflow_metadata=workflow_metadata, config=config)
    assert plain_analyzer.get_tables_from_file("unexpected_name.csv") == []
    
```

###### FILE: test/metadata/parsing/analyzer/test_raise_error_if_delta_and_empty.py ######

```py
import re
from collections import defaultdict
import azfr_fsspec_utils as fspath
from test.metadata.parsing import AnalyzerTest as Analyzer
from azfr_skywalker_utils.metadata.parsing.metadata import FileDetailedStatus, FileStatus, get_now_UTC
from azfr_skywalker_utils.metadata.parsing.workflowconfig import FileConfig, TableConfig
from test.metadata.parsing import ConfigTest, FileFormatTest
from test.metadata.parsing.helpers import dummy_parse


class TestRaiseErrorIfDeltaAndEmpty:
    file_identifier = "file_A"
    tables = ["table_1", "table_2"]
    file_basename = "file_A-20241025-20241025235959.txt"
    date = "20241025"

    def setup_config(self, dir_paths, table_configs):
        files_configs = {
            "file_A": FileConfig(
                file_identifier="file_A",
                overdue_time="18:15",
                mode="daily",
                min_expected_date="",
                period_checked=30,
                working_days_only=True,
                tables=["table_1", "table_2"],
                table_configs=table_configs
                )
        }

        return ConfigTest(
                metadata_dir=dir_paths['metadata'],
                archive_dir=dir_paths['archive'],
                input_dir=dir_paths['landing'],
                workflow_name="TEST-WORKFLOW",
                file_pattern=re.compile(r"(?P<file_identifier>[a-zA-Z_]+)-(?P<date>[0-9]{8})-(?P<timestamp>[0-9]{14})\.txt"),
                uncompressed_file_pattern=re.compile(r"(?P<name>[a-zA-Z_0-9]+)\.txt"),
                files_configs=files_configs,
                tables_to_format={"table_1": FileFormatTest(extraction_type="delta"),
                                "table_2": FileFormatTest(extraction_type="delta")}
        )

    def prepare_test_env(self, test_config, dir_paths, workflow_metadata_class, table_configs_dict):
        table_configs = defaultdict(TableConfig)
        for table in self.tables:
            cfg = table_configs_dict.get(table, {})
            table_configs[table] = TableConfig(**cfg)
        test_config2 = self.setup_config(dir_paths, table_configs if table_configs else None)
        file_path = fspath.join(test_config2.archive_dir, self.file_basename)
        workflow_metadata = workflow_metadata_class()
        workflow_metadata.start(test_config2, write_start_event=False)
        analyzer = Analyzer(workflow_metadata=workflow_metadata, config=test_config2)
        analyzer.parse_results = {}
        return table_configs, file_path, analyzer

    def test_no_table_specific_configs(self, test_config, dir_paths, analyzer):
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, {})
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_delta_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is False, "Expected False when raise_error_if_empty_delta_mode is False by default"
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_NO_ROWS.value,
                            'message': "Fichier file_A-20241025 : Toutes les tables sont vides"},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_raise_error_if_empty_delta_mode_disabled(self, test_config, dir_paths, analyzer):
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_delta_mode": False},
            "table_2": {"raise_error_if_empty_delta_mode": False}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_delta_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is False, "Expected False when raise_error_if_empty_delta_mode is explicitly False"
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_NO_ROWS.value,
                            'message': "Fichier file_A-20241025 : Toutes les tables sont vides"},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_table_specific_configs_no_raise_error_date(self, test_config, dir_paths, analyzer):
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_delta_mode": True, "raise_error_if_empty_after": None},
            "table_2": {"raise_error_if_empty_delta_mode": False, "raise_error_if_empty_after": None}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_delta_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())  # type: ignore
        assert result is True, "Expected True when raise_error_if_empty_delta_mode is True and no raise_error_if_empty_after date"
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': ["table_1"],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_raise_error_date_after_functional_date(self, test_config, dir_paths, analyzer):
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_delta_mode": True, "raise_error_if_empty_after": "20251212"}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_delta_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())  # type: ignore
        assert result is False, "Expected False when raise_error_if_empty_after date > functional date"
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_NO_ROWS.value,
                            'message': "Fichier file_A-20241025 : Toutes les tables sont vides"},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_raise_error_date_before_functional_date(self, test_config, dir_paths, analyzer):
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_delta_mode": True, "raise_error_if_empty_after": "20240101"}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_delta_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())  # type: ignore
        assert result is True, "Expected True when raise_error_if_empty_after date < functional date"
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': ["table_1"],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

```

###### FILE: test/metadata/parsing/analyzer/test_raise_error_if_full_and_empty.py ######

```py
import re
from collections import defaultdict
import azfr_fsspec_utils as fspath

from test.metadata.parsing import AnalyzerTest as Analyzer
from azfr_skywalker_utils.metadata.parsing.metadata import FileDetailedStatus, FileStatus, get_now_UTC
from azfr_skywalker_utils.metadata.parsing.workflowconfig import FileConfig, TableConfig
from test.metadata.parsing import ConfigTest, FileFormatTest
from test.metadata.parsing.helpers import dummy_parse


class TestRaiseErrorIfFullAndEmpty:
    file_identifier = "file_A"
    tables = ["table_1", "table_2"]
    file_basename = "file_A-20241025-20241025235959.txt"
    date = "20241025"

    def setup_config(self, dir_paths, table_config):
        files_configs = {
            "file_A": FileConfig(
                file_identifier="file_A",
                overdue_time="18:15",
                mode="daily",
                min_expected_date="",
                period_checked=30,
                working_days_only=True,
                tables=["table_1", "table_2"],
                table_configs=table_config
                )
        }

        return ConfigTest(
                metadata_dir=dir_paths['metadata'],
                archive_dir=dir_paths['archive'],
                input_dir=dir_paths['landing'],
                workflow_name="TEST-WORKFLOW",
                file_pattern=re.compile(r"(?P<file_identifier>[a-zA-Z_]+)-(?P<date>[0-9]{8})-(?P<timestamp>[0-9]{14})\.txt"),
                uncompressed_file_pattern=re.compile(r"(?P<name>[a-zA-Z_0-9]+)\.txt"),
                files_configs=files_configs,
                tables_to_format={"table_1": FileFormatTest(extraction_type="full"),
                                  "table_2": FileFormatTest(extraction_type="full")}
        )

    def prepare_test_env(self, test_config, dir_paths, workflow_metadata_class, table_configs_dict):
        table_configs = defaultdict(TableConfig)
        for table in self.tables:
            cfg = table_configs_dict.get(table, {})
            table_configs[table] = TableConfig(**cfg)
        test_config = self.setup_config(dir_paths, table_configs if table_configs else None)
        file_path = fspath.join(test_config.archive_dir, self.file_basename)
        workflow_metadata = workflow_metadata_class()
        workflow_metadata.start(test_config, write_start_event=False)
        analyzer = Analyzer(workflow_metadata=workflow_metadata, config=test_config)
        analyzer.parse_results = {}
        return table_configs, file_path, analyzer

    def test_no_table_specific_configs(self, test_config, dir_paths, analyzer):
        """Default TableConfig: raise_error_if_empty_full_mode=True"""
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, {})
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is True, "Expected True when table-specific configs is not given"
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': ["table_1", "table_2"],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_raise_error_if_empty_full_mode_disabled(self, test_config, dir_paths, analyzer):
        """Test when raise_error_if_empty_full_mode is explicitly set to False"""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_full_mode": False},
            "table_2": {"raise_error_if_empty_full_mode": False}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is False, "Expected False when raise_error_if_empty_full_mode is explicitly False"
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_NO_ROWS.value,
                            'message': "Fichier file_A-20241025 : Toutes les tables sont vides"},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_mixed_table_configurations(self, test_config, dir_paths, analyzer):
        """Test mixed configs: one table enabled, one disabled"""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_full_mode": False},
            "table_2": {"raise_error_if_empty_full_mode": True}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result1 = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        result2 = analyzer.raise_error_if_full_and_empty("table_2", 0, 0, self.date, table_configs["table_2"], file_path)
        assert result1 is False, "Expected False for table_1 with disabled flag"
        assert result2 is True, "Expected True for table_2 with enabled flag"

    def test_non_empty_table_full_mode_enabled(self, test_config, dir_paths, analyzer):
        """Test that non-empty tables don't trigger error even when raise_error_if_empty_full_mode=True"""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_full_mode": True}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        # Test with non-empty table (parsed_lines > 0)
        result = analyzer.raise_error_if_full_and_empty("table_1", 5, 0, self.date, table_configs["table_1"], file_path)
        assert result is False, "Expected False when table is not empty (has parsed lines)"
        # Test with rejected lines but no parsed lines
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 3, self.date, table_configs["table_1"], file_path)
        assert result is False, "Expected False when table has rejected lines (not completely empty)"

    def test_table_specific_configs_no_raise_error_date(self, test_config, dir_paths, analyzer):
        """Test when raise_error_if_empty_after is None (should raise error if empty and flag is True)"""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_after": None},
            "table_2": {"raise_error_if_empty_after": None}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is True, "Expected True when no raise_error_if_empty_date"
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': ["table_1", "table_2"],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

    def test_raise_error_date_after_functional_date(self, test_config, dir_paths, analyzer):
        """Test when raise_error_if_empty_after date is in the future."""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_after": "20251212"},
            "table_2": {"raise_error_if_empty_after": "20251212"}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is False, "Expected False when raise_error_if_empty_date > functional date"
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
        analyzer.analyze_file_after_parse(file_path, get_now_UTC())
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        # After post-processing, file status is SUCCESS as no table triggered error
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.SUCCESS.value,
                            'detailed_status': FileDetailedStatus.FILE_NO_ROWS.value,
                            'message': "Fichier file_A-20241025 : Toutes les tables sont vides"},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': [],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}
    
    def test_raise_error_date_before_functional_date(self, test_config, dir_paths, analyzer):
        """Test when raise_error_if_empty_after date is in the past (should raise error for both tables)"""
        table_configs_dict = {
            "table_1": {"raise_error_if_empty_after": "20240101"}
        }
        table_configs, file_path, analyzer = self.prepare_test_env(
            test_config, dir_paths,
            analyzer.workflow_metadata.__class__, table_configs_dict)
        state = dummy_parse(0, 0, False, return_state=True)
        result = analyzer.raise_error_if_full_and_empty("table_1", 0, 0, self.date, table_configs["table_1"], file_path)
        for table in self.tables:
            analyzer.analyze_parse_table(state, self.file_identifier, table, self.date, file_path, get_now_UTC())
        assert result is True, "Expected True when raise_error_if_empty_date < functional date"
        assert list(analyzer.parse_results.keys()) == [self.file_basename]
        assert analyzer.parse_results[self.file_basename].__dict__ == {
            'file_status': {'status': FileStatus.FAILED.value,
                            'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value,
                            'message': 'Fichier file_A-20241025 : erreur table(s)'},
            'total_rejected_lines': 0,
            'total_parsed_lines': 0,
            'fullmode_empty_tables': ["table_1", "table_2"],
            'deltamode_empty_tables': [],
            'rejected_tables': [],
            'file_identifier': 'file_A',
            'date': '20241025'}

```

###### FILE: test/metadata/parsing/analyzer/test_register_missing_files.py ######

```py
import logging
import re
from datetime import datetime, timedelta

import polars as pl
from deltalake.exceptions import TableNotFoundError
import azfr_fsspec_utils as fspath
import pytest

from azfr_skywalker_utils.metadata.parsing.metadata import (
    WORKFLOW_FILE_STATUS,
    FileDetailedStatus,
    FileStatus,
    WorkflowMetadata,
    register_missing_files,
)
from azfr_skywalker_utils.metadata.parsing.workflowconfig import FileConfig
from test.metadata.parsing import ConfigTest



class TestRegisterMissingFiles:
    def setup_method(self):
        self.run_date = datetime.now()
        self.min_expected_date_1 = (self.run_date - timedelta(days=10)).strftime("%Y%m%d")
        self.min_expected_date_2 = (self.run_date - timedelta(days=100)).strftime("%Y%m%d")
        self.dates_success = [(self.run_date - timedelta(days=x)).strftime("%Y%m%d") for x in [5, 20, 100]]

    def _get_files_configs(self, min_expected_date, mode='daily', period_checked=30):
        expected_days = [(self.run_date - timedelta(days=10)).day]
        return {
            "file_A": FileConfig(
                file_identifier="file_A",
                overdue_time="23:59",
                mode=mode,
                min_expected_date=min_expected_date,
                period_checked=period_checked,
                working_days_only=False,
                expected_days=expected_days,
                tables=["table_1", "table_2", "table_3"]
            ),
            "file_B": FileConfig(
                file_identifier="file_B",
                overdue_time="23:59",
                mode=mode,
                min_expected_date=min_expected_date,
                period_checked=period_checked,
                working_days_only=False,
                expected_days=expected_days,
                tables=["table_1", "table_2", "table_3"]
            )
        }

    def _get_config_dict(self, metadata_path, files_configs):
        return ConfigTest(
            metadata_dir=metadata_path,
            workflow_name="TEST-WORKFLOW",
            file_pattern=re.compile("(?P<file_identifier>[a-zA-Z_]+)-(?P<date>[0-9]{8})-(?P<timestamp>[0-9]{14})\\.txt"),
            files_configs=files_configs
        )

    def test_write_file_status_success(self, dir_paths):
        metadata_path = dir_paths["metadata"]
        files_configs = self._get_files_configs(self.min_expected_date_1)
        test_config = self._get_config_dict(metadata_path, files_configs)
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)

        for date_success in self.dates_success:
            for file_identifier in files_configs:
                workflow_metadata.write_file_status(
                    file_identifier, date_success, FileStatus.SUCCESS,
                    FileDetailedStatus.FILE_SUCCESS,
                    f"Fichier {file_identifier}-{date_success} ingéré avec succès",
                    [], workflow_metadata.run_start_ts)

        metadata_file_path = fspath.join(metadata_path, WORKFLOW_FILE_STATUS)
        metadata_file = pl.scan_delta(metadata_file_path)
        metadata_file_results = metadata_file.select(["file_identifier", "version", "global_status", "detailed_status"]).collect().to_dicts()
        nbr_results_file = len(metadata_file_results)
        nbr_results_file_expected = len(self.dates_success) * len(files_configs)
        for file_identifier in test_config.files_configs:
            for date_success in self.dates_success:
                assert {
                    'file_identifier': file_identifier,
                    'version': date_success,
                    'global_status': FileStatus.SUCCESS.value,
                    'detailed_status': FileDetailedStatus.FILE_SUCCESS.value
                } in metadata_file_results
        assert nbr_results_file == nbr_results_file_expected

    def test_register_missing_files_waiting_and_failed(self, dir_paths):
        metadata_path = dir_paths["metadata"]
        files_configs = self._get_files_configs(self.min_expected_date_1)
        test_config = self._get_config_dict(metadata_path, files_configs)
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)

        for date_success in self.dates_success:
            for file_identifier in files_configs:
                workflow_metadata.write_file_status(
                    file_identifier, date_success, FileStatus.SUCCESS,
                    FileDetailedStatus.FILE_SUCCESS,
                    f"Fichier {file_identifier}-{date_success} ingéré avec succès",
                    [], workflow_metadata.run_start_ts)

        register_missing_files(files_configs, metadata_path, workflow_metadata, self.run_date)

        metadata_file_path = fspath.join(metadata_path, WORKFLOW_FILE_STATUS)
        metadata_file = pl.scan_delta(metadata_file_path)
        metadata_file_results = metadata_file.select(["file_identifier", "version", "global_status", "detailed_status"]).collect().to_dicts()
        nbr_results_file_expected = len(self.dates_success) * len(files_configs)

        for file_identifier in test_config.files_configs:
            assert {
                'file_identifier': file_identifier,
                'version': self.run_date.strftime("%Y%m%d"),
                'global_status': FileStatus.WAITING.value,
                'detailed_status': FileDetailedStatus.FILE_WAITING.value
            } in metadata_file_results
            nbr_results_file_expected += 1
            dates_to_check = [
                (self.run_date - timedelta(days=x)).strftime("%Y%m%d")
                for x in range(1, 11)
            ]
            dates_to_check = [d for d in dates_to_check if d not in self.dates_success]
            for date in dates_to_check:
                assert {
                    'file_identifier': file_identifier,
                    'version': date,
                    'global_status': FileStatus.FAILED.value,
                    'detailed_status': FileDetailedStatus.FILE_NOT_RECEIVED.value
                } in metadata_file_results
            nbr_results_file_expected += len(dates_to_check)
        assert len(metadata_file_results) == nbr_results_file_expected

    def test_register_missing_files_late_failed(self, dir_paths):
        metadata_path = dir_paths["metadata"]
        files_configs = self._get_files_configs(self.min_expected_date_1)
        test_config = self._get_config_dict(metadata_path, files_configs)
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)

        for date_success in self.dates_success:
            for file_identifier in files_configs:
                workflow_metadata.write_file_status(
                    file_identifier, date_success, FileStatus.SUCCESS,
                    FileDetailedStatus.FILE_SUCCESS,
                    f"Fichier {file_identifier}-{date_success} ingéré avec succès",
                    [], workflow_metadata.run_start_ts)

        for file_identifier in test_config.files_configs:
            test_config.files_configs[file_identifier].min_expected_date = self.min_expected_date_2
            test_config.files_configs[file_identifier].overdue_time = "00:00"
        register_missing_files(test_config.files_configs, metadata_path, workflow_metadata, self.run_date)

        metadata_file_path = fspath.join(metadata_path, WORKFLOW_FILE_STATUS)
        metadata_file = pl.scan_delta(metadata_file_path)
        metadata_file_results = metadata_file.select(["file_identifier", "version", "global_status", "detailed_status"]).collect().to_dicts()
        nbr_results_file_expected = len(self.dates_success) * len(files_configs)
        for file_identifier in test_config.files_configs:
            dates_to_check = [
                (self.run_date - timedelta(days=x)).strftime("%Y%m%d")
                for x in range(1, test_config.files_configs[file_identifier].period_checked)
            ]
            dates_to_check = [d for d in dates_to_check if d not in self.dates_success]
            dates_to_check.append(self.run_date.strftime("%Y%m%d"))
            nbr_results_file_expected += len(dates_to_check)
            for date in dates_to_check:
                assert {
                    'file_identifier': file_identifier,
                    'version': date,
                    'global_status': FileStatus.FAILED.value,
                    'detailed_status': FileDetailedStatus.FILE_NOT_RECEIVED.value
                } in metadata_file_results
        assert len(metadata_file_results) == nbr_results_file_expected

    def test_register_missing_files_no_period_checked(self, dir_paths):
        metadata_path = dir_paths["metadata"]
        files_configs = self._get_files_configs(self.min_expected_date_1, mode='monthly', period_checked=0)
        test_config = self._get_config_dict(metadata_path, files_configs)
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)

        register_missing_files(test_config.files_configs, metadata_path, workflow_metadata, self.run_date)

        metadata_file_path = fspath.join(metadata_path, WORKFLOW_FILE_STATUS)
        if not fspath.exists(metadata_file_path):
            assert not fspath.exists(metadata_file_path)
        else:
            with pytest.raises(TableNotFoundError):
                pl.scan_delta(metadata_file_path).collect()


```

###### FILE: test/metadata/parsing/archive_analyzers/test_tar_analyzer.py ######

```py
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.analyzer import TarAnalyzer
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata
from test.metadata.parsing.conftest import dir_paths, test_config


class TestTarAnalyzer:
    def setup_method(self):
        self.tables = ["table_1.txt", "table_2.txt", "table_3.txt", "table_4.txt"]

    def _prepare(self, loader, test_config):
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)
        analyzer = TarAnalyzer(workflow_metadata=workflow_metadata, config=test_config)
        data_path = loader.dest("data")
        fspath.makedirs(data_path, exist_ok=True)
        for table in self.tables:
            fspath.touch(fspath.join(data_path, table))
        return analyzer, data_path

    def test_tar_analyzer_nomismatch(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240101-20240101000000.tar"
        file = fspath.join(data_path, file_name)
        with fspath.FsspecTarFile(file, mode="w") as t:
            for table in self.tables[:3]:
                t.add(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)
        assert not analyzer.is_mismatch_tables(file, "20240101", "file_A",files_tables)

    def test_tar_analyzer_missing_table(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240102-20240102000000.tar"
        file = fspath.join(data_path, file_name)
        with fspath.FsspecTarFile(file, mode="w") as t:
            for table in self.tables[:2]:
                t.add(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)
        assert analyzer.is_mismatch_tables(file, "20240102", "file_A", files_tables)

    def test_tar_analyzer_extra_table(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240103-20240103000000.tar"
        file = fspath.join(data_path, file_name)
        with fspath.FsspecTarFile(file, mode="w") as t:
            for table in self.tables:
                t.add(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)
        assert analyzer.is_mismatch_tables(file, "20240103", "file_A", files_tables)

```

###### FILE: test/metadata/parsing/archive_analyzers/test_zip_analyzer.py ######

```py
import azfr_fsspec_utils as fspath
from azfr_fsspec_utils.zipfile import FsspecZipFile

from azfr_skywalker_utils.metadata.parsing.analyzer import ZipAnalyzer
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata
from test.metadata.parsing.conftest import loader, dir_paths, test_config


class TestZipAnalyzer:
    def setup_method(self):
        self.tables = ["table_1.txt", "table_2.txt", "table_3.txt", "table_4.txt"]

    def _prepare(self, loader, test_config):
        workflow_metadata = WorkflowMetadata()
        workflow_metadata.start(test_config, write_start_event=False)
        analyzer = ZipAnalyzer(workflow_metadata=workflow_metadata, config=test_config)
        data_path = loader.dest("data")
        fspath.makedirs(data_path, exist_ok=True)
        for table in self.tables:
            fspath.touch(fspath.join(data_path, table))
        return analyzer, data_path

    def test_zip_analyzer_nomismatch(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240101-20240101000000.zip"
        file = fspath.join(data_path, file_name)
        with FsspecZipFile(file, mode="w") as z:
            for table in self.tables[:3]:
                z.write(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)                
        assert not analyzer.is_mismatch_tables(file, "20240101", "file_A", files_tables)

    def test_zip_analyzer_missing_table(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240102-20240102000000.zip"
        file = fspath.join(data_path, file_name)
        with FsspecZipFile(file, mode="w") as z:
            for table in self.tables[:2]:
                z.write(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)
        assert analyzer.is_mismatch_tables(file, "20240102", "file_A", files_tables)

    def test_zip_analyzer_extra_table(self, loader, dir_paths, test_config):
        analyzer, data_path = self._prepare(loader, test_config)
        file_name = "file_A-20240103-20240103000000.zip"
        file = fspath.join(data_path, file_name)
        with FsspecZipFile(file, mode="w") as z:
            for table in self.tables:
                z.write(fspath.join(data_path, table), arcname=table)
        files_tables = analyzer.get_tables_from_file(file)
        assert analyzer.is_mismatch_tables(file, "20240103", "file_A", files_tables)

```

###### FILE: test/metadata/parsing/calendar/test_calendar.py ######

```py
from datetime import datetime as dt
from datetime import timedelta

import pytest

from azfr_skywalker_utils.metadata.parsing.calendar import get_expected_dates, is_expected_date, is_future_date


@pytest.mark.parametrize(
    ("date", "mode", "offset", "expected_days", "working_days_only", "expected"),
    [("20240921", "daily", 0, None, True, False),
     ("20240922", "daily", 0, None, True, False),
     ("20240923", "daily", 0, None, True, True),
     ("20241101", "daily", 0, None, True, False),
     ("20241111", "daily", 0, None, True, False),
     ("20241225", "daily", 0, None, True, False),
     ("20250101", "daily", 0, None, True, False),
     ("20250421", "daily", 0, None, True, False),
     ("20250501", "daily", 0, None, True, False),
     ("20250508", "daily", 0, None, True, False),
     ("20250609", "daily", 0, None, True, False),
     ("20250714", "daily", 0, None, True, False),
     ("20250815", "daily", 0, None, True, False),
     ("20400521", "daily", 0, None, True, False),
     ("20240921", "daily", 1, None, True, True),
     ("20240922", "daily", 1, None, True, False),
     ("20240923", "daily", 1, None, True, False),
     ("20241101", "daily", 1, None, True, True),
     ("20241102", "daily", 1, None, True, False),
     ("20240921", "daily", 0, None, False, True),
     ("20240922", "daily", 0, None, False, True),
     ("20240923", "daily", 0, None, False, True),
     ("20241101", "daily", 0, None, False, True),
     ("20241102", "daily", 0, None, False, True),
     ("20240924", "daily", 0, None, False, True),
     ("20241101", "weekly", 0, [5], True, False),
     ("20241102", "weekly", 0, [5], True, False),
     ("20241103", "weekly", 0, [5], True, False),
     ("20241104", "weekly", 0, [5], True, True),
     ("20241108", "weekly", 0, [5], True, True),
     ("20241109", "weekly", 0, [5], True, False),
     ("20241110", "weekly", 0, [5], True, False),
     ("20240923", "weekly", 0, [5], True, False),
     ("20241101", "weekly", 1, [5], True, True),
     ("20241108", "weekly", 1, [5], True, True),
     ("20240510", "weekly", 1, [5], True, False),
     ("20240923", "weekly", 1, [5], True, False),
     ("20241101", "weekly", 0, [5], False, True),
     ("20241108", "weekly", 0, [5], False, True),
     ("20240510", "weekly", 0, [5], False, True),
     ("20240923", "weekly", 0, [5], False, False),
     ("20240924", "weekly", 0, [2, 5], True, True),
     ("20240508", "weekly", 0, [2, 5], True, False),
     ("20240927", "weekly", 0, [2, 5], True, True),
     ("20240815", "weekly", 0, [2, 5], True, False),
     ("20240925", "weekly", 0, [2, 5], True, False),
     ("20270506", "monthly", 0, [6], True, False),
     ("20240906", "monthly", 0, [6], True, True),
     ("20241008", "monthly", 0, [6], True, False),
     ("20240706", "monthly", 0, [6], True, False),
     ("20241006", "monthly", 0, [6], True, False),
     ("20241106", "monthly", 1, [6], True, True),
     ("20241006", "monthly", 1, [6], True, False),
     ("20240506", "monthly", 1, [6], True, False),
     ("20280606", "monthly", 1, [6], True, False),
     ("20241108", "monthly", 1, [6], True, False),
     ("20270506", "monthly", 0, [6], False, True),
     ("20240906", "monthly", 0, [6], False, True),
     ("20241008", "monthly", 0, [6], False, False),
     ("20240706", "monthly", 0, [6], False, True),
     ("20241006", "monthly", 0, [6], False, True),
     ("20241106", "monthly", 0, [6], False, True),
     ("20240506", "monthly", 0, [6], False, True),
     ("20240606", "monthly", 0, [6], False, True),
     ("20270506", "monthly", 0, [6, 15], True, False),
     ("20270507", "monthly", 0, [6, 15], True, True),
     ("20270507", "monthly", 0, [6, 15], False, False),
     ("20270508", "monthly", 0, [6, 15], True, False),
     ("20240906", "monthly", 0, [6, 15], True, True),
     ("20240907", "monthly", 0, [6, 15], True, False),
     ("20240706", "monthly", 0, [6, 15], True, False),
     ("20240707", "monthly", 0, [6, 15], True, False),
     ("20240707", "monthly", 0, [6, 15], False, False),
     ("20240708", "monthly", 0, [6, 15], True, True),
     ("20240708", "monthly", 0, [6, 15], False, False),
     ("20241006", "monthly", 0, [6, 15], True, False),
     ("20240815", "monthly", 0, [6, 15], True, False),
     ("20241115", "monthly", 0, [6, 15], True, True),
     ("20250215", "monthly", 0, [6, 15], True, False),
     ("20240915", "monthly", 0, [6, 15], True, False),
     ("20241108", "monthly", 0, [6, 15], True, False),
     ]
)
def test_is_expected_date(date, mode, offset, expected_days, working_days_only, expected):
    assert is_expected_date(date, mode, offset, expected_days, working_days_only) == expected


@pytest.mark.parametrize(
    ("min_date", "mode", "offset", "expected_days", "working_days_only", "max_date", "expected"),
    [((dt.now() - timedelta(10)).strftime("%Y%m%d"), "daily", 0, None, False, dt.now().strftime("%Y%m%d"), [(dt.now() - timedelta(days=x)).strftime("%Y%m%d") for x in range(11)]),
     ("20240429", "daily", 0, None, True, "20240505", ['20240429', '20240430', '20240502', '20240503']),
     ("20240429", "daily", 1, None, True, "20240505", ['20240430', '20240501', '20240503', '20240504']),
     ("20240429", "daily", 0, None, False, "20240505", ['20240429', '20240430', '20240501', '20240502', '20240503', '20240504', '20240505']),
     ("20240429", "weekly", 0, [1, 3, 7], True, "20240505", ['20240429', '20240502']),
     ("20240429", "weekly", 1, [1, 3, 7], True, "20240505", ['20240430', '20240501']),
     ("20240429", "weekly", 0, [1, 3, 7], False, "20240505", ['20240429', '20240501', '20240505']),
     ("20240429", "monthly", 0, [1, 5, 30], True, "20240510", ['20240430', '20240502', '20240506']),
     ("20240429", "monthly", 1, [1, 5, 30], True, "20240510", ['20240430', '20240501', '20240507']),
     ("20240429", "monthly", 0, [1, 5, 30], False, "20240510", ['20240430', '20240501', '20240505']),
     ]
)
def test_get_expected_dates(min_date, mode, offset, expected_days, working_days_only, max_date, expected):
    assert sorted(get_expected_dates(min_date, mode, offset, expected_days, working_days_only, max_date)) == sorted(expected)


@pytest.mark.parametrize(
    ("date_to_check", "offset", "max_date", "expected"),
    [(dt.now().strftime("%Y%m%d"), 0, dt.now().strftime("%Y%m%d"), False),
     (dt.now().strftime("%Y%m%d"), 0, None, False),
     (dt.now().strftime("%Y%m%d"), 1, dt.now().strftime("%Y%m%d"), True),
     ("20240920", 0, "20240919", True),
     ]
)
def test_is_future_date(date_to_check, offset, max_date, expected):
    assert is_future_date(date_to_check, offset, max_date) == expected


def test_is_expected_date_invalid_mode_raises():
    """Test that invalid mode raises ValueError."""
    with pytest.raises(ValueError, match="Mode unsupported_mode not supported"):
        is_expected_date("20240921", "unsupported_mode", 0, None, False)


def test_is_expected_date_integer_expected_days_normalization():
    """Test that integer expected_days is normalized to list for monthly mode."""
    assert is_expected_date("20240906", "monthly", 0, 6, False) is True


def test_is_expected_date_weekly_with_int_expected_days():
    """Test weekly mode with integer expected_days normalization."""
    assert is_expected_date("20240924", "weekly", 0, 2, False) is True


def test_get_expected_dates_with_max_date_before_min_date():
    """Test that empty list is returned when max_date < min_date."""
    result = get_expected_dates("20240510", "daily", 0, None, False, "20240505")
    assert result == []


def test_is_first_working_day_after_expected_no_match():
    """Test _is_first_working_day_after_expected when no expected day is matched in lookback."""
    from azfr_skywalker_utils.metadata.parsing.calendar import _is_first_working_day_after_expected
    result = _is_first_working_day_after_expected(dt(2024, 9, 20), "weekly", 0, [7])
    assert result is False


def test_is_expected_date_working_days_only_with_first_working_day_match():
    """Test working_days_only flag with first working day after expected match."""
    assert is_expected_date("20240930", "weekly", 0, [1], True) is True


def test_matches_expected_day_daily_mode_returns_false():
    """_matches_expected_day with mode='daily' falls through all conditions and returns False (line 21)."""
    from azfr_skywalker_utils.metadata.parsing.calendar import _matches_expected_day
    result = _matches_expected_day(dt(2024, 9, 24), "daily", None)
    assert result is False


def test_get_expected_dates_no_max_date_defaults_to_today():
    """get_expected_dates with no max_date uses today as end date (covers the else branch at line 75)."""
    min_date = (dt.now() - timedelta(3)).strftime("%Y%m%d")
    result = get_expected_dates(min_date, "daily", 0, None, False)
    assert len(result) >= 1
    assert min_date in result


```

###### FILE: test/metadata/parsing/conftest.py ######

```py
import os
import re
import tempfile
import shutil
import pytest

import azfr_fsspec_utils
import azfr_fsspec_utils as fspath

from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata
from azfr_skywalker_utils.metadata.parsing.workflowconfig import FileConfig

from test.metadata.parsing import AnalyzerTest as Analyzer, FileFormatTest
from test.metadata.parsing import ConfigTest


class TestLoader(object):
    def __init__(self, test_dir, target_dir):
        self.test_dir = test_dir
        self.target_dir = target_dir

    def root_dir(self):
        return self.target_dir

    @classmethod
    def mk_dirs_with_file_system(cls, file_system, folder, raise_if_exists=True):
        file_system.mkdir(folder, raise_if_exists=raise_if_exists)

    @classmethod
    def mk_dirs(cls, folder):
        azfr_fsspec_utils.makedirs(folder, exist_ok=True)

    def path(self, path):
        return azfr_fsspec_utils.join(self.test_dir, path)

    def dest(self, path, target_dir=None):
        if target_dir is not None:
            target_dir = target_dir.format(self.target_dir)
        else:
            target_dir = self.target_dir
        result = azfr_fsspec_utils.join(target_dir, path)
        return result

    def copy_path_with_file_system(self, file_system, path, dest=None):
        src = self.path(path)
        dest = self.dest(dest or path)
        self.mk_dirs_with_file_system(file_system, azfr_fsspec_utils.dirname(dest))
        file_system.copy(src, dest)
        return dest

    def copy_path(self, path, dest=None):
        src = self.path(path)
        dest = self.dest(path) if dest is None else dest
        self.mk_dirs(azfr_fsspec_utils.dirname(dest))
        azfr_fsspec_utils.copy(src, dest)
        return dest


@pytest.fixture
def loader():
    # Use temporary directories for test and target
    test_dir = tempfile.mkdtemp(prefix="testloader_test_")
    target_dir = tempfile.mkdtemp(prefix="testloader_target_")
    loader = TestLoader(test_dir, target_dir)
    yield loader
    # Cleanup after test
    shutil.rmtree(test_dir, ignore_errors=True)
    shutil.rmtree(target_dir, ignore_errors=True)


def _find_project_root():
    """Find the project root by looking for setup.py"""
    current_path = os.path.abspath(os.path.dirname(__file__))
    while current_path != os.path.dirname(current_path):  # Until we reach the filesystem root
        if os.path.exists(os.path.join(current_path, 'setup.py')):
            return current_path
        current_path = os.path.dirname(current_path)
    raise RuntimeError("Could not find project root (setup.py not found)")


PROJECT_ROOT = _find_project_root()
TEST_DIR = os.path.join(PROJECT_ROOT, "test")
TARGET_DIR = os.path.join(PROJECT_ROOT, "target")
TARGET_TEST = os.path.join(TARGET_DIR, "tests")


def _paths(request):
    path = "{}".format(request.fspath)
    folder, file = os.path.split(path)
    test_dir = azfr_fsspec_utils.abspath(folder)
    rel_path = azfr_fsspec_utils.relpath(test_dir, TEST_DIR)
    target_dir = azfr_fsspec_utils.abspath(
        azfr_fsspec_utils.join(
            TARGET_DIR, rel_path, file.replace(".py", ""), request.function.__name__),
    )
    return test_dir, target_dir


@pytest.fixture(scope='session', autouse=True)
def remove_test_data():
    paths = [
        'file://{}',
    ]
    for path in paths:
        uri = path.format(TARGET_DIR)
        try:
            azfr_fsspec_utils.removedirs(path=uri)
        except FileNotFoundError:
            pass


def setup_test_directories(loader, *dir_names):
    """Setup multiple test directories"""
    paths = {}
    for dir_name in dir_names:
        paths[dir_name] = loader.dest(dir_name)
        if dir_name != "metadata":  # metadata dir is created by WorkflowMetadata
            fspath.makedirs(paths[dir_name], exist_ok=True)
    return paths


@pytest.fixture
def dir_paths(loader):
    return setup_test_directories(loader, "metadata", "landing", "archive")


@pytest.fixture
def test_config(dir_paths):
    files_configs = {
        "file_A": FileConfig(
            file_identifier="file_A",
            overdue_time="18:15",
            mode="daily",
            min_expected_date="",
            period_checked=30,
            working_days_only=True,
            tables=["table_1", "table_2", "table_3"])
    }
    return ConfigTest(
        metadata_dir=dir_paths['metadata'],
        archive_dir=dir_paths['archive'],
        input_dir=dir_paths['landing'],
        workflow_name="TEST-WORKFLOW",
        file_pattern=re.compile(r"(?P<file_identifier>[a-zA-Z_]+)-(?P<date>[0-9]{8})-(?P<timestamp>[0-9]{14})\.txt"),
        uncompressed_file_pattern=re.compile(r"(?P<name>[a-zA-Z_0-9]+)\.txt"),
        files_configs=files_configs,
        tables_to_format={"table_1": FileFormatTest(extraction_type="full"),
                          "table_2": FileFormatTest(extraction_type="delta")
                          # table_3 is not defined, but it doesn't break the tests
                          }
    )


@pytest.fixture
def analyzer(dir_paths, test_config):
    fspath.makedirs(fspath.join(dir_paths["archive"], "20240102"))
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(test_config, write_start_event=False)
    return Analyzer(workflow_metadata=workflow_metadata,config=test_config)

```

###### FILE: test/metadata/parsing/helpers.py ######

```py
from typing import Tuple, Optional


class MockState:
    """Mock Prefect-like state satisfying StateProtocol for use in tests."""

    def __init__(self, result: Optional[Tuple[int, int]] = None, completed: bool = True, message: str = ""):
        self._result = result
        self._completed = completed
        self._message = message

    def is_completed(self) -> bool:
        """Return whether the mock state represents a completed run."""
        return self._completed

    def result(self, raise_on_failure: bool = True, retry_result_failure: bool = True) -> Optional[Tuple[int, int]]:
        """Return the mock result tuple."""
        return self._result

    @property
    def message(self) -> str:
        """Return the mock message, formatted as Prefect does for failed states."""
        if not self._completed and self._message:
            return f"Task run encountered an exception Exception: {self._message}"
        return self._message


def dummy_parse(nb_parsed_lines: int, nb_rejected_lines: int, ko: bool, return_state: bool = False):
    """Return a MockState or raw result tuple for testing parse logic without Prefect API."""
    if return_state:
        if ko:
            return MockState(completed=False, message="dummy parse ko")
        return MockState(result=(nb_parsed_lines, nb_rejected_lines), completed=True)
    if ko:
        raise Exception("dummy parse ko")
    return nb_parsed_lines, nb_rejected_lines

```

###### FILE: test/metadata/parsing/workflow_metadata/test_workflow_metadata.py ######

```py
import tempfile
import types
import importlib
import re
from enum import Enum
from contextlib import contextmanager
import polars as pl
import azfr_fsspec_utils as fspath
import pytest

from datetime import datetime as _datetime
from azfr_skywalker_utils.metadata.parsing.metadata import (
    WORKFLOW_EVENTS, WORKFLOW_FILE_STATUS, WORKFLOW_TABLE_STATUS,
    FileDetailedStatus, FileStatus, TableDetailedStatus, TableStatus,
    WorkflowMetadata, get_now_UTC, get_registered_info,
    _is_effectively_finished, _process_missing_dates_for_file,
)
from deltalake.exceptions import TableNotFoundError
from azfr_skywalker_utils.metadata.parsing.exception import (
    RejectedRowsError,
    DeltaModeEmptyTableError,
    FullModeEmptyTableError,
    AnalyzerValidationError,
)
from azfr_skywalker_utils.metadata.parsing.workflowconfig import TableConfig, FileConfig, WorkflowBaseConfig
from azfr_skywalker_utils.trino.common import TrinoConfig
from azfr_skywalker_utils.utils.file import make_gen, count_file_lines
from azfr_skywalker_utils.utils.sql import qualified_table_name
from test.metadata.parsing import ConfigTest
from test.metadata.parsing.conftest import loader

class TestHelper:
    
    @staticmethod
    def write_test_events_and_statuses(workflow_metadata: WorkflowMetadata):
        """Write test events and status data for testing"""
        # Write file statuses
        workflow_metadata.write_file_status(
            "file_identifier_A", "20240920", FileStatus.SUCCESS,
            FileDetailedStatus.FILE_SUCCESS, "message", ["path/to/file"], get_now_UTC()
        )
        workflow_metadata.write_file_status(
            "file_identifier_A", "20240921", FileStatus.WAITING,
            FileDetailedStatus.FILE_WAITING, "message", ["path/to/file"], get_now_UTC()
        )
        
        # Write table status
        workflow_metadata.write_table_status(
            "file_identifier_A", "20240920", "table_A", TableStatus.SUCCESS,
            TableDetailedStatus.TABLE_SUCCESS, 100, 0, "OK", "path/to/file", get_now_UTC()
        )
    
    @staticmethod
    def get_expected_metadata_events():
        """Get expected metadata events for testing"""
        return [
            {
                'workflow_name': 'TEST-WORKFLOW', 
                'event_type': 'WORKFLOW_START',
                'event_info_json': None, 
                'wf_config_json': '{"param_1": "value_1"}'
            },
            {
                'workflow_name': 'TEST-WORKFLOW', 
                'event_type': 'WORKFLOW_END',
                'event_info_json': None, 
                'wf_config_json': '{"param_1": "value_1"}'
            }
        ]

    @staticmethod
    def get_expected_file_results():
        """Get expected file results"""
        return [
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_identifier_A',
             'version': '20240921', 'global_status': 'SUCCESS',
             'detailed_status': 'FILE_SUCCESS', 'message': 'message',
             'file_paths': ['path/to/file']},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_identifier_A',
             'version': '20240921', 'global_status': 'FAILED',
             'detailed_status': 'FILE_ERROR_TABLES', 'message': 'message',
             'file_paths': ['path/to/file']},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_identifier_A',
             'version': '20240921', 'global_status': 'WAITING',
             'detailed_status': 'FILE_WAITING', 'message': 'message',
             'file_paths': ['path/to/file']},
            {'workflow_name': 'TEST-WORKFLOW', 'file_identifier': 'file_identifier_A',
             'version': '20240920', 'global_status': 'SUCCESS',
             'detailed_status': 'FILE_SUCCESS', 'message': 'message',
             'file_paths': ['path/to/file']}
        ]

    @staticmethod
    def get_expected_table_results():
        """Get expected table results for testing"""
        return [
            {
                'workflow_name': 'TEST-WORKFLOW', 
                'file_identifier': 'file_identifier_A',
                'version': '20240920', 
                'table': 'table_A', 
                'global_status': 'SUCCESS',
                'detailed_status': 'TABLE_SUCCESS', 
                'nb_parsed_lines': 100, 
                'nb_rejected_lines': 0,
                'message': 'OK', 
                'file_path': 'path/to/file'
            }
        ]

    @staticmethod
    def assert_metadata_results(df_query, expected_results: list[dict], expected_count: int):
        """Assert metadata query results match expectations"""
        results = df_query.collect().to_dicts()
        assert len(results) == expected_count
        for expected in expected_results:
            assert expected in results


class TestWorkflowMetadata:

    def init(self, loader):
        self.loader = loader
        # Create a unique temp directory for metadata for each test
        self.metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
        self.test_config = ConfigTest(metadata_dir=self.metadata_path, workflow_name="TEST-WORKFLOW")
        self.workflow_metadata = WorkflowMetadata()

    def test_initial_state(self, loader):
        self.init(loader)
        assert get_registered_info(fspath.join(self.metadata_path, WORKFLOW_FILE_STATUS), ["file_identifier_A"]) == {'file_identifier_A': {}}

    def test_get_registered_info(self, loader):
        self.init(loader)
        self.workflow_metadata.start(self.test_config, wf_config_json='{\"param_1\": \"value_1\"}')
        TestHelper.write_test_events_and_statuses(self.workflow_metadata)
        expected_after_initial = {
            'file_identifier_A': {
                '20240920': {'detailed_status': FileDetailedStatus.FILE_SUCCESS.value, 'global_status': FileStatus.SUCCESS.value},
                '20240921': {'detailed_status': FileDetailedStatus.FILE_WAITING.value, 'global_status': FileStatus.WAITING.value}
            }
        }
        assert get_registered_info(fspath.join(self.metadata_path, WORKFLOW_FILE_STATUS), ["file_identifier_A"]) == expected_after_initial

        self.workflow_metadata.write_file_status("file_identifier_A", "20240921", FileStatus.FAILED,
                                                FileDetailedStatus.FILE_ERROR_TABLES,
                                                "message", ["path/to/file"], get_now_UTC())
        expected_after_failed = {
            'file_identifier_A': {
                '20240920': {'detailed_status': FileDetailedStatus.FILE_SUCCESS.value, 'global_status': FileStatus.SUCCESS.value},
                '20240921': {'detailed_status': FileDetailedStatus.FILE_ERROR_TABLES.value, 'global_status': FileStatus.FAILED.value}
            }
        }
        assert get_registered_info(fspath.join(self.metadata_path, WORKFLOW_FILE_STATUS), ["file_identifier_A"]) == expected_after_failed

        self.workflow_metadata.write_file_status("file_identifier_A", "20240921", FileStatus.SUCCESS,
                                                FileDetailedStatus.FILE_SUCCESS,
                                                "message", ["path/to/file"], get_now_UTC())
        expected_final = {
            'file_identifier_A': {
                '20240920': {'detailed_status': FileDetailedStatus.FILE_SUCCESS.value, 'global_status': FileStatus.SUCCESS.value},
                '20240921': {'detailed_status': FileDetailedStatus.FILE_SUCCESS.value, 'global_status': FileStatus.SUCCESS.value}
            }
        }
        assert get_registered_info(fspath.join(self.metadata_path, WORKFLOW_FILE_STATUS), ["file_identifier_A"]) == expected_final

        expected_date_specific = {
            'file_identifier_A': {
                '20240921': {'detailed_status': FileDetailedStatus.FILE_SUCCESS.value, 'global_status': FileStatus.SUCCESS.value}
            }
        }
        assert get_registered_info(fspath.join(self.metadata_path, WORKFLOW_FILE_STATUS), ["file_identifier_A"], "20240921") == expected_date_specific

    def test_workflow_events(self, loader):
        self.init(loader)
        self.workflow_metadata.start(self.test_config, wf_config_json='{"param_1": "value_1"}')
        TestHelper.write_test_events_and_statuses(self.workflow_metadata)
        self.workflow_metadata.end(wf_config_json='{"param_1": "value_1"}')

        df_metadata_events = pl.scan_delta(fspath.join(self.test_config.metadata_dir, WORKFLOW_EVENTS))
        expected_events = TestHelper.get_expected_metadata_events()
        TestHelper.assert_metadata_results(df_metadata_events.select(["workflow_name", "event_type", "event_info_json", "wf_config_json"]), expected_events, 2)

    def test_workflow_files(self, loader):
        self.init(loader)
        self.workflow_metadata.start(self.test_config, wf_config_json='{"param_1": "value_1"}')
        TestHelper.write_test_events_and_statuses(self.workflow_metadata)
        self.workflow_metadata.write_file_status("file_identifier_A", "20240921", FileStatus.FAILED, FileDetailedStatus.FILE_ERROR_TABLES,"message", ["path/to/file"], get_now_UTC())
        self.workflow_metadata.write_file_status("file_identifier_A", "20240921", FileStatus.SUCCESS, FileDetailedStatus.FILE_SUCCESS, "message", ["path/to/file"], get_now_UTC())
        self.workflow_metadata.end(wf_config_json='{"param_1": "value_1"}')

        df_metadata_file = pl.scan_delta(fspath.join(self.test_config.metadata_dir, WORKFLOW_FILE_STATUS))
        expected_file_results = TestHelper.get_expected_file_results()
        TestHelper.assert_metadata_results(df_metadata_file.select(["workflow_name", "file_identifier", "version", "global_status", "detailed_status", "message", "file_paths"]), expected_file_results, 4)

    def test_workflow_tables(self, loader):
        self.init(loader)
        self.workflow_metadata.start(self.test_config, wf_config_json='{"param_1": "value_1"}')
        TestHelper.write_test_events_and_statuses(self.workflow_metadata)
        self.workflow_metadata.end(wf_config_json='{"param_1": "value_1"}')

        df_metadata_table = pl.scan_delta(fspath.join(self.test_config.metadata_dir, WORKFLOW_TABLE_STATUS))
        expected_table_results = TestHelper.get_expected_table_results()
        TestHelper.assert_metadata_results(df_metadata_table.select(["workflow_name", "file_identifier", "version", "table", "global_status", "detailed_status", "nb_parsed_lines", "nb_rejected_lines", "message", "file_path"]), expected_table_results, 1)


def test_qualified_table_name_with_catalog_and_schema():
    assert qualified_table_name("catalog_a", "schema_b", "table_c") == "catalog_a.schema_b.table_c"


def test_qualified_table_name_with_schema_only():
    assert qualified_table_name(None, "schema_b", "table_c") == "schema_b.table_c"


def test_qualified_table_name_with_table_only():
    assert qualified_table_name(None, None, "table_c") == "table_c"


@pytest.mark.parametrize("table", ["", None])
def test_qualified_table_name_raises_when_table_missing(table):
    with pytest.raises(ValueError, match="Table must be provided"):
        qualified_table_name("catalog_a", "schema_b", table)


def test_make_gen_yields_all_chunks():
    chunks = [b"abc", b"def", b"", b"ignored"]

    def reader(_size):
        return chunks.pop(0)

    assert list(make_gen(reader)) == [b"abc", b"def"]


def test_count_file_lines_without_offset(tmp_path):
    path = tmp_path / "sample.txt"
    path.write_bytes(b"row1\nrow2\nrow3\n")

    assert count_file_lines(str(path)) == 3


def test_count_file_lines_with_offset(tmp_path):
    path = tmp_path / "sample.txt"
    path.write_bytes(b"row1\nrow2\n")

    assert count_file_lines(str(path), offset=1) == 3


def test_parsing_exceptions_messages_and_payloads():
    assert str(RejectedRowsError(["table_1"])) == "Table with rejected rows detected. 1 table."
    assert str(RejectedRowsError(["table_1", "table_2"])) == "Table with rejected rows detected. 2 tables."

    delta_exc = DeltaModeEmptyTableError(["table_1", "table_2"])
    assert str(delta_exc) == "Table in delta mode but empty detected. 2 tables."

    full_exc = FullModeEmptyTableError(["table_1"])
    assert str(full_exc) == "Table in full mode but empty detected. 1 table."

    validation_exc = AnalyzerValidationError(["err_1", "err_2"])
    assert validation_exc.errors == ["err_1", "err_2"]
    assert "List of errors for this run" in str(validation_exc)


def _build_trino_test_config() -> TrinoConfig:
    return TrinoConfig(
        host="trino.local",
        port=443,
        http_scheme="https",
        ssl_verify=True,
        access_token_scope="api://scope/.default",
        user="john.doe+svc",
        catalog="catalog_a",
        schema_name="schema_b",
    )


def test_build_trino_sqlalchemy_url_quotes_user():
    from azfr_skywalker_utils.trino.sql_alchemy import build_trino_sqlalchemy_url

    config = _build_trino_test_config()
    assert build_trino_sqlalchemy_url(config) == "trino://john.doe%2Bsvc@trino.local:443/catalog_a/schema_b"


def test_trino_engine_disposes_after_context(monkeypatch):
    import azfr_skywalker_utils.trino.sql_alchemy as trino_sql_alchemy

    captured = {}

    class DummyEngine:
        def __init__(self):
            self.disposed = False

        def dispose(self):
            self.disposed = True

    engine = DummyEngine()

    class DummyCredential:
        def get_token(self, _scope):
            return types.SimpleNamespace(token="token_abc")

    def fake_create_engine(url, connect_args):
        captured["url"] = url
        captured["connect_args"] = connect_args
        return engine

    monkeypatch.setattr(trino_sql_alchemy, "AzfrDefaultCredential", lambda: DummyCredential())
    monkeypatch.setattr(trino_sql_alchemy, "create_engine", fake_create_engine)

    with trino_sql_alchemy.trino_engine(_build_trino_test_config()) as returned_engine:
        assert returned_engine is engine

    assert engine.disposed is True
    assert captured["url"].startswith("trino://")
    assert captured["connect_args"]["http_scheme"] == "https"


def test_trino_session_rollback_on_error(monkeypatch):
    import azfr_skywalker_utils.trino.sql_alchemy as trino_sql_alchemy

    class DummySession:
        def __init__(self):
            self.closed = False
            self.rolled_back = False

        def rollback(self):
            self.rolled_back = True

        def close(self):
            self.closed = True

    created = {}

    def fake_sessionmaker(bind):
        assert bind == "dummy_engine"

        def factory():
            created["session"] = DummySession()
            return created["session"]

        return factory

    monkeypatch.setattr(trino_sql_alchemy, "sessionmaker", fake_sessionmaker)

    with pytest.raises(RuntimeError):
        with trino_sql_alchemy.trino_session("dummy_engine"):
            raise RuntimeError("boom")

    assert created["session"].rolled_back is True
    assert created["session"].closed is True


def test_dbapi_trino_connection_and_cursor(monkeypatch):
    from contextlib import contextmanager
    import azfr_skywalker_utils.trino.dbapi as trino_dbapi

    captured = {}

    class DummyCredential:
        def get_token(self, _scope):
            return types.SimpleNamespace(token="token_abc")

    class DummyCursor:
        def __init__(self):
            self.closed = False

        def close(self):
            self.closed = True

    class DummyConnection:
        def __init__(self):
            self.cursor_instance = DummyCursor()

        def cursor(self):
            return self.cursor_instance

    @contextmanager
    def fake_connect(**kwargs):
        captured.update(kwargs)
        yield DummyConnection()

    monkeypatch.setattr(trino_dbapi, "AzfrDefaultCredential", lambda: DummyCredential())
    monkeypatch.setattr(trino_dbapi, "connect", fake_connect)

    with trino_dbapi.trino_connection(_build_trino_test_config()) as connection:
        assert connection is not None
        with trino_dbapi.get_cursor(connection) as cursor:
            assert cursor is connection.cursor_instance
        assert connection.cursor_instance.closed is True

    assert captured["catalog"] == "catalog_a"
    assert captured["schema"] == "schema_b"
    assert captured["user"] == "john.doe+svc"


def test_ibis_connection_disconnect_called(monkeypatch):
    pytest.importorskip("ibis")
    import azfr_skywalker_utils.ibis.trino as ibis_trino

    class DummyCredential:
        def get_token(self, _scope):
            return types.SimpleNamespace(token="token_abc")

    class DummyIbisConn:
        def __init__(self):
            self.disconnected = False

        def disconnect(self):
            self.disconnected = True

    dummy_conn = DummyIbisConn()

    def fake_ibis_connect(**kwargs):
        assert kwargs["database"] == "catalog_a"
        assert kwargs["schema"] == "schema_b"
        return dummy_conn

    monkeypatch.setattr(ibis_trino, "AzfrDefaultCredential", lambda: DummyCredential())
    monkeypatch.setattr(ibis_trino.ibis.trino, "connect", fake_ibis_connect)

    with ibis_trino.ibis_connection(_build_trino_test_config()) as connection:
        assert connection is dummy_conn

    assert dummy_conn.disconnected is True


def test_archive_helpers_cover_main_paths(monkeypatch):
    import azfr_skywalker_utils.utils.archive as archive_module

    moved = []

    monkeypatch.setattr(archive_module.fspath, "makedirs", lambda *_args, **_kwargs: None)
    monkeypatch.setattr(archive_module.fspath, "basename", lambda p: p.split("/")[-1])
    monkeypatch.setattr(archive_module.fspath, "join", lambda a, b: f"{a}/{b}")
    monkeypatch.setattr(archive_module.fspath, "exists", lambda p: p.endswith("existing.csv"))
    monkeypatch.setattr(archive_module.fspath, "move", lambda src, dst: moved.append((src, dst)))

    with pytest.raises(FileExistsError):
        archive_module._archive("landing/existing.csv", "archive")

    archived = archive_module._archive("landing/new.csv", "archive")
    assert archived == "archive/new.csv"
    assert moved == [("landing/new.csv", "archive/new.csv")]


def test_rollback_moves_only_non_existing_landing_files(monkeypatch):
    import azfr_skywalker_utils.utils.archive as archive_module

    class DummyLogger:
        def info(self, _msg):
            return None

    moved = []
    monkeypatch.setattr(archive_module, "get_run_logger", lambda: DummyLogger())
    monkeypatch.setattr(archive_module.fspath, "listdir", lambda _p: ["archive/a.csv", "archive/b.csv"])
    monkeypatch.setattr(archive_module.fspath, "basename", lambda p: p.split("/")[-1])
    monkeypatch.setattr(archive_module.fspath, "join", lambda a, b: f"{a}/{b}")
    monkeypatch.setattr(archive_module.fspath, "exists", lambda p: p.endswith("a.csv"))
    monkeypatch.setattr(archive_module.fspath, "move", lambda src, dst: moved.append((src, dst)))

    archive_module._rollback("archive", "landing")
    assert moved == [("archive/b.csv", "landing/b.csv")]


def test_uncompress_dispatch_and_unsupported_format(monkeypatch):
    import azfr_skywalker_utils.utils.archive as archive_module

    monkeypatch.setattr(archive_module, "untar", lambda _in, _out: ["tar-file"])
    monkeypatch.setattr(archive_module, "unzip", lambda _in, _out: ["zip-file"])

    assert archive_module.uncompress.fn("input.tar", "out", "tar") == ["tar-file"]
    assert archive_module.uncompress.fn("input.zip", "out", "zip") == ["zip-file"]
    with pytest.raises(ValueError, match="Unsupported compression format"):
        archive_module.uncompress.fn("input.rar", "out", "rar")


def test_untar_and_unzip_extract_and_return_paths(monkeypatch):
    import azfr_skywalker_utils.utils.archive as archive_module

    extracted = {"tar": None, "zip": []}
    writes = {}

    class DummyTar:
        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def getmembers(self):
            return [types.SimpleNamespace(name="a.csv"), types.SimpleNamespace(name="b.csv")]

        def extractall(self, out_dir):
            extracted["tar"] = out_dir

    class DummyZip:
        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def infolist(self):
            return [types.SimpleNamespace(filename="x.csv"), types.SimpleNamespace(filename="y.csv")]

        def read(self, member):
            return member.filename.encode()

    class DummyWriter:
        def __init__(self, path):
            self.path = path

        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def write(self, data):
            writes[self.path] = data

    @contextmanager
    def fake_open(path, compression=None, mode="rb"):
        if compression == "tar":
            yield DummyTar()
        else:
            # unzip uses fspath.open(extract_file, 'wb') where 'wb' is passed positionally
            # and ends up as the second argument in this fake.
            open_mode = compression if compression in {"wb", "rb"} else mode
            assert open_mode == "wb"
            yield DummyWriter(path)

    monkeypatch.setattr(archive_module.fspath, "open", fake_open)
    monkeypatch.setattr(archive_module.fspath, "join", lambda a, b: f"{a}/{b}")
    monkeypatch.setattr(archive_module.fspath, "makedirs", lambda *_args, **_kwargs: None)
    monkeypatch.setattr(archive_module, "FsspecZipFile", lambda _path: DummyZip())

    tar_paths = archive_module.untar("archive.tar", "out")
    zip_paths = archive_module.unzip("archive.zip", "out")

    assert tar_paths == ["out/a.csv", "out/b.csv"]
    assert extracted["tar"] == "out"
    assert zip_paths == ["out/x.csv", "out/y.csv"]
    assert writes["out/x.csv"] == b"x.csv"
    assert writes["out/y.csv"] == b"y.csv"


def test_archive_file_rolls_back_and_reraises(monkeypatch):
    import azfr_skywalker_utils.utils.archive as archive_module

    class DummyLogger:
        def error(self, _msg):
            return None

    rolled_back = {}

    monkeypatch.setattr(archive_module, "get_run_logger", lambda: DummyLogger())
    monkeypatch.setattr(archive_module, "_archive", lambda *_args, **_kwargs: (_ for _ in ()).throw(RuntimeError("boom")))
    monkeypatch.setattr(archive_module.fspath, "dirname", lambda p: p.rsplit("/", 1)[0])
    monkeypatch.setattr(
        archive_module,
        "_rollback",
        lambda src, dst: rolled_back.update({"archive_path": src, "landing_path": dst}),
    )

    with pytest.raises(RuntimeError, match="boom"):
        archive_module.archive_file.fn("landing/data.csv", "archive")

    assert rolled_back == {"archive_path": "archive", "landing_path": "landing"}


def test_main_flow_with_validation_registers_and_raises(monkeypatch):
    flow_module = importlib.import_module("azfr_skywalker_utils.metadata.parsing.flow")

    class DummyLogger:
        def info(self, _msg):
            return None

    class DummyWorkflowMetadata:
        def __init__(self):
            self.started = False
            self.ended = False

        def start(self, _config, write_start_event=True):
            self.started = write_start_event

        def end(self):
            self.ended = True

    class DummyAnalyzer:
        def __init__(self, **_kwargs):
            self.errors = ["invalid file"]

        def analyze_landing_files(self, landing_files, check_mismatch_tables=True):
            assert check_mismatch_tables is True
            return landing_files

    calls = {"register_missing_files": 0, "archive": []}

    config = types.SimpleNamespace(
        to_json=lambda: "{}",
        input_dir="landing",
        file_pattern="*.csv",
        validate_files=True,
        files_configs=[{"id": "file_A"}],
        metadata_dir="metadata",
        date_format="%Y%m%d",
        time_format="%H%M%S",
    )

    monkeypatch.setattr(flow_module, "get_run_logger", lambda: DummyLogger())
    monkeypatch.setattr(flow_module, "WorkflowMetadata", DummyWorkflowMetadata)
    monkeypatch.setattr(flow_module, "get_files", lambda *_args, **_kwargs: ["landing/f1.csv"])
    monkeypatch.setattr(
        flow_module,
        "register_missing_files",
        lambda *_args, **_kwargs: calls.update({"register_missing_files": calls["register_missing_files"] + 1}),
    )

    def archive_and_parse(cfg, landing_file, analyzer, return_state=True):
        assert cfg is config
        assert return_state is True
        assert analyzer.errors == ["invalid file"]
        calls["archive"].append(landing_file)

    with pytest.raises(AnalyzerValidationError):
        flow_module.main_flow.fn(config, None, archive_and_parse, DummyAnalyzer)

    assert calls["archive"] == ["landing/f1.csv"]
    assert calls["register_missing_files"] == 1


def test_main_flow_without_validation_skips_register_and_analyzer(monkeypatch):
    flow_module = importlib.import_module("azfr_skywalker_utils.metadata.parsing.flow")

    class DummyLogger:
        def info(self, _msg):
            return None

    class DummyWorkflowMetadata:
        def start(self, _config, write_start_event=True):
            return None

        def end(self):
            return None

    config = types.SimpleNamespace(
        to_json=lambda: "{}",
        input_dir="landing",
        file_pattern="*.csv",
        validate_files=False,
        files_configs=[],
        metadata_dir="metadata",
        date_format="%Y%m%d",
        time_format="%H%M%S",
    )

    monkeypatch.setattr(flow_module, "get_run_logger", lambda: DummyLogger())
    monkeypatch.setattr(flow_module, "WorkflowMetadata", DummyWorkflowMetadata)
    monkeypatch.setattr(flow_module, "get_files", lambda *_args, **_kwargs: [])

    def fail_if_called(*_args, **_kwargs):
        raise AssertionError("should not be called")

    monkeypatch.setattr(flow_module, "register_missing_files", fail_if_called)

    class DummyAnalyzer:
        def __init__(self, **_kwargs):
            raise AssertionError("analyzer should not be constructed when validation is disabled")

    flow_module.main_flow.fn(config, None, fail_if_called, DummyAnalyzer)


def test_workflow_run_service_from_dict_filters_extra_and_can_create_table():
    abstract_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_run.abstract")

    class DummyDetailedStatus(str, Enum):
        SUCCESS = "SUCCESS"
        TECHNICAL_ERROR = "TECHNICAL_ERROR"

    class DummyColumns:
        @staticmethod
        def keys():
            return ["workflow_name", "version"]

    class DummyTable:
        columns = DummyColumns()

    class DummyMetadata:
        __table__ = DummyTable()
        __tablename__ = "dummy_table"

        def __init__(self, **kwargs):
            self.kwargs = kwargs
            self.run_id = None
            self.workflow_name = kwargs.get("workflow_name")
            self.version = kwargs.get("version")
            self.global_status = None
            self.detailed_status = None
            self.detailed_status_enum = DummyDetailedStatus
            self.create_table_called_with = None

        def create_table(self, engine):
            self.create_table_called_with = engine

    class DummyService(abstract_module.AbstractWorkflowRunService):
        @classmethod
        def get_metadata_class(cls):
            return DummyMetadata

    service = DummyService.from_dict(
        {
            "workflow_name": "wf_a",
            "version": "20260413",
            "ignored": "x",
            "create_metadata_tables": True,
        },
        engine="engine_a",
    )

    assert service.metadata.kwargs == {"workflow_name": "wf_a", "version": "20260413"}
    assert service.metadata.create_table_called_with == "engine_a"


def test_workflow_run_service_status_lifecycle_and_save(monkeypatch):
    abstract_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_run.abstract")

    class DummyDetailedStatus(str, Enum):
        SUCCESS = "SUCCESS"
        TECHNICAL_ERROR = "TECHNICAL_ERROR"
        BUSINESS_ERROR = "BUSINESS_ERROR"

    class DummyMetadata:
        run_id = None
        workflow_name = "wf"
        version = "v1"
        global_status = None
        detailed_status = None
        detailed_status_enum = DummyDetailedStatus
        start_ts = None
        end_ts = None
        creator_github_repository = None
        creator_commit_id = None
        __tablename__ = "dummy_table"

        def create_table(self, _engine):
            return None

    class DummyService(abstract_module.AbstractWorkflowRunService):
        @classmethod
        def get_metadata_class(cls):
            raise NotImplementedError

    class DummySession:
        def __init__(self):
            self.added = None
            self.committed = False

        def add(self, value):
            self.added = value

        def commit(self):
            self.committed = True

    session = DummySession()

    @contextmanager
    def fake_trino_session(_engine):
        yield session

    monkeypatch.setattr(abstract_module, "trino_session", fake_trino_session)
    monkeypatch.setenv("GITHUB_REPOSITORY", "org/repo")
    monkeypatch.setenv("GITHUB_SHA", "abc123")
    monkeypatch.setattr(abstract_module.uuid, "uuid4", lambda: "run-123")

    service = DummyService(DummyMetadata(), engine="engine_x")
    entered = service.__enter__()
    assert entered is service
    assert service.metadata.run_id == "run-123"
    assert service.metadata.creator_github_repository == "org/repo"
    assert service.metadata.creator_commit_id == "abc123"

    service.mark_success()
    assert service.metadata.global_status == abstract_module.WorkflowGlobalStatus.SUCCESS
    assert service.metadata.detailed_status == DummyDetailedStatus.SUCCESS

    service.set_detailed_status(DummyDetailedStatus.BUSINESS_ERROR)
    assert service.metadata.detailed_status == DummyDetailedStatus.SUCCESS

    service.set_detailed_status(DummyDetailedStatus.BUSINESS_ERROR, overwrite=True)
    assert service.metadata.detailed_status == DummyDetailedStatus.BUSINESS_ERROR

    service.mark_failed(DummyDetailedStatus.TECHNICAL_ERROR)
    assert service.metadata.global_status == abstract_module.WorkflowGlobalStatus.FAILED

    service.save()
    assert session.added is service.metadata
    assert session.committed is True

    with pytest.raises(ValueError):
        service.set_detailed_status("bad_status")


def test_workflow_run_service_fail_if_not_completed_and_exit_paths(monkeypatch):
    abstract_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_run.abstract")

    class DummyDetailedStatus(str, Enum):
        SUCCESS = "SUCCESS"
        TECHNICAL_ERROR = "TECHNICAL_ERROR"
        BUSINESS_ERROR = "BUSINESS_ERROR"

    class DummyMetadata:
        run_id = None
        global_status = None
        detailed_status = None
        detailed_status_enum = DummyDetailedStatus
        end_ts = None
        __tablename__ = "dummy_table"

        def create_table(self, _engine):
            return None

    class DummyService(abstract_module.AbstractWorkflowRunService):
        @classmethod
        def get_metadata_class(cls):
            raise NotImplementedError

    class DummyState:
        def __init__(self, completed):
            self._completed = completed

        def is_completed(self):
            return self._completed

    monkeypatch.setattr(abstract_module, "State", DummyState)

    service = DummyService(DummyMetadata(), engine="engine_x")

    with pytest.raises(TypeError):
        service.fail_if_not_completed(object(), DummyDetailedStatus.BUSINESS_ERROR)

    service.fail_if_not_completed(DummyState(completed=True), DummyDetailedStatus.BUSINESS_ERROR)
    assert service.metadata.global_status is None

    service.fail_if_not_completed(DummyState(completed=False), DummyDetailedStatus.BUSINESS_ERROR, raise_exception=False)
    assert service.metadata.global_status == abstract_module.WorkflowGlobalStatus.FAILED
    assert service.metadata.detailed_status == DummyDetailedStatus.BUSINESS_ERROR

    with pytest.raises(RuntimeError):
        service.fail_if_not_completed(DummyState(completed=False), DummyDetailedStatus.BUSINESS_ERROR, raise_exception=True)

    saved = {"count": 0}
    monkeypatch.setattr(service, "save", lambda: saved.update({"count": saved["count"] + 1}))
    service.__exit__(None, None, None)
    assert saved["count"] == 1
    assert service.metadata.global_status == abstract_module.WorkflowGlobalStatus.SUCCESS

    service.metadata.detailed_status = None
    service.__exit__(RuntimeError, RuntimeError("boom"), None)
    assert service.metadata.global_status == abstract_module.WorkflowGlobalStatus.FAILED
    assert service.metadata.detailed_status == DummyDetailedStatus.TECHNICAL_ERROR


@pytest.mark.parametrize(
    "value, expected",
    [
        (20260413, ["20260413"]),
        ("20260413, 20260414", ["20260413", "20260414"]),
        (["20260413", "20260414"], ["20260413", "20260414"]),
    ],
)
def test_parse_versions_supported_inputs(value, expected):
    dbt_common = importlib.import_module("azfr_skywalker_utils.dbt.common")
    assert dbt_common.parse_versions(value) == expected


def test_parse_versions_invalid_input_type():
    dbt_common = importlib.import_module("azfr_skywalker_utils.dbt.common")
    with pytest.raises(TypeError):
        dbt_common.parse_versions({"not": "valid"})


def test_prepare_dbt_command_with_and_without_vars():
    dbt_common = importlib.import_module("azfr_skywalker_utils.dbt.common")

    cmd_with_vars = dbt_common.prepare_dbt_command(
        dbt_command="run",
        dbt_vars={"threads": 4, "region": "eu"},
        additional_args="--select model_a",
        project_dir="dbt-macros",
    )
    assert "dbt run --project-dir dbt-macros" in cmd_with_vars
    assert "--threads 4" in cmd_with_vars
    assert "--vars" in cmd_with_vars
    assert "--select model_a" in cmd_with_vars

    cmd_without_vars = dbt_common.prepare_dbt_command("test", None)
    assert cmd_without_vars.startswith("dbt test --project-dir dbt_project")
    assert "--vars" not in cmd_without_vars


def test_run_shell_command_task(monkeypatch):
    dbt_common = importlib.import_module("azfr_skywalker_utils.dbt.common")

    seen = {"commands": None, "logs": []}

    class DummyLogger:
        def info(self, message):
            seen["logs"].append(message)

    class DummyShellOperation:
        def __init__(self, commands):
            seen["commands"] = commands

        def run(self):
            return {"status": "ok"}

    monkeypatch.setattr(dbt_common, "get_run_logger", lambda: DummyLogger())
    monkeypatch.setattr(dbt_common, "ShellOperation", DummyShellOperation)

    output = dbt_common.run_shell_command.fn("echo hello")
    assert output == {"status": "ok"}
    assert seen["commands"] == ["echo hello"]
    assert any("Running command" in msg for msg in seen["logs"])


def test_workflow_run_datavault_create_table_and_service_class():
    datavault_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_run.datavault")

    executed = {"sql": None}

    class DummyConn:
        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def execute(self, stmt):
            executed["sql"] = str(stmt)

    class DummyEngine:
        def connect(self):
            return DummyConn()

    metadata = datavault_module.WorkflowRunDatavault()
    assert metadata.detailed_status_enum is datavault_module.DatasetDetailedStatus

    metadata.create_table(DummyEngine())
    assert "CREATE TABLE IF NOT EXISTS" in executed["sql"]
    assert "dataset varchar" in executed["sql"]

    assert datavault_module.DatavaultWorkflowRunService.get_metadata_class() is datavault_module.WorkflowRunDatavault


def test_workflow_run_extraction_create_table_and_service_class():
    extraction_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_run.extraction")

    executed = {"sql": None}

    class DummyConn:
        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def execute(self, stmt):
            executed["sql"] = str(stmt)

    class DummyEngine:
        def connect(self):
            return DummyConn()

    metadata = extraction_module.WorkflowRunExtract()
    assert metadata.detailed_status_enum is extraction_module.ExtractionDetailedStatus

    metadata.create_table(DummyEngine())
    assert "CREATE TABLE IF NOT EXISTS" in executed["sql"]
    assert "generated_files array(varchar)" in executed["sql"]

    assert extraction_module.ExtractionWorkflowRunService.get_metadata_class() is extraction_module.WorkflowRunExtract


def test_workflow_registry_getters_and_validation_branches(tmp_path):
    registry_module = importlib.import_module("azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry")

    config = {
        "workflows": {
            "parsing.source_a": {
                "layer": "parsing",
                "metadata_catalog": "cat_a",
                "metadata_schema": "sch_a",
            },
            "extraction.target": {
                "layer": "extraction",
                "metadata_catalog": "cat_b",
                "metadata_schema": "sch_b",
                "depends_on": ["parsing.source_a"],
            },
        }
    }

    registry = registry_module.WorkflowRegistry.from_dict(config)
    assert registry.get_workflows_by_layer(registry_module.LayerType.PARSING) == ["parsing.source_a"]
    assert registry.get_workflows_depending_on("parsing.source_a") == ["extraction.target"]

    with pytest.raises(ValueError, match="Workflow not found"):
        registry.get_workflow_config("missing.workflow")

    with pytest.raises(ValueError, match="Dependency dict must have exactly one workflow name key"):
        registry_module.WorkflowEntry(
            name="bad.workflow",
            layer=registry_module.LayerType.EXTRACTION,
            metadata_catalog="cat",
            metadata_schema="sch",
            depends_on=[{"a": {"optional": True}, "b": {"optional": False}}],
        )

    yaml_path = tmp_path / "registry.yml"
    yaml_path.write_text(
        """
workflows:
  parsing.source_a:
    layer: parsing
    metadata_catalog: cat_a
    metadata_schema: sch_a
""".strip(),
        encoding="utf-8",
    )
    yaml_registry = registry_module.WorkflowRegistry.from_yaml(str(yaml_path))
    assert yaml_registry.get_workflow_config("parsing.source_a").name == "parsing.source_a"


def test_workflow_metadata_create_additional_columns_shapes(loader):
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    additional_columns, metadata_expr = workflow_metadata.create_additional_columns("20260413", "file_a.txt")
    assert len(additional_columns) == 2
    assert additional_columns[0].name == "__functional_date__"
    assert additional_columns[1].name == "__version__"
    assert metadata_expr is not None


def test_get_registered_info_handles_table_not_found(monkeypatch):
    import azfr_skywalker_utils.metadata.parsing.metadata as metadata_module

    def raise_table_not_found(_path):
        raise TableNotFoundError("missing table")

    monkeypatch.setattr(metadata_module.pl, "scan_delta", raise_table_not_found)
    result = metadata_module.get_registered_info("dummy/path", ["file_A", "file_B"])
    assert result == {"file_A": {}, "file_B": {}}


def _build_workflowconfig_file_config(**overrides):
    base = {
        "file_identifier": "file_A",
        "overdue_time": "10:30",
        "mode": "daily",
        "min_expected_date": "20240101",
        "period_checked": 7,
    }
    base.update(overrides)
    return FileConfig(**base)


def test_workflowconfig_table_config_date_parsing_and_error_branch():
    table_cfg = TableConfig(raise_error_if_empty_after=20240131)
    assert table_cfg.raise_error_if_empty_after.isoformat() == "2024-01-31"

    with pytest.raises(ValueError, match="YYYYMMDD"):
        TableConfig(raise_error_if_empty_after="2024-01-31")


def test_workflowconfig_file_config_invalid_overdue_time_raises():
    with pytest.raises(ValueError, match="Invalid format for overdue_time"):
        _build_workflowconfig_file_config(overdue_time="25:99")


def test_workflowconfig_pattern_validator_non_string_and_to_json():
    file_cfg = _build_workflowconfig_file_config()
    workflow_cfg = WorkflowBaseConfig(
        workflow_name="WF-A",
        metadata_dir="/tmp/metadata",
        file_pattern=re.compile(r"^.*$"),
        uncompressed_file_pattern=re.compile(r"^.*\.csv$"),
        files_configs={"file_A": file_cfg},
        archive_dir="/tmp/archive",
        input_dir="/tmp/input",
        tables_to_format={"table_a": "csv"},
    )

    assert isinstance(workflow_cfg.file_pattern, re.Pattern)
    assert isinstance(workflow_cfg.uncompressed_file_pattern, re.Pattern)

    as_json = workflow_cfg.to_json()
    assert "tables_to_format" not in as_json


def test_workflow_metadata_get_registered_info_with_date_filter(loader):
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    workflow_metadata.write_file_status("file_id_1", "20240101", FileStatus.SUCCESS,
                                       FileDetailedStatus.FILE_SUCCESS, "msg", ["p1"], get_now_UTC())
    workflow_metadata.write_file_status("file_id_1", "20240102", FileStatus.FAILED,
                                       FileDetailedStatus.FILE_ERROR_TABLES, "msg", ["p2"], get_now_UTC())

    result = get_registered_info(fspath.join(metadata_path, WORKFLOW_FILE_STATUS), ["file_id_1"], "20240102")
    assert "20240102" in result["file_id_1"]
    assert result["file_id_1"]["20240102"]["global_status"] == "FAILED"


def test_workflow_metadata_get_registered_info_empty_filters(loader):
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    workflow_metadata.write_file_status("file_id_x", "20240101", FileStatus.SUCCESS,
                                       FileDetailedStatus.FILE_SUCCESS, "msg", ["p"], get_now_UTC())

    result = get_registered_info(fspath.join(metadata_path, WORKFLOW_FILE_STATUS), [])
    assert result == {}


def test_workflow_metadata_write_file_status_with_end_ts_none(loader):
    """Test write_file_status with end_ts=None (defaults to now)."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    workflow_metadata.write_file_status("file_id_y", "20240102", FileStatus.SUCCESS,
                                       FileDetailedStatus.FILE_SUCCESS, "msg", ["p"], get_now_UTC(), end_ts=None)
    
    result = get_registered_info(fspath.join(metadata_path, WORKFLOW_FILE_STATUS), ["file_id_y"])
    assert "20240102" in result["file_id_y"]
    assert result["file_id_y"]["20240102"]["global_status"] == "SUCCESS"


def test_workflow_metadata_write_table_status_with_end_ts_none(loader):
    """Test write_table_status with end_ts=None (defaults to now)."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    workflow_metadata.write_table_status("file_id_z", "20240103", "table_z", TableStatus.SUCCESS,
                                       TableDetailedStatus.TABLE_SUCCESS, 100, 0, "OK", "path", get_now_UTC(), end_ts=None)
    
    df = pl.scan_delta(fspath.join(metadata_path, WORKFLOW_TABLE_STATUS))
    results = df.filter(pl.col("table") == "table_z").collect().to_dicts()
    assert len(results) == 1
    assert results[0]["global_status"] == "SUCCESS"


def test_workflow_metadata_create_additional_columns_returns_valid_structure(loader):
    """Test that create_additional_columns returns columns and metadata expression."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    cols, metadata_expr = workflow_metadata.create_additional_columns("20240104", "test_file.txt")
    assert len(cols) == 2
    assert cols[0].name == "__functional_date__"
    assert cols[1].name == "__version__"
    assert metadata_expr is not None


def test_workflow_metadata_write_table_status_with_explicit_end_ts(loader):
    """Test write_table_status with an explicit end_ts: covers the False branch of 'if end_ts is None'."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    explicit_end_ts = get_now_UTC()
    workflow_metadata.write_table_status(
        "file_explicit_ts", "20240201", "tbl_explicit",
        TableStatus.SUCCESS, TableDetailedStatus.TABLE_SUCCESS,
        50, 0, "ok", "/path", get_now_UTC(), end_ts=explicit_end_ts,
    )

    df = pl.scan_delta(fspath.join(metadata_path, WORKFLOW_TABLE_STATUS))
    rows = df.filter(pl.col("table") == "tbl_explicit").collect().to_dicts()
    assert len(rows) == 1
    assert rows[0]["global_status"] == "SUCCESS"


def test_workflow_metadata_write_file_status_with_explicit_end_ts(loader):
    """Test write_file_status with an explicit end_ts: covers the False branch of 'if end_ts is None'."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    explicit_end_ts = get_now_UTC()
    workflow_metadata.write_file_status(
        "file_explicit_end", "20240202",
        FileStatus.SUCCESS, FileDetailedStatus.FILE_SUCCESS,
        "msg", ["/p"], get_now_UTC(), end_ts=explicit_end_ts,
    )

    result = get_registered_info(fspath.join(metadata_path, WORKFLOW_FILE_STATUS), ["file_explicit_end"])
    assert "20240202" in result["file_explicit_end"]
    assert result["file_explicit_end"]["20240202"]["global_status"] == "SUCCESS"


def test_workflow_metadata_write_event_with_explicit_timestamps(loader):
    """Test write_event with explicit event_ts and run_end_ts: covers both False branches."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    explicit_ts = get_now_UTC()
    workflow_metadata.write_event(
        "EXPLICIT_EVENT", None,
        event_info_json='{"key": "value"}',
        run_end_ts=explicit_ts,
        event_ts=explicit_ts,
    )

    df = pl.scan_delta(fspath.join(metadata_path, WORKFLOW_EVENTS))
    rows = df.filter(pl.col("event_type") == "EXPLICIT_EVENT").collect().to_dicts()
    assert len(rows) == 1


def test_is_effectively_finished_with_present_non_ignored_status():
    """Covers the False branch of 'if not date_info' in _is_effectively_finished (returns True)."""
    registered_info = {"20240101": {"detailed_status": FileDetailedStatus.FILE_SUCCESS.value}}
    assert _is_effectively_finished(registered_info, "20240101") is True


def test_is_effectively_finished_with_present_ignored_status():
    """Covers the False branch of 'if not date_info' where status is ignored (returns False)."""
    registered_info = {"20240101": {"detailed_status": FileDetailedStatus.FILE_WAITING.value}}
    assert _is_effectively_finished(registered_info, "20240101") is False


def test_process_missing_dates_when_overdue(loader):
    """Covers the False branch of 'if is_today_not_overdue_yet' (time is past overdue_time)."""
    metadata_path = tempfile.mkdtemp(prefix="test_metadata_")
    config = ConfigTest(metadata_dir=metadata_path, workflow_name="TEST-WORKFLOW")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config, write_start_event=False)

    file_config = FileConfig(
        file_identifier="overdue_file",
        overdue_time="00:01",  # very early overdue_time
        mode="daily",
        min_expected_date="20240918",
        period_checked=7,
    )

    # 23:59 > 00:01 -> is_today_not_overdue_yet=False -> jump directly to for loop
    current_time = _datetime(2024, 9, 24, 23, 59, 0)
    _process_missing_dates_for_file(
        file_identifier="overdue_file",
        file_config=file_config,
        registered_info={},
        workflow_metadata=workflow_metadata,
        current_time=current_time,
        date_format="%Y%m%d",
        time_format="%H:%M",
    )

    result = get_registered_info(fspath.join(metadata_path, WORKFLOW_FILE_STATUS), ["overdue_file"])
    assert "20240918" in result["overdue_file"]
    assert result["overdue_file"]["20240918"]["global_status"] == FileStatus.FAILED.value


def test_workflowconfig_table_config_none_date_field():
    """Covers the True branch of 'if value is None' in validate_date_format."""
    table_cfg = TableConfig()  # raise_error_if_empty_after defaults to None -> validator returns None
    assert table_cfg.raise_error_if_empty_after is None


def test_workflowconfig_pattern_validator_with_string_input():
    """Covers the True branch of 'if isinstance(value, str)' in validate_pattern."""
    file_cfg = _build_workflowconfig_file_config()
    workflow_cfg = WorkflowBaseConfig(
        workflow_name="WF-STR",
        metadata_dir="/tmp/metadata",
        file_pattern="^(?P<file_identifier>[a-z]+)-(?P<date>[0-9]{8})\\.txt$",  # string, not re.Pattern
        files_configs={"file_cfg": file_cfg},
        archive_dir="/tmp/archive",
        input_dir="/tmp/input",
        tables_to_format={},
    )
    assert isinstance(workflow_cfg.file_pattern, re.Pattern)

```

###### FILE: test/metadata/workflow_dependency/scenarios/end_to_end_scenarios.md ######

```md
# End-to-End Scenarios for get_versions_auto

This document describes the expected behavior of the `get_versions_auto` function in end-to-end scenarios. These scenarios are documented here for reference and can be used for manual testing or future integration test implementation.

## Test Database State

For these scenarios, assume the following database state:

### datavault.main.workflow
- **last_successful**: "20240102" (Main workflow processed up to day 2)
- **versions**:
  - {"version": "20240101", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}
  - {"version": "20240102", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}

### parsing.dependency1
- **versions**:
  - {"version": "20240101", "domain_status": "SUCCESS"}
  - {"version": "20240102", "domain_status": "SUCCESS"}
  - {"version": "20240103", "domain_status": "SUCCESS"}     # Success
  - {"version": "20240104", "domain_status": "SUCCESS"}     # Success
  - {"version": "20240105", "domain_status": "FAILED"}      # Failed - stops ENSURE_ORDER
  - {"version": "20240106", "domain_status": "SUCCESS"}     # Success after failure

### datavault.dependency2
- **versions**:
  - {"version": "20240101", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}
  - {"version": "20240102", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}
  - {"version": "20240103", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}
  - {"version": "20240104", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}
  - {"version": "20240105", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}  # Success - different from dep 1
  - {"version": "20240106", "global_status": "SUCCESS", "detailed_status": "SUCCESS"}

## Scenario 1: ENSURE_ORDER strategy without min_version

**Setup:**
- Workflow: "datavault.main.workflow" with dependencies ["parsing.dependency1", "datavault.dependency2"]
- Strategy: VersionStrategy.ENSURE_ORDER
- min_version: None

**Expected Behavior:**
- Effective min_version = 20240102 (from main workflow last successful)
- Dep1: ENSURE_ORDER stops at 20240105 failure → [20240103, 20240104] after 20240102
- Dep2: All successful → [20240103, 20240104, 20240105, 20240106] after 20240102
- **Result: [20240103, 20240104]** (intersection)

## Scenario 2: ENSURE_ORDER strategy with min_version

**Setup:**
- Workflow: "datavault.main.workflow" with dependencies ["parsing.dependency1", "datavault.dependency2"]
- Strategy: VersionStrategy.ENSURE_ORDER
- min_version: "20240103"

**Expected Behavior:**
- Effective min_version = max(20240102, 20240103) = 20240103
- Dep1: ENSURE_ORDER stops at 20240105 failure → [20240104] after 20240103
- Dep2: All successful → [20240104, 20240105, 20240106] after 20240103
- **Result: [20240104]** (intersection)

## Scenario 3: ALL_AVAILABLE strategy without min_version

**Setup:**
- Workflow: "datavault.main.workflow" with dependencies ["parsing.dependency1"]
- Strategy: VersionStrategy.ALL_AVAILABLE
- min_version: None

**Expected Behavior:**
- Already processed (successful): [20240101, 20240102]
- Dependency available (successful): [20240101, 20240102, 20240103, 20240104, 20240106]
- Valid dependency versions (intersection): [20240101, 20240102, 20240103, 20240104, 20240106]
- **Result = valid - already_processed = [20240103, 20240104, 20240106]**

## Scenario 4: ALL_AVAILABLE strategy with min_version

**Setup:**
- Workflow: "datavault.main.workflow" with dependencies ["parsing.dependency1"]
- Strategy: VersionStrategy.ALL_AVAILABLE
- min_version: "20240103"

**Expected Behavior:**
- Already processed (successful): [20240101, 20240102] → filter >20240103 → []
- Dependency available (successful): [20240101, 20240102, 20240103, 20240104, 20240106] → filter >20240103 → [20240104, 20240106]
- Valid dependency versions (intersection): [20240104, 20240106]
- **Result = valid - already_processed = [20240104, 20240106]**

## Usage

These scenarios can be used for:
1. **Manual Testing**: Verify behavior against a test database with this data
2. **Integration Tests**: Use as test cases for future integration test implementation
3. **Documentation**: Understanding the expected behavior of different strategy/parameter combinations
4. **Regression Testing**: Ensure changes don't break expected behavior

```

###### FILE: test/metadata/workflow_dependency/version_provider/conftest.py ######

```py
"""Shared fixtures for workflow_dependency tests."""
import pytest
from typing import Optional, List, Dict, Any
from unittest.mock import MagicMock

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import WorkflowVersionProvider
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import WorkflowEntry, LayerType


class MockRowMapping(dict):
    """Lightweight mock of SQLAlchemy RowMapping with attr access and index access.

    This class extends dict to provide both attribute-style and integer index access,
    mimicking the behavior of SQLAlchemy's RowMapping objects commonly returned by
    database queries.

    Supported access patterns:
    - Dictionary access: row['column_name']
    - Attribute access: row.column_name  
    - Integer indexing: row[0] (gets value at first key position)
    """
    def __getattr__(self, key):
        """Enable attribute-style access (row.column_name) to mimic SQLAlchemy RowMapping behavior."""
        try:
            return self[key]
        except KeyError as e:
            raise AttributeError(key) from e
    
    def __getitem__(self, key):
        """Support both dictionary access and integer indexing to match SQLAlchemy RowMapping interface.
        When key is an integer, it converts the index to a key name. 
        When key is not an integer, it falls back to normal dictionary access.
        """
        if isinstance(key, int):
            keys = list(self.keys())
            if 0 <= key < len(keys):
                return super().__getitem__(keys[key])
            else:
                raise IndexError(f"Row index {key} out of range")
        else:
            # Normal dictionary access
            return super().__getitem__(key)


class DummyProvider(WorkflowVersionProvider):
    """Dummy provider for testing base WorkflowVersionProvider functionality.
    
    Example usage:
        from unittest.mock import MagicMock
        
        mock_engine = MagicMock()
        custom_versions = [
            {"version": "20230101", "global_status": "success"},
            {"version": "20230102", "global_status": "failed"},
            {"version": "20230103", "global_status": "success"},
        ]
        provider = DummyProvider("test.workflow", mock_engine, custom_versions=custom_versions)
    """
    
    def __init__(self, workflow_name: str, engine, custom_versions: Optional[List[Dict[str, Any]]] = None, custom_dependencies: Optional[Any] = None):
        super().__init__(workflow_name, engine)
        self.custom_versions = custom_versions or []
        self.custom_dependencies = custom_dependencies

    def get_last_successful_version(self) -> Optional[str]:
        successful_versions = [
            v["version"] for v in self.custom_versions 
            if v.get("global_status") == "success"
        ]
        return max(successful_versions) if successful_versions else None

    def _collect_all_versions(self, min_version=None) -> List[MockRowMapping]:
        if min_version is None:
            filtered_versions = self.custom_versions
        else:
            filtered_versions = [v for v in self.custom_versions if v["version"] > min_version]
        
        # Convert dict to MockRowMapping so values can be accessed by key or attribute
        return [MockRowMapping(version_data) for version_data in filtered_versions]

    def _is_version_successful(self, version_data) -> bool:
        return version_data["global_status"] == "success"
    
    def _is_empty(self, detailed_status_list: list[str]) -> bool:
        """Check if version is empty. Default implementation for dummy provider."""
        return False


@pytest.fixture
def mock_engine():
    """Fixture for a mock database engine."""
    return MagicMock()


@pytest.fixture(scope="module")
def mock_workflow_no_deps():
    """Mock parsing workflow entry without dependencies."""
    return WorkflowEntry(
        name="parsing.test.workflow",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=[]
    )


@pytest.fixture(scope="module")
def mock_workflow_empty_deps():
    """Mock parsing workflow entry with empty dependencies list."""
    return WorkflowEntry(
        name="parsing.test.workflow",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=[]
    )


@pytest.fixture(scope="module")
def mock_workflow_single_dep():
    """Mock datavault workflow entry with single parsing dependency."""
    return WorkflowEntry(
        name="datavault.test.workflow",
        layer=LayerType.DATAVAULT,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=["parsing.dependency1"]
    )


@pytest.fixture(scope="module")
def mock_workflow_two_deps():
    """Mock extraction workflow entry with two parsing dependencies."""
    return WorkflowEntry(
        name="extraction.test.workflow",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=["parsing.dependency1", "datavault.dependency2"]
    )


@pytest.fixture(scope="module")
def mock_workflow_three_deps():
    """Mock extraction workflow entry with mixed dependencies."""
    return WorkflowEntry(
        name="extraction.test.workflow",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=["parsing.dependency1", "datavault.dependency2", "extraction.dependency3"]
    )


@pytest.fixture(scope="module")
def mock_workflow_with_optional_deps():
    """Mock extraction workflow with 1 required and 2 optional dependencies."""
    return WorkflowEntry(
        name="extraction.test",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=[
            "datavault.required",
            {"datavault.optional1": {"optional": True}},
            {"datavault.optional2": {"optional": True}}
        ]
    )


@pytest.fixture(scope="module")
def mock_workflow_all_optional_deps():
    """Mock extraction workflow with all optional dependencies."""
    return WorkflowEntry(
        name="extraction.test",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
        depends_on=[
            {"datavault.optional1": {"optional": True}},
            {"datavault.optional2": {"optional": True}},
            {"datavault.optional3": {"optional": True}}
        ]
    )


```

###### FILE: test/metadata/workflow_dependency/version_provider/get_versions_auto/test_get_versions_auto.py ######

```py
"""Tests for the get_versions_auto function."""
import pytest
from unittest.mock import patch, MagicMock

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import get_versions_auto, VersionStrategy
from azfr_skywalker_utils.utils.helpers import max_with_none

class TestGetVersionsAutoEnsureOrder:
    """Test get_versions_auto function with ENSURE_ORDER strategy."""

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_ensure_order_strategy_basic_flow(self, mock_create_provider, mock_engine):
        """Test ENSURE_ORDER strategy with basic successful flow."""
        # Mock provider
        mock_provider = MagicMock()
        mock_provider.get_last_successful_version.return_value = "20230101"
        mock_provider.get_valid_dependencies_versions.return_value = [
            {'version': "20230102", 'dependencies_status': 'SUCCESS'},
            {'version': "20230103", 'dependencies_status': 'SUCCESS'},
            {'version': "20230104", 'dependencies_status': 'SUCCESS'}
        ]
        mock_create_provider.return_value = mock_provider
        
        _ = get_versions_auto(
            workflow_name="datavault.test.workflow",
            engine=mock_engine,
            strategy=VersionStrategy.ENSURE_ORDER,
            min_version="20230101"
        )
        
        mock_create_provider.assert_called_once_with("datavault.test.workflow", mock_engine)
        mock_provider.get_last_successful_version.assert_called_once()
        # Should use max of workflow last successful (20230101) and min_version (20230101)
        mock_provider.get_valid_dependencies_versions.assert_called_once_with("20230101", VersionStrategy.ENSURE_ORDER)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.max_with_none")
    def test_ensure_order_strategy_max_with_none_usage(self, mock_max_with_none, mock_create_provider, mock_engine):
        """Test ENSURE_ORDER strategy uses max_with_none correctly."""
        # Mock provider
        mock_provider = MagicMock()
        mock_provider.get_last_successful_version.return_value = "20230105"
        mock_provider.get_valid_dependencies_versions.return_value = [
            {'version': "20230106", 'dependencies_status': 'SUCCESS'},
            {'version': "20230107", 'dependencies_status': 'SUCCESS'}
        ]
        mock_create_provider.return_value = mock_provider
        mock_max_with_none.return_value = "20230105"
        
        _ = get_versions_auto(
            workflow_name="datavault.test.workflow",
            engine=mock_engine,
            strategy=VersionStrategy.ENSURE_ORDER,
            min_version="20230101"
        )
        
        mock_max_with_none.assert_called_once_with("20230105", "20230101")
        mock_provider.get_valid_dependencies_versions.assert_called_once_with("20230105", VersionStrategy.ENSURE_ORDER)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_ensure_order_strategy_no_last_successful_version(self, mock_create_provider, mock_engine):
        """Test ENSURE_ORDER strategy when workflow has no last successful version."""
        # Mock provider
        mock_provider = MagicMock()
        mock_provider.get_last_successful_version.return_value = None
        mock_provider.get_valid_dependencies_versions.return_value = [
            {'version': "20230102", 'dependencies_status': 'SUCCESS'},
            {'version': "20230103", 'dependencies_status': 'SUCCESS'}
        ]
        mock_create_provider.return_value = mock_provider
        
        _ = get_versions_auto(
            workflow_name="datavault.test.workflow",
            engine=mock_engine,
            strategy=VersionStrategy.ENSURE_ORDER,
            min_version="20230101"
        )
        
        # Should use min_version when workflow last successful is None
        mock_provider.get_valid_dependencies_versions.assert_called_once_with("20230101", VersionStrategy.ENSURE_ORDER)


class TestGetVersionsAutoAllAvailable:
    """Test get_versions_auto function with ALL_AVAILABLE strategy."""
    
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_all_available_strategy_method_calls(self, mock_create_provider, mock_engine):
        """Test ALL_AVAILABLE strategy calls the correct methods with correct parameters."""
        # Mock provider
        mock_provider = MagicMock()
        mock_provider.get_all_successful_versions.return_value = [
            {'version': "20230101", 'dependencies_status': 'SUCCESS'},
            {'version': "20230102", 'dependencies_status': 'SUCCESS'}
        ]
        mock_provider.get_valid_dependencies_versions.return_value = [
            {'version': "20230101", 'dependencies_status': 'SUCCESS'},
            {'version': "20230102", 'dependencies_status': 'SUCCESS'},
            {'version': "20230103", 'dependencies_status': 'SUCCESS'},
            {'version': "20230104", 'dependencies_status': 'SUCCESS'}
        ]
        mock_create_provider.return_value = mock_provider
        
        _ = get_versions_auto(
            workflow_name="datavault.test.workflow",
            engine=mock_engine,
            strategy=VersionStrategy.ALL_AVAILABLE,
            min_version=None
        )
        
        # Verify correct method calls with correct parameters
        mock_provider.get_all_successful_versions.assert_called_once_with(None, VersionStrategy.ALL_AVAILABLE)
        mock_provider.get_valid_dependencies_versions.assert_called_once_with(None, VersionStrategy.ALL_AVAILABLE)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    @pytest.mark.parametrize("scenario", [
        {
            "name": "all_valid_versions_already_processed",
            "already_processed": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'},
                {'version': "20230103", 'dependencies_status': 'SUCCESS'}
            ],
            "valid_versions": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'},
                {'version': "20230103", 'dependencies_status': 'SUCCESS'}
            ],
            "expected": []
        },
        {
            "name": "no_versions_already_processed",
            "already_processed": [],
            "valid_versions": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'},
                {'version': "20230103", 'dependencies_status': 'SUCCESS'}
            ],
            "expected": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'},
                {'version': "20230103", 'dependencies_status': 'SUCCESS'}
            ]
        },
        {
            "name": "partial_overlap",
            "already_processed": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'}
            ],
            "valid_versions": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'},
                {'version': "20230103", 'dependencies_status': 'SUCCESS'},
                {'version': "20230104", 'dependencies_status': 'SUCCESS'}
            ],
            "expected": [
                {'version': "20230103", 'dependencies_status': 'SUCCESS'},
                {'version': "20230104", 'dependencies_status': 'SUCCESS'}
            ]
        },
        {
            "name": "no_overlap",
            "already_processed": [
                {'version': "20230101", 'dependencies_status': 'SUCCESS'},
                {'version': "20230102", 'dependencies_status': 'SUCCESS'}
            ],
            "valid_versions": [
                {'version': "20230103", 'dependencies_status': 'SUCCESS'},
                {'version': "20230104", 'dependencies_status': 'SUCCESS'},
                {'version': "20230105", 'dependencies_status': 'SUCCESS'}
            ],
            "expected": [
                {'version': "20230103", 'dependencies_status': 'SUCCESS'},
                {'version': "20230104", 'dependencies_status': 'SUCCESS'},
                {'version': "20230105", 'dependencies_status': 'SUCCESS'}
            ]
        }
    ], ids=lambda scenario: scenario["name"])
    def test_all_available_strategy_version_difference_scenarios(self, mock_create_provider, mock_engine, scenario):
        """Test ALL_AVAILABLE strategy version difference logic with various scenarios."""
        # Mock provider
        mock_provider = MagicMock()
        mock_provider.get_all_successful_versions.return_value = scenario["already_processed"]
        mock_provider.get_valid_dependencies_versions.return_value = scenario["valid_versions"]
        mock_create_provider.return_value = mock_provider
        
        result = get_versions_auto(
            workflow_name="datavault.test.workflow",
            engine=mock_engine,
            strategy=VersionStrategy.ALL_AVAILABLE,
            min_version="20230101"
        )
        
        assert result == scenario["expected"], f"Failed for scenario: {scenario['name']}"


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
def test_get_versions_auto_unsupported_strategy_raises(mock_create_provider, mock_engine):
    mock_create_provider.return_value = MagicMock()
    with pytest.raises(ValueError, match="Unsupported version strategy"):
        get_versions_auto("datavault.test.workflow", mock_engine, "unsupported")


@pytest.mark.parametrize(
    "a,b,expected",
    [
        (None, None, None),
        (None, "20230101", "20230101"),
        ("20230102", None, "20230102"),
        ("20230101", "20230102", "20230102"),
    ],
)
def test_max_with_none_cases(a, b, expected):
    assert max_with_none(a, b) == expected

```

###### FILE: test/metadata/workflow_dependency/version_provider/providers/test_create_provider.py ######

```py
import pytest
from unittest.mock import MagicMock, patch
from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import create_provider, ParsingVersionProvider, DatavaultVersionProvider, ExtractionVersionProvider
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import WorkflowEntry, LayerType


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_create_provider_parsing(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = create_provider("parsing.test", engine=MagicMock())
    assert isinstance(provider, ParsingVersionProvider)


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_create_provider_datavault(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="datavault.test",
        layer=LayerType.DATAVAULT,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = create_provider("datavault.test", engine=MagicMock())
    assert isinstance(provider, DatavaultVersionProvider)

@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_create_provider_extraction(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="extraction.test",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = create_provider("extraction.test", engine=MagicMock())
    assert isinstance(provider, ExtractionVersionProvider)

@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_create_provider_invalid(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="invalid.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    with pytest.raises(ValueError):
        create_provider("invalid.test", engine=MagicMock())


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider')
@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_version_provider_filter_successful_versions_empty_list(mock_registry, mock_create):
    """Test _filter_successful_versions with empty versions list."""
    mock_workflow_entry = WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = ParsingVersionProvider("parsing.test", engine=MagicMock())
    result = provider._filter_successful_versions([], None)
    assert result == []


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_version_provider_build_version_condition_with_min_version(mock_registry):
    """Test _build_version_condition with min_version."""
    mock_workflow_entry = WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = ParsingVersionProvider("parsing.test", engine=MagicMock())
    condition = provider._build_version_condition("20240101")
    assert condition == "version > '20240101'"


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_version_provider_build_version_condition_none(mock_registry):
    """Test _build_version_condition with None min_version."""
    mock_workflow_entry = WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = ParsingVersionProvider("parsing.test", engine=MagicMock())
    from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import ALL_VERSIONS_CONDITION
    condition = provider._build_version_condition(None)
    assert condition == ALL_VERSIONS_CONDITION


@patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY')
def test_version_provider_get_all_successful_versions_empty(mock_registry):
    """Test get_all_successful_versions with empty versions list."""
    mock_workflow_entry = WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    
    provider = ParsingVersionProvider("parsing.test", engine=MagicMock())
    provider._collect_all_versions = MagicMock(return_value=[])
    from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
    result = provider.get_all_successful_versions(None, VersionStrategy.ALL_AVAILABLE)
    assert result == []



```

###### FILE: test/metadata/workflow_dependency/version_provider/providers/test_datavault_version_provider.py ######

```py
from typing import cast
from unittest.mock import MagicMock, patch
from sqlalchemy.engine import RowMapping

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import DatavaultVersionProvider
from azfr_skywalker_utils.metadata.workflow_run.abstract import WorkflowGlobalStatus
from azfr_skywalker_utils.metadata.workflow_run.datavault import DatasetDetailedStatus
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import LayerType, WorkflowEntry


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
def test_is_version_successful(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="datavault.test",
        layer=LayerType.DATAVAULT,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    engine = MagicMock()
    provider = DatavaultVersionProvider("datavault.test", engine)

    successful_version = cast(RowMapping, {
        "version": "20240701",
        "global_status": WorkflowGlobalStatus.SUCCESS.value,
        "detailed_status": DatasetDetailedStatus.SUCCESS.value
    })
    assert provider._is_version_successful(successful_version)

    disabled_version = cast(RowMapping, {
        "version": "20240702",
        "global_status": WorkflowGlobalStatus.SUCCESS.value,
        "detailed_status": DatasetDetailedStatus.INSERT_OR_POST_TEST_DISABLED.value
    })
    assert not provider._is_version_successful(disabled_version)

    failed_version = cast(RowMapping, {
        "version": "20240703",
        "global_status": WorkflowGlobalStatus.FAILED.value,
        "detailed_status": DatasetDetailedStatus.POST_TEST_FAILED.value
    })
    assert not provider._is_version_successful(failed_version)


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
def test_get_last_successful_version_paths(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="datavault.test",
        layer=LayerType.DATAVAULT,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry

    class DummyConn:
        def __init__(self, row):
            self.row = row

        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def execute(self, _query):
            return self

        def fetchone(self):
            return self.row

    class DummyEngine:
        def __init__(self, row):
            self.row = row

        def connect(self):
            return DummyConn(self.row)

    provider_no_result = DatavaultVersionProvider("datavault.test", DummyEngine(None))
    assert provider_no_result.get_last_successful_version() is None

    provider_with_result = DatavaultVersionProvider("datavault.test", DummyEngine(("20240101",)))
    assert provider_with_result.get_last_successful_version() == "20240101"


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
def test_collect_all_versions_and_is_empty(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="datavault.test",
        layer=LayerType.DATAVAULT,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry

    class DummyResult:
        def __init__(self, rows):
            self._rows = rows

        def mappings(self):
            return self

        def fetchall(self):
            return self._rows

    class DummyConn:
        def __init__(self, rows):
            self._rows = rows

        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def execute(self, _query):
            return DummyResult(self._rows)

    class DummyEngine:
        def __init__(self, rows):
            self._rows = rows

        def connect(self):
            return DummyConn(self._rows)

    rows = [
        {"version": "20240101", "global_status": "SUCCESS", "detailed_status": "SUCCESS"},
        {"version": "20240102", "global_status": "FAILED", "detailed_status": "POST_TEST_FAILED"},
    ]
    provider = DatavaultVersionProvider("datavault.test", DummyEngine(rows))
    assert provider._collect_all_versions() == rows
    assert provider._is_empty("anything") is False
```

###### FILE: test/metadata/workflow_dependency/version_provider/providers/test_extraction_version_provider.py ######

```py
from typing import cast
from unittest.mock import MagicMock, patch
from sqlalchemy.engine import RowMapping

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import ExtractionVersionProvider
from azfr_skywalker_utils.metadata.workflow_run.abstract import WorkflowGlobalStatus
from azfr_skywalker_utils.metadata.workflow_run.extraction import ExtractionDetailedStatus
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import LayerType, WorkflowEntry


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
def test_is_version_successful(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="extraction.test",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry
    engine = MagicMock()
    provider = ExtractionVersionProvider("extraction.test", engine)

    successful_version = cast(RowMapping, {
        "version": "20240701",
        "global_status": WorkflowGlobalStatus.SUCCESS.value,
        "detailed_status": ExtractionDetailedStatus.SUCCESS.value
    })
    assert provider._is_version_successful(successful_version)

    failed_version = cast(RowMapping, {
        "version": "20240703",
        "global_status": WorkflowGlobalStatus.FAILED.value,
    })
    assert not provider._is_version_successful(failed_version)


@patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
def test_extraction_collect_versions_and_empty_logic(mock_registry):
    mock_workflow_entry = WorkflowEntry(
        name="extraction.test",
        layer=LayerType.EXTRACTION,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema"
    )
    mock_registry.get_workflow_config.return_value = mock_workflow_entry

    class DummyResult:
        def __init__(self, rows):
            self._rows = rows

        def mappings(self):
            return self

        def fetchall(self):
            return self._rows

    class DummyConn:
        def __init__(self, rows):
            self._rows = rows

        def __enter__(self):
            return self

        def __exit__(self, exc_type, exc, tb):
            return False

        def execute(self, _query):
            return DummyResult(self._rows)

    class DummyEngine:
        def __init__(self, rows):
            self._rows = rows

        def connect(self):
            return DummyConn(self._rows)

    rows = [
        {
            "version": "20240101",
            "global_status": WorkflowGlobalStatus.SUCCESS.value,
            "detailed_status": ExtractionDetailedStatus.NO_CHANGES.value,
        },
        {
            "version": "20240102",
            "global_status": WorkflowGlobalStatus.SUCCESS.value,
            "detailed_status": ExtractionDetailedStatus.SUCCESS.value,
        },
    ]

    provider = ExtractionVersionProvider("extraction.test", DummyEngine(rows))
    statuses = provider._collect_all_versions()
    assert statuses[0]["detailed_status"] == "EMPTY"
    assert statuses[1]["detailed_status"] == ExtractionDetailedStatus.SUCCESS.value

    assert provider._is_empty(ExtractionDetailedStatus.DEPENDENCIES_EMPTY.value) is True
    assert provider._is_empty([ExtractionDetailedStatus.NO_CHANGES.value]) is True
    assert provider._is_empty([ExtractionDetailedStatus.NO_CHANGES.value, ExtractionDetailedStatus.SUCCESS.value]) is False
```

###### FILE: test/metadata/workflow_dependency/version_provider/providers/test_parsing_version_provider.py ######

```py
import pytest
from typing import cast
from unittest.mock import MagicMock, patch
from sqlalchemy import RowMapping

from azfr_skywalker_utils.metadata.parsing.metadata import FileStatus, FileDetailedStatus, ParsingDomainStatus, ParsingDetailedStatus
from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import ParsingVersionProvider
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import LayerType, WorkflowEntry


def make_file_status(version, status, file_id, detailed_status=None) -> MagicMock:
    """Helper function to create a mock file status object."""
    mock = MagicMock()
    mock.version = version
    mock.global_status = status
    mock.file_identifier = file_id
    mock.detailed_status = detailed_status if detailed_status is not None else FileDetailedStatus.FILE_SUCCESS.value
    return mock


@pytest.fixture(scope="module")
def mock_parsing_workflow_entry() -> WorkflowEntry:
    """Fixture to create a mock parsing workflow entry."""
    return WorkflowEntry(
        name="parsing.test",
        layer=LayerType.PARSING,
        metadata_catalog="test_catalog",
        metadata_schema="test_schema",
    )


@pytest.fixture
def mock_workflow_registry(mock_parsing_workflow_entry):
    """Fixture that creates a mock registry configured with a parsing workflow."""
    with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
        mock_registry.get_workflow_config.return_value = mock_parsing_workflow_entry
        yield mock_registry


def test_compute_domain_status_5_days_with_file_identifiers(mock_workflow_registry):
    """
    Simulate 5 days of runs (versions), each with 3 files:
    - Day 1 (20240701): All SUCCESS      => domain SUCCESS
    - Day 2 (20240702): All SUCCESS      => domain SUCCESS
    - Day 3 (20240703): 2 SUCCESS, 1 FAILED => domain FAILED
    - Day 4 (20240704): 2 SUCCESS, 1 WAITING => domain WAITING
    - Day 5 (20240705): All SUCCESS      => domain SUCCESS
    """
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = [
        # Day 1 - All SUCCESS
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_A"),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_B"),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_C"),

        # Day 2 - 2 SUCCESS, 1 FAILED
        make_file_status("20240702", FileStatus.SUCCESS.value, "file_A"),
        make_file_status("20240702", FileStatus.SUCCESS.value, "file_B"),
        make_file_status("20240702", FileStatus.FAILED.value, "file_C"),

        # Day 3 - 2 SUCCESS, 1 WAITING
        make_file_status("20240703", FileStatus.SUCCESS.value, "file_A"),
        make_file_status("20240703", FileStatus.WAITING.value, "file_B"),
        make_file_status("20240703", FileStatus.SUCCESS.value, "file_C"),
    ]

    expected = [
        {
            "version": "20240701",
            "domain_status": ParsingDomainStatus.SUCCESS.value,
            "domain_detail_status": ParsingDetailedStatus.SUCCESS.value
        },
        {
            "version": "20240702",
            "domain_status": ParsingDomainStatus.FAILED.value,
            "domain_detail_status": ParsingDetailedStatus.FAILED.value
        },
        {
            "version": "20240703",
            "domain_status": ParsingDomainStatus.WAITING.value,
            "domain_detail_status": ParsingDetailedStatus.WAITING.value
        },
    ]

    result = provider._compute_domain_status(file_status_list)
    assert result == expected


def test_compute_domain_status_with_unknown_status_raises_error(mock_workflow_registry):
    """ Test that _compute_domain_status raises ValueError when encountering unknown file status. """
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = [
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_A"),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_B"),
        make_file_status("20240701", "UNKNOWN", "file_C"),  # Unknown status
    ]

    with pytest.raises(ValueError):
        provider._compute_domain_status(file_status_list)
    

def test_is_version_successful(mock_workflow_registry):
    """Test _is_version_successful logic for parsing workflow."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    successful_version = cast(RowMapping, {"version": "20240701", "domain_status": FileStatus.SUCCESS.value})
    assert provider._is_version_successful(successful_version)

    failed_version = cast(RowMapping, {"version": "20240703", "domain_status": FileStatus.FAILED.value})
    assert not provider._is_version_successful(failed_version)

    waiting_version = cast(RowMapping, {"version": "20240704", "domain_status": FileStatus.WAITING.value})
    assert not provider._is_version_successful(waiting_version)


def test_compute_domain_status_empty_file_status_list(mock_workflow_registry):
    """Test _compute_domain_status with an empty file status list."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = []

    result = provider._compute_domain_status(file_status_list)
    
    assert result == []


def test_compute_domain_status_all_files_empty_detailed_status(mock_workflow_registry):
    """Test _compute_domain_status when all files have SUCCESS with EMPTY detailed status."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = [
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_A", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_B", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_C", FileDetailedStatus.FILE_NO_ROWS.value),
    ]

    expected = [
        {
            "version": "20240701",
            "domain_status": ParsingDomainStatus.SUCCESS.value,
            "domain_detail_status": ParsingDetailedStatus.EMPTY.value
        }
    ]

    result = provider._compute_domain_status(file_status_list)
    assert result == expected


def test_compute_domain_status_mixed_empty_and_success_detailed_status(mock_workflow_registry):
    """Test _compute_domain_status with mixed EMPTY and SUCCESS detailed statuses."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = [
        # Version 1: All SUCCESS global, all empty detailed → EMPTY domain_detail_status
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_A", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_B", FileDetailedStatus.FILE_NO_ROWS.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_C", FileDetailedStatus.FILE_EMPTY.value),

        # Version 2: All SUCCESS global, mixed detailed → SUCCESS domain_detail_status
        make_file_status("20240702", FileStatus.SUCCESS.value, "file_A", FileDetailedStatus.FILE_SUCCESS.value),
        make_file_status("20240702", FileStatus.SUCCESS.value, "file_B", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240702", FileStatus.SUCCESS.value, "file_C", FileDetailedStatus.FILE_NO_ROWS.value),

        # Version 3: All SUCCESS global, all success detailed → SUCCESS domain_detail_status
        make_file_status("20240703", FileStatus.SUCCESS.value, "file_A", FileDetailedStatus.FILE_SUCCESS.value),
        make_file_status("20240703", FileStatus.SUCCESS.value, "file_B", FileDetailedStatus.FILE_SUCCESS.value),
        make_file_status("20240703", FileStatus.SUCCESS.value, "file_C", FileDetailedStatus.FILE_SUCCESS.value),
    ]

    expected = [
        {
            "version": "20240701",
            "domain_status": ParsingDomainStatus.SUCCESS.value,
            "domain_detail_status": ParsingDetailedStatus.EMPTY.value
        },
        {
            "version": "20240702",
            "domain_status": ParsingDomainStatus.SUCCESS.value,
            "domain_detail_status": ParsingDetailedStatus.SUCCESS.value
        },
        {
            "version": "20240703",
            "domain_status": ParsingDomainStatus.SUCCESS.value,
            "domain_detail_status": ParsingDetailedStatus.SUCCESS.value
        },
    ]

    result = provider._compute_domain_status(file_status_list)
    assert result == expected


def test_compute_domain_status_failed_ignores_detailed_status(mock_workflow_registry):
    """Test that FAILED global status ignores detailed status."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    file_status_list = [
        make_file_status("20240701", FileStatus.FAILED.value, "file_A", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_B", FileDetailedStatus.FILE_EMPTY.value),
        make_file_status("20240701", FileStatus.SUCCESS.value, "file_C", FileDetailedStatus.FILE_EMPTY.value),
    ]

    expected = [
        {
            "version": "20240701",
            "domain_status": ParsingDomainStatus.FAILED.value,
            "domain_detail_status": ParsingDetailedStatus.FAILED.value
        }
    ]

    result = provider._compute_domain_status(file_status_list)
    assert result == expected


def test_is_version_successful_with_empty_detailed_status(mock_workflow_registry):
    """Test _is_version_successful considers EMPTY detailed status as successful."""
    engine = MagicMock()
    provider = ParsingVersionProvider("parsing.test", engine)

    # SUCCESS domain_status with EMPTY domain_detail_status should be successful
    empty_version = cast(RowMapping, {
        "version": "20240701", 
        "domain_status": ParsingDomainStatus.SUCCESS.value,
        "domain_detail_status": ParsingDetailedStatus.EMPTY.value
    })
    assert provider._is_version_successful(empty_version)

    # SUCCESS domain_status with SUCCESS domain_detail_status should be successful
    success_version = cast(RowMapping, {
        "version": "20240702",
        "domain_status": ParsingDomainStatus.SUCCESS.value,
        "domain_detail_status": ParsingDetailedStatus.SUCCESS.value
    })
    assert provider._is_version_successful(success_version)

```

###### FILE: test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_collect_dependencies_versions.py ######

```py
"""Tests for the _collect_dependencies_versions method."""
import pytest
from unittest.mock import patch, MagicMock

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
from ..conftest import DummyProvider


class TestCollectDependenciesVersions:
    """Test class for _collect_dependencies_versions method."""

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_collect_dependencies_versions_single_dependency(self, mock_create_provider, mock_registry, mock_workflow_single_dep, mock_engine):
        """Test _collect_dependencies_versions with single dependency."""
        mock_registry.get_workflow_config.return_value = mock_workflow_single_dep
        
        # Mock dependency provider
        mock_dep_provider = MagicMock()
        mock_dep_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'},
            {'version': "20230102", 'detailed_status': 'SUCCESS'}
        ]
        mock_dep_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'},
            {'version': "20230102", 'dependency_status': 'SUCCESS'}
        ]
        mock_create_provider.return_value = mock_dep_provider
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ENSURE_ORDER)
        
        expected = {
            "parsing.dependency1": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        assert result == expected
        mock_create_provider.assert_called_once_with("parsing.dependency1", mock_engine)
        mock_dep_provider._collect_all_versions.assert_called_once_with(None)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_collect_dependencies_versions_two_dependencies(self, mock_create_provider, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test basic functionality of _collect_dependencies_versions."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        # Mock dependency providers
        mock_dep1_provider = MagicMock()
        mock_dep1_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'},
            {'version': "20230102", 'detailed_status': 'SUCCESS'},
            {'version': "20230103", 'detailed_status': 'SUCCESS'}
        ]
        mock_dep1_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'},
            {'version': "20230102", 'dependency_status': 'SUCCESS'},
            {'version': "20230103", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_dep2_provider = MagicMock()
        mock_dep2_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'},
            {'version': "20230103", 'detailed_status': 'SUCCESS'},
            {'version': "20230104", 'detailed_status': 'SUCCESS'}
        ]
        mock_dep2_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'},
            {'version': "20230103", 'dependency_status': 'SUCCESS'},
            {'version': "20230104", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_create_provider.side_effect = [mock_dep1_provider, mock_dep2_provider]
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        expected = {
            "parsing.dependency1": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'}
            ],
            "datavault.dependency2": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'},
                {'version': "20230104", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        assert result == expected
        
        # Verify create_provider was called for each dependency
        assert mock_create_provider.call_count == 2
        mock_create_provider.assert_any_call("parsing.dependency1", mock_engine)
        mock_create_provider.assert_any_call("datavault.dependency2", mock_engine)
        
        # Verify _collect_all_versions was called with correct parameters
        mock_dep1_provider._collect_all_versions.assert_called_once_with(None)
        mock_dep2_provider._collect_all_versions.assert_called_once_with(None)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_collect_dependencies_versions_empty_versions(self, mock_create_provider, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test _collect_dependencies_versions when dependencies return empty versions."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        # Mock dependency providers returning empty lists
        mock_dep1_provider = MagicMock()
        mock_dep1_provider._collect_all_versions.return_value = []
        mock_dep1_provider._filter_successful_versions.return_value = []
        
        mock_dep2_provider = MagicMock()
        mock_dep2_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'}
        ]
        mock_dep2_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_create_provider.side_effect = [mock_dep1_provider, mock_dep2_provider]
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        expected = {
            "parsing.dependency1": [],
            "datavault.dependency2": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        assert result == expected

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_collect_dependencies_versions_no_dependencies(self, mock_registry, mock_workflow_no_deps, mock_engine):
        """Test _collect_dependencies_versions when workflow has no dependencies."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        # Should return empty dict when no dependencies
        assert result == {}

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_collect_dependencies_versions_empty_dependencies(self, mock_registry, mock_workflow_empty_deps, mock_engine):
        """Test _collect_dependencies_versions when workflow has empty dependencies list."""
        mock_registry.get_workflow_config.return_value = mock_workflow_empty_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        # Should return empty dict when dependencies list is empty
        assert result == {}

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_collect_dependencies_versions_dependency_error(self, mock_create_provider, mock_registry, mock_workflow_single_dep, mock_engine):
        """Test _collect_dependencies_versions when dependency provider raises an error."""
        mock_registry.get_workflow_config.return_value = mock_workflow_single_dep
        
        # Mock dependency provider that raises an exception
        mock_dep_provider = MagicMock()
        mock_dep_provider._collect_all_versions.side_effect = Exception("Database connection failed")
        mock_create_provider.return_value = mock_dep_provider
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        
        # Should raise RuntimeError with helpful message
        with pytest.raises(RuntimeError) as exc_info:
            provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        assert "Dependency parsing.dependency1 - Error while trying to get versions" in str(exc_info.value)
        assert "Database connection failed" in str(exc_info.value)

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.create_provider")
    def test_collect_dependencies_versions_extraction_workflow_mixed_deps(self, mock_create_provider, mock_registry, mock_workflow_three_deps, mock_engine):
        """Test _collect_dependencies_versions with extraction workflow having mixed dependency types."""
        mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
        
        # Mock dependency providers for different types
        mock_parsing_provider = MagicMock()
        mock_parsing_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'},
            {'version': "20230102", 'detailed_status': 'SUCCESS'}
        ]
        mock_parsing_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'},
            {'version': "20230102", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_datavault_provider = MagicMock()
        mock_datavault_provider._collect_all_versions.return_value = [
            {'version': "20230101", 'detailed_status': 'SUCCESS'},
            {'version': "20230103", 'detailed_status': 'SUCCESS'}
        ]
        mock_datavault_provider._filter_successful_versions.return_value = [
            {'version': "20230101", 'dependency_status': 'SUCCESS'},
            {'version': "20230103", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_extraction_provider = MagicMock()
        mock_extraction_provider._collect_all_versions.return_value = [
            {'version': "20230102", 'detailed_status': 'SUCCESS'},
            {'version': "20230104", 'detailed_status': 'SUCCESS'}
        ]
        mock_extraction_provider._filter_successful_versions.return_value = [
            {'version': "20230102", 'dependency_status': 'SUCCESS'},
            {'version': "20230104", 'dependency_status': 'SUCCESS'}
        ]
        
        mock_create_provider.side_effect = [mock_parsing_provider, mock_datavault_provider, mock_extraction_provider]
        
        provider = DummyProvider("extraction.test.workflow", mock_engine)
        result = provider._collect_dependencies_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        expected = {
            "parsing.dependency1": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'}
            ],
            "datavault.dependency2": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'}
            ],
            "extraction.dependency3": [
                {'version': "20230102", 'dependency_status': 'SUCCESS'},
                {'version': "20230104", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        assert result == expected
        
        # Verify create_provider was called for each dependency type
        assert mock_create_provider.call_count == 3
        mock_create_provider.assert_any_call("parsing.dependency1", mock_engine)
        mock_create_provider.assert_any_call("datavault.dependency2", mock_engine)
        mock_create_provider.assert_any_call("extraction.dependency3", mock_engine)

```

###### FILE: test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_find_valid_versions_across_dependencies.py ######

```py
"""Tests for the _find_valid_versions_across_dependencies method."""
from unittest.mock import MagicMock, patch

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
from ..conftest import DummyProvider


class TestFindValidVersionsAcrossDependencies:
    """Test class for _find_valid_versions_across_dependencies method."""

    def test_empty_dependencies_returns_empty_list(self, mock_workflow_empty_deps):
        """Test that empty dependencies dictionary returns empty list."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_empty_deps
            mock_engine = MagicMock()
            provider = DummyProvider("parsing.test.workflow", mock_engine)
            
            dependencies_versions = {}
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            assert result == []

    def test_single_dependency_returns_all_versions(self, mock_workflow_single_dep):
        """Test that with a single dependency, all its versions are returned."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_single_dep
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.test.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            assert [v['version'] for v in result] == ["20230101", "20230102", "20230103"]

    def test_multiple_dependencies_intersection_all_available(self, mock_workflow_three_deps):
        """Test intersection of versions across multiple dependencies with ALL_AVAILABLE strategy."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.test.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.dependency2": [
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            # Only version present in all dependencies
            assert [v['version'] for v in result] == ["20230103"]

    def test_multiple_dependencies_intersection_ensure_order(self, mock_workflow_three_deps):
        """Test intersection of versions with ENSURE_ORDER strategy - stops at first missing version."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.test.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230103
                "datavault.dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # Should stop at 20230103 since it's missing in dependency1
            assert [v['version'] for v in result] == ["20230101", "20230102"]

    def test_no_common_versions_all_available(self, mock_workflow_three_deps):
        """Test behavior when no versions are common across all dependencies with ALL_AVAILABLE."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.test.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.dependency2": [
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230105", 'dependency_status': 'SUCCESS'},
                    {'version': "20230106", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            assert result == []

    def test_no_common_versions_ensure_order(self, mock_workflow_three_deps):
        """Test behavior when no versions are common across all dependencies with ENSURE_ORDER."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.test.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.dependency2": [
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230105", 'dependency_status': 'SUCCESS'},
                    {'version': "20230106", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            assert result == []

    def test_partial_overlap_all_available(self, mock_workflow_three_deps):
        """Test with partial overlap between dependencies using ALL_AVAILABLE strategy."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.dependency2": [
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230106", 'dependency_status': 'SUCCESS'},
                    {'version': "20230107", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            # Only 20230103 is present in all dependencies
            assert [v['version'] for v in result] == ["20230103"]

    def test_partial_overlap_ensure_order(self, mock_workflow_three_deps):
        """Test with partial overlap using ENSURE_ORDER strategy."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "parsing.dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230103
                "datavault.dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],
                "extraction.dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # Should include 20230101, 20230102, then stop at 20230103 since it's missing in parsing.dependency1
            assert [v['version'] for v in result] == ["20230101", "20230102"]

    def test_identical_versions_across_dependencies(self, mock_workflow_three_deps):
        """Test when all dependencies have identical versions."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],
                "dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            for strategy in [VersionStrategy.ALL_AVAILABLE, VersionStrategy.ENSURE_ORDER]:
                result = provider._find_valid_versions_across_dependencies(dependencies_versions, strategy)
                assert [v['version'] for v in result] == ["20230101", "20230102", "20230103"]

    def test_single_version_in_all_dependencies(self, mock_workflow_three_deps):
        """Test when all dependencies have only one common version."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [{"version": "20230101", 'dependency_status': 'SUCCESS'}],
                "dependency2": [{"version": "20230101", 'dependency_status': 'SUCCESS'}],
                "dependency3": [{"version": "20230101", 'dependency_status': 'SUCCESS'}]
            }
            
            for strategy in [VersionStrategy.ALL_AVAILABLE, VersionStrategy.ENSURE_ORDER]:
                result = provider._find_valid_versions_across_dependencies(dependencies_versions, strategy)
                assert [v['version'] for v in result] == ["20230101"]

    def test_ensure_order_stops_at_first_gap(self, mock_workflow_three_deps):
        """Test that ENSURE_ORDER strategy stops at the first version gap."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230103
                "dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # Should stop at 20230103 and not include 20230104, 20230105 even though they're available
            assert [v['version'] for v in result] == ["20230101", "20230102"]

    def test_ensure_order_with_multiple_gaps(self, mock_workflow_three_deps):
        """Test ENSURE_ORDER strategy with multiple gaps - should stop at first one."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230102, 20230104
                "dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ],
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230104", 'dependency_status': 'SUCCESS'},
                    {'version': "20230105", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # Should only include 20230101, then stop at 20230102 which is missing in dependency1
            assert [v['version'] for v in result] == ["20230101"]

    def test_versions_are_sorted_in_result(self, mock_workflow_three_deps):
        """Test that returned versions are sorted."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "dependency2": [
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                ],
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            for strategy in [VersionStrategy.ALL_AVAILABLE, VersionStrategy.ENSURE_ORDER]:
                result = provider._find_valid_versions_across_dependencies(dependencies_versions, strategy)
                assert [v['version'] for v in result] == ["20230101", "20230102", "20230103"]

    def test_empty_dependency_list_all_available(self, mock_workflow_three_deps):
        """Test with one dependency having empty version list using ALL_AVAILABLE."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],
                "dependency2": [],  # Empty list
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            # No versions should be valid since dependency2 has no versions
            assert result == []

    def test_empty_dependency_list_ensure_order(self, mock_workflow_three_deps):
        """Test with one dependency having empty version list using ENSURE_ORDER."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],
                "dependency2": [],  # Empty list
                "dependency3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # No versions should be valid since dependency2 has no versions
            assert result == []

    @patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.logger')
    def test_logging_warning_for_missing_versions_all_available(self, mock_logging, mock_workflow_two_deps):
        """Test that warnings are logged for missing versions with ALL_AVAILABLE strategy."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230102
                "dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            # Should log warning for 20230102 missing in dependency1
            mock_logging.warning.assert_called()
            warning_calls = [call.args[0] for call in mock_logging.warning.call_args_list]
            assert any("Version 20230102 will be ignored because required dependency" in call and "dependency1" in call for call in warning_calls)
            assert [v['version'] for v in result] == ["20230101", "20230103"]

    @patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.logger')
    def test_logging_warning_for_missing_versions_ensure_order(self, mock_logging, mock_workflow_two_deps):
        """Test that warnings are logged for missing versions with ENSURE_ORDER strategy."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {
                "dependency1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ],  # Missing 20230102
                "dependency2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'},
                    {'version': "20230103", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ENSURE_ORDER)
            
            # Should log warning and stop at missing version
            mock_logging.warning.assert_called()
            warning_calls = [call.args[0] for call in mock_logging.warning.call_args_list]
            assert any("Version 20230102 will be ignored because required dependency" in call and "dependency1" in call for call in warning_calls)
            assert [v['version'] for v in result] == ["20230101"]

    @patch('azfr_skywalker_utils.metadata.workflow_dependency.version_provider.logger')
    def test_logging_warning_for_no_versions_found(self, mock_logging, mock_workflow_empty_deps):
        """Test that warning is logged when no versions are found across dependencies."""
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_empty_deps
            mock_engine = MagicMock()
            provider = DummyProvider("datavault.workflow", mock_engine)
            
            dependencies_versions = {}
            
            result = provider._find_valid_versions_across_dependencies(dependencies_versions, VersionStrategy.ALL_AVAILABLE)
            
            mock_logging.warning.assert_called_with("No versions found across all dependencies")
            assert result == []

```

###### FILE: test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_get_all_successful_versions.py ######

```py
"""Tests for the get_all_successful_versions method."""
from unittest.mock import patch, MagicMock

from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy, WorkflowVersionProvider


class SimpleTestProvider(WorkflowVersionProvider):
    """Simplified test provider that focuses only on what we need to test."""
    
    def __init__(self, workflow_name: str, engine, test_versions=None):
        super().__init__(workflow_name, engine)
        # Simple list of version dictionaries with success/failure status
        self.test_versions = test_versions or []

    def get_last_successful_version(self):
        # Not needed for testing get_all_successful_versions
        return "20230101"

    def _collect_all_versions(self, min_version=None):
        # Filter versions if min_version is provided
        if min_version is None:
            return self.test_versions
        return [v for v in self.test_versions if v["version"] > min_version]

    def _is_version_successful(self, version_data):
        # Simple check: successful if status is "success"
        return version_data["status"] == "success"
    
    def _is_empty(self, detailed_status_list: list[str]) -> bool:
        """Default implementation - always returns False for test provider."""
        return False


class TestGetAllSuccessfulVersions:
    """Test class for get_all_successful_versions method with both strategies."""

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_ensure_order_strategy_stops_at_first_failure(self, mock_registry, mock_workflow_no_deps):
        """Test ENSURE_ORDER strategy stops processing at first unsuccessful version."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Simple test data: success, success, fail, success, fail
        test_versions = [
            {"version": "20230101", "status": "success"},
            {"version": "20230102", "status": "success"},
            {"version": "20230103", "status": "failed"},
            {"version": "20230104", "status": "success"},
            {"version": "20230105", "status": "failed"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        result = provider.get_all_successful_versions(None, VersionStrategy.ENSURE_ORDER)
        
        # Should return only the first two successful versions before the failure
        assert [v['version'] for v in result] == ["20230101", "20230102"]

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_ensure_order_strategy_all_successful(self, mock_registry, mock_workflow_no_deps):
        """Test ENSURE_ORDER strategy when all versions are successful."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: all successful
        test_versions = [
            {"version": "20230101", "status": "success"},
            {"version": "20230102", "status": "success"},
            {"version": "20230103", "status": "success"},
            {"version": "20230104", "status": "success"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        result = provider.get_all_successful_versions(None, VersionStrategy.ENSURE_ORDER)
        
        # Should return all versions
        assert [v['version'] for v in result] == ["20230101", "20230102", "20230103", "20230104"]

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_all_available_strategy_skips_failures(self, mock_registry, mock_workflow_no_deps):
        """Test ALL_AVAILABLE strategy includes all successful versions regardless of failures."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: success, fail, success, fail, success
        test_versions = [
            {"version": "20230101", "status": "success"},
            {"version": "20230102", "status": "failed"},
            {"version": "20230103", "status": "success"},
            {"version": "20230104", "status": "failed"},
            {"version": "20230105", "status": "success"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        result = provider.get_all_successful_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        # Should return all successful versions, skipping failures
        assert [v['version'] for v in result] == ["20230101", "20230103", "20230105"]

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_with_min_version_filter(self, mock_registry, mock_workflow_no_deps):
        """Test both strategies with min_version filter."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: success, success, fail, success, success
        test_versions = [
            {"version": "20230101", "status": "success"},
            {"version": "20230102", "status": "success"},
            {"version": "20230103", "status": "failed"},
            {"version": "20230104", "status": "success"},
            {"version": "20230105", "status": "success"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        
        # Test ENSURE_ORDER with min_version
        result_ensure = provider.get_all_successful_versions("20230102", VersionStrategy.ENSURE_ORDER)
        # Should return empty list because version 20230103 fails (first version after min_version)
        assert result_ensure == []
        
        # Test ALL_AVAILABLE with min_version
        result_all = provider.get_all_successful_versions("20230102", VersionStrategy.ALL_AVAILABLE)
        # Should return successful versions after min_version (20230104, 20230105)
        assert [v['version'] for v in result_all] == ["20230104", "20230105"]

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_empty_versions_list(self, mock_registry, mock_workflow_no_deps):
        """Test both strategies with empty versions list."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: empty list
        test_versions = []
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        
        # Both strategies should return empty list
        result_ensure = provider.get_all_successful_versions(None, VersionStrategy.ENSURE_ORDER)
        result_all = provider.get_all_successful_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        assert result_ensure == []
        assert result_all == []
        
    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_version_sorting(self, mock_registry, mock_workflow_no_deps):
        """Test that versions are properly sorted regardless of input order."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: unsorted versions, all successful
        test_versions = [
            {"version": "20230103", "status": "success"},
            {"version": "20230101", "status": "success"},
            {"version": "20230105", "status": "success"},
            {"version": "20230102", "status": "success"},
            {"version": "20230104", "status": "success"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        
        # Both strategies should return sorted versions
        result_ensure = provider.get_all_successful_versions(None, VersionStrategy.ENSURE_ORDER)
        result_all = provider.get_all_successful_versions(None, VersionStrategy.ALL_AVAILABLE)
        
        expected = ["20230101", "20230102", "20230103", "20230104", "20230105"]
        assert [v['version'] for v in result_ensure] == expected
        assert [v['version'] for v in result_all] == expected

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_first_version_fails_ensure_order(self, mock_registry, mock_workflow_no_deps):
        """Test ENSURE_ORDER strategy when first version fails."""
        mock_registry.get_workflow_config.return_value = mock_workflow_no_deps
        mock_engine = MagicMock()
        
        # Test data: first version fails
        test_versions = [
            {"version": "20230101", "status": "failed"},
            {"version": "20230102", "status": "success"},
            {"version": "20230103", "status": "success"},
        ]
        
        provider = SimpleTestProvider("test.workflow", mock_engine, test_versions)
        result = provider.get_all_successful_versions(None, VersionStrategy.ENSURE_ORDER)
        
        # Should return empty list since first version fails
        assert result == []

```

###### FILE: test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_optional_dependencies.py ######

```py
"""Tests for optional dependency support in workflow registry."""
from azfr_skywalker_utils.metadata.workflow_dependency.workflow_registry import (
    WorkflowRegistry,
    Dependency,
    WorkflowEntry,
    LayerType
)


class TestOptionalDependencies:
    """Test suite for optional dependency configuration."""

    def test_dependency_with_optional_flag(self):
        """Test that optional flag can be set on dependencies."""
        dep = Dependency(workflow_name="parsing.test", optional=True)
        assert dep.optional is True
        assert dep.workflow_name == "parsing.test"
        assert dep.ignore_empty_version is False

    def test_dependency_optional_defaults_to_false(self):
        """Test that optional flag defaults to False."""
        dep = Dependency(workflow_name="parsing.test")
        assert dep.optional is False

    def test_workflow_entry_with_optional_dependencies(self):
        """Test WorkflowEntry can have both required and optional dependencies."""
        entry = WorkflowEntry(
            name="extraction.test",
            layer=LayerType.EXTRACTION,
            metadata_catalog="test_catalog",
            metadata_schema="test_schema",
            depends_on=[
                "datavault.source1",
                {"datavault.source2": {"optional": True}},
                {"datavault.source3": {"optional": True}}
            ]
        )
        
        assert len(entry.depends_on) == 3
        assert entry.depends_on[0].optional is False
        assert entry.depends_on[1].optional is True
        assert entry.depends_on[2].optional is True

    def test_yaml_format_with_optional_dependency(self):
        """Test parsing YAML format with optional dependencies."""
        config_dict = {
            "workflows": {
                "parsing.source1": {
                    "layer": "parsing",
                    "metadata_catalog": "catalog1",
                    "metadata_schema": "schema1"
                },
                "parsing.source2": {
                    "layer": "parsing",
                    "metadata_catalog": "catalog2",
                    "metadata_schema": "schema2"
                },
                "extraction.test": {
                    "layer": "extraction",
                    "metadata_catalog": "catalog_ext",
                    "metadata_schema": "schema_ext",
                    "depends_on": [
                        "parsing.source1",
                        {"parsing.source2": {"optional": True}}
                    ]
                }
            }
        }
        
        registry = WorkflowRegistry.from_dict(config_dict)
        extraction_config = registry.get_workflow_config("extraction.test")
        
        assert len(extraction_config.depends_on) == 2
        assert extraction_config.depends_on[0].workflow_name == "parsing.source1"
        assert extraction_config.depends_on[0].optional is False
        
        assert extraction_config.depends_on[1].workflow_name == "parsing.source2"
        assert extraction_config.depends_on[1].optional is True

    def test_yaml_format_with_multiple_flags(self):
        """Test parsing YAML with both optional and ignore_empty_version flags."""
        config_dict = {
            "workflows": {
                "parsing.source1": {
                    "layer": "parsing",
                    "metadata_catalog": "catalog1",
                    "metadata_schema": "schema1"
                },
                "extraction.test": {
                    "layer": "extraction",
                    "metadata_catalog": "catalog_ext",
                    "metadata_schema": "schema_ext",
                    "depends_on": [
                        {
                            "parsing.source1": {
                                "optional": True,
                                "ignore_empty_version": True
                            }
                        }
                    ]
                }
            }
        }
        
        registry = WorkflowRegistry.from_dict(config_dict)
        extraction_config = registry.get_workflow_config("extraction.test")
        
        assert len(extraction_config.depends_on) == 1
        assert extraction_config.depends_on[0].workflow_name == "parsing.source1"
        assert extraction_config.depends_on[0].optional is True
        assert extraction_config.depends_on[0].ignore_empty_version is True

    def test_mixed_dependency_formats(self):
        """Test mixing simple strings and dict formats in depends_on."""
        config_dict = {
            "workflows": {
                "parsing.a": {"layer": "parsing", "metadata_catalog": "cat", "metadata_schema": "sch"},
                "parsing.b": {"layer": "parsing", "metadata_catalog": "cat", "metadata_schema": "sch"},
                "parsing.c": {"layer": "parsing", "metadata_catalog": "cat", "metadata_schema": "sch"},
                "extraction.test": {
                    "layer": "extraction",
                    "metadata_catalog": "catalog_ext",
                    "metadata_schema": "schema_ext",
                    "depends_on": [
                        "parsing.a",  # Simple string - required
                        {"parsing.b": {"optional": True}},  # Optional
                        {"parsing.c": {"ignore_empty_version": True}}  # Required but ignore empty
                    ]
                }
            }
        }
        
        registry = WorkflowRegistry.from_dict(config_dict)
        extraction_config = registry.get_workflow_config("extraction.test")
        
        assert extraction_config.depends_on[0].workflow_name == "parsing.a"
        assert extraction_config.depends_on[0].optional is False
        assert extraction_config.depends_on[0].ignore_empty_version is False
        
        assert extraction_config.depends_on[1].workflow_name == "parsing.b"
        assert extraction_config.depends_on[1].optional is True
        assert extraction_config.depends_on[1].ignore_empty_version is False
        
        assert extraction_config.depends_on[2].workflow_name == "parsing.c"
        assert extraction_config.depends_on[2].optional is False
        assert extraction_config.depends_on[2].ignore_empty_version is True


class TestOptionalDependencyLogic:
    """Test suite for optional dependency behavior logic."""

    def test_optional_dependency_scenario_all_updated(self, mock_workflow_with_optional_deps):
        """
        Scenario: All 3 sources (1 required + 2 optional) are updated
        Expected: Processing should proceed with all dependencies
        """
        from unittest.mock import MagicMock, patch
        from ..conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_with_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.required": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # All versions should be valid since all dependencies are present
            assert len(result) == 2
            assert [v['version'] for v in result] == ["20230101", "20230102"]

    def test_optional_dependency_scenario_required_and_partial_optional(self, mock_workflow_with_optional_deps):
        """
        Scenario: Required source updated, 1 optional updated, 1 optional delayed
        Expected: Processing should proceed (optional with delay is acceptable)
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_with_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.required": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional2": [
                    # Missing version 20230102 - delayed
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Both versions should be valid - optional2 delay is acceptable
            assert len(result) == 2
            assert [v['version'] for v in result] == ["20230101", "20230102"]

    def test_optional_dependency_scenario_optional_failed(self, mock_workflow_with_optional_deps):
        """
        Scenario: One optional source has FAILED status
        Expected: Processing should NOT proceed (even optional dependencies cannot have FAILED status)
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_with_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.required": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'FAILED'}  # FAILED
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Only 20230101 should be valid, 20230102 blocked by optional1 FAILED
            assert len(result) == 1
            assert [v['version'] for v in result] == ["20230101"]

    def test_optional_dependency_scenario_required_failed(self, mock_workflow_with_optional_deps):
        """
        Scenario: Required source has FAILED status
        Expected: Processing should NOT proceed
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_with_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.required": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'FAILED'}  # FAILED
                ],
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Only 20230101 should be valid, 20230102 blocked by required FAILED
            assert len(result) == 1
            assert [v['version'] for v in result] == ["20230101"]

    def test_optional_dependency_scenario_required_delayed(self, mock_workflow_with_optional_deps):
        """
        Scenario: Required source is delayed/missing for a version
        Expected: Processing should NOT proceed (required must be present)
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_with_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.required": [
                    # Missing version 20230102 - delayed
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Only 20230101 should be valid, 20230102 blocked by missing required dependency
            assert len(result) == 1
            assert [v['version'] for v in result] == ["20230101"]

    def test_all_dependencies_optional_none_present(self, mock_workflow_all_optional_deps):
        """
        Scenario: All dependencies are optional and none are present for a version
        Expected: Processing should NOT proceed (need at least one present)
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_all_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                    # Missing version 20230102
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                    # Missing version 20230102
                ],
                "datavault.optional3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                    # Missing version 20230102
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Only 20230101 should be valid (all present)
            # 20230102 blocked because all optional and none present
            assert len(result) == 1
            assert [v['version'] for v in result] == ["20230101"]

    def test_all_dependencies_optional_at_least_one_present(self, mock_workflow_all_optional_deps):
        """
        Scenario: All dependencies are optional and at least one is present
        Expected: Processing should proceed
        """
        from unittest.mock import MagicMock, patch
        from test.metadata.workflow_dependency.version_provider.conftest import DummyProvider
        from azfr_skywalker_utils.metadata.workflow_dependency.version_provider import VersionStrategy
        
        with patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY") as mock_registry:
            mock_registry.get_workflow_config.return_value = mock_workflow_all_optional_deps
            mock_engine = MagicMock()
            provider = DummyProvider("extraction.test", mock_engine)
            
            dependencies_versions = {
                "datavault.optional1": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'},
                    {'version': "20230102", 'dependency_status': 'SUCCESS'}  # Present
                ],
                "datavault.optional2": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                    # Missing version 20230102 - delayed
                ],
                "datavault.optional3": [
                    {'version': "20230101", 'dependency_status': 'SUCCESS'}
                    # Missing version 20230102 - delayed
                ]
            }
            
            result = provider._find_valid_versions_across_dependencies(
                dependencies_versions, 
                VersionStrategy.ALL_AVAILABLE
            )
            
            # Both versions should be valid
            # 20230102 has at least one optional dependency present (optional1)
            assert len(result) == 2
            assert [v['version'] for v in result] == ["20230101", "20230102"]



class TestWorkflowRegistryBranches:
    """Targeted branch tests for WorkflowRegistry to increase branch coverage."""

    def test_from_dict_flat_format_without_workflows_key(self):
        """from_dict with flat dict (no 'workflows' key) covers the False branch of 'if workflows in config'."""
        flat_dict = {
            "parsing.a": {
                "layer": "parsing",
                "metadata_catalog": "cat_a",
                "metadata_schema": "sch_a",
            },
        }
        registry = WorkflowRegistry.from_dict(flat_dict)
        assert "parsing.a" in registry.workflows
        assert registry.workflows["parsing.a"].layer == LayerType.PARSING

    def test_from_dict_config_already_has_name(self):
        """from_dict where config dict already has 'name' key covers the False branch of 'if name not in config'."""
        config_with_name = {
            "workflows": {
                "parsing.b": {
                    "name": "parsing.b",  # 'name' already present
                    "layer": "parsing",
                    "metadata_catalog": "cat_b",
                    "metadata_schema": "sch_b",
                },
            }
        }
        registry = WorkflowRegistry.from_dict(config_with_name)
        assert registry.workflows["parsing.b"].name == "parsing.b"

    def test_get_workflow_config_unknown_raises(self):
        """get_workflow_config raises ValueError for unknown workflow name."""
        import pytest
        registry = WorkflowRegistry(workflows={
            "parsing.known": WorkflowEntry(
                name="parsing.known",
                layer=LayerType.PARSING,
                metadata_catalog="cat",
                metadata_schema="sch",
            ),
        })
        with pytest.raises(ValueError, match="Workflow not found"):
            registry.get_workflow_config("unknown.workflow")

    def test_get_workflows_by_layer_filtering(self):
        """get_workflows_by_layer returns workflows for specified layer."""
        registry = WorkflowRegistry(workflows={
            "parsing.p1": WorkflowEntry(
                name="parsing.p1",
                layer=LayerType.PARSING,
                metadata_catalog="cat",
                metadata_schema="sch",
            ),
            "parsing.p2": WorkflowEntry(
                name="parsing.p2",
                layer=LayerType.PARSING,
                metadata_catalog="cat",
                metadata_schema="sch",
            ),
            "datavault.d1": WorkflowEntry(
                name="datavault.d1",
                layer=LayerType.DATAVAULT,
                metadata_catalog="cat",
                metadata_schema="sch",
            ),
        })
        parsing_workflows = registry.get_workflows_by_layer(LayerType.PARSING)
        assert "parsing.p1" in parsing_workflows
        assert "parsing.p2" in parsing_workflows
        assert "datavault.d1" not in parsing_workflows



```

###### FILE: test/metadata/workflow_dependency/version_provider/workflow_version_provider/test_unique_versions.py ######

```py
"""Tests for the _unique_versions method."""
from unittest.mock import patch

from ..conftest import DummyProvider


class TestUniqueVersions:
    """Test class for _unique_versions method."""

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_unique_versions_empty_dependencies(self, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test _unique_versions with empty dependencies dictionary."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        dependencies_versions = {}
        
        result = provider._unique_versions(dependencies_versions)
        
        assert result == set()

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_unique_versions_single_dependency(self, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test _unique_versions with single dependency."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        dependencies_versions = {
            "parsing.dependency1": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        result = provider._unique_versions(dependencies_versions)
        
        expected = {"20230101", "20230102", "20230103"}
        assert result == expected

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_unique_versions_multiple_dependencies_with_overlap(self, mock_registry, mock_workflow_three_deps, mock_engine):
        """Test _unique_versions with multiple dependencies having overlapping versions."""
        mock_registry.get_workflow_config.return_value = mock_workflow_three_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        dependencies_versions = {
            "parsing.dependency1": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'}
            ],
            "datavault.dependency2": [
                {'version': "20230102", 'dependency_status': 'SUCCESS'},
                {'version': "20230103", 'dependency_status': 'SUCCESS'},
                {'version': "20230104", 'dependency_status': 'SUCCESS'}
            ],
            "extraction.dependency3": [
                {'version': "20230103", 'dependency_status': 'SUCCESS'},
                {'version': "20230105", 'dependency_status': 'SUCCESS'},
                {'version': "20230106", 'dependency_status': 'SUCCESS'}
            ]
        }
        
        result = provider._unique_versions(dependencies_versions)
        
        expected = {"20230101", "20230102", "20230103", "20230104", "20230105", "20230106"}
        assert result == expected

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_unique_versions_empty_version_lists(self, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test _unique_versions with dependencies having empty version lists."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        dependencies_versions = {
            "parsing.dependency1": [],
            "datavault.dependency2": [
                {'version': "20230101", 'dependency_status': 'SUCCESS'},
                {'version': "20230102", 'dependency_status': 'SUCCESS'}
            ],
        }
        
        result = provider._unique_versions(dependencies_versions)
        
        expected = {"20230101", "20230102"}
        assert result == expected

    @patch("azfr_skywalker_utils.metadata.workflow_dependency.version_provider.WORKFLOW_REGISTRY")
    def test_unique_versions_single_dependency_empty_list(self, mock_registry, mock_workflow_two_deps, mock_engine):
        """Test _unique_versions with single dependency having empty version list."""
        mock_registry.get_workflow_config.return_value = mock_workflow_two_deps
        
        provider = DummyProvider("datavault.test.workflow", mock_engine)
        dependencies_versions = {
            "parsing.dependency1": []
        }
        
        result = provider._unique_versions(dependencies_versions)
        
        expected = set()
        assert result == expected

```
