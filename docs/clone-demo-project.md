# Get the Daipe demo project

## Cloning project from repository

Cloning a **Daipe demo project** by running the following command:

```
git clone https://github.com/daipe-ai/daipe-demo-databricks.git
```

!!! info "Prerequisites"
    The following software needs to be installed first:

      - [Miniconda package manager](https://docs.conda.io/en/latest/miniconda.html)
      - [Git for Windows](https://git-scm.com/download/win) or standard Git in Linux (_apt-get install git_)
      
    We recommend using the following IDEs:
    
      - [PyCharm Community or Pro](https://www.jetbrains.com/pycharm/download/) with the [EnvFile plugin](https://plugins.jetbrains.com/plugin/7861-envfile) installed
      - [Visual Studio Code](https://code.visualstudio.com/download) with the [PYTHONPATH setter extension](https://marketplace.visualstudio.com/items?itemName=datasentics.pythonpath-setter) installed

    Tu run commands, use **Git Bash** on Windows or standard Terminal on **Linux/Mac**

**Configure the local project**

In ==src/daipedemo/_config/config_dev.yaml== fill Databricks URL.

![](images/bricks_config.png)

Create `.env` file from `.env.dist` and fill Databricks Personal Access Token.

![](images/bricks_env_file.png)

**Initialize local environment**

Now run `./env-init.sh` which will initialize your local environment.

**Activate the environment**

Now activate the Conda environment for your new project

```bash
$ conda activate $PWD/.venv
```

or use a shortcut

```bash
$ ca
```