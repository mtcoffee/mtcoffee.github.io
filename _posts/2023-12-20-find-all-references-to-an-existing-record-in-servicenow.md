---
title: Find all references to an existing record in ServiceNow
---

# Summary
Sometimes you need to know "which records are pointing to this record that I am about to work with". For example a core_company record that you want to consolidate or you want to know if any active records are pointing to an aged out group. The [servicenowguru.com](https://servicenowguru.com/system-definition/find-references-specific-record/) blog has a good interactive solution but I needed to generate a very large list.
# Solution
This background script will find all referencing records to a specified table/sys_id combination and save the results to a CSV in the sys_data_source table.

<script src="https://gist.github.com/mtcoffee/15845837f92f3b6f71431ce2497d9528.js"></script>

Sample result of all records referencing the core_company "Microsoft" record.
![]({{ 'assets/images/reference_list.PNG' | relative_url }})
