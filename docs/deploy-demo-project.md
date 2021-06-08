# Deploy to Databricks

Now you can push your project to Databricks Workspace using following command

```bash
$ console dbx:deploy
```

And you should see the pushed project in Databricks workspace

![](images/pushed_daipedemo_project.png)


into the **Environment Variables**.

**Important scripts:**

1. ```poe flake8``` - checks coding standards
1. ```poe container-check``` - check app container consistency (if configured properly)
