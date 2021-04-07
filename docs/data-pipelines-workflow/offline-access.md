# Offline Databricks access

## Introduction

This section is maninly for those who are working in some strict environment where you may not have full internet access.

However there are couple of assumptions

- Git for Windows installed
- Python 3.7 or Conda installed
- Ability to clone or download Github repository
- Ability to call Databricks API
- Databricks Personal Access Token

## Cloning repository with project skeleton

- First clone usual daipe project skeleton `git clone https://github.com/daipe-ai/skeleton-databricks.git`
- Then clone repository with offline access files `git clone https://github.com/daipe-ai/offline-access.git`
- Add files from this repository to your daipe project

**Project structure**  
The project structure is very similar to normal daipe project.

There are just some addinational files which helps you operate with minimal inretnet connection.

- `dependencies/` - directory where all the dependencies (.whl files) are stored so we don't have to install from PyPI
- `.poetry/` - directory where poetry package manager is stored so we don't have to install it from internet
- `env-init-offline.sh` - offline environment initialization script
- `activate.sh` - environment activation script if you don't want to use conda based environment
- `deactivate.sh` - environment deactivation script
- `azure-pipelines.yml` - simple offline devops pipeline

**Note**  
In this skeleton project we included dependencies for Windows/Linux and Python 3.7. So we are able to develop bricksflow project on local Windows machine and also use it on some ci/cd Linux agent. Also very important thing is that our target Databricks runtime is DBR 7.5 which is Linux with Python 3.7. If you want to make some changes, e.g. add some python package it's your responsibility to add appropriate wheels in the `dependencies/` direcotry.

## Configuration
There are two new configuration options in `src/__myproject__/_config/bundles/dbxdeploy.yaml`

- offline_install - which tells dbx-deploy to upload approptiate wheels to dbfs and install them offline from here
- dependencies_deploy_path - path where wheels are uploaded, default `dbfs:/FileStore/jars/__myproject__/dependencies/dependencyName.whl`

```yaml
parameters:
  dbxdeploy:
    target:
      package:
        offline_install: True
        dependencies_deploy_path: '%dbxdeploy.target.package.baseDir%/{packageName}/dependencies/{packageFilename}'
```

## Local environment initialization

Environment initialization is basically same as in normal daipe project. Just use offline init script `./env-init-offline.sh`.

Set your development Databricks URL in `src/__myproject__/_config/config_dev.yaml`.

Create `.env` file from `.env.dist` and set your Databricks Personal Access Token here.

There is a slight difference in activating/deactivating python virtual environment if you don't want to use conda.

- `source activate.sh`
- `source deactivate.sh`

After activating virtual environment you should be able to run standard bricksflow commands like `console dbx:deploy`.

**Edge case**  
One edge case we run into in one very strict environment is that we were not able to run `console` command because it is an executable and only defined set of executables was allowed to run. To avoid this issue we can run console command in this way `python .venv/Lib/site-packages/consolebundle/CommandRunner.py dbx:deploy`. To make life easier we can add following line in the `.bashrc` - `alias console='python .venv/Lib/site-packages/consolebundle/CommandRunner.py'`. Be aware that this way the `console` command will work only from project root.

## How to get dependencies

**Databricks dependencies**

To get dependencies that are needed to run application on databricks you can use command `console dbx:build-dependencies`. This command will build dependencies on remote Databricks cluster and the download them in your local `dependecies/` folder. You can configure on which databricks workspace and cluster you want to build dependencies in dbx-deploy config.

```yaml
parameters:
  dbxdeploy:
    target:
      package:
        build:
          databricks:
            host: '%dbxdeploy.databricks.host%'
            token: '%dbxdeploy.databricks.token%'
            job_cluster_definition:
              spark_version: '7.3.x-scala2.12'
              node_type_id: 'Standard_DS3_v2'
              num_workers: 1
```

**Note 1**  
To build the dependencies the cluster must have internet access. We assume that if you want to build dependecies with ease you will have some non-productional Databricks workspace with internet access.

**Note 2**  
Dependencies built with `console dbx:build-dependencies` are just dependencies that are needed to run application itself excluding development dependencies like `flake8` etc. If you also want to build development dependecies you can pass `--dev` flag. Dependencies built this way should be runable on most linux platforms with appropriate python version that was used in Databricks runtime.

**Local dependencies**

To get local dependencies that are specific to your platform you can use this sequence of commands.

```
poetry export --dev --without-hashes -o dev-requirements.txt
python -m pip wheel -r dev-requirements.txt -w dependencies/
rm dev-requirements.txt
```
