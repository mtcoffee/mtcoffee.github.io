---
title: ServiceNow Request Management "Shims"
---

# The Problem
Sometimes new ServiceNow customers are migrating from an aging system that was primarily work note driven. For example their workflow for requests consisted of the user submitting a "ticket", using it as a discussion thread, manually reassigning it between groups until it was completed. While this is not a great practice nor one to try and replicate, some teams really need the ability to look at running thread of all work done between each team. If you are using ServiceNow Request Management, the work notes only exist on each task, so they have to visit each task to see what the other group did. As well, they want to see the customer updates on their tasks too. So while these tricks are not highly recommend, sometimes they are necessary.

# The Solution
These are few tricks of mine that can help.
## Auto Add Catalog Tasks Work notes to Request Item
On the sc_task table create a new business rule

Name:Copy work notes to RITM  
When: before insert, updates  
Advanced Condition: current.work_notes.changes()  
Advanced Script:
```
(function executeRule(current, previous /*null when async*/) {

//fetch RITM URL details and append to work notes variable
var url = 'sc_task.do?sys_id=' + current.sys_id;
var tasknum = current.number.getDisplayValue();
	
var work_notes = current.work_notes.getJournalEntry(1);
var i = work_notes.indexOf('\n'); 
if (i>0)
{
// taking everything after the first line break, he-he.
var work_notes_tidy = work_notes.substring(i+1, work_notes.length);
}	
	
var item = new GlideRecord('sc_req_item');
item.get(current.request_item);
//var worknote_title = 'From task ' + current.number + '\n' ; 
var worknote_title = "[code]<h4>From task " + "<a target='_blank' href='" + url + "'>" + tasknum + "</a></h4>[/code]";
item.work_notes = worknote_title + work_notes_tidy;
item.update();
	


})(current, previous);
```

Result:
![]({{ 'assets/images/Tasks Work notes to Request Item.PNG' | relative_url }})
## Auto Add Request Item Comments to Catalog Tasks
On the sc_req_item table create a new business rule  
Name:Copy RITM Comment to Active task  
When: before insert, updates  
Advanced Condition: current.comments.changes()  
Advanced Script:
```
(function executeRule(current, previous /*null when async*/ ) {

    // Add your code here
    updateTasks();

 function updateTasks() {
     //fetch RITM URL details and append to work notes variable
     var url = 'sc_req_item.do?sys_id=' + current.sys_id;
     var ritmNum = current.number.getDisplayValue();

     var sctask = new GlideRecord('sc_task');
     sctask.addQuery('request_item', current.sys_id);
     sctask.addQuery('active', true);
     sctask.query();
     while (sctask.next()) {
         sctask.comments = "[code]<h4>Additional comment from Request Item - To reply to this comment, open " + "<a target='_blank' href='" + url + "'>" + ritmNum + "</a></h4>[/code]" + current.comments;
         //sctask.comments = "test " + current.comments; 
         sctask.update();
     }
 }
})(current, previous);
```

Result:![]({{ 'assets/images/RITM comment to task.PNG' | relative_url }})
## Bonus - Add related Knowledge Articles to Work Notes
This rule will find all knowledge articles associated to the Catalog Item and present them in any new task with a clickable link to the knowledge portal. Helpful if you have a Runbook associated to the work related to the associated request.

On the sc_task table create a new business rule  
Name:Add Informational Work Notes  
When: before insert  
Advanced Script:
```
(function executeRule(current, previous /*null when async*/) {

	// Add your code here
	
//fetch RITM URL details and append to work notes variable
var url = 'sc_req_item.do?sys_id=' + current.request_item.sys_id;
var ritmNum = current.request_item.number.getDisplayValue();

var work_notes = "[code]<h3>System Generated Work Note - Task Fulfiller Information</h3>[/code]";
work_notes += "To contact the requester please [code]<b>update the request item comments in the request item</b>[/code] " +"[code]<a target='_blank' href='" + url + "'>" + ritmNum + "</a> or use their preferred method of contact stated in the request.<br><br>[/code]";
	
//fetch related Knowledge Article Details and append to work notes variable
work_notes += "Please review any RITM related knowledge articles listed below for fullfilment instructions and please add or update the instructions as needed. As a fulfiler, you should have full edit access to the KB article with exception to published End User articles. To have a new article added to this list, just link your KB article to this catalog item on the Related Catalog Items list." + "[code]<br>[/code]";

var idR = current.request_item.cat_item.sys_id;
var kbrel = new GlideRecord('sc_2_kb');
kbrel.addQuery('sc_cat_item',idR);
kbrel.query();
if (kbrel.hasNext()) {
work_notes += "\n[code]<b>Catalog Item Linked Articles</b>[/code]\n";	
}
while(kbrel.next()){
var kburl = 'kb_view.do?sys_kb_id=' + kbrel.kb_knowledge.sys_id;
var kbNum = kbrel.kb_knowledge.getDisplayValue();
var kbName = kbrel.kb_knowledge.short_description.getDisplayValue();
//add each related kb article	
work_notes += "[code]<a target='_blank' href='" + kburl + "'>" + kbNum + "</a>[/code]" + " - " + kbName + "[code]<br>[/code]";
}

//repeat for kb_2_sc table
var idR2 = current.request_item.cat_item.sys_id;
var kbrel2 = new GlideRecord('kb_2_sc');
kbrel2.addQuery('sc_cat_item',idR2);
kbrel2.orderByDesc('sys_created_on');
kbrel2.setLimit(1);
kbrel2.query();
if (kbrel2.hasNext()) {
  work_notes += "\n[code]<b>Knowledge Articles Linked to this Catalog Item</b>[/code]\n";
}		
while(kbrel2.next()){
var kburl2 = 'kb_view.do?sys_kb_id=' + kbrel2.kb_knowledge.sys_id;
var kbNum2 = kbrel2.kb_knowledge.getDisplayValue();
var kbName2 = kbrel2.kb_knowledge.short_description.getDisplayValue();
//add each related kb article	
work_notes += "[code]<a target='_blank' href='" + kburl2 + "'>" + kbNum2 + "</a>[/code]" + " - " + kbName2 + "[code]<br>[/code]";
}
	
//write work notes
current.work_notes = work_notes;
	
})(current, previous);
```
Result:![]({{ 'assets/images/kbworknote.PNG' | relative_url }})
