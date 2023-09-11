---
title: ServiceNow Data Table from Instance widget to Modal
---

# Summary
A fun task I received recently was to have a list of records show for the user in the portal, but when they click on it, it must show a modal instead of navigating away and opening the record in a 
new page. We can leverage the OOB "Data Table from Instance Definition" to accomplish this

# Solution
1. Clone the widget the OOB "Data Table from Instance Definition" using the "Clone Widget" UI action.
2. Update name to Data Table from Instance - Modal
3. Update the client script in the new custom widget
```
function actions($scope, spUtil, $location, spAriaFocusManager, spModal) {
	$scope.$on('data_table.click', function(e, parms){
		var p = $scope.data.page_id || 'form';
		var s = {id: p, table: parms.table, sys_id: parms.sys_id, view: 'sp'};
		//var newURL = $location.search(s);
		//spAriaFocusManager.navigateToLink(newURL.url());
        spModal.open({
            title: "Record",
            widget: 'widget-form',
            widgetInput: {
                sys_id: parms.sys_id,
                table: parms.table
            },
            buttons: [
                {
                    label: "✔ Ok",
                    primary: true
                }
            ],
        }).then(function() {
            //
        });
	});
}
```
5. Add the widget to your page and set the table/filter options.

Sample result
![]({{ 'assets/images/SPModalDataTable.PNG' | relative_url }})
