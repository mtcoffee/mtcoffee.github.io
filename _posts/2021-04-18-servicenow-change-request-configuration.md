---
title: ServiceNow Change Request Configuration
---

Recently I had a need to alter the OOB fuctionality of ServiceNow Change Management. I was given the following requirements.
1. Change the workflow titles (process flow  formatter) at top of the issue from Assess/Authorize to Assignment Group Assess/CAB Authorize
2. Change the approval mail so that it includes the change details instead of the OOB approver email for all approvals
3. Exclude cab members from the approval email since they will do their approval at CAB meets.


# Change Process flow formatter
To change the [process flow formatter](https://docs.servicenow.com/bundle/quebec-platform-administration/page/administer/form-administration/reference/r_ProcessFlowFormatter.html), 

1. Navigate to System UI > Process Flow.
2. Under label search for "Assess"
3. Open the record for "Normal Change - Assess state"
4. Change the label value to "Assignment Group Assess"
5. Repeat these steps for the Authorize/CAB Authorize label

Example Result
![]({{ 'assets/images/change_formatter.PNG' | relative_url }})
# Change approval mail content
First we need to prevent the OOB "Approval Request" notification that fires for all approvals to not fire change requests since we want unique content. This is achived with a simple condition on the "when to send" tab of the notification. Use the script below in "Advanced condition"
```
//Do not send email if this is a change request approval. We will use another notification for this.
if (current.sysapproval.sys_class_name == 'change_request') {
	answer = false;
} else {
	answer = true;
}
```
Don't for get to save. Now, from the hamburger menu, lets "insert and stay" to create a copy of the "Approval Request"  and we'll rename it to "Approval Request Change".  Next lets invert the Advanced condition so that it only fires when the approval is for a change request.
```
//Only trigger this if the approval is for a change request
if (current.sysapproval.sys_class_name == 'change_request') {
	answer = true;
} else {
	answer = false;
}
```
Now we have a notfication that is unique to change. We can alter the "what it will contain" content to retrieve and include addtional data, exclusive to change requests.

# Change approval email to exclude CAB members
Finally, we want to exclude cab members from the approval email since they will do their approval at CAB meets. This prevents them from approving the requests in their email accidentally. We dot walk the condition on the "When to send" tab and instruct it to not send when assignment group is CAB approval.

1. Click choose field and scroll to bottom
2. Select Show Related Fields
3. Select Group Approval Fields
4. Select Assignment Group
5. Set "is not CAB Approval"
![]({{ 'assets/images/change_exclude_cab.PNG' | relative_url }})

Now approval emails will not be sent to members of this group when it is the approver assignment. *This won't prevent  users in the cab group from getting the approval email for other assignment groups they might be in.*

Some folks might prefer to alter the event that triggers this notification, however I like this approach for simplifying upgrades. As is always with ServiceNow, there are many ways to achieve the same outcome.
