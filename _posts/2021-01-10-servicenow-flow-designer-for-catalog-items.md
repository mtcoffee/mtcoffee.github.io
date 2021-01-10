---
title: ServiceNow Flow Designer for Catalog Items
---

ServiceNow has been pushing everyone to make the move from the classic Workflow Editor to Flow Designer. Flow Designer is a modern low code solution (similar to Microsoft Flow) for creating workflows. It can also be[ used in place of business rules](https://developer.servicenow.com/dev.do#!/guides/paris/now-platform/pro-dev-guide/pd-build-logic##flow-designer-vs-business-rules). Their general position is for any new process flow requirements, ServiceNow recommends using Flow Designer over the legacy workflow for almost all circumstances. Welp, lets give this a go. The easy place to start is a with a simple catalog item. Let's create a basic single task catalog item. 

## Create  a New Flow Designer Flow
1. Open Flow Designer
2. In the upper right corner, click New -> Flow
    *  Title it MY - Catalog Item Request - Single Task
    * Add a description
    * choose run as "System User" *Important Note - [Flow can not create service catalog task when it's initiated by an ITIL user](https://hi.service-now.com/kb_view.do?sysparm_article=KB0754165)
    * Click submit

3. Under Trigger click the "+" sign
    * AdSearch for "Service Catalog" and choose "Service Catalog"
    * Accept the defaults

## Create an update statement to add data to the RITM fields
1. Under Action click the "+" sign and choose "Actions".
2. Search for and select "Updated Record".
    *  For Record, click the data pill icon and choose "Trigger -Service Catalog ->Requested Item Record"
    *  For Fields, set Assignment Group, click the data pill icon and choose "Trigger -Service Catalog ->Requested Item Record->Item ->Fulfillment group->Sys ID"
    *  Again for Fields, set Description, however this time we'll choose scripted by click the "f(x)" button. Use the script below:

```
var itemDescription = fd_data.trigger.request_item.cat_item.description.replace(/<li>/g, '').replace(/\//g, '').replace(/<p>/g, '').replace(/<ul>/g, '').replace(/<li>/g, '\n').replace(/&#34;/g, '"').replace(/<strong>/g, '').replace(/<span>/g, '').replace(/<span style="text-decoration: underline;">/g, '');
return itemDescription;
```
## Create a Catalog Task

1. Under Action click the "+" sign and choose "Actions".
2. Search for and select "Create Catalog Task".
    *  For Requested Item, click the data pill icon and choose "Trigger -Service Catalog ->Requested Item Record"
    *  For Short Description, click the data pill icon and choose "Trigger -Service Catalog ->Requested Item Record -> Short Description"
    *  For Fields, set Assignment Group, click the data pill icon and choose "Trigger -Service Catalog ->Requested Item Record->Item ->Fulfillment group->Sys ID"
    *  Again for Fields, set Description and use this text block.
```
Please see KB articles referenced in the task work notes for fullfilment instructions and please add or update the instructions as needed.
```

## Add a log step
We need one final step after the task is complete to set staging on. The simplest method is to add a log step.

1. Under Action click the "+" sign and choose "Actions".
2. Search for and select "Log".
3. Set message to "Set Stage to completed after last task is closed".

## Set stages
For Request Item states to be updated as the task is closed, you must import the states and set them in the necessary spots. 
1. Click the 3 dots in the top right and choose stages.
2. Click Select a stages set and choose Requested Item.
3. Reorder the stages as as you would like to appear on the portal. You need this to be in the correct order so they appear correctly in the portal.

Now we need to add the stages to the correct spots. See example below:
![]({{ 'assets/images/FD_BASIC_CAT_ITEM.PNG' | relative_url }})

**Now save the flow!**

## TIP - Catalog Item Variables on Task Form
You can configure which variables you want to show on the form in the Catalog Task action of the workflow, but then you have to create a new flow for every catalog item. Yuck! Instead, configure your variables to be "Global" and they will show on every Task automatically.
![]({{ 'assets/images/global variables.PNG' | relative_url }})

## Finishing Up
Now you're all set to add this Flow to your Catlog Item.  You may need to add the "Flow" field to your Form Layout on the sc_cat_item table.
![]({{ 'assets/images/flow on sc_cat_item.PNG' | relative_url }})

Also, this flow is setup to use the fullfilment group on the catalog item, so make sure that is also visible and set.

Don't forget to read **[Best practices for using the Flow Designer](https://community.servicenow.com/community?id=community_blog&sys_id=09becbf3db589b403882fb651f961986)**
