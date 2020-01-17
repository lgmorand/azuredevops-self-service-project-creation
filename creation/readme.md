# Creation of the project

As a second step, we need now to prepare the release (or releases), responsible to handle the creation of the project.

## Create a release

Once again, we are going to leverage Azure DevOps. So, in the "global" or "governance" project, create a (release) pipeline (/!\ not a build one. Not that it changes anything but since it creates something, I consider it a release, more than a build)

![Create release](./media/creating%201.png)

Then in the *variables* tab, add some variables like the name of the project and its type.

> Don't forget to scroll to the right and check the variable to be settable at release creation

![Add variables](./media/creating%202.png)

You now have to add one PowerShell task in which we call [Azure DevOps CLI](https://docs.microsoft.com/en-us/azure/devops/cli/?view=azure-devops) commands

### Prepare the CLI

First, we need to be sure that CLI is here (it should be present by default on Hosted Agents) and then, we need to ensure the azure-devops extension is also present (which should also be the base by default)

```powershell
az extension add --name azure-devops
```

Then we need to authenticate against our organization. For that, you have to create a PAT token, and add the value in the variables of your pipeline and click on the small lock, to make it a secret (secure). You can also use [Azure Keyvault integration](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml).

```powershell
# setting up the PAT which will be used by commands
$env:AZURE_DEVOPS_EXT_PAT = '$(PAT)'

# authenticate against the desired org
az devops login --org "https://lgmorand.visualstudio.com/"
```

After that, it's time to create a project

```powershell
# create a new project
az devops project create --name $(ProjectName) --org "https://lgmorand.visualstudio.com/"

# set defaults so we don't have to repeat org and projects parameters for following commands
az devops configure --defaults organization=https://lgmorand.visualstudio.com/ project=$(ProjectName)
```

Then we want to customize the project, by creating a repo from a repo template (which you have to create yourself, it can be in Azure DevOps!), add some branching strategy, rights access and also create the first pipeline

Here is the full and final script:

```powershell
# Ensure that Azure DevOps CLI extension is installed
az extension add --name azure-devops

# setting up the PAT which will be used by commands
$env:AZURE_DEVOPS_EXT_PAT = '$(PAT)'

# authenticate against the desired org
az devops login --org "https://lgmorand.visualstudio.com/"

# create a new project
az devops project create --name $(ProjectName) --org "https://lgmorand.visualstudio.com/"

# set defaults so we don't have to repeat org and projects parameters for following commands
az devops configure --defaults organization=https://lgmorand.visualstudio.com/ project=$(ProjectName)

### Security ###
# retrieve the ID of the admin group of the new created project
$projectAdminGroup = az devops security group list --project test1 --query "graphGroups[?displayName=='Contributors'].descriptor" | Out-String

Write-Host $projectAdminGroup

# add the creator as the admin of the new created project
az devops security group membership add --group-id $projectAdminGroup --member-id '$(Creator)'

### Repository ###
# initialize the repository with a default content and retrieve the ID of the repo
$repositoryID = az repos import create --repository $(ProjectName) --git-source-url "https://github.com/lgmorand/azuredevops-defaultrepo" --query repository.id -o json | Out-String

# branch policy on master => enforce workitem linking
az repos policy work-item-linking create --blocking true --enabled true --branch master --repository $repositoryID

# branch policy on master => enforce comment resolution
az repos policy comment-required create --blocking true --enabled true --branch master --repository $repositoryID

# branch policy on master => enforce code reviewing
az repos policy approver-count create --blocking true --enabled true --allow-downvotes false --reset-on-source-push false --minimum-approver-count 1 --creator-vote-counts true --branch master --repository $repositoryID

```

When executed you get:

- a new project is created
- for which the new admin is the creator of the *Demand* ticket
- a repo is already present
- best practices for branching strategy are also in place

It's not perfect but that's a pretty good start for a scaffolded project, isn't it?

## Next steps

It's now time to [process our tickets](../processing/readme.md).
