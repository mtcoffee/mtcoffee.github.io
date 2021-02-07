---
title: Creating an “ITIL LITE” role in ServiceNow
---

This seems to be overlooked in ServiceNow's out of the box role design. 
# Problem
OOB, users with the ITIL role can do almost everything including write to the cmdb. I was recently in a call with a ServiceNow consultant who said, “you probably don’t want your help desk editing the cmdb”.  Fortunately we can leverage the [ITSM roles plugin ](https://docs.servicenow.com/bundle/paris-it-service-management/page/product/incident-management/task/req-itsm-roles-inci-mgmt.html/) to address this situation.
# Solution
1. Navigate to User Administration > Roles and click New. Create a new role called itil_lite
2. Open the role and open the related list "Contains Roles". Click edit and add the following roles:
```
sn_request_write
sn_incident_write
cmdb_read
sn_change_read
sn_problem_read
knowledge
dependency_views
agent_workspace_user
template_editor
view_changer
app_service_user
email_composer
```
3. Open sys_properties.list and update the property "glide.ui.can_search" to include itil_lite. Repeat this step for:
```
glide.history.role
glide.knowman.section.view_roles.review
glide.knowman.show_flag.roles
glide.ui.activity.email_roles
glide.ui.personalize_form.role
```
4. Add access to the caller table (optional if not using the caller module)
    * Open System Definition ->Tables
    * Search for "call" and open the definition for "new_call" table
    * Create new ACL's create/write/delete/read operations and set requires role to "itil_lite"
5. Add access to the Service Desk and Caller menu
     * Open System Definition -> Application Menus an open the definition for "Service Desk" 
     * Add "qu_its_itsm_base"  to roles
     * Scroll down to to the modules related list and notice the column "roles".
     * Where ever you see "itil" add the role "itil_lite".
6. Assign this role to the necessary groups.

This may not perfectly fit your situtation, but does address the cmdb write issue. One important detail to note, is the sn_change_write role includes ACL's for write access to the CMDB. So if you want to permit users to submit change requests but not have cmdb write access, you'll need to modify the cmdb_ci table ACL's .
