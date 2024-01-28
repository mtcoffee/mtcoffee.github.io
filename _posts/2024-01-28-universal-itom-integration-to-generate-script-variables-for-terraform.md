---
title: Universal ServiceNow ITOM Integration to Generate Script Variables for Terraform
---

# Summary
IT Staff like the idea of having users fill out a ServiceNow request for some type of infrastructure (like a server) that automatically creates the underlying infrastructure through an integration, but often are either not able to supply an API to integrate with or are simply  not comfortable with the idea of a user facing system, generating new infrastructure without their direct control.

There is an easy way to get around this challenge. In absence of an API, we can generate something that can quickly be dropped into describer language file like Hashicorp Terraform or into a script like Python or PowerShell. What makes this universal is that it can work with any infrastructure language that accepts variables.

Here are couple of options that can be easily tailored for use with of any language or CI/CD pipeline. The examples below will generate content, compabatible with a Terraform tfvars file.
# Method 1
A simple UI Action driven client script that can collect all variables on a catalog item and create a key/value file to be used in a script by the IT Operations or DevOps teams.  Create a simple UI client action that is conditional based on the catalog item. It will collect the variables and present them in a modal for quick copying.

```
var array =[];

for (var index = 0; index < g_form.nameMap.length; index++) {
     var variable_name = g_form.nameMap[index].prettyName 
     if (g_form.getDisplayBox(variable_name)) {
     var value = g_form.getDisplayBox(g_form.resolveNameMap(variable_name)).value;
     } else{
     var value = g_form.getValue('variables.' + variable_name)
     }
     //set your key/value separator here
     array.push(variable_name + ' = ' + value);
}

// Create a GlideModal to present the key pairs
var gm = new GlideModal('Variables');
//Sets the dialog title
gm.setTitle('Variable List');
//Set up valid custom HTML to be displayed

//We'll use the windows object to ensure our code is accessible from the modal dialog
top.window.CopytoClipboard = function() {
  var copyText = document.getElementById("myvars").innerHTML;
  if (navigator.clipboard.writeText(copyText)) {
                g_form.addInfoMessage("Copied successfully!");
            }
}

var modalContent = '<pre id="myvars">' + array.join('\n') + '</pre>'
var buttonContent = '<div class="input-group-btn"><button onclick="top.window.CopytoClipboard()" class="btn btn-default" >Copy to Clipboard</button></div>'
gm.renderWithContent( modalContent + buttonContent);
```

**Sample Result**   
IT Staff can quickly copy the key/value pair and run "terraform plan" with their Terraform project.   
![]({{ 'assets/images/universal itom1.png' | relative_url }})

# Method 2
Add a hidden multiline variable to the catlog item that only shows on request items or tasks.Then add this on submit client script to auto populate the multiline variable.
```
function onSubmit() {
    //Type appropriate comment here, and begin script below
    var array = [];

    for (var index = 0; index < g_form.nameMap.length; index++) {
        var variable_name = g_form.nameMap[index].prettyName;
        if (g_form.getDisplayBox(variable_name)) {
            var value = g_form.getDisplayBox(g_form.resolveNameMap(variable_name)).value;
        } else {
            var value = g_form.getValue('variables.' + variable_name);
        }
        //set your key/value separator here
        if (variable_name != 'script_variables') {
            array.push(variable_name + ' = ' + value);
        }
    }
    g_form.setValue('script_variables', array.join('\n'));

}
```
**Sample Result**   
IT Staff can quickly copy the key/value pair and run "terraform plan" with their Terraform project.
![]({{ 'assets/images/universal itom2.png' | relative_url }})

Certainly not a fully automated solution, but one that can easily be dropped in for any organization.
