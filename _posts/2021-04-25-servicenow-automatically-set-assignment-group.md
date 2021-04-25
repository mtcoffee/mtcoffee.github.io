---
title: ServiceNow Automatically set Assignment Group
---

# Summary
This is a useful tidbit for auto populating assignment groups. ServiceNow does have a few features that make it possible to auto assign groups based on criteria such as:
* [Assignment Lookup Rules](https://docs.servicenow.com/bundle/paris-platform-administration/page/administer/task-table/concept/c_AssignmentLookupRulesExample.html)
* [Data lookup definitions](https://docs.servicenow.com/bundle/quebec-platform-administration/page/administer/field-administration/concept/c_DataLookRecMatchSupport.html)

Howevever, sometimes a simple client script can offer less overhead. In this case I had the following requirements.
1. When a requester picks a group it will auto set the assignment group to the Configuration Item approver group
2. The requester can accept this or choose to override it
3. If a group is not found on the CI, a pop up will alert them to select it manually

# Solution

To accomplish this the client script was as follows:

**Name** : Populate Assignment Group  
**Type** : onChange  
**Field Name** : cmdb_ci  (Configuration Item)

Code:
```
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '')
        return;
    var ref = g_form.getReference('cmdb_ci');
    var refgroup = ref.change_control; // change_control is the name of the field on cmdb to get the group

    if (refgroup) {
        g_form.setValue('assignment_group', refgroup); 
    } else {
  g_form.addInfoMessage("Assignment Group from Configuration Item not found, please select manually");
}

}
```

Set that on your table (incident/change etc) and your all set!


# Why not Assignment Rule?
I was asked, why not create an assignment rule, after all its built into the product. For example a simple rule under System Policy -> Assignment defined using this script snippet work work. Sample snippet:
```
if (JSUtil.notNil(current.cmdb_ci)) {
    current.assignment_group = current.cmdb_ci.support_group;
}
```

However it is server side and does not set until  after the user saves the record. This would be ideal if you wanted to "Hard enforce" that the assignment group must be set by the CI, however if your requirement is to allow the user to override then a client script is the preferred route.
