---
title: We have a Knowledge Portal in ServiceNow?
---

This is one that comes up for me regulary.
# The Problem
Often we overlook that ITIL users get busy with their own jobs and struggle to find time to learn all of the tooling available to them in ServiceNow. One item that comes up often for me is "Why is the Knowledge Article viewer so ugly" to which I'll reply, have you looked at it in the Knowledge Portal? The response of course is "We have a Knowledge Portal?". 
# The Solution
Viewing Knowledge Articles on the portal is much nicer than the platform UI. To help intuitively broadcast this to the team, we can make a couple of small changes!
## Part 1 -Update a single UI Macro
Customizing delivered UI Macro's, is not everyone's favourite choice but this is a very small and trackable change that can easily be rolled back if needed.
1. Open the System UI--> UI Macros and open the UI Macro "**kb_view_common_header_banner_image**"
2. After line 47, add the code snippet below. **Ignore the first and last lines, they're just there to help identify location**

```
<j:if test="${state!=''}">$[HTML:state]</j:if>
	<!-- Added by ME 2021 -->
	 <j:if test="${knowledgeRecord.workflow_state == 'published'}">
	<span style="float: right;"><a href="/kb?id=kb_article${AMP}sys_id=${knowledgeRecord.sys_id}" target="_blank">View on Knowledge Portal</a></span>	
	</j:if>  
	<!-- END of Added by ME 2021--> 
</div>
```
Save the change and view open a published knowledge article. You should have a new Hyperlink on the page that will open the article in new window on the portal!  
*Inspiration from this post if you would like to learn more. - [https://community.servicenow.com/community?id=communityblog&sysid=39eb190bdb178850190dfb24399619b2](https://community.servicenow.com/community?id=community_blog&sys_id=39eb190bdb178850190dfb24399619b2)*

**Result:**
![]({{ 'assets/images/View KB on portal UI macro.PNG' | relative_url }})

## Part 2 - Add a UI Action to the Knowledge Table
Similar to my previous post we can add a simple UI action to the kb_knowledge table. Here is a screenshot of the UI action. Script is further down.
![]({{ 'assets/images/View Article in Portal UI Action.PNG' | relative_url }})

**The script**

```
function openRecordInKBPortal() {
    var url = 'kb?id=kb_article&sys_id=' + g_form.getUniqueValue();
    g_navigation.openPopup(url);
}
```
**Result**
![]({{ 'assets/images/view kb in portal ui action result.PNG' | relative_url }})
