---
title: ServiceNow Agent Workspace or Standard Platform? Why not both?
---

The ServiceNow Agent Workspace offers a few advantages for ITSM fulfillers, such as rich text journal entries and an interface for multi-tasking. That said, it's always a challenge to get users to adopt a new tool, even if it offers an advantage. So, let them have both and make it easy to switch between! From the incident interface we can add a simple UI action to open the current record in the Agent Workspace. The example below will create a UI action button that will open the current incident in a new tab using the Agent Workspace.

# The Solution
Here is a screenshot of the UI action. Script is further down.
![]({{ 'assets/open in agent workspace ui action.PNG' | relative_url }})

The script
```
function openRecordInAgentWorkspace(){
var url = 'now/workspace/agent/record/incident/' + g_form.getUniqueValue();
g_navigation.openPopup(url);
}
```
# Result
Very simple and reusable!
![]({{ 'assets/open in agent workspace ui action_result.PNG' | relative_url }})
