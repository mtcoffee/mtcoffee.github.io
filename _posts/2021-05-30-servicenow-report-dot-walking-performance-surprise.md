---
title: ServiceNow Report dot-walking Performance Surprise
---

This week I was working with user who had a dashboard that contained a report with some dot walking in the listed fields. Per [ServiceNow](https://docs.servicenow.com/bundle/paris-now-intelligence/page/use/reporting/concept/extended-table-fields-dot-walking.html), there is nothing wrong with doing this, just keep the chain link to three levels. 

# The Problem
In this case the user was using their report to test some data in a sub production instance and it would **not** return before the timeout interval. Sure, sub production is slower but this report returns in a under a second in production. It turns out, dot walking on a particular field, really affected the performance in the cloned sub production instance.

# The Solution
![]({{ 'assets/images/stage.PNG' | relative_url }})

After removing the stage field, the report returned right away.
