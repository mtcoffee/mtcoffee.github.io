---
title: ServiceNow Service Catalog Platform Page "Redirect"
---

# Summary
A ServiceNow Service Catalog can be accessed from both the Platform UI Page and the Service Portal. This can be problematic if the catalog items have not been developed for both, but you may also not be able to deny access to the Platform view for integration reasons like Universal Request. One simple catch all is to remind users when they visit the UI page is to nicely suggest the open the item in the portal.
# Solution
1. Update the UI page "com.glideapp.servicecatalog_cat_item_view"
2. Add this block to the top of the HTML. Instructs jelly to call the client script in the function.

```
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
	<!-- call client script to show pop up message -->
	<script>
	addLoadEvent( function() {
	onLoadFunction();
	});
	</script>
```

3.Add this modal script block to the client script.

```
//client script to run onload of ui page
function onLoadFunction() {
    var item_sys_id = g_form.getUniqueValue('sys_id');
    var gm = new GlideModal("", true, 600);
    gm.setTitle("Warning");
    var html = '<div style="padding:15px"><p class="text-primary"><b>For best experience use the Service Portal</b>.</p> <p>It is always recommended to use the Service Portal for submitting catalog items. Unless you have specific need, please open this item in the portal view.</p><div style="padding:5px;float:right">' +
        '<button style="padding:5px;margin-right:10px" onclick="top.window.TaskAction(this.innerHTML,jQuery(\'#taskAction\').val())" class="btn btn-default">I have a specific need</button>' +
        '<button  style="padding:5px" class="btn btn-primary" onclick="top.window.TaskAction(this.innerHTML,jQuery(\'#taskAction\').val())">Open this Item in Portal</button></div></div>';
    gm.renderWithContent(html);
    //We'll use the windows object to ensure our code is accessible from the modal dialog
    top.window.TaskAction = function(thisButton) {

        //Close the glide modal dialog window
        gm.destroy();

        //Submit to the back-end
        if (thisButton == 'Open this Item in Portal') {
            //open new link
            window.open('/sp?id=sc_cat_item&sys_id=' + item_sys_id, '_blank');
        }
    };
    return false; //prevents the form from submitting when the dialog first loads
}
```

Result
![]({{ 'assets/images/scwarning.PNG' | relative_url }})
