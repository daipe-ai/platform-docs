# Creating project from skeleton

Create a **new Daipe project** by running the following command:

```
curl -s https://raw.githubusercontent.com/daipe-ai/project-creator/master/create_project.sh | bash -s skeleton-databricks
```

!!! info "Prerequisites"
    The following software needs to be installed first:

      - [Miniconda package manager](https://docs.conda.io/en/latest/miniconda.html)
        - **IMPORTANT!** - To avoid Anaconda's [Terms of Service](https://www.anaconda.com/terms-of-service) run:
            - `conda config channels --remove defaults`
            - `conda config channels --append conda-forge`
        - This sets up a community-driven [conda-forge](https://conda-forge.org/) as the only conda repository.
      - [Git for Windows](https://git-scm.com/download/win) or standard Git in Linux (_apt-get install git_)
      
    We recommend using the following IDEs:
    
      - [PyCharm Community or Pro](https://www.jetbrains.com/pycharm/download/) with the [EnvFile plugin](https://plugins.jetbrains.com/plugin/7861-envfile) installed
      - [Visual Studio Code](https://code.visualstudio.com/download) with the [PYTHONPATH setter extension](https://marketplace.visualstudio.com/items?itemName=datasentics.pythonpath-setter) installed

    Tu run commands, use **Git Bash** on Windows or standard Terminal on **Linux/Mac**

!!! summary "What does the command do?"
    1. Asks for project & directory name of your new project 
    2. Download the [Daipe project skeleton template](https://github.com/daipe-ai/skeleton-databricks)
    3. Create the new project skeleton based on the template
    4. Runs the Daipe [development environment initialization script](https://github.com/daipe-ai/benvy)
