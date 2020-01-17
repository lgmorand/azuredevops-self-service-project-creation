# Improvments

This auto-creation is perfectly working but here some ideas I did not implemented here to keep simple and that you could add to your process:

- you can retrieve the creator information (name and email) inside Microsoft Flow and you can send him an email telling him that its project is now ready. It can also send a slack/teams message
- Ticketing could be done in an external ticketing service like serviceNow
- you coud have different releases and trigger the right one depending on the type of desired project by the creator. the IF ELSE control can be done in Microsoft Flow (and thus having several pipelines in AzDO), or in AzDO and having conditional tasks in a single pipeline
- there are some actions which are not possible through the CLI, you may have to use the API.
