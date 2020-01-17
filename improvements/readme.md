# Improvements

This self-service is perfectly working but here some ideas I did not implement here to keep simple and that you could add to your process:

- you can retrieve the creator information (name and email) inside Microsoft Flow and you can send him an email telling him that its project is now ready. It can also send a slack/teams message
- Ticketing could be done in an external ticketing service like ServiceNow
- you could have different releases and trigger the right one depending on the type of the desired project by the creator. the IF ELSE control can be done in Microsoft Flow (and thus having several pipelines in AzDO), or in AzDO and having conditional tasks in a single pipeline
- some actions are not possible through the CLI (yet), you may have to use the API.
- it's possible to create YAML file in the template repo which can then be used to create a default pipeline in the project. very useful for teams which do not know well Azure DevOps
