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

### Example: Adding or updating a project dependency

#### Adding a dependency

Suppose we are developing some notebook in Databricks and now we need some new python package (dependency), e.g. `scipy`.
In order to do that we need to follow a series of steps.
Add it to `pyproject.toml`, build master package again and upload it to Databricks.

1. To add `scipy` to `pyproject.toml` we need to run `poetry add scipy`
1. Now we don't want to do `console dbx:deploy` because it would overwrite our notebooks in Databricks.
1. Instead we want to only update master package. To do that you can use command `console dbx:deploy-master-package`
1. Now we run `%run install_master_package` cell again.
1. Now you should be able to import `scipy` module
1. __Important:__ The updated project should then be pushed to a central repository so that other team members can pull it and have the same dependencies.
1. The dependencies are installed __automatically__ after running `git pull`

#### Updating a dependency

Let's assume that we want to update a depency, e. g. `datalake-bundle`. We need to follow a series of steps similar to the previous case.

1. We need to check `pyproject.toml` that the dependency has the correct version defined e. g. `datalake-bundle = "^1.0"` will only update to versions `1.*` but not to `2.*` or higher.
1. Then we run `poetry update datalake-bundle`. __DO NOT__ run `poetry update` without an argument. It updates all packages which might break the entire project.
1. For now on we follow the steps 3 - 6 from the previous example. We use command `console dbx:deploy-master-package`
1. Now run `%run install_master_package` cell again.
1. Now you should be able to use the updated `datalake-bundle` module
1. __Important:__ The updated project should then be pushed to a central repository so that other team members can pull it and have the same dependencies.
1. The dependencies are installed __automatically__ after running `git pull`