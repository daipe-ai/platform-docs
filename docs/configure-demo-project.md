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