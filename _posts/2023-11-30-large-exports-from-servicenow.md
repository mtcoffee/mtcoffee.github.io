---
title: Large Exports from ServiceNow
---

# Summary
As ServiceNow points out in [KB0727636](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0727636), by default, ServiceNow has a max limit of 10,000 records that can be exported from a table when using the user interface, the Table API or the Export URL method, i.e. *https://instancename.service-now.com/cmdb_ci_computer.do?CSV&sysparmdefaultexportfields=all*

# Solution
A simple PowerShell script that will query the Table API for the selected table and loop through blocks of 10,000 until all records are retrieved.
It then converts the JSON payload to a CSV.

<script src="https://gist.github.com/mtcoffee/d07f7982d7714a5064c35449b7f4bfbc.js"></script>
