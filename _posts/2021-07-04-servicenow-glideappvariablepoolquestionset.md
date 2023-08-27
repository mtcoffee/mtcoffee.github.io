---
title: ServiceNow GlideappVariablePoolQuestionSet
---

# Summary
Recently I was tasked with adding all variables from a ServiceNow Request to an email. This is where you can use the function GlideappVariablePoolQuestionSet.  I'll use of a few examples that leverage the OOB items in a ServiceNow Personal Developer Instanace.

# Add all RITM variables to an email sent to the Requester
In this example, we will place all variables for all RITM records in the Request in an email to the requester. 
1. Create a mail script named "add RITM variables"
<script src="https://gist.github.com/mtcoffee/9bac5b0bf1ab8b6a6ed829f929c3be9b.js"></script>
2.  On the OOB request.general notification template, add the mail script to the bottom. Note that this notification is on the sc_request table


**Notification Change**     
![]({{ 'assets/images/gpvs001.PNG' | relative_url }})     

**Sample Result**   
![]({{ 'assets/images/gpvs002.PNG' | relative_url }})

# Add all RITM variables to an email sent to the Catalog Task Fulfiller
In this example, we will place all variables for all RITM records in the Request, in an email to the Catalog Task Fulfiller. 
1. Create a mail script named "add RITM variables to task"
<script src="https://gist.github.com/mtcoffee/b469f6cd16b6c50cb600cee661431ee9.js"></script>
2. On the OOB sc_task.itil.role notification template, add the mail script to the bottom. Note that this notification is on the sc_task table.

**Notification Change**     
![]({{ 'assets/images/gpvs003.PNG' | relative_url }})     

**Sample Result**   
![]({{ 'assets/images/gpvs004.PNG' | relative_url }})

# Bonus - Add variables to the subject line
While doing this I realized you can also add variables to subject lines. For example, this will override the default subject by adding this script to the sc_task.itil.role notification template.  

**Mail Script**
<script src="https://gist.github.com/mtcoffee/e5f6b9167d308d5464e8664488fef082.js"></script>

**Notification Change**     
![]({{ 'assets/images/gpvs005.PNG' | relative_url }})     

**Sample Result**   
![]({{ 'assets/images/gpvs006.PNG' | relative_url }})
