# Clone Daipe Project

!!! info "Prerequisites"
      - Enable 'Files in Repos' in your Databricks workspace at *Settings -> Admin Console -> Workspace Settings*
      - Set up a GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
         - In your Databricks workspace at *Settings -> User Settings -> Git Integration* select GitHub as a provider and use your new token here

1. Under Repos open your personal folder and press "Add Repo":
![](images/create-daipe-step1.png)
    - Enter your project HTTPS Clone Url and confirm

2. Run notebook at *src/daipeproject/app/bootstrap* to validate your setup
![](images/create-daipe-step2.png)
