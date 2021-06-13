---
title: Enhancing ServiceNow Visual Task Boards
---

ServiceNow Visual Task Boards are a great way to organize your self or your team. Similar to Kanban or Microsoft Planner boards, you can quickly assign tasks and move them between lanes to adjust their states. 
![]({{ 'assets/images/vtb board.PNG' | relative_url }})
# The Problem
There are 2 main types, stateful  and freeform.  Stateful is useful for managing tasks on existing structured tables (INC/PRB) where as freeform is best for general task management. I did however notice some significant shortcomings on freeform

1. The PTSK tasks created on freeform boards remain in a state of "Open", even after you mark it done or archive it. This floods users "My Work" filters with all of these open tasks and messes with their reports.
2. When you mark a task with a due date, there are no OOB notifications to notify users that an assigned task is over due.


# The Solution
First we'll tell the PTSK to close all freeform tasks where the freeform board has marked it done. We can do this with a business rule on the vtb_card table.  

## Business Rule to close tasks

**Name**: Change state on PTSK lane change Done  
**Table**: vtb_card  
**When**: before update
**Conditions**:  
Lane.Name is Done AND  
Board.Lane field is empty AND  
Task.Task type is Private Task

**Script**:
```
(function executeRule(current, previous /*null when async*/ ) {
    // Add your code here
    var ptaskGR = new GlideRecord('vtb_task');
    ptaskGR.get(current.task.toString());
    ptaskGR.setValue('state', '3');
    ptaskGR.update();
})(current, previous);
```
Now when all tasks are marked Done in the board, the task will be closed using state 3. 

## Bonus option to reopen tasks
We can also reverse this in the event the task is reopened. Just flip the lane name logic and set the state to 1 in the script.  
![]({{ 'assets/images/vtbnotdone.PNG' | relative_url }})

## Send Notification on due date
A simple scheduled Flow Designer Flow that looks up all vtb_task records and based on the due date criteria, sends an email to the assignee. You can do an additional lookup to get the board id too.  Building the email in FD was tricky and may be better done as notification action. If deploying to production, you'll need to add validation step, to make sure that the task has an assignee, else the flow will error when it can't send an email to no address.
![]({{ 'assets/images/vtb_notify_flow.PNG' | relative_url }})
