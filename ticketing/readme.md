# Ticketing System

I could create a system working with a bot framework, based on email or even a Teams/Slack message but I like the notion of tracking the history but also because ticketing also a user-friendly way to create a demand and also is more ITIL ready.

## Creating a root project

In my case, I want to use my current organization to host all my tooling. I'll then create one project with access to all employess and use this project to host my tickets.

I won't detail how to do that but if you want to give access to all employees, the easiest way to do it if to use [AAD groups](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/manage-azure-active-directory-groups).

> Before rushing into creating the project, read the next steps first !

## Creating a ticketing workitem

I could reuse or customize an existing User Story/PBI workitem but to make it more simpler, I recommend to create a custom work item called: *Demand*

For that, you first need to create a custom [inherited process](https://docs.microsoft.com/en-us/azure/devops/organizations/settings/work/customize-process?view=azure-devops) and then you create a Demand workitem with the fields you need.

### Creating a new process

![Create process](./media/ticketing%201.png)

### Creating our custom workitem

Create a new workitem type called *Demand*, add all the fields you fill relevant and don't hesitate to use rules and options to customize the experience.
![Create item](./media/ticketing%202.png)


In my case, my *Demand* ticket has:

- A title which be automatically changed thanks to rules, depending on the value of the fields filled by the use
- A demand type : (ProjectCreation or Other)
- A name of the project: free field but mandatory
- A type of project: picklist, because I want to be able to scaffold different type of projects
- All other fields are removed/hidden when possible

Here is the final result:

![Create item](./media/ticketing%204.png)

### Creating the project

Don't forget to use the process with just created

![Create project](./media/ticketing%203.png)

That's all. Whenever someone wants to create a project, he just has to create a *User Story* workitem and fills the fields accordingly.

## Next steps

It's now time to [automate the creation of the project](../creation/readme.md).
