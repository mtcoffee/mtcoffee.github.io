---
title: Data Validation in ServiceNow Import Sets
---

# The Problem
I recently ran into an issue with an import source that where the source system had ran into an issue providing its exported data. In this case the source system was exporting its contents to a CSV and publishing to an SFTP host for pick up by the ServiceNow Import Set. The problem, was that the system started creating zero byte files with no data. This does not trigger an error in the import set since it the file is still provided, it just happens to be empty. We need a way to validate that our source data exists.

# The Solution
An onstart script in the transform! ServiceNow does provide some interesting examples [here](https://docs.servicenow.com/bundle/paris-platform-administration/page/script/server-scripting/reference/r_TransformationScriptVariables.html). But lets get to the meat and potatoes.

In the transform that your data source uses, create an "onStart" script and give it an order value to ensure it runs first. The script below will trigger an error if the row count is zero in the data source. This is one of many ways, you can check the data in the source before running the transform.
```
// Add your code here
info ="########Checking Source Row Count########, row count is: ";
rowcount=source.getRowCount();
info += rowcount; 
log.info( info );	
if(rowcount <= 0 ){
	error = true;
	error_message = "#########No data in source!!!!!! Check source file.###########";
}
```

To monitor this, you can create a notification that triggers when the sys_import_set_run table updates with "state complete with errors"
