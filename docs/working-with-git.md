# Working with GIT

## Create feature branch and work on it

One of the most common cases is that you want to create feature branch out of master and make some changes.

First activate conda environment
```bash
$ ca
```

Now pull latest changes from master branch
```bash
$ git pull origin master
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

You can now code some awesome new features right in Databricks workspace!

![](images/bricks_feature_branch.png)

## Commit your work to GIT repository

Once you're happy with what you've done in your feature branch you probably want to commit and push your changes to git repository.


First download all of your work in Databricks workspace to your local machine using following command
```bash
$ console dbx:workspace:export --env=dev
```

Now you can commit and push your changes to repository
```bash
$ git add .
$ git commit -m "Awesome new feature"
$ git push origin feature-new-great-stuff
```

## Getting in sync with same with existing feature branch folder in Databricks

If you are Data Engineer and Data Scientist provided the Databricks folder for you:

- Create a feature branch on local with the same name as the folder in databricks. 
- Use `console dbx:workspace:export` to sync the notebooks to the local
- commit the changes and push them to git service (GitHub, Devops, ...)

## Updating The Master Package

```bash
$ console dbx:deploy-master-package --env=dev
```

You have certainly noticed, that in the local Daipe project there is a way more code than in Databricks, where only notebooks exists. It's mainly some `yaml` and `toml` configuration files which are uploaded in master package which is installed in each notebook. So if we want to make change to those files our only option is to edit them locally.

**(example) Adding new project dependency**

- Suppose we are developing some notebook in Databricks and now we need some new python package (dependency), e.g. `scipy`.
- In order to do that we need to add it to `pyproject.toml`, build master package again and upload it to Databricks.
- To add `scipy` to `pyproject.toml` we need to run `poetry add scipy`
- Now we don't want to do `console dbx:deploy` because it would overwrite our notebooks in Databricks.
- Instead we want to only update master package. To do that you can use command `console dbx:deploy-master-package`
- Now to take effect you need to detach and reattach the Databricks notebook and run `%run install_master_package` cell again.
- Now you should be able to import `scipy` module
