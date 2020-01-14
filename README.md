# Azure DevOps: Automatic creation of a new project

## Context

In large companies, there is often one central team responsible to provide services to the developers, like a software factory platform. When they are using Azure DevOps, they generally create one Azure DevOps organization and then create one project per team or per project.

## Today

There are different ways to handle the creation of the project and of course, it depends on the actual process in place in the company. Sometimes, teams just send an email, sometimes they have to create a serviceNow ticket, sometimes they need to bribe the team to get a project canvas. There is one common process:
Dev team --> new demand --> send to AzureDevOps team --> manual creation of the project

There are pros and cons in this very simple process

**pros**

- centralized demand
- possibility of implementing specific controls

**cons**

- slow project creation
- time consuming for the Azure DevOps teams 
- not very reactive (only during work hours)
- not scalable
- not "DevOps"-mindset !

## Tomorrow

What I'm discussing here is how to automate a big part of the process, from receiving the demand to create a project and at the same time scaffold some good practices.

There are of course different ways of doing it. I could implement a chat bot experience to build a conversational agent to receive the demand. I could also develop my custom self service portal. In both cases it requires custom development and for each evolution, it would require rewriting. **I would like to leverage Azure DevOps as much as possible** in order to be flexible but also in order to not require external infrastructure to maintain.

My solution is:

1. using Azure DevOps as a ticketing system (could be replaced easily by Jira, ServiceNow or anything else with an exposed Rest API)
2. using Microsoft Flow to monitor and process the tickets and create controls
3. using Azure DevOps pipeline to drive the project creation
