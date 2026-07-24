# Code dump: azfr-data-cetip-reporting

Generated from `/c/Users/LARBANI/repos_git/azfr-data-cetip-reporting`.

## File tree

```
.condarc
.github/workflows/build_parsing_workflow.yaml
.gitignore
activate_venv
constraints.txt
Dockerfile
ensure_conda
Makefile
model/format/migration_sante/extraction_ace_af.yaml
model/format/migration_sante/extraction_ace_ct.yaml
model/format/migration_sante/extraction_hace_af.yaml
model/format/migration_sante/extraction_hace_ct.yaml
model/format/ref_prevoyance/extraction_ape_nape_mig.yaml
model/format/ref_prevoyance/ref_prevoyance_rsr_nature_financiere.yaml
model/format/reporting/balances_comptes_echeances_schema.yaml
model/format/reporting/decaiss_enrichi_mensuels_schema.yaml
model/format/reporting/differentiel_assures_contrats_gt_schema.yaml
model/format/reporting/emission_cotisations_benef_schema.yaml
model/format/reporting/encaiss_ventil_valeur_schema.yaml
model/format/reporting/encaissements_schema.yaml
model/format/reporting/ent_contrat_pop_schema.yaml
model/format/reporting/ent_contrat_schema.yaml
model/format/reporting/ent_couverture_pop_schema.yaml
model/format/reporting/ent_salaries_schema.yaml
model/format/reporting/fichier_assures_contrats_gt_schema.yaml
model/format/reporting/indus_schema.yaml
model/format/reporting/operations_contrat_schema.yaml
model/format/reporting/precontentieux_collectif_v2_schema.yaml
model/format/reporting/precontentieux_individuel_v2_schema.yaml
model/README.md
model/setup.py
model/src/azfr_parsing_cetip_model/__init__.py
pip.conf
README.md
requirements.txt
requirements-dev.txt
scripts/fix_raw_data.py
setup.cfg
setup.py
tests/__init__.py
tests/base/__init__.py
tests/base/loader.py
tests/conftest.py
tests/test_migration_sante/__init__.py
tests/test_migration_sante/data/__init__.py
tests/test_migration_sante/data/test_file_generator.py
tests/test_migration_sante/test_functional.py
tests/test_prevoyance_rsr/__init__.py
tests/test_prevoyance_rsr/data/__init__.py
tests/test_prevoyance_rsr/data/test_file_generator.py
tests/test_prevoyance_rsr/test_functional.py
tests/test_prevoyance_rsr/test_unit_task.py
tests/test_reporting/__init__.py
tests/test_reporting/data/__init__.py
tests/test_reporting/data/test_file_generator.py
tests/test_reporting/test_functional.py
tests/test_reporting/test_unit_task.py
workflow/__init__.py
workflow/config.py
workflow/migration_sante.py
workflow/parse/__init__.py
workflow/parse/archive.py
workflow/parse/clean.py
workflow/parse/parse.py
workflow/parse/processing.py
workflow/parse/unzip.py
workflow/ref_prevoyance.py
workflow/reporting.py
workflow/utils/__init__.py
```

## File contents

###### FILE: .condarc ######

```condarc
ssl_verify: false
```

###### FILE: .github/workflows/build_parsing_workflow.yaml ######

```yaml
name: Build Prefect workflow

env:
  # Docker image name (lower-cased-kebab-case, without tag)
  DOCKER_IMAGE_NAME: prodazfrz6sh.azurecr.io/azfr-data-cetip-reporting
  # Docker image tag (default: ref name (branch name or tag name))
  DOCKER_IMAGE_TAG: ${{ github.ref_name }}
  # Run tests
  RUN_TESTS: "true"
  # Build and publish Docker image
  PUBLISH_ARTIFACTS: "true"
  # Credentials - don't touch
  AZFR_CI_USERNAME: ${{ secrets.AZFR_CI_USERNAME }}
  AZFR_CI_PASSWORD: ${{ secrets.AZFR_CI_PASSWORD }}
  DOCKER_CONFIG_JSON: ${{ secrets.DOCKER_CONFIG_JSON }}
  AZFR_PYPI_HOSTED_URL: ${{ secrets.AZFR_PYPI_HOSTED_URL }}

on:
  workflow_dispatch:
    inputs:
      run_tests:
        description: "Run tests"
        required: false
        type: choice
        options: [ "true", "false" ]
      publish_artifacts:
        description: "Publish Docker image"
        required: false
        type: choice
        options: [ "true", "false" ]
      disable_cache_read:
        description: "Not use cache data"
        required: true
        type: choice
        options: [ "true", "false" ]
        default: "false"
  push: {}

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
          docker: true
          sonar: true
          disable_cache_read: ${{ inputs.disable_cache_read == 'true' }}

      - name: Test
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        run: make update reports

      - name: Run SonarScanner
        uses: azf-h1-datascience/sonar-scanner/run@release
        if: ${{ inputs.run_tests == 'true' || (inputs.run_tests == '' && env.RUN_TESTS == 'true') }}
        with:
          host_url: ${{ secrets.SONAR_HOST_URL }}
          token: ${{ secrets.SONAR_TOKEN }}
          sources: model,workflow
          tests: tests
          params: |
            sonar.python.version: 3
            sonar.python.coverage.reportPaths: target/report/coverage.xml
            sonar.exclusions: "**/*.yaml,**/*.yml,workflow/query/**"
      
      - uses: azf-h1-datascience/use-docker@release
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        with:
          buildx: true

      - name: Build Docker image
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        run: |
          IMAGE=${DOCKER_IMAGE_NAME}:$(sanitize "${DOCKER_IMAGE_TAG}")
          BUILD_ARGS=""
          BUILD_ARGS="${BUILD_ARGS} --build-arg _GITHUB_REPOSITORY=${GITHUB_REPOSITORY}"
          BUILD_ARGS="${BUILD_ARGS} --build-arg _GITHUB_SHA=${GITHUB_SHA}"
          docker-build-push ${IMAGE} ${BUILD_ARGS} . 

      - name: Publish model module
        if: ${{ inputs.publish_artifacts == 'true' || (inputs.publish_artifacts == '' && env.PUBLISH_ARTIFACTS == 'true') }}
        run: |
          make update
          make deploy-model \
            PYPI_REPO_URL=$AZFR_PYPI_HOSTED_URL \
            PYPI_REPO_USER=$AZFR_CI_USERNAME \
            PYPI_REPO_PASS=$AZFR_CI_PASSWORD

```

###### FILE: .gitignore ######

```gitignore
# do not version log folder nor log files
*.log

# do not version tmp file
.~*
*~
*.*~
*.back
*tmp*

.idea/


#tools for dev
.tools

#python
venv

*.pyc
*.pyo
.pytest_cache
.coverage
__pycache__
.eggs/
*.egg
*.egg-info
build/
.tox/
/docs/_build/
coverage.xml
workspace.xml
.coverage.*
report

#pyspark
*/metastore_db/*

#================
#dotenv file
.env

#spark
spark-warehouse
directory
metastore_db/

#jupyter
.jupyter
*checkpoint.ipynb

# compiled output
tools/
tmp/
dist/
target/
*/tmp/
*/dist/
*/target/
*/build/
*.rst

# Java
*.class
*.jar
*.war
*.ear

# nodejs
node_modules/
/e2e/*.js
/e2e/*.map

# IDEs and editors
.idea
*.iml
.c9/
*.launch
*.sublime-workspace

# IDE - VSCode
.vscode/*
vcs.xml

# Eclipse
.workspace.xml
.classpath
.project
.settings

# misc
.cache/
/.sass-cache
/connect.lock
/coverage
/typings

# System Files
.DS_Store
Thumbs.db

#build virtual
provision.retry
.vagrant

#nohup
nohup.out



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
azfr-prefect-utils>=0.0.4,<0.1.0
azfr-parsing-utils>=0.5.0,<1.0.0
azfr-skywalker-utils>=7.3.7,<8
pydantic>=2.12.5,<3
polars>=1.0,<1.30.0
fsspec>=2026.2.0,<2027
fastapi>=0.135.2,<0.136
```

###### FILE: Dockerfile ######

```
FROM devazfrsg14.azurecr.io/pod-base:prefect-py312

# Use conda's libraries (including libstdc++) instead of system libraries
ENV LD_LIBRARY_PATH=/opt/conda/lib

RUN mkdir -p /work/model/format /work/workflow

#COPY pip.conf /etc/pip.conf
COPY requirements.txt constraints.txt setup.py /work/

RUN pip install --no-cache-dir -e . -r requirements.txt -c constraints.txt

COPY model/format /work/model/format
COPY workflow /work/workflow/

ARG _GITHUB_REPOSITORY
ARG _GITHUB_SHA

ENV GITHUB_REPOSITORY=${_GITHUB_REPOSITORY}
ENV GITHUB_SHA=${_GITHUB_SHA}

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
.PHONY: all update test generate coverage cov reports clean-reports deploy-model deploy-docs-model docs clean distclean

all: info

info:  ## Show this infomation
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

PYTHON_VERSION=3.12

REPORT_OUTPUT_DIR = ./target/report
DOCUMENT_OUTPUT_DIR = ./doc/_build
MODEL_FOLDER = model

venv:  ## Create virtualenv by Conda
	. ./ensure_conda && conda create -yq -p venv python=$(PYTHON_VERSION)
	cp -f pip.conf venv/pip.conf
	cp -f pip.conf venv/pip.ini
	. ./activate_venv && \
		python -m pip install -U -q -c constraints.txt pip && \
		pip install -U -q -c constraints.txt -e . -r requirements-dev.txt

update: venv  ## Update dependencies
	cp -f pip.conf venv/pip.conf
	cp -f pip.conf venv/pip.ini
	. ./activate_venv && pip install -U -q -c constraints.txt -e . -r requirements-dev.txt

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

generate: venv  ## Generate Model
	. ./activate_venv && generate_model -f $(MODEL_FOLDER)/format -o $(MODEL_FOLDER)/src/azfr_parsing_cetip_model

deploy-model: venv  ## Deploy library packages for model
	. ./activate_venv && \
		pip install -q twine wheel && \
		cd $(MODEL_FOLDER) && \
		python setup.py sdist bdist_wheel && \
		twine upload \
			--repository-url $(PYPI_REPO_URL) \
			--username $(PYPI_REPO_USER) \
			--password $(PYPI_REPO_PASS) \
			dist/*

deploy-docs-model: docs  ## Deploy API documents for model
	. ./activate_venv && \
		azfr-document-deploy \
			-i docs/_build/html \
			-o $(DOC_REPO_ROOT) \
			-n `python MODEL_FOLDER/setup.py --name` \
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

###### FILE: model/format/reporting/ent_couverture_pop_schema.yaml ######

```yaml
columns:
- data_type: DATE
  name: DATE_DONNEES
  format: DD/MM/YYYY
- data_type: STRING
  name: CODE_ENTITERATTACHEMENT
- data_type: STRING
  name: CODE_CENTREGESTION
- data_type: STRING
  name: NUMERO_CONTRAT_COLLECTIF
- data_type: STRING
  name: GROUPE_POPULATION
- data_type: STRING
  name: LIB_GP_POPULATION
- data_type: STRING
  name: RAISON_SOCIALE
- data_type: STRING
  name: SIREN
- data_type: STRING
  name: NIC
  name_in_header: CODE_NIC
- data_type: STRING
  name: APE
  name_in_header: CODE_APE
- data_type: STRING
  name: REFERENCE_EXTERNE
- data_type: DATE
  name: DATE_SOUSCRIPTION
  format: DD/MM/YYYY
- data_type: DATE
  name: DATE_RESILIATION
  name_in_header: DATE_RESILITATION
  format: DD/MM/YYYY
- data_type: STRING
  name: CODE_OFFRE
  name_in_header: OFFRE
- data_type: STRING
  name: CODE_FORMULE
- data_type: STRING
  name: LIB_FORMULE
- data_type: STRING
  name: CODE_PACK
- data_type: STRING
  name: LIB_PACK
- data_type: STRING
  name: CODE_SS_PACK
- data_type: STRING
  name: LIB_SS_PACK
- data_type: STRING
  name: CODE_GARANTIETECHNIQUE
- data_type: STRING
  name: LIB_GARANTIETECHNIQUE
- data_type: STRING
  name: LIB_LG_GARANTIE_TECHNIQUE
- data_type: DATE
  name: DATE_ADHESION_GT
  format: DD/MM/YYYY
- data_type: DATE
  name: DATE_RADIATION_GT
  format: DD/MM/YYYY
- data_type: STRING
  name: CODE_AGENCE
- data_type: STRING
  name: LIB_AGENCE
- data_type: STRING
  name: CODE_RESEAU
- data_type: STRING
  name: LIB_RESEAU
- data_type: STRING
  name: CODE_PRODUCTEUR
- data_type: STRING
  name: LIB_PRODUCTEUR
- data_type: STRING
  name: NATURE_COUVERTURE
- data_type: INTEGER
  name: NUM_ENTREPRISE
- data_type: INTEGER
  name: NUM_ETABLISSEMENT
name: ENT_COUVERTURE_POP
sep: ';'
header: True
encoding: cp1252
quoted_strings_can_be_null: True
extraction_type: 'full'
file_identifier: "ENT_COUVERTURE_POP"
```

###### FILE: model/README.md ######

```md
# AZFR Data azfr-parsing-cetip - Model Part
```

###### FILE: model/setup.py ######

```py
from setuptools import setup, find_packages

setup(
    name='azfr-parsing-cetip-model',
    description='azfr-parsing-cetip model',
    url='https://github.developer.allianz.io/azf-h1-datascience/azfr-data-azfr-parsing-cetip.git',

    long_description='file: README.md',
    long_description_content_type='text/markdown',

    author='Allianz France AI/Big Data Team',
    license='UNLICENSED',

    package_dir={'': 'src'},
    packages=find_packages(where='src'),
    zip_safe=False,

    include_package_data=True,

    python_requires='>=3.10',

    use_azfr_versioning=True
)
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
### **_Project Specific Configurations:_**

**_workflow/config.py_**

Workflow parameters can be edited here

**_model/format/_**

Respective yaml files containing the information of the tables are placed here with respective columns and details as follows,

    columns:
        date_type: <<date type of value of table column>>
        name: <<name of table column>>
        format: <<specific format if required, note tht this is optional>>

    name: <<table name>>
	sep: <<separator value>>
	header: <<Boolean>>
	encoding: <<file encoding>>
	quoted_strings_can_be_null: <<Boolean>>

Schema contains 2 additional parameters: **mode** that specify if the file is a delta or a full, they will be registered in separate folders, **parsed_name** that will be the name of the delta table if specified (if not specified it will use the default name).

**_workflow/parse/parse.py_**

Configuration for data type customisation,additional columns if required are configured under column_parsers and additional_columns
Parsing logic defined here is utilised in workflow/main.py

**_workflow/main.py_**

The prefect workflow is orchestrated here

**_tests/data/test_file_generator.py_**

Testing related file generation configuration can be edited here, ensure the column customisation if any also refelcts here

**_Requirements:_**

Refer Makefile, venv and update commands (Executing make venv or make update, creates or updates venv )

**_Generation of the model :_**

To deploy an model, you need to generate python file containing pydantic object with model of tables. Refer Makefile, generate command to know more. (Execute make generate, would generate models based on configuration in Make file for the input and output. Also refer azfr-parsing-utils/src/azfr_parsing_utils/model_generator/cli/generate_model.py for more details on model generation )

**_Local testing :_**

Execute make test

**_Template used:_**

	https://github.developer.allianz.io/azf-h1-datascience/azfr-data-parsing-prefect-template
	
On repositories that need to include the template :     `git remote add template [URL of the template repo]`

To fetch template level updates:  `git fetch --all`

To merge changes from template level :  `git merge template/[branch to merge] --allow-unrelated-histories`
```

###### FILE: requirements.txt ######

```txt
polars
fsspec
pyyaml
uuid7
azfr-fsspec-abfs
azfr-prefect-utils
azfr-parsing-utils
azfr-skywalker-utils

```

###### FILE: requirements-dev.txt ######

```txt
-r requirements.txt

azfr_release_utils
azfr-parsing-utils

pytest
pytest-html
pytest-forked
pytest-cov
pytest-xdist
pytest-html

flake8
flake8-codeclimate
flake8-html
safety

sphinx
m2r2
sphinx_rtd_theme
zipp

```

###### FILE: scripts/fix_raw_data.py ######

```py
import polars as pl
import zipfile
import os
from charset_normalizer import detect
import subprocess
from io import BytesIO

# Paths
input_directory = r"new_data"  
output_directory = r"new_data/output"  

def detect_encoding(file_path):
    file_content = file_path.read()
    result = detect(file_content)
    return result["encoding"]

def compare_files(zip_path_1, zip_path_2):
     with zipfile.ZipFile(zip_path_1, 'r') as z1, zipfile.ZipFile(zip_path_2, 'r') as z2:
        # Get the list of files inside each zip
        files1 = z1.namelist()
        files2 = z2.namelist()
        
        # Compare the file names in both zips
        for file1, file2 in zip(files1, files2):
            print(f"Comparing {file1} and {file2}")
            
            with z1.open(file1) as f1, z2.open(file2) as f2:  
                file1_lines = f1.readlines()
                file2_lines = f2.readlines()
                # Ensure both files have the same number of lines
                max_lines = max(len(file1_lines), len(file2_lines))

                for i in range(max_lines):
                    line1 = file1_lines[i].strip() if i < len(file1_lines) else ""
                    line2 = file2_lines[i].strip() if i < len(file2_lines) else ""

                    # Compare lines
                    if line1 != line2:
                        print(f"\nDifference found on line {i + 1}:")
                        print(f"File1: {line1}")
                        print(f"File2: {line2}")

# Function to update the code_reseau column
def update_code_reseau(df, zip_filename):
    # Apply conditional logic to update code_reseau

    if 'LIB_RESEAU' in df.columns:
        df = df.with_columns(
        pl.when( pl.col(reseau) == "SAL").then(pl.lit("CONSEILLER AZ"))
        .when( pl.col(reseau) == "AGT").then(pl.lit("AGENT ALLIANZ"))
        .when( pl.col(reseau) == "CRT").then(pl.lit("AZ COURTAGE"))
        .otherwise(pl.col("LIB_RESEAU"))
        .alias("LIB_RESEAU"))
    return df


# Loop through all ZIP files in the directory
for zip_filename in os.listdir(input_directory):
    if zip_filename.endswith('.zip'):  # Ensure the file is a ZIP file
        input_zip_path = os.path.join(input_directory, zip_filename)
        output_zip_path = os.path.join(output_directory, zip_filename)

        # Extract and read the CSV file from the input zip archive with the detected encoding
        with zipfile.ZipFile(input_zip_path, 'r') as z:
            # Assuming the zip contains a single CSV file
            csv_file_name = z.namelist()[0]
            with z.open(csv_file_name) as csv_file:
                #detected_encoding = detect_encoding(csv_file)
                #print(f"Detected encoding for {csv_file_name}: {detected_encoding}")
                
                # Read the CSV directly from the zip file using BytesIO and Polars
                df = pl.read_csv(BytesIO(csv_file.read()), separator=";", has_header=True, encoding="iso-8859-1",
                                 infer_schema=False, infer_schema_length=0)
        
        # Apply transformations if necessary (e.g., updating `code_reseau`)
        updated_df = update_code_reseau(df, zip_filename)

        # Write the updated CSV to a temporary file with UTF-8 encoding
        temp_csv_output_path = os.path.join(output_directory, f"temp_output_{zip_filename}")
        updated_df.write_csv(temp_csv_output_path, quote_char='"', quote_style="always", separator=";")

        # Convert the CSV file from UTF-8 to ISO-8859-1 (cp1252)
        output_csv_path = os.path.join(output_directory, f"output_{zip_filename}")
        with open(temp_csv_output_path, "r", encoding="utf-8") as utf8_file:
            with open(output_csv_path, "w", encoding="iso-8859-1") as cp1252_file:
                cp1252_file.write(utf8_file.read())
        
        # Optionally, use sed to modify the header or other transformations
        subprocess.run(f"sed -i '1s/\"//g' {output_csv_path}", shell=True)

        # Compare the files (original and updated)

        # Create a new zip file containing the updated CSV
        with zipfile.ZipFile(output_zip_path, 'w', zipfile.ZIP_DEFLATED) as z:
            z.write(output_csv_path, arcname=csv_file_name)
        #compare_files(input_zip_path, output_zip_path)

        # Clean up temporary files
        os.remove(temp_csv_output_path)
        os.remove(output_csv_path)

        print(f"Updated CSV file compressed into: {output_zip_path}")

```

###### FILE: setup.cfg ######

```cfg
[bdist_wheel]
universal = 1

[easy_install]
index_url = https://nexus-azfr-bigdata.devops-services.ew3.aws.aztec.cloud.allianz/repository/pypi-public/simple

[tool:pytest]
minversion = 3.0
testpaths = tests

[coverage:run]
branch = True
source = workflow/
```

###### FILE: setup.py ######

```py
from setuptools import find_packages, setup

setup(
    name='azfr-parsing-cetip-parse',
    description='Allianz FR Data Parsing azfr-parsing-cetip with Prefect',
    url='https://github.developer.allianz.io/azf-h1-datascience/azfr-data-azfr-parsing-cetip.git',

    long_description='file: README.md',

    author='Allianz France AI/Big Data Team',
    author_email='azfrbd@allianz.fr',

    license='UNLICENSED',

    packages=find_packages(include='workflow'),
    python_requires='>=3.10',

    use_azfr_versioning=True
)

```

###### FILE: tests/__init__.py ######

```py
import azfr_parsing_utils.azure

azfr_parsing_utils.azure.use()

```

###### FILE: tests/base/loader.py ######

```py
# coding: utf-8

import logging

import azfr_fsspec_utils

logger = logging.getLogger(__name__)


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
```

###### FILE: tests/conftest.py ######

```py
# coding: utf-8
import azfr_fsspec_utils
import os
import pytest
from tests.base.loader import TestLoader
from prefect.testing.utilities import prefect_test_harness

CURRENT_DIR = os.path.abspath(os.path.dirname(__file__))
TEST_DIR = os.path.abspath(os.path.dirname(CURRENT_DIR))
TARGET_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))


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


@pytest.fixture(autouse=True)
def disable_prefect_api_url():
    """Disable Prefect server connections for tests."""
    old_value = os.environ.get("PREFECT_API_URL")
    os.environ["PREFECT_API_URL"] = ""
    os.environ["PREFECT_LOGGING_LEVEL"] = "CRITICAL"
    yield
    if old_value is not None:
        os.environ["PREFECT_API_URL"] = old_value
    else:
        os.environ.pop("PREFECT_API_URL", None)

@pytest.fixture(scope="session", autouse=True)
def prefect_test_mode():
    """Enable Prefect test mode for all tests."""
    with prefect_test_harness():
        yield

"""
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

"""
@pytest.fixture(autouse=True)
def loader(request):
    test_dir, target_dir = _paths(request)

    loader = TestLoader(
        test_dir=test_dir,
        target_dir=target_dir,
    )

    return loader

```

###### FILE: tests/test_migration_sante/__init__.py ######

```py
# coding: utf-8
import os
from workflow.config import MigrationSanteConfig
from azfr_skywalker_utils.utils.mail import EmailConfig


empty_config_dict = {
        "workflow_name": "",
        "input_dir": "",
        "archive_dir": "",
        "parsed_dir": "",
        "rejected_dir": "",
        "file_pattern": "",
        "metadata_dir": "",
        "files_configs": {"EXTRACTION.ACE.AF": {"file_identifier": "EXTRACTION.ACE.AF", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "20991231", "period_checked": 0},
                        "EXTRACTION.ACE.CT": {"file_identifier": "EXTRACTION.ACE.CT", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "20991231", "period_checked": 0},
                        "EXTRACTION.HACE.AF": {"file_identifier": "EXTRACTION.HACE.AF", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "20991231", "period_checked": 0},
                        "EXTRACTION.HACE.CT": {"file_identifier": "EXTRACTION.HACE.CT", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "20991231", "period_checked": 0},
                        }
}
empty_config = MigrationSanteConfig(**empty_config_dict)

empty_mail_config = EmailConfig(**{
    "sender": "",
    "to": "",
    "subject": "",
    "host": "",
    "port": 2525,
    "ssl": False,
    "color": "",  # Use it if you need a new color
    "error_receivers": ""  # Use it if you need to receive only errors
})

TABLES = empty_config.tables

TARGET_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), "../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))

```

###### FILE: tests/test_migration_sante/data/__init__.py ######

```py
# coding: utf-8

import os

CURRENT_DIR = os.path.abspath(os.path.dirname(__file__))

TARGET_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))
TARGET_TMP_DIR = os.path.abspath(os.path.join(TARGET_TEST, "tmp"))
```

###### FILE: tests/test_migration_sante/data/test_file_generator.py ######

```py
# coding: utf-8

import datetime
import random
import string
from datetime import datetime as dt
from random import randint
from urllib.parse import urlparse, urlunparse
import os
import pytz
import re
import azfr_fsspec_utils
from azfr_fsspec_utils.zipfile import FsspecZipFile
import tarfile

random.seed(4587)

letters = string.ascii_letters + string.ascii_uppercase + ' \t'
decimal_pattern = r"^decimal\((?P<length>[0-9]+),(?P<precision>[0-9]+)\)$"


def random_date(pattern):
    """Generate a random date """
    null = random.randint(0, 20)
    if null == 0:
        return ""

    end = datetime.datetime(2019, 1, 1).timestamp()

    epoch = random.uniform(0.01, 1) * end

    return pattern.format(datetime.datetime.fromtimestamp(epoch).astimezone(tz=pytz.timezone("Europe/Paris")))


def random_string():
    """Generate a random string """
    null = randint(0, 20)
    if null == 0:
        return ""

    length = randint(1, 30)
    letters = string.ascii_lowercase
    return ''.join(random.choice(letters) for i in range(length))


def random_int(custom_column=None):
    """Generate a random int """
    null = randint(0, 20)
    if null == 0:
        return ""
    if custom_column == "INTEGER_SPACED":
        return "{:,d}".format(randint(0, 100000)).replace(",", " ")
    return "{:d}".format(randint(0, 300))


def random_double(length=3, precision=2, decimal_point=None, custom_column=None):
    """Generate a random normal double type """
    null = randint(0, 20)
    if null == 0:
        return ""

    if not decimal_point:
        decimal_point = "."

    if custom_column == "DECIMAL_SPACED":
        return "{:,}".format(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(",", " ")\
            .replace(".", decimal_point)

    return str(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(".", decimal_point)


def random_boolean():
    """Generate a random boolean"""
    return "True" if random.getrandbits(1) else "False"


def generate_test_file(clz, file_format, folder, epoch=None, nbr_rows=None, max_rows=30):
    try:
        azfr_fsspec_utils.makedirs(folder)
    except FileExistsError:
        pass
    now = dt.fromtimestamp(epoch)
    file_name = "{name}.{date}.csv"
    gz = azfr_fsspec_utils.join(
        folder, file_name.format(
            name=clz.upper(), date="{:%Y%m%d}".format(now),
        ),
    )

    encoding = file_format.encoding
    delimiter = file_format.sep
    decimal_point = file_format.decimal_point
    rows = []
    empty=False
    if not nbr_rows:
        empty = True
        nbr_rows = randint(1, max_rows)
    for i in range(nbr_rows):
        values = []
        for column in file_format.columns:
            ctype = column.data_type.lower()
            custom_format = column.format
            # If you have custom format, please replace the values below
            if custom_format == "YYYY-MM-DD":
                values.append(random_date("{:%Y-%m-%d}"))
            elif custom_format == "DD/MM/YYYY":
                values.append(random_date("{:%d/%m/%Y}"))                
            elif custom_format == "DD/MM/YYYY HH:mm:ss":
                values.append(random_date("{:%d/%m/%Y %H:%M:%S}"))
            elif custom_format == "INTEGER_SPACED":
                values.append(random_int(custom_column="INTEGER_SPACED"))
            elif custom_format == "DECIMAL_SPACED":
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point, custom_column="DECIMAL_SPACED"))
            elif custom_format:
                raise ValueError("Unknown custom format {}. "
                                 "Please add custom data creation for this format"
                                 .format(custom_format))
            elif ctype == "integer" or ctype == "bigint":
                values.append(random_int())
            elif ctype == "double" or ctype == "float":
                values.append(random_double())
            elif re.match(decimal_pattern, ctype):
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point))
            elif ctype == "date":
                values.append(random_date("{:%Y-%m-%d}"))
            elif ctype == "datetime":
                values.append(random_date("{:%Y-%m-%d %H:%M:%S}"))
            elif ctype == "string":
                values.append(random_string())
            elif ctype == "boolean":
                values.append(random_boolean())
            else:
                raise ValueError("Unknown type {}".format(ctype))
        rows.append(delimiter.join(values))
    rows.append("")
    scheme, netloc, url, params, query, fragment = urlparse(gz)
    if scheme in ['file', '']:
        gz_url = url
    else:
        gz_url = azfr_fsspec_utils.join('.', azfr_fsspec_utils.basename(gz))

    with azfr_fsspec_utils.open(gz_url, 'wb') as f:
        if file_format.header == True:
            headers = [column.name_in_header or column.name for column in file_format.columns]
            if empty:
                lines = delimiter.join(headers) + "\n"        
            else:
                lines = delimiter.join(headers) + "\n" + "\n".join(rows)
        else:
            lines = "\n".join(rows)
        f.write(lines.encode(encoding))

    if scheme not in ['file', '']:
        full_path = urlunparse([scheme, netloc, url, params, query, fragment])
        azfr_fsspec_utils.move(gz_url, full_path)
    return gz


def generate_zipped_file(clz, file_format, folder, epoch=None, nbr_rows=None):
    now = dt.fromtimestamp(epoch)
    file = generate_test_file(clz, file_format, folder, epoch, nbr_rows)
    folder = azfr_fsspec_utils.dirname(file)
    zip_name = "{name}.{date}.zip".format(name=clz.upper(), date="{:%Y%m%d}".format(now))
    zip_path = azfr_fsspec_utils.join(folder, zip_name)
    with FsspecZipFile(zip_path, "w") as z:
        z.write(file, arcname=azfr_fsspec_utils.basename(file))
    azfr_fsspec_utils.remove(file)
    return zip_path

```

###### FILE: tests/test_migration_sante/test_functional.py ######

```py
# coding: utf-8
from datetime import datetime as dt
from workflow.migration_sante import migration_sante_flow
from tests.test_migration_sante import empty_config_dict, empty_mail_config
# coding: utf-8
import azfr_fsspec_utils as fspath
from workflow.config import MigrationSanteConfig
from tests.test_migration_sante.data import test_file_generator
import datetime


def test_migration_sante_flow(loader):
    now = dt.now()
    dates = [now - datetime.timedelta(days=1), now]
    nbr_rows = 25

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"], 
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata")
    }
    for path in config_dict.values():
        fspath.makedirs(path, exist_ok=True)

    config_dict["files_configs"] = empty_config_dict["files_configs"]
    config_dict["file_pattern"] = r"^(?P<file_identifier>[a-zA-Z0-9_\.]+)\.(?P<date>[0-9]{8})\.(?P<suffix>zip)$"
    test_config = MigrationSanteConfig(**config_dict)

    tables = test_config.tables
    for date in dates:
        for clz in tables:
            test_file_generator.generate_zipped_file(clz, test_config.tables_to_format[clz], test_config.input_dir, dt.timestamp(date), nbr_rows)
    state = migration_sante_flow(test_config, empty_mail_config, return_state=True)
    assert state.is_completed()

```

###### FILE: tests/test_prevoyance_rsr/__init__.py ######

```py
# coding: utf-8
import os
from workflow.config import RefPrevoyanceConfig
from azfr_skywalker_utils.utils.mail import EmailConfig


TARGET_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), "../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))


empty_config_dict = {
        "workflow_name": "",
        "input_dir": "",
        "archive_dir": "",
        "parsed_dir": "",
        "rejected_dir": "",
        "metadata_dir": "",
        "file_pattern": "",
        "files_configs": {
            "REF.SCPREV_RSR_NATURE_FINANCIERE": {"file_identifier": "REF.SCPREV_RSR_NATURE_FINANCIERE", "overdue_time": "18:15", "mode": "daily",
                                             "min_expected_date": "", "period_checked": 2},
            "EXTRACTION.APE-NAPE.CT": {"file_identifier": "EXTRACTION.APE-NAPE.CT", "overdue_time": "18:15", "mode": "daily",
                                       "min_expected_date": "", "period_checked": 2},
        },
    }

empty_config = RefPrevoyanceConfig(**empty_config_dict)

_rsr_tables = [t for t in empty_config.tables if empty_config.tables_to_format[t].file_identifier == "SCPREV_RSR_NATURE_FINANCIERE"]
_ape_nape_tables = [t for t in empty_config.tables if empty_config.tables_to_format[t].file_identifier == "EXTRACTION.APE-NAPE.CT"]

file_specs = {
    "ref_prevoyance_rsr": {
        "file_name": "REF.{file_identifier}.{date}",
        "pattern": "^(?P<prefix>REF\\.)(?P<file_identifier>[a-zA-Z0-9_]+)\\.(?P<date>[0-9]{8})(?P<suffix>\\.zip)$",
        "tables": _rsr_tables,
    },
    "ref_prevoyance_ape_nape": {
        "file_name": "{file_identifier}.{date}",
        "pattern": "^(?P<file_identifier>[a-zA-Z0-9_.\\-]+)\\.(?P<date>[0-9]{8})(?P<suffix>\\.zip)$",
        "tables": _ape_nape_tables,
    },
}

empty_mail_config = EmailConfig(**{
    "sender": "",
    "to": "",
    "subject": "",
    "host": "",
    "port": 2525,
    "ssl": False,
    "color": "",  # Use it if you need a new color
    "error_receivers": ""  # Use it if you need to receive only errors
})

```

###### FILE: tests/test_prevoyance_rsr/data/__init__.py ######

```py
# coding: utf-8

import os

CURRENT_DIR = os.path.abspath(os.path.dirname(__file__))

TARGET_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))
TARGET_TMP_DIR = os.path.abspath(os.path.join(TARGET_TEST, "tmp"))
```

###### FILE: tests/test_prevoyance_rsr/data/test_file_generator.py ######

```py
# coding: utf-8

import datetime
import random
import string
from datetime import datetime as dt
from random import randint
from urllib.parse import urlparse, urlunparse
import os
import pytz
import re
import azfr_fsspec_utils
from azfr_fsspec_utils.zipfile import FsspecZipFile

random.seed(4587)

letters = string.ascii_letters + string.ascii_uppercase + ' \t'
decimal_pattern = r"^decimal\((?P<length>[0-9]+),(?P<precision>[0-9]+)\)$"


def random_date(pattern):
    """Generate a random date """
    null = random.randint(0, 20)
    if null == 0:
        return ""

    end = datetime.datetime(2019, 1, 1).timestamp()

    epoch = random.uniform(0.01, 1) * end

    return pattern.format(datetime.datetime.fromtimestamp(epoch).astimezone(tz=pytz.timezone("Europe/Paris")))


def random_string():
    """Generate a random string """
    null = randint(0, 20)
    if null == 0:
        return ""

    length = randint(1, 30)
    letters = string.ascii_lowercase
    return ''.join(random.choice(letters) for i in range(length))


def random_int(custom_column=None):
    """Generate a random int """
    null = randint(0, 20)
    if null == 0:
        return ""
    if custom_column == "INTEGER_SPACED":
        return "{:,d}".format(randint(0, 100000)).replace(",", " ")
    return "{:d}".format(randint(0, 300))


def random_double(length=3, precision=2, decimal_point=None, custom_column=None):
    """Generate a random normal double type """
    null = randint(0, 20)
    if null == 0:
        return ""

    if not decimal_point:
        decimal_point = "."

    if custom_column == "DECIMAL_SPACED":
        return "{:,}".format(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(",", " ")\
            .replace(".", decimal_point)

    return str(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(".", decimal_point)


def random_boolean():
    """Generate a random boolean"""
    return "True" if random.getrandbits(1) else "False"


def generate_test_file(file_identifier, file_format, folder, file_name, epoch=None, nbr_rows=None, max_rows=30):
    try:
        azfr_fsspec_utils.makedirs(folder)
    except FileExistsError:
        pass
    now = dt.fromtimestamp(epoch)
    gz = azfr_fsspec_utils.join(
        folder, file_name.format(
            file_identifier=file_identifier, date="{:%Y%m%d}".format(now) + ".csv",
        ),
    )

    encoding = file_format.encoding
    delimiter = file_format.sep
    decimal_point = file_format.decimal_point
    rows = []
    if not nbr_rows:
        nbr_rows = randint(1, max_rows)
    for i in range(nbr_rows):
        values = []
        for column in file_format.columns:
            ctype = column.data_type.lower()
            custom_format = column.format
            # If you have custom format, please replace the values below
            if custom_format == "DD/MM/YYYY" or custom_format == "DD/MM/YYYY-WIDE":
                values.append(random_date("{:%d/%m/%Y}"))
            elif custom_format == "DD/MM/YYYY HH:mm:ss":
                values.append(random_date("{:%d/%m/%Y %H:%M:%S}"))
            elif custom_format == "INTEGER_SPACED":
                values.append(random_int(custom_column="INTEGER_SPACED"))
            elif custom_format == "DECIMAL_SPACED":
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point, custom_column="DECIMAL_SPACED"))
            elif custom_format:
                raise ValueError("Unknown custom format {}. "
                                 "Please add custom data creation for this format"
                                 .format(custom_format))
            elif ctype == "integer" or ctype == "bigint":
                values.append(random_int())
            elif ctype == "double" or ctype == "float":
                values.append(random_double())
            elif re.match(decimal_pattern, ctype):
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point))
            elif ctype == "date":
                values.append(random_date("{:%Y-%m-%d}"))
            elif ctype == "datetime":
                values.append(random_date("{:%Y-%m-%d %H:%M:%S}"))
            elif ctype == "string":
                values.append(random_string())
            elif ctype == "boolean":
                values.append(random_boolean())
            else:
                raise ValueError("Unknown type {}".format(ctype))
        rows.append(delimiter.join(values))
    rows.append("")
    scheme, netloc, url, params, query, fragment = urlparse(gz)
    if scheme in ['file', '']:
        gz_url = url
    else:
        gz_url = azfr_fsspec_utils.join('.', azfr_fsspec_utils.basename(gz))

    with azfr_fsspec_utils.open(gz_url, 'wb') as f:
        if file_format.header == True:
            headers = [column.name_in_header or column.name for column in file_format.columns]
            lines = delimiter.join(headers) + "\n" + "\n".join(rows)
        else:
            lines = "\n".join(rows)
        f.write(lines.encode(encoding))

    if scheme not in ['file', '']:
        full_path = urlunparse([scheme, netloc, url, params, query, fragment])
        azfr_fsspec_utils.move(gz_url, full_path)
    return gz


def generate_zipped_file(folder, file_name, file_specs, file_identifier, formats, epoch, nbr_rows=None):
    now = dt.fromtimestamp(epoch)
    files = []
    for table_name in file_specs[file_identifier].tables:
        file = generate_test_file(file_identifier, formats[table_name], folder, file_name, epoch, nbr_rows)
        files.append(file)
    zip_name = file_name.format(file_identifier=file_identifier, date="{:%Y%m%d}".format(now)) + ".zip"
    zip_path = azfr_fsspec_utils.join(folder, zip_name)
    with FsspecZipFile(zip_path, "w") as z:
        for file in files:
            z.write(file, arcname=azfr_fsspec_utils.basename(file))
            azfr_fsspec_utils.remove(file)
    return zip_path


def generate_test_files(files_configs, folder, file_name, formats, epoch, nbr_rows=None):
    files_generated = []
    file_identifiers = list(set([formats[table].file_identifier for table in formats]))
    for file_identifier in file_identifiers:
        files_generated.append(generate_zipped_file(folder, file_name, files_configs,
                                                    file_identifier, formats, epoch, nbr_rows))
    return files_generated

```

###### FILE: tests/test_prevoyance_rsr/test_functional.py ######

```py
# coding: utf-8
from datetime import datetime as dt
from workflow.ref_prevoyance import ref_prevoyance_flow
from azfr_parsing_utils.csv import CsvColumn
from tests.test_prevoyance_rsr import empty_config_dict, empty_mail_config, file_specs
# coding: utf-8
import azfr_fsspec_utils as fspath
from workflow.config import RefPrevoyanceConfig
from tests.test_prevoyance_rsr.data import test_file_generator
from deltalake import DeltaTable
import datetime


def test_main_task_one_file(loader):
    """ fast one file test"""
    date = dt.now()
    nbr_rows = 2

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }
    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict["file_pattern"] = file_specs["ref_prevoyance_rsr"]["pattern"]
    test_config = RefPrevoyanceConfig(**config_dict)

    state = ref_prevoyance_flow(test_config, empty_mail_config, return_state=True)
    assert state.is_completed()


def test_main_task(loader):
    now = dt.now()
    dates = [now - datetime.timedelta(days=1), now]
    nbr_rows = 25

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }
    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict["file_pattern"] = file_specs["ref_prevoyance_rsr"]["pattern"]
    test_config = RefPrevoyanceConfig(**config_dict)

    tables = file_specs["ref_prevoyance_rsr"]["tables"]
    for date in dates:
        test_file_generator.generate_test_files(test_config.files_configs,
                                                test_config.input_dir,
                                                file_specs["ref_prevoyance_rsr"]["file_name"],
                                                test_config.tables_to_format,
                                                dt.timestamp(date),
                                                nbr_rows)

    ref_prevoyance_flow(test_config, empty_mail_config)

    expected_versions = ["__version__=" + date.strftime("%Y%m%d") for date in dates]
    expected_versions.sort()

    for table in tables:
        file_format = test_config.tables_to_format[table]
        if file_format.parsed_name:
            table_name = file_format.parsed_name.lower()
        else:
            table_name = table.lower()
        delta_table = DeltaTable(fspath.abspath(fspath.join(test_config.parsed_dir, file_format.extraction_type, table_name)))
        output_versions = [fspath.dirname(item) for item in delta_table.files()]
        output_versions.sort()
        assert output_versions == expected_versions
        assert delta_table.to_pyarrow_table().shape[0] == nbr_rows*len(dates)

    # Empty rerun
    empty_run = ref_prevoyance_flow(test_config, empty_mail_config, return_state=True)
    assert empty_run.is_completed()

def test_overwrite_schema(loader):
    """Test that a column can be added when overwrite_schema is set to True"""
    now = dt.now()
    yest = now - datetime.timedelta(days=1)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict[
        "file_pattern"] = file_specs["ref_prevoyance_rsr"]["pattern"]
    test_config = RefPrevoyanceConfig(**config_dict)
    test_config.overwrite_schema = True

    # Standard run
    test_file_generator.generate_test_files(test_config.files_configs,
                                            test_config.input_dir,
                                            file_specs["ref_prevoyance_rsr"]["file_name"],
                                            test_config.tables_to_format,
                                            dt.timestamp(yest))
    ref_prevoyance_flow(test_config, empty_mail_config)

    rsr_tables = file_specs["ref_prevoyance_rsr"]["tables"]

    # Add new column to schema
    for clz in rsr_tables:
        test_config.tables_to_format[clz].columns.append(CsvColumn(name="TEST_OVERWRITE_SCHEMA", data_type="STRING"))

    # Generate files with new column
    test_file_generator.generate_test_files(test_config.files_configs,
                                            test_config.input_dir,
                                            file_specs["ref_prevoyance_rsr"]["file_name"],
                                            test_config.tables_to_format,
                                            dt.timestamp(now))

    ref_prevoyance_flow(test_config, empty_mail_config)

    # Ensure new column is included in deltatable
    for table in rsr_tables:
        file_format = test_config.tables_to_format[table]
        if file_format.parsed_name:
            table_name = file_format.parsed_name.lower()
        else:
            table_name = table.lower()
        delta_table = DeltaTable(fspath.abspath(fspath.join(test_config.parsed_dir, file_format.extraction_type, table_name)))
        expected_cols = [item.name for item in file_format.columns] + ["__version__", "__functional_date__", "__metadata__"]
        expected_cols.sort()
        output_cols = [item.name for item in delta_table.schema().fields]
        output_cols.sort()
        assert expected_cols == output_cols
```

###### FILE: tests/test_prevoyance_rsr/test_unit_task.py ######

```py

# coding: utf-8
import azfr_fsspec_utils
import datetime
import os
import pytest
import yaml
from workflow.config import RefPrevoyanceConfig
from datetime import datetime as dt
from workflow.parse.archive import archive_file
from workflow.parse.parse import parse_file
from workflow.parse.unzip import unzip
from prefect.logging import disable_run_logger
from tests.test_prevoyance_rsr.data import test_file_generator
from tests.test_prevoyance_rsr import file_specs, empty_config_dict
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata


def test_task_archive(loader):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": "",
        "rejected_dir": "",
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = RefPrevoyanceConfig(**config_dict)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")
    landing_files = test_file_generator.generate_test_files(test_config.files_configs,
                                                            test_config.input_dir,
                                                            file_specs["ref_prevoyance_rsr"]["file_name"],
                                                            test_config.tables_to_format,
                                                            epoch)

    archive_folder = azfr_fsspec_utils.abspath(azfr_fsspec_utils.join(test_config.archive_dir, date))

    with disable_run_logger():
        for landing_file in landing_files:
            archive_file.fn(landing_file, archive_folder)

    for path in landing_files:
        assert azfr_fsspec_utils.exists(
            azfr_fsspec_utils.join(
                test_config.archive_dir,
                date,
                azfr_fsspec_utils.basename(path),
            ),
        )
    files_rested_input_dir = azfr_fsspec_utils.listdir(test_config.input_dir)
    assert len(files_rested_input_dir) == 0


def test_task_unzip(loader):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": "",
        "archive_dir": loader.dest("archive"),
        "parsed_dir": "",
        "rejected_dir": "",
        "unzipped_dir": loader.dest("unzipped"),
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = RefPrevoyanceConfig(**config_dict)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")

    archive_folder = azfr_fsspec_utils.abspath(azfr_fsspec_utils.join(test_config.archive_dir, date))

    paths = test_file_generator.generate_test_files(test_config.files_configs,
                                                    archive_folder,
                                                    file_specs["ref_prevoyance_rsr"]["file_name"],
                                                    test_config.tables_to_format,
                                                    epoch)
    with disable_run_logger():
        for path in paths:
            unzip.fn(path, test_config.unzipped_dir)

    assert len(azfr_fsspec_utils.listdir(test_config.unzipped_dir)) == len(test_config.tables)
    assert all([f.endswith(".csv") for f in azfr_fsspec_utils.listdir(test_config.unzipped_dir)])


@pytest.mark.parametrize("table", file_specs["ref_prevoyance_rsr"]["tables"])
def test_task_parse(loader, table):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = RefPrevoyanceConfig(**config_dict)
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(test_config, write_start_event=False)
    azfr_fsspec_utils.makedirs(test_config.input_dir, exist_ok=True)
    azfr_fsspec_utils.makedirs(test_config.rejected_dir, exist_ok=True)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")
    archived_dir = azfr_fsspec_utils.join(test_config.archive_dir,
                                          date)
    path = test_file_generator.generate_test_file(table,
                                                  test_config.tables_to_format[table],
                                                  archived_dir,
                                                  file_specs["ref_prevoyance_rsr"]["file_name"],
                                                  epoch,
                                                  max_rows=30)

    file_format = test_config.tables_to_format[table]
    rejected_file = azfr_fsspec_utils.join(test_config.rejected_dir, f"{date}_{table.lower()}.txt")
    parsed_dir = azfr_fsspec_utils.join(test_config.parsed_dir, table.lower())

    with disable_run_logger():
        parse_file.fn(path, parsed_dir, rejected_file, file_format, date, workflow_metadata)

    outputs = azfr_fsspec_utils.listdir(
        azfr_fsspec_utils.join(
            test_config.parsed_dir, table.lower()
        )
    )
    expected = ['_delta_log', "__version__=" + date].sort()
    assert expected == outputs.sort()

```

###### FILE: tests/test_reporting/__init__.py ######

```py
# coding: utf-8
import os
from workflow.config import ReportingConfig
from azfr_skywalker_utils.utils.mail import EmailConfig


TARGET_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), "../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))


empty_config_dict = {
        "workflow_name": "",
        "input_dir": "",
        "archive_dir": "",
        "parsed_dir": "",
        "rejected_dir": "",
        "metadata_dir": "",
        "file_pattern": "",
        "files_configs": {"BALANCES_COMPTES_ECHEANCES": {"file_identifier": "BALANCES_COMPTES_ECHEANCES", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "DECAISS_ENRICHI_MENSUELS": {"file_identifier": "DECAISS_ENRICHI_MENSUELS", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "DIFFERENTIEL_ASSURES_CONTRATS_GT": {"file_identifier": "DIFFERENTIEL_ASSURES_CONTRATS_GT", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "EMISSION_COTISATIONS_BENEF": {"file_identifier": "EMISSION_COTISATIONS_BENEF", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "ENCAISS_VENTIL_VALEUR": {"file_identifier": "ENCAISS_VENTIL_VALEUR", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "ENCAISSEMENTS": {"file_identifier": "ENCAISSEMENTS", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "ENT_CONTRAT_POP": {"file_identifier": "ENT_CONTRAT_POP", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "ENT_CONTRAT": {"file_identifier": "ENT_CONTRAT", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "ENT_COUVERTURE_POP": {"file_identifier": "ENT_COUVERTURE_POP", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "ENT_SALARIES": {"file_identifier": "ENT_SALARIES", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "FICHIER_ASSURES_CONTRATS_GT": {"file_identifier": "FICHIER_ASSURES_CONTRATS_GT", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "INDUS": {"file_identifier": "INDUS", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "OPERATIONS_CONTRAT": {"file_identifier": "OPERATIONS_CONTRAT", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        "PRECONTENTIEUX_COLLECTIF_V2": {"file_identifier": "PRECONTENTIEUX_COLLECTIF_V2", "overdue_time": "18:15", "mode": "daily",
                                    "min_expected_date": "", "period_checked": 2},
                        "PRECONTENTIEUX_INDIVIDUEL_V2": {"file_identifier": "PRECONTENTIEUX_INDIVIDUEL_V2", "overdue_time": "18:15", "mode": "daily",
                                   "min_expected_date": "", "period_checked": 2},
                        },
    }

empty_config = ReportingConfig(**empty_config_dict)

file_specs = {
    "reporting": {
            "file_name": "AZCTPC.ENT.Q.{file_identifier}.{date}",
            "pattern": "^(?P<prefix>AZCTPC\\.ENT\\.Q\\.)(?P<file_identifier>[a-zA-Z0-9_]+)\\.(?P<date>[0-9]{8})(?P<suffix>\\.zip)$",
            "tables": empty_config.tables}
}

empty_mail_config = EmailConfig(**{
    "sender": "",
    "to": "",
    "subject": "",
    "host": "",
    "port": 2525,
    "ssl": False,
    "color": "",  # Use it if you need a new color
    "error_receivers": ""  # Use it if you need to receive only errors
})

```

###### FILE: tests/test_reporting/data/__init__.py ######

```py
# coding: utf-8

import os

CURRENT_DIR = os.path.abspath(os.path.dirname(__file__))

TARGET_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../../target"))
TARGET_TEST = os.path.abspath(os.path.join(TARGET_DIR, "tests"))
TARGET_TMP_DIR = os.path.abspath(os.path.join(TARGET_TEST, "tmp"))
```

###### FILE: tests/test_reporting/data/test_file_generator.py ######

```py
# coding: utf-8

import datetime
import random
import string
from datetime import datetime as dt
from random import randint
from urllib.parse import urlparse, urlunparse
import os
import pytz
import re
import azfr_fsspec_utils
from azfr_fsspec_utils.zipfile import FsspecZipFile

random.seed(4587)

letters = string.ascii_letters + string.ascii_uppercase + ' \t'
decimal_pattern = r"^decimal\((?P<length>[0-9]+),(?P<precision>[0-9]+)\)$"


def random_date(pattern):
    """Generate a random date """
    null = random.randint(0, 20)
    if null == 0:
        return ""

    end = datetime.datetime(2019, 1, 1).timestamp()

    epoch = random.uniform(0.01, 1) * end

    return pattern.format(datetime.datetime.fromtimestamp(epoch).astimezone(tz=pytz.timezone("Europe/Paris")))


def random_string():
    """Generate a random string """
    null = randint(0, 20)
    if null == 0:
        return ""

    length = randint(1, 30)
    letters = string.ascii_lowercase
    return ''.join(random.choice(letters) for i in range(length))


def random_int(custom_column=None):
    """Generate a random int """
    null = randint(0, 20)
    if null == 0:
        return ""
    if custom_column == "INTEGER_SPACED":
        return "{:,d}".format(randint(0, 100000)).replace(",", " ")
    return "{:d}".format(randint(0, 300))


def random_double(length=3, precision=2, decimal_point=None, custom_column=None):
    """Generate a random normal double type """
    null = randint(0, 20)
    if null == 0:
        return ""

    if not decimal_point:
        decimal_point = "."

    if custom_column == "DECIMAL_SPACED":
        return "{:,}".format(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(",", " ")\
            .replace(".", decimal_point)

    return str(round(random.uniform(0.0, 10**(length-precision)), precision)).replace(".", decimal_point)


def random_boolean():
    """Generate a random boolean"""
    return "True" if random.getrandbits(1) else "False"


def generate_test_file(file_identifier, file_format, folder, file_name, epoch=None, nbr_rows=None, max_rows=30):
    try:
        azfr_fsspec_utils.makedirs(folder)
    except FileExistsError:
        pass
    now = dt.fromtimestamp(epoch)
    gz = azfr_fsspec_utils.join(
        folder, file_name.format(
            file_identifier=file_identifier, date="{:%Y%m%d}".format(now) + ".csv",
        ),
    )

    encoding = file_format.encoding
    delimiter = file_format.sep
    decimal_point = file_format.decimal_point
    rows = []
    if not nbr_rows:
        nbr_rows = randint(1, max_rows)
    for i in range(nbr_rows):
        values = []
        for column in file_format.columns:
            ctype = column.data_type.lower()
            custom_format = column.format
            # If you have custom format, please replace the values below
            if custom_format == "DD/MM/YYYY" or custom_format == "DD/MM/YYYY-WIDE":
                values.append(random_date("{:%d/%m/%Y}"))
            elif custom_format == "DD/MM/YYYY HH:mm:ss":
                values.append(random_date("{:%d/%m/%Y %H:%M:%S}"))
            elif custom_format == "INTEGER_SPACED":
                values.append(random_int(custom_column="INTEGER_SPACED"))
            elif custom_format == "DECIMAL_SPACED":
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point, custom_column="DECIMAL_SPACED"))
            elif custom_format:
                raise ValueError("Unknown custom format {}. "
                                 "Please add custom data creation for this format"
                                 .format(custom_format))
            elif ctype == "integer" or ctype == "bigint":
                values.append(random_int())
            elif ctype == "double" or ctype == "float":
                values.append(random_double())
            elif re.match(decimal_pattern, ctype):
                length = int(re.match(decimal_pattern, ctype).group("length"))
                precision = int(re.match(decimal_pattern, ctype).group("precision"))
                values.append(random_double(length, precision, decimal_point))
            elif ctype == "date":
                values.append(random_date("{:%Y-%m-%d}"))
            elif ctype == "datetime":
                values.append(random_date("{:%Y-%m-%d %H:%M:%S}"))
            elif ctype == "string":
                values.append(random_string())
            elif ctype == "boolean":
                values.append(random_boolean())
            else:
                raise ValueError("Unknown type {}".format(ctype))
        rows.append(delimiter.join(values))
    rows.append("")
    scheme, netloc, url, params, query, fragment = urlparse(gz)
    if scheme in ['file', '']:
        gz_url = url
    else:
        gz_url = azfr_fsspec_utils.join('.', azfr_fsspec_utils.basename(gz))

    with azfr_fsspec_utils.open(gz_url, 'wb') as f:
        if file_format.header == True:
            headers = [column.name_in_header or column.name for column in file_format.columns]
            lines = delimiter.join(headers) + "\n" + "\n".join(rows)
        else:
            lines = "\n".join(rows)
        f.write(lines.encode(encoding))

    if scheme not in ['file', '']:
        full_path = urlunparse([scheme, netloc, url, params, query, fragment])
        azfr_fsspec_utils.move(gz_url, full_path)
    return gz


def generate_zipped_file(folder, file_name, file_specs, file_identifier, formats, epoch, nbr_rows=None):
    now = dt.fromtimestamp(epoch)
    files = []
    for table_name in file_specs[file_identifier].tables:
        file = generate_test_file(file_identifier, formats[table_name], folder, file_name, epoch, nbr_rows)
        files.append(file)
    zip_name = file_name.format(file_identifier=file_identifier, date="{:%Y%m%d}".format(now)) + ".zip"
    zip_path = azfr_fsspec_utils.join(folder, zip_name)
    with FsspecZipFile(zip_path, "w") as z:
        for file in files:
            z.write(file, arcname=azfr_fsspec_utils.basename(file))
            azfr_fsspec_utils.remove(file)
    return zip_path


def generate_test_files(files_configs, folder, file_name, formats, epoch, nbr_rows=None):
    files_generated = []
    file_identifiers = list(set([formats[table].file_identifier for table in formats]))
    for file_identifier in file_identifiers:
        files_generated.append(generate_zipped_file(folder, file_name, files_configs,
                                                    file_identifier, formats, epoch, nbr_rows))
    return files_generated

```

###### FILE: tests/test_reporting/test_functional.py ######

```py
# coding: utf-8
from datetime import datetime as dt
from workflow.reporting import reporting_flow
from azfr_parsing_utils.csv import CsvColumn
from tests.test_reporting import empty_config_dict, empty_mail_config, file_specs
# coding: utf-8
import azfr_fsspec_utils as fspath
from workflow.config import ReportingConfig
from tests.test_reporting.data import test_file_generator
from deltalake import DeltaTable
import datetime


def test_main_task_one_file(loader):
    """ fast one file test"""
    date = dt.now()
    nbr_rows = 2

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }
    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict["file_pattern"] = file_specs["reporting"]["pattern"]
    test_config = ReportingConfig(**config_dict)

    state = reporting_flow(test_config, empty_mail_config, return_state=True)
    assert state.is_completed()


def test_main_task(loader):
    now = dt.now()
    dates = [now - datetime.timedelta(days=1), now]
    nbr_rows = 25

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }
    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict["file_pattern"] = file_specs["reporting"]["pattern"]
    test_config = ReportingConfig(**config_dict)

    tables = test_config.tables
    for date in dates:
        test_file_generator.generate_test_files(test_config.files_configs,
                                                test_config.input_dir,
                                                file_specs["reporting"]["file_name"],
                                                test_config.tables_to_format,
                                                dt.timestamp(date),
                                                nbr_rows)

    reporting_flow(test_config, empty_mail_config)

    expected_versions = ["__version__=" + date.strftime("%Y%m%d") for date in dates]
    expected_versions.sort()

    for table in tables:
        file_format = test_config.tables_to_format[table]
        if file_format.parsed_name:
            table_name = file_format.parsed_name.lower()
        else:
            table_name = table.lower()
        delta_table = DeltaTable(fspath.abspath(fspath.join(test_config.parsed_dir, file_format.extraction_type, table_name)))
        output_versions = [fspath.dirname(item) for item in delta_table.files()]
        output_versions.sort()
        assert output_versions == expected_versions
        assert delta_table.to_pyarrow_table().shape[0] == nbr_rows*len(dates)

    # Empty rerun
    empty_run = reporting_flow(test_config, empty_mail_config, return_state=True)
    assert empty_run.is_completed()

def test_overwrite_schema(loader):
    """Test that a column can be added when overwrite_schema is set to True"""
    now = dt.now()
    yest = now - datetime.timedelta(days=1)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "unzipped_dir": loader.dest("unzipped"),
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    for key in config_dict:
        if "_dir" in key:
            path = config_dict[key]
            fspath.makedirs(path, exist_ok=True)

    config_dict[
        "file_pattern"] = file_specs["reporting"]["pattern"]
    test_config = ReportingConfig(**config_dict)
    test_config.overwrite_schema = True

    # Standard run
    test_file_generator.generate_test_files(test_config.files_configs,
                                            test_config.input_dir,
                                            file_specs["reporting"]["file_name"],
                                            test_config.tables_to_format,
                                            dt.timestamp(yest))
    reporting_flow(test_config, empty_mail_config)

    # Add new column to schema
    for clz in test_config.tables_to_format:
        test_config.tables_to_format[clz].columns.append(CsvColumn(name="TEST_OVERWRITE_SCHEMA", data_type="STRING"))

    # Generate files with new column
    test_file_generator.generate_test_files(test_config.files_configs,
                                            test_config.input_dir,
                                            file_specs["reporting"]["file_name"],
                                            test_config.tables_to_format,
                                            dt.timestamp(now))

    reporting_flow(test_config, empty_mail_config)

    # Ensure new column is included in deltatable
    for table in test_config.tables_to_format:
        file_format = test_config.tables_to_format[table]
        if file_format.parsed_name:
            table_name = file_format.parsed_name.lower()
        else:
            table_name = table.lower()
        delta_table = DeltaTable(fspath.abspath(fspath.join(test_config.parsed_dir, file_format.extraction_type, table_name)))
        expected_cols = [item.name for item in file_format.columns] + ["__version__", "__functional_date__", "__metadata__"]
        expected_cols.sort()
        output_cols = [item.name for item in delta_table.schema().fields]
        output_cols.sort()
        assert expected_cols == output_cols
```

###### FILE: tests/test_reporting/test_unit_task.py ######

```py

# coding: utf-8
import azfr_fsspec_utils
import datetime
import os
import pytest
import yaml
from workflow.config import ReportingConfig
from datetime import datetime as dt
from workflow.parse.archive import archive_file
from workflow.parse.parse import parse_file
from workflow.parse.unzip import unzip
from prefect.logging import disable_run_logger
from tests.test_reporting.data import test_file_generator
from tests.test_reporting import file_specs, empty_config_dict
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata


def test_task_archive(loader):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": "",
        "rejected_dir": "",
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = ReportingConfig(**config_dict)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")
    landing_files = test_file_generator.generate_test_files(test_config.files_configs,
                                                            test_config.input_dir,
                                                            file_specs["reporting"]["file_name"],
                                                            test_config.tables_to_format,
                                                            epoch)

    archive_folder = azfr_fsspec_utils.abspath(azfr_fsspec_utils.join(test_config.archive_dir, date))

    with disable_run_logger():
        for landing_file in landing_files:
            archive_file.fn(landing_file, archive_folder)

    for path in landing_files:
        assert azfr_fsspec_utils.exists(
            azfr_fsspec_utils.join(
                test_config.archive_dir,
                date,
                azfr_fsspec_utils.basename(path),
            ),
        )
    files_rested_input_dir = azfr_fsspec_utils.listdir(test_config.input_dir)
    assert len(files_rested_input_dir) == 0


def test_task_unzip(loader):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": "",
        "archive_dir": loader.dest("archive"),
        "parsed_dir": "",
        "rejected_dir": "",
        "unzipped_dir": loader.dest("unzipped"),
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = ReportingConfig(**config_dict)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")

    archive_folder = azfr_fsspec_utils.abspath(azfr_fsspec_utils.join(test_config.archive_dir, date))

    paths = test_file_generator.generate_test_files(test_config.files_configs,
                                                    archive_folder,
                                                    file_specs["reporting"]["file_name"],
                                                    test_config.tables_to_format,
                                                    epoch)
    with disable_run_logger():
        for path in paths:
            unzip.fn(path, test_config.unzipped_dir)

    assert len(azfr_fsspec_utils.listdir(test_config.unzipped_dir)) == len(test_config.tables)
    assert all([f.endswith(".csv") for f in azfr_fsspec_utils.listdir(test_config.unzipped_dir)])


@pytest.mark.parametrize("table", file_specs["reporting"]["tables"])
def test_task_parse(loader, table):
    now = dt.now()
    epoch = dt.timestamp(now)

    config_dict = {
        "workflow_name": empty_config_dict["workflow_name"],        
        "input_dir": loader.dest("landing"),
        "archive_dir": loader.dest("archive"),
        "parsed_dir": loader.dest("parsed"),
        "rejected_dir": loader.dest("rejected"),
        "file_pattern": "",
        "metadata_dir": loader.dest("_metadata"),
        "files_configs": empty_config_dict["files_configs"]
    }

    test_config = ReportingConfig(**config_dict)
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(test_config, write_start_event=False)
    azfr_fsspec_utils.makedirs(test_config.input_dir, exist_ok=True)
    azfr_fsspec_utils.makedirs(test_config.rejected_dir, exist_ok=True)

    today = datetime.date.today()
    date = today.strftime("%Y%m%d")
    archived_dir = azfr_fsspec_utils.join(test_config.archive_dir,
                                          date)
    path = test_file_generator.generate_test_file(table,
                                                  test_config.tables_to_format[table],
                                                  archived_dir,
                                                  file_specs["reporting"]["file_name"],
                                                  epoch,
                                                  max_rows=30)

    file_format = test_config.tables_to_format[table]
    rejected_file = azfr_fsspec_utils.join(test_config.rejected_dir, f"{date}_{table.lower()}.txt")
    parsed_dir = azfr_fsspec_utils.join(test_config.parsed_dir, table.lower())

    with disable_run_logger():
        parse_file.fn(path, parsed_dir, rejected_file, file_format, date, workflow_metadata)

    outputs = azfr_fsspec_utils.listdir(
        azfr_fsspec_utils.join(
            test_config.parsed_dir, table.lower()
        )
    )
    expected = ['_delta_log', "__version__=" + date].sort()
    assert expected == outputs.sort()

```

###### FILE: workflow/__init__.py ######

```py
import azfr_parsing_utils.azure

azfr_parsing_utils.azure.use()

```

###### FILE: workflow/config.py ######

```py
import re
from pydantic import model_validator
from workflow.utils import map_tables_to_format, CustomCsvFileFormat
from azfr_skywalker_utils.metadata.parsing.workflowconfig import WorkflowBaseConfig
import os


CURRENT_DIR = os.path.abspath(os.path.dirname(__file__))
REPORTING_SCHEMA_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../model/format/reporting"))
MIGRATION_SANTE_SCHEMA_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../model/format/migration_sante"))
REF_PREVOYANCE_SCHEMA_DIR = os.path.abspath(os.path.join(CURRENT_DIR, "../model/format/ref_prevoyance"))


class WorkflowConfig(WorkflowBaseConfig):
    parsed_dir: str
    rejected_dir: str
    unzipped_dir: str = "unzipped"
    metadata_dir: str
    # Move in deploy
    date_regex: re.Pattern = re.compile(r"^\d{8}$")
    date_format: str = "%Y%m%d"
    time_format: str = "%H:%M"
    validate_files: bool = True
    overwrite_schema: bool = False
    retention_hours: int = 744  # 31 days by default

    @property
    def tables(self):
        return list(self.tables_to_format.keys())

    @property
    def file_pattern(self):
        return re.compile(self.file_str_pattern)
    
    def to_json(self, exclude={"tables_to_format"}) -> str:
        return self.model_dump_json(indent=2, exclude=exclude)
    
    @model_validator(mode='after')
    def set_files_configs(self) -> 'WorkflowConfig':
        tables_to_format = self.tables_to_format
        files_configs = self.files_configs if hasattr(self, 'files_configs') else {}
        table_map = {file_identifier: set() for file_identifier in files_configs}

        for table in tables_to_format:
            file_identifier = tables_to_format[table].file_identifier
            table_map[file_identifier].add(table)

        for file_identifier, tables in table_map.items():
            files_configs[file_identifier].tables = list(tables)

        self.files_configs = files_configs
        return self
class ReportingConfig(WorkflowConfig):
    tables_to_format: dict = map_tables_to_format(REPORTING_SCHEMA_DIR, CustomCsvFileFormat)

class MigrationSanteConfig(WorkflowConfig):
    tables_to_format: dict = map_tables_to_format(MIGRATION_SANTE_SCHEMA_DIR, CustomCsvFileFormat)

class RefPrevoyanceConfig(WorkflowConfig):
    tables_to_format: dict = map_tables_to_format(REF_PREVOYANCE_SCHEMA_DIR, CustomCsvFileFormat)


```

###### FILE: workflow/migration_sante.py ######

```py
from azfr_skywalker_utils.utils.mail import EmailConfig

from workflow.config import MigrationSanteConfig
from workflow.parse.processing import processing_flow
from prefect import flow


@flow
def migration_sante_flow(config: MigrationSanteConfig, email_config: EmailConfig):

    processing_flow(config, email_config, return_state=True)
```

###### FILE: workflow/parse/archive.py ######

```py
from prefect import task, get_run_logger
import azfr_fsspec_utils as fspath


@task(task_run_name="archive_file({source_file})")
def archive_file(source_file, out_dir):
    """
    Move source_file to out_dir
    :param source_file: List of path to the files to archive
    :param out_dir: Folder that will contain archived data
    :return: Path to file that has been archived
    """
    logger = get_run_logger()
    try:
        logger.info("Archive file", extra={"input_src": str(source_file),
                                                 "archive_dest": out_dir})
        fspath.makedirs(fspath.dirname(source_file), exist_ok=True)
        archived_file = _archive(source_file, out_dir)
        return archived_file
    except Exception as e:
        logger.error("Failed to archive : " + str(e))
        if source_file:
            _rollback(out_dir, fspath.dirname(source_file))
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
        for file in files:
            landing_file = fspath.join(
                dest_dir, fspath.basename(file),
            )
            logger.info(
                "Roll back table from archive",
                extra={
                    "landing_path": landing_file,
                },
            )
            if not fspath.exists(landing_file):
                fspath.move(file, landing_file)


def _archive(file, archive_path) -> str:
    """
    Move file to archive_path
    :param file: Path to file
    :param archive_path: Archive directory
    """
    fspath.makedirs(archive_path, exist_ok=True)
    dest = fspath.join(archive_path, fspath.basename(file))
    if fspath.exists(dest):
        if not fspath.exists(file):
            return dest  # archived already
        else:
            fspath.remove(dest)  # un-atomic file, delete
    fspath.move(file, dest)
    return dest

```

###### FILE: workflow/parse/clean.py ######

```py
from prefect import task
import azfr_fsspec_utils as fspath


@task(task_run_name="clean({dir_to_clean})")
def clean(dir_to_clean: str) -> None:
    """
    Remove recursively the directory that needs to be clean
    :return:
    """
    fspath.remove(dir_to_clean, recursive=True)


```

###### FILE: workflow/parse/parse.py ######

```py
from datetime import datetime
from pathlib import Path
from prefect import task, get_run_logger
from azfr_parsing_utils.csv.polars import parsed_csv_data, ColumnParser
from azfr_parsing_utils.deltalake import write_deltalake
from azfr_parsing_utils.utils import checkpoint
from azfr_skywalker_utils.utils.format_extension import CsvFileFormatExtended
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata

from workflow.utils import rawgencount
import azfr_fsspec_utils as fspath
import polars as pl


# DECIMAL_SPACED is done this way to handle natural numbers ('1','2','3', ...)
column_parsers = {
        "DD/MM/YYYY": ColumnParser(
                            test=lambda c: c.str.strptime(pl.Date,  format="%d/%m/%Y", strict=False).is_not_null(),
                            value=lambda c: c.str.strptime(pl.Date, format="%d/%m/%Y")
                        ),
        "YYYY-MM-DD": ColumnParser(
                            test=lambda c: c.str.strptime(pl.Date,  format="%Y-%m-%d", strict=False).is_not_null(),
                            value=lambda c: c.str.strptime(pl.Date, format="%Y-%m-%d")
                        ),                        
        "DD/MM/YYYY HH:mm:ss": ColumnParser(
                            test=lambda c: c.str.strptime(pl.Datetime, format="%d/%m/%Y %H:%M:%S", strict=False).is_not_null(),
                            value=lambda c: c.str.strptime(pl.Datetime, format="%d/%m/%Y %H:%M:%S").dt.replace_time_zone("Europe/Paris")
                        ),
        "INTEGER_SPACED": ColumnParser(
                            test=r"^[\d\ ]*$",
                            value=lambda c: c.str.replace_all(" ", "")
                        ),
        "DECIMAL_SPACED": ColumnParser(
                            test=r"^(-?[\d\ ]+(,[\d]+)?)?$",
                            value=lambda c: pl.when(c.is_not_null() & c.str.contains(",").not_()).then(pl.concat_str(c, pl.lit("."))).otherwise(c).str.replace_all(" ", "").str.replace_all(",", ".")
                        ),
    }


@task(task_run_name="parse_file({input_file}, {version})")
def parse_file(input_file: str, out_dir: str, rejected_file: str,
               file_format: CsvFileFormatExtended, version: str, 
               workflow_metadata: WorkflowMetadata, overwrite_schema: bool = False) -> tuple[int, int]:

    """
    Parse data from input_file according to file_format in out_dir partitioned by version.
    Data that does not match expected regex from column parsers is rejected and written in rejected_file.
    :param input_file: Path to the file to parse
    :param file_format: CsvFileFormat object that contains the format of input_file
    :param out_dir: Directory where the parquet will be written
    :param version: Functional date of parsed data
    :param rejected_file: Path to the file where the rejected data will be written
    :param mode: Mode that will be used for writing deltalake
    :param compression: compression mode
    :param partition_filters: Optional list of tuples to filters when overwriting
    :param overwrite_schema: If True, allows updating the schema of the table
    """
    logger = get_run_logger()
    # technical metadata
    file_name = Path(input_file).name
    additional_columns, metadata = workflow_metadata.create_additional_columns(version, file_name)

    # Check if input columns in header match expected columns
    if file_format.header:
        with fspath.open(input_file, encoding=file_format.encoding) as f:
            input_columns = [col.replace("\n", "") for col in f.readline().split(file_format.sep)]
        expected_input_columns = [col.name_in_header or col.name for col in file_format.columns]
        if expected_input_columns != input_columns:
            raise ValueError(
                f"Columns in input_file don't match expected columns {expected_input_columns} != {input_columns}")

    with parsed_csv_data(source=input_file, format=file_format,
                         column_parsers=column_parsers,
                         rejected_file=rejected_file,
                         additional_columns=additional_columns,
                         transform=lambda df: df.with_columns(metadata.alias("__metadata__"))
                         ) as parsed:
        with checkpoint(parsed) as result:
            # Check if nbr of rows parsed is equal to number of rows in input_file
            nbr_input_rows = rawgencount(input_file)
            nbr_parsed_rows = parsed.processed_lines + parsed.rejected_lines
            if nbr_parsed_rows != nbr_input_rows:
                raise ValueError(
                    f"Number of rows in input file: {nbr_input_rows} but number of parsed rows {nbr_parsed_rows}")

            # Check if parsed columns match expected columns
            expected_parsed_columns = [col.name for col in file_format.columns]
            parsed_columns = parsed.schema.names
            parsed_columns.remove("__functional_date__")
            parsed_columns.remove("__version__")
            parsed_columns.remove("__metadata__")
            if expected_parsed_columns != parsed_columns:
                raise ValueError(
                    f"Parsed columns does not match expected columns: {parsed_columns} != {expected_parsed_columns}")

            if parsed.rejected_lines:
                logger.warning(f"{parsed.rejected_lines} rejected lines from {file_format.name} for version {version}")

            if parsed.processed_lines == 0 and parsed.rejected_lines == 0:
                logger.warning(f"file {input_file} for version {version} is empty")

            write_deltalake(data=result, path=out_dir,
                                mode="overwrite",
                                partition_by=['__version__'],
                                partition_filters=[("__version__", "=", str(version))],
                                merge_schema=overwrite_schema,
                                overwrite_schema=overwrite_schema)
            
            return parsed.processed_lines, parsed.rejected_lines

```

###### FILE: workflow/parse/processing.py ######

```py
from datetime import datetime
from prefect import flow, get_run_logger

import azfr_fsspec_utils as fspath
import azfr_parsing_utils.azure
from azfr_parsing_utils.utils import get_files
from azfr_skywalker_utils.utils.date import get_now_UTC
from azfr_skywalker_utils.metadata.parsing import register_missing_files
from azfr_skywalker_utils.metadata.parsing.metadata import WorkflowMetadata
from azfr_skywalker_utils.utils.mail import EmailConfig
from azfr_skywalker_utils.metadata.parsing import (
    FileDetailedStatus,
    FileStatus,
)
from azfr_skywalker_utils.utils.archive import archive_file, clean
from azfr_skywalker_utils.metadata.parsing.analyzer import PlainFileAnalyser

from workflow.config import WorkflowConfig
from workflow.parse.parse import parse_file
from workflow.parse.unzip import unzip

azfr_parsing_utils.azure.use()


@flow
def processing_flow(config: WorkflowConfig, email_config: EmailConfig):
    logger = get_run_logger()
    logger.info(f"using config: {config.to_json()}")
    workflow_metadata = WorkflowMetadata()
    workflow_metadata.start(config)
    landing_files = get_files(config.input_dir, config.file_pattern)
    logger.info("Files detected from {} : {}".format(str(config.input_dir),
                                                     str(landing_files)))
    if config.validate_files:
        # Keep only dates from landing that are valid
        analyzer = PlainFileAnalyser(workflow_metadata=workflow_metadata, config=config)
        analyzer.analyze_landing_files(landing_files, check_mismatch_tables=False)
        valid_landing_files = analyzer.valid_landing_files
        logger.info("Files that are parsed are validated")
    else:
        valid_landing_files = landing_files
        logger.info("File validation is disabled !")
    for landing_file in valid_landing_files:
        archive_and_parse(config, landing_file, analyzer, return_state=True)
    if config.validate_files:
        register_missing_files(config.files_configs, config.metadata_dir, workflow_metadata, datetime.now(),
                               config.date_format, config.time_format)
    workflow_metadata.end()
    if config.validate_files and analyzer.errors:
        raise RuntimeError(f"List of Errors for this run: {str(analyzer.errors)}")
    
@flow(flow_run_name="archive_and_parse({landing_file})")
def archive_and_parse(config: WorkflowConfig, landing_file: str, analyzer: PlainFileAnalyser):
    logger = get_run_logger()
    archived_file = None
    decompressed_dir = None
    fullmode_empty_tables = []
    rejected_tables = []
    file_start_ts = get_now_UTC()
    file_basename = fspath.basename(landing_file)
    match = config.file_pattern.match(file_basename)
    date = match.group("date")
    file_identifier = match.group("file_identifier")
    try:
        table = file_identifier
        # Move files from input_dir to archive_dir
        archive_folder = fspath.abspath(fspath.join(config.archive_dir, date))
        archived_file = archive_file(landing_file, archive_folder)
        # Unzip the files
        decompressed_dir = fspath.join(config.unzipped_dir, table, date)
        decompressed_file = unzip(archived_file, decompressed_dir)[0]
        # Parse files
        table_start_ts = get_now_UTC()
        file_format = config.tables_to_format[table]
        rejected_file = fspath.join(config.rejected_dir, f"rejected_{date}_{table}.txt")
        parsed_name = file_format.parsed_name

        # Handle file naming for assures_contrats_gt (2 files, full & delta)
        if parsed_name:
            table_name = parsed_name.lower()
        else:
            table_name = table.lower()

        out_dir = fspath.join(config.parsed_dir, config.tables_to_format[table].extraction_type, table_name)
        state = parse_file(input_file=decompressed_file, out_dir=out_dir, rejected_file=rejected_file, file_format=file_format, version=date,
                            workflow_metadata=analyzer.workflow_metadata, overwrite_schema=config.overwrite_schema, return_state=True)

        analyzer.analyze_parse_table(state, file_identifier, table, date, archived_file, table_start_ts)
        analyzer.analyze_file_after_parse(archived_file, file_start_ts)
        fullmode_empty_tables = analyzer.parse_results[file_basename].fullmode_empty_tables
        rejected_tables = analyzer.parse_results[file_basename].rejected_tables
    except Exception:
        # file already received
        message = f"Fichier {file_identifier}-{date} : erreur technique"
        logger.info(message)
        analyzer.workflow_metadata.write_file_status(file_identifier, date, FileStatus.FAILED,
                                            FileDetailedStatus.FILE_TECHNICAL_ERROR,
                                            message, [archived_file or landing_file], file_start_ts)
        raise 
    finally:
        # Clean decompressed files if needed
        if decompressed_dir:
            clean(decompressed_dir)
    raise_exception_if(fullmode_empty_tables, "Table in full mode but empty detected.")
    raise_exception_if(rejected_tables, "Table with rejected rows detected.")


def raise_exception_if(table_list, error_message):
    if table_list:
        count = len(table_list)
        table_word = "table" if count == 1 else "tables"
        raise ValueError(f"{error_message} {count} {table_word}.")

```

###### FILE: workflow/parse/unzip.py ######

```py
import azfr_fsspec_utils as fspath
from azfr_fsspec_utils.zipfile import FsspecZipFile
from prefect import task


@task(task_run_name="unzip({input_file})")
def unzip(input_file, out_dir):
    extracted = []
    with FsspecZipFile(input_file) as f:
        csv_files = f.infolist()
        for csv_file in csv_files:
            extract_file = fspath.join(out_dir, csv_file.filename)
            extracted.append(extract_file)
            fspath.makedirs(out_dir, exist_ok=True)
            with fspath.open(extract_file, 'wb') as extracted_file:
                extracted_file.write(f.read(csv_file))
    return extracted

```

###### FILE: workflow/ref_prevoyance.py ######

```py
from azfr_skywalker_utils.utils.mail import EmailConfig

from workflow.config import RefPrevoyanceConfig
from workflow.parse.processing import processing_flow
from prefect import flow


@flow
def ref_prevoyance_flow(config: RefPrevoyanceConfig, email_config: EmailConfig):

    processing_flow(config, email_config, return_state=True)
```

###### FILE: workflow/reporting.py ######

```py
from azfr_skywalker_utils.utils.mail import EmailConfig

from workflow.config import ReportingConfig
from workflow.parse.processing import processing_flow
from prefect import flow


@flow
def reporting_flow(config: ReportingConfig, email_config: EmailConfig):

    processing_flow(config, email_config, return_state=True)
```

###### FILE: workflow/utils/__init__.py ######

```py
from typing import List, Literal, Optional
import re
import yaml
from datetime import datetime
import pytz

from pydantic import Field, TypeAdapter

from azfr_skywalker_utils.utils.format_extension import CsvFileFormatExtended
from azfr_parsing_utils.csv.format import CsvFileFormat
from azfr_parsing_utils.utils import get_files
import azfr_fsspec_utils as fspath


class CustomCsvFileFormat(CsvFileFormatExtended):
    """ Custom CSV file format """
    parsed_name: Optional[str] = Field(
        title="parsed_name",
        description="Optional name of the parsed table",
        default=None
    )


def get_now_utc():
    return datetime.now(pytz.UTC)


def map_tables_to_format(format_folder: str, ref_format=CsvFileFormat) -> dict:
    """
    Map the tables with their format extracted from yaml.
    Parameters
    ----------
    format_folder: The path of the folder containing the yaml file of the format
    Returns
    -------
    Dict containing the mapping of the table name to their corresponding CsvFileFormat object
    """
    tables_to_format = {}
    schemas = [file for file in fspath.listdir(format_folder) if file.endswith(".yaml")]
    for file in schemas:
        schema_path = fspath.abspath(fspath.join(format_folder, file))
        with fspath.open(schema_path) as f:
            file_format = TypeAdapter(ref_format).validate_python(yaml.load(f, yaml.FullLoader))
            tables_to_format[file_format.name] = file_format
    return tables_to_format


def is_containing_file_for_table(dir_to_check: str, table: str, file_pattern: re.Pattern) -> bool:
    """
    Check if dir contain files that match pattern for table
    Parameters
    ----------
    dir_to_check: The path of the folder where pattern must be searched
    table: Name of table to check
    file_pattern: Pattern to search
    Returns
    -------
    Boolean. True if directory contains any file with given pattern
    """
    for file in fspath.listdir(dir_to_check):
        if file_pattern.match(file):
            if file_pattern.match(file).group("name").upper() == table.upper():
                return True
    return False


def get_files_for_table(path: str, table: str, pattern: re.Pattern) -> List[str]:
    """Get a list of files from the given path which match the given pattern and the given date.
    Parameters
    ----------
    path: The path of the folder.
    table: The given name used to match files' names.
    pattern: The file pattern.
    Returns
    -------
    The list of files ( absolute pah ) which correspond to the given date and the given pattern.
    """
    files = get_files(path, pattern)
    if not files:
        return []
    return [
        f for f in files
        if pattern.match(fspath.basename(f)).group('name').upper() == table.upper()
    ]


def make_gen(reader):
    """
    Make a generator from reader
    """
    b = reader(1024 * 1024)
    while b:
        yield b
        b = reader(1024*1024)


def rawgencount(file):
    """
    Count the number of new lines in file
    """
    with fspath.open(file, 'rb') as f:
        f_gen = make_gen(f.raw.read)
        # Remove 1 because of empty line at the end of the file
        return sum(buf.count(b'\n') for buf in f_gen)-1
```

