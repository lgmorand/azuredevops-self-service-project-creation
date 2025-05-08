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
$projectAdminGroup = az devops security group list --project test1 --query "graphGroups[?displayName=='Administrattors'].descriptor" | Out-String

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

## Custom role (optional)

Referential

|Key| Bit flag | Role name dans l'interface|
|-----|-----|------|
| GENERIC_READ |1 | View project-level information |
| GENERIC_WRITE | 2 |Edit project-level information |
| DELETE | 4| Delete team project |
| PUBLISH_TEST_RESULTS | 8 | Create test runs|
| ADMINISTER_BUILD | 16 |Administer a build |
| START_BUILD | 32|Start a build  |
| EDIT_BUILD_STATUS | 64|Edit build quality   |
| UPDATE_BUILD | 128|Write to build operational store|
| DELETE_TEST_RESULTS | 256|Delete test runs|
| VIEW_TEST_RESULTS | 512|View test runs|
| MANAGE_TEST_ENVIRONMENTS | 2048| Manage test environments|
| MANAGE_TEST_CONFIGURATIONS | 4096| Manage test configurations|
| WORK_ITEM_DELETE | 8192| Delete and restore work items|
| WORK_ITEM_MOVE | 16384| Move work items out of this project|
| WORK_ITEM_PERMANENTLY_DELETE | 32768| Permanently delete work items|
| RENAME | 65536| Rename team project |
| MANAGE_PROPERTIES | 131072 | Manage project properties|
| MANAGE_SYSTEM_PROPERTIES | 262144|Manage system project properties |
| BYPASS_PROPERTY_CACHE | 524288|Bypass project property cache |
| BYPASS_RULES | 1048576 | Bypass rules on work item updates |
| SUPPRESS_NOTIFICATIONS | 2097152 | Suppress notifications for work item updates |
| UPDATE_VISIBILITY | 4194304| Update project visibility |
| CHANGE_PROCESS | 8388608 | Change process of team project|
| AGILETOOLS_BACKLOG | 16777216|Agile backlog management|
| DELIVERY_PLANS |33554432|Manage delivery plans|

```bash
az devops security groupe create --name $projectName --description 'blablalba with CPM ID' --project $projectName

# retrieve the ID of the admin group of the new created project
$projectDescriptor = az devops security group list --project $projectName  --query "graphGroups[?displayName=='$groupName'].descriptor" | ConvertFrom-Json
$projectId = az devops project show --project $projectName --query "id" | ConvertFrom-Json

# check if the descriptiof is correct
az devops security group show --id $projectDescriptor

# list permisssions of the group
$token = '$PROJECT:vstfs:///Classification/TeamProject/'+$projectId
# namespace-id for project collection (list is documented here: https://learn.microsoft.com/fr-fr/azure/devops/organizations/security/namespace-reference?view=azure-devops)
az devops security permission list --namespace-id '52d39943-cb85-4d7f-8fa8-c6baac873819' --subject $projectDescriptor --token $token --output table

# add permissions to the group
# either you can add permission one by one (using merge=true parameter) or you can add all permissions at once (combining big flags)
az devops security permission update --namespace-id '52d39943-cb85-4d7f-8fa8-c6baac873819' --subject $projectDescriptor --token $token --allow-bit 7  --merge false --output table
az devops security permission update --namespace-id '52d39943-cb85-4d7f-8fa8-c6baac873819' --subject $projectDescriptor --token $token --deny-bit 16 --merge false --output table

# add member to the group
az devops security group membership add --group-id $projectDescriptor --member-id 'user@email.com'

```

## Next steps

It's now time to [process our tickets](../processing/readme.md).
