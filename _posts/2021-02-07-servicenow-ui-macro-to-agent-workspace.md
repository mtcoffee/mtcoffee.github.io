---
title: ServiceNow UI Macro to Agent Workspace
---

In a [previous post](/servicenow-agent-workspace-or-standard-platform-why-not-both/) I suggested adding a UI action on the incident form to make it easy for ITIL folks to switch over to the Agent Workspace from Platform UI. Someone suggested, why not do that as a UI Macro and then the button is in the form instead of across the top. Well, they're not wrong... so lets do that.
# UI Macro
First create lets create the UI macro. Under System UI -> UI Macros, create a new record. Use the code snippet below.
```
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
<j:set var="jvar_n" value="open_agent_workspace"/>
<g:reference_decoration id="${jvar_n}"
  onclick="openRecordInAgentWorkspace(); "
  title="${gs.getMessage('Open Record In Agent Workspace')}" image="images/icons/tasks.gifx" icon="icon-open-document-new-tab"/>
<script>
// show record in the agent workspace
function openRecordInAgentWorkspace() {
	try {
        var table = g_form.getTableName();
        var url = 'now/workspace/agent/record/' + table + '/' + g_form.getUniqueValue();
        g_navigation.openPopup(url);
	} catch (e) {
		jslog('error showing workspace list');
		jslog(e);
	}
}

</script>
</j:jelly> 
```

Save as "open_in_agent_workspace"
# Dictionary attribute
Next we need to add a ref_contribution to the dictionary attribute for the Incident table.
1. Open the Incident Table
2. Right click number and choose configure dictionary
3. Copy the contents of the attributes field to you clipboard, we'll need this later
4. Scroll down to Dictionary Overrides and click New
5. Set the Table to Incident, check of "Override attributes" and paste in the value you copied in step #3.
6. Append a comma and add <ref_contributions=open_in_agent_workspace>
7. Save
# Result
You should now have a button next to the Number field that will open the related Incident in a new Agent Workspace tab.

![]({{ 'assets/images/agent_ui_macro.PNG' | relative_url }})

You may choose to repeat this for other task tables like catalog task, requested item, problem, etc
