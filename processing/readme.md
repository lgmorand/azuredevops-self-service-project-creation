# Processing tickets

Now that ticketing process is ready and release exist, we just need to plug them together.

To be transparent, in this example, once the demand to create a new project is created, we'll process it directly. **In a real world, you could have a workflow in order to say that only tickets in a certain state can be processed.** The workflow could be in the workitem process itself (only process tickets with "Accepted" status), could be done in Microsoft Flow, maybe calling external service (like teams, ServiceNow, etc.), or even in the release itself.

## Leveraging Microsoft Flow

To my knowledge, there is no way to trigger a script or a pipeline in Azure DevOps **from** the creation or the update of a workitem. You can use webhook to post a message in a Teams/Slack channel but triggering a pipeline is something totally different. Moreover, I'd like to be able to pass dynamic parameters to the build.

My idea is to use [Microsoft Flow](https://flow.microsoft.com) because it's cheap, totally serverless and very quick to set up. It'll take less than 60 secondes to build the workflow I need.

## Building the workflow

First, create a Microsoft Flow account and create a new **empty** flow. Secondly, we need to look for the Azure DevOps connector and especially the trigger when a workitem is created (or updated if you want to have a workflow first in Azure DevOps, like validating manually the tickets before getting them processed).

![Create trigger](./media/processing%201.png)

Then fill the minimum fields: organization, project and the work item we created: *Demand*. From that simple "box", the workflow will be triggered and most of the fields of the workitem, will be retrieved.

![Retrieve data](./media/processing%202.png)

Then add a second step with the task **Azure DevOps: trigger a release** and the select the release we just created.

![trigger release](./media/processing%203.png)

That's ALL ! If everything was configured properly, whenever you create a new *Demand* ticket, after few seconds you should see in Microsoft Flow a iteration of your flow and if everything went fine, you should discover a new project in your Azure DevOps organization.

It's not perfect yet and I listed here some ideas of [improvments](../creation/readme.md).
