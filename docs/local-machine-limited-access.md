# Local machine has limited internet access

In this case we assume that local machine has limited internet access access, e.g. can't do `pip install package`.

For this case we have prepared repository with files and scripts that support this situation.

We use basic python virtual environment instead of conda, because conda needs to reach internet when creating virtual env.

You can get this repository at `https://github.com/daipe-ai/offline-access.git`
and copy following files/directories to your daipe project.

- `dependencies/` - directory where all the dependencies (.whl files) are stored so we don't have to install them from PyPI
- `.poetry/` - directory where poetry package manager is stored so we don't have to install it from internet
- `env-init-offline.sh` - offline environment initialization script
- `activate.sh` - environment activation script
- `deactivate.sh` - environment deactivation script
- `azure-pipelines.yml` - simple offline devops pipeline

You can now do `./env-init-offline.sh` which will initialize your local environment without touching the internet.

You can activate/deactivate environment using following commands

- `source activate.sh`
- `source deactivate.sh`

After activating virtual environment you should be able to run standard Daipe commands like `console dbx:deploy`.

**Note**  
In dependencies directory we included dependencies for Windows/Linux and Python 3.7. So we are able to develop Daipe project on local Windows machine and also use it on some ci/cd Linux agent. Also very important thing is that our target Databricks runtime is DBR 7.3 which is Linux with Python 3.7. If you want to make some changes, e.g. add some python package it's your responsibility to add appropriate wheels in the `dependencies/` direcotry.

**Edge case**  
One edge case we run into in one very strict environment is that we were not able to run `console` command because it is an executable and only defined set of executables was allowed to run. To avoid this issue we can run console command in this way `python .venv/Lib/site-packages/consolebundle/CommandRunner.py dbx:deploy`. To make life easier we can add following line in the `.bashrc` - `alias console='python .venv/Lib/site-packages/consolebundle/CommandRunner.py'`. Be aware that this way the `console` command will work only from project root.

## How to get dependencies

**Databricks dependencies**

To get dependencies that are needed to run application on databricks you can use command `console dbx:build-dependencies` as it was documented in first section of this page.

**Note**  
Dependencies built with `console dbx:build-dependencies` are just dependencies that are needed to run application itself excluding development dependencies like `flake8` etc. If you also want to build development dependecies you can pass `--dev` flag. Dependencies built this way should be runable on most linux platforms with appropriate python version that was used in Databricks runtime.

**Local dependencies**

To get local dependencies that are specific to your platform you can use this sequence of commands.

```
poetry export --dev --without-hashes -o dev-requirements.txt
python -m pip wheel -r dev-requirements.txt -w dependencies/
rm dev-requirements.txt
```
