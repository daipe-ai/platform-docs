# Syncing local project with Databricks

## Local -> Databricks

First activate conda environment
```bash
$ ca
```

Then create a feature branch
```bash
$ git checkout -b feature-new-great-stuff
```

Finally upload your feature branch to Databricks workspace using following command
```bash
$ console dbx:deploy --env=dev
```

Your feature branch will be deployed to the DEV Databricks workspace.

![](../images/bricks_feature_branch.png)

You can now code some awesome new features right in Databricks workspace!

## Databricks -> Local

When you're happy with what you've done, open the project folder in terminal and use the following command

```bash
$ console dbx:workspace:export --env=dev
```

This will download all Databricks notebooks from that feature branch workspace folder back to your computer.

Now you can commit your changes

```bash
$ git add .
$ git commit -m "Awesome new feature"
$ git push origin feature-new-great-stuff
```

## Updating Master Package

You have certainly noticed, that in the local bricksflow project there is a way more code than in Databricks, where only notebooks exists. It's mainly some `yaml` and `toml` configuration files which are uploaded in master package which is installed in each notebook. So if we want to make change to those files our only option is to edit them locally.

**Example**

- Suppose we are developing some notebook in Databricks and now we need some new python package (dependency), e.g. `scipy`.
- In order to do that we need to add it to `pyproject.toml`, build master package again and upload it to Databricks.
- To add `scipy` to `pyproject.toml` we need to run `poetry add scipy`
- Now we don't want to do `console dbx:deploy` because it would overwrite our notebooks in Databricks.
- Instead we want to only update master package. To do that you can use command `console dbx:deploy-master-package`
- Now to take effect you need to detach and reattach the Databricks notebook and run `%run install_master_package` cell again.
- Now you should be able to import `scipy` module
