---
title: ServiceNow Workspace Redirect with Navigation Handlers
---

# Summary
Now that the **ServiceNow Workspaces** are becoming more usable, many teams are transitioning to them as their primary interface. However, this creates a challenge: **ensuring users are consistently routed to the Workspace even when they click legacy links** (like those in old email notifications) or manually navigate to the Platform UI. 

Instead of manually updating every notification and link, you can use an old but still useful feature called **Navigation Handlers**.  
# View Rule vs Navigation Handler
You may be thinking, doesn't this conflict with View Rules?   
**Key Differences of View Rule vs Navigation Handler**:

* **View Rules**: Configured via UI, best for simple, rule-based view changes (e.g., "If user has role X, show role Y").
* **Navigation Handlers**: Requires JavaScript, used for complex logic, such as forcing a specific view regardless of user preference, or acting as a "backdoor" to redirect users, often taking precedence over regular View Rules.
* **Execution Order**: Controlled by the system property `glide.ui.view_rule.check_after_nav_handler`  
If true, handlers run first; if false, View Rules take precedence.
* **Use Case**: Use View Rules for straightforward, policy-based view changes (e.g., UI Policy). Use Navigation Handlers for complex scenarios (e.g., switching between Workspace and Core UI based on record data or user role).

# Creating Navigation Handlers
To get started review the OOB entry in the **sys_navigator** table.  Look at the entry for **sys_hub_flow** that opens Flows in Flow Designer

## To redirect all requests from the Incident Platform UI to the SOW Workspace
1. Create a new record in the sys_navigator table
2. Set Table to Incident
3. Add Script
<script src="https://gist.github.com/mtcoffee/638c9b87077163878925d842310cd3a5.js"></script>

## To redirect all requests from the Incident Platform UI to the SOW Workspace if the view is not specified
1. Create a new record in the sys_navigator table
2. Set Table to Incident
3. Add Script
<script src="https://gist.github.com/mtcoffee/7f7bf1d7da59f6d72e7b51575f4129c9.js"></script>

**References**
* [https://www.servicenow.com/docs/r/platform-user-interface/c_NavigationHandler.html ](https://www.servicenow.com/docs/r/platform-user-interface/c_NavigationHandler.html )
* [https://snpro.dev/2023/10/18/workspace-redirect-from-record-in-core-ui/](https://snpro.dev/2023/10/18/workspace-redirect-from-record-in-core-ui/)
