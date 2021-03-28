---
title: ServiceNow Client Scripts, GlideAjax and Record creation
---

# Summary
One of the most common mistakes ServiceNow developers make is using gliderecord in client scripts. Per ServiceNow, this is neither supported nor [recommended](https://docs.servicenow.com/bundle/quebec-application-development/page/script/client-scripts/concept/client-script-best-practices.html), however it does work. So why not run glide record queries/inserts in client scripts? A few reasons include:

* Performance - [Community post](https://community.servicenow.com/community?id=community_question&sys_id=78edb1a3db1d53804837f3231f96193c)
* Validation - Executing server side will peform the necessary validation, more [here](https://www.basicoservicenowlearning.in/2019/12/glideajax-in-servicenow.html)
* Security  - user access to record may prevent the action from working
* Blowback - eventually you will get burned by this practice

To demonstrate, lets create a ui action pop up on the services form to create an incident.

# UI Page
First we will create a UI Page. Sample code blocks.
## UI Page HTML
```
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
	
<script>
		//to set value of element id service to read only
       $j(document).ready(function(){
        var service_readonly = gel('sys_display.service');
		service_readonly.setAttribute('disabled','true');
		var service_nolookup = gel('lookup.service');
		service_nolookup.setAttribute('disabled','true');
         });
</script> 
	
  <!-- Get values from dialog preferences passed in -->
  <j:set var="jvar_name" value="${RP.getWindowProperties().get('name')}" />
  <j:set var="jvar_assignment_group" value="${RP.getWindowProperties().get('assignment_group')}" />
	
	   <!-- Set up form fields and labels -->
   <table width="100%">
	        <tr>
	   <td width="50%">
	   <j:set var="jvar_mandatory" value="true"/>
       <div class="form-group is-required form-horizontal">
	     <div class="col-md-5 text-right">
		   <g:form_label>
			 Service
		   </g:form_label>
	     </div>
	     <div class="col-md-7">
			<g:evaluate var="jvar_request" jelly = "true">
             var serviceId = jelly.jvar_name;
	         var appservice = new GlideRecord("cmdb_ci_service");
               appservice.addQuery("name", serviceId);
               appservice.query();
               appservice.next();
               appservice;
           </g:evaluate>
             <g:ui_reference id="service" name="service" table="cmdb_ci_service" mandatory="true" value="${appservice.sys_id}" displayValue="${appservice.name}"/>
	     </div>
       </div>
	   </td>

	   <td width="50%">
	   <j:set var="jvar_mandatory" value="true"/>
       <div class="form-group is-required form-horizontal">
	     <div class="col-md-5 text-right">
		   <g:form_label>
			 Assignment Group
		   </g:form_label>
	     </div>
	     <div class="col-md-7">
			<g:evaluate var="jvar_request" jelly = "true">
             var groupId = jelly.jvar_assignment_group;
	         var appGroup = new GlideRecord("sys_user_group");
               appGroup.addQuery("sys_id", groupId);
               appGroup.query();
               appGroup.next();
               appGroup;
           </g:evaluate>
             <g:ui_reference id="assignment_group" name="assignment_group" table="sys_user_group" mandatory="true" value="${appGroup.sys_id}" displayValue="${appGroup.name}" /> 
	     </div>
       </div>
	   </td>
	   </tr>
	   <tr><td style="height: 10px"></td></tr>
	   
	   <tr>
	   	   <td colspan="2" >
	   <j:set var="jvar_mandatory" value="false"/>
             <g:ui_input_field id="short_description" name="short_description" label="Short Description" value="${jvar_name} is having issues" size="115"/> 
	   </td>
	   </tr>
	   
	   	   <tr><td style="height: 4px"></td></tr>
	   
		<tr>	
		<td colspan="2">
	   <j:set var="jvar_mandatory" value="true"/>
             <g:ui_multiline_input_field name="description" id="description" label="Description" value="${jvar_name} is having issues"/>
	   </td>
	   </tr>

	     <tr id="dialog_buttons">
        <td colspan="2" align="right">
       
           <input type="hidden" name="current_sys_id" value="${RP.getWindowProperties().get('sys_id')}"/>
		<g:dialog_buttons_ok_cancel ok_text="Create Incident" ok="return CreateIncident();" cancel="window.GlideModalForm.prototype.locate(this).destroy(); return false"/>	
       </td>
     </tr>
	   
  </table>	
</j:jelly>
```
## UI Page Client Script
In this clident script we will insert a new record form the user input. This is a prime example of what not to do.
```
function CreateIncident() {

    var description = document.getElementById("description").value;
    var short_description = document.getElementById("short_description").value;
    var assignment_group = document.getElementById("assignment_group").value;
    //var service = document.getElementById("service").value;
    //gel is a short cut to document.getElementById
    var service = gel('service').value;

    //confirm input
    if (assignment_group == '' || service == '') {
        //If terms are false stop submission
        alert('Please fill out all mandatory fields.');
        return false;
    }

    var gr = new GlideRecord('incident');
    gr.initialize();
    gr.description = description;
    gr.short_description = short_description;
    gr.assignment_group = assignment_group;
    gr.business_service = service;
    var id = gr.insert();
    GlideDialogWindow.get().destroy(); //Close the dialog window

    //get related info and display
    var inc = new GlideRecord('incident');
    inc.get(id);
    var servicename = new GlideRecord('cmdb_ci_service');
    servicename.get(service);
    var url = '/incident.do?sys_id=' + inc.sys_id;
    g_form.addInfoMessage('<a href="' + url + '">' + inc.number + '</a>' + ' has been generated for ' + servicename.name);

}
```
# UI Action 
Create this on the cmdb_ci_service form

Values to set:
* Show update = true
* client = true
* form button = true
* form style = primary
* isolate script = false
* onclick = openPopUp()

Script
```
function openPopUp() {
	var dialogClass = window.GlideModal ? GlideModal : GlideDialogWindow;
	var dialog = new dialogClass("create_incident");
	dialog.setWidth("800");
	dialog.setTitle(getMessage("Create an Incident for this Service"));
	dialog.setPreference("sys_id", g_form.getUniqueValue());
	dialog.setPreference("name",g_form.getValue('name'));
	dialog.setPreference("assignment_group",g_form.getValue('assignment_group'));
	assetSetDomainParameters(dialog);
	dialog.render(); //Open the dialog
}
```
# Demo
At this point you should have button on the form to create an incident. Example:
![]({{ 'assets/images/uipage dialog create incident.PNG' | relative_url }})

While this works it will be a big sluggish. Lets convert this to Glide Ajax. First we need to create a script include.
# Script Include for Glide Ajax
* Make sure it is client callable 


```
var CreateIncidentfromServices = Class.create();
CreateIncidentfromServices.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    CreateIncidentfromServices: function() {

        var short_description = this.getParameter('sysparm_short_description');
        var assignment_group = this.getParameter('sysparm_assignment_group');
        var service = this.getParameter('sysparm_service');
        var description = this.getParameter('sysparm_description');

        //create new incident 
        var gr = new GlideRecord('incident');
        gr.initialize();
        gr.description = description;
        gr.short_description = short_description;
        gr.assignment_group = assignment_group;
        gr.business_service = service;
        //var id = gr.insert();
        gr.insert();

        //return data for client script
        var object = {};
        object.id = gr.getUniqueValue();
        object.number = gr.getDisplayValue('number');
        object.service = gr.getDisplayValue('business_service');
        var json = new JSON();
        var data = json.encode(object); //JSON formatted string

        return data;

    },

    type: 'CreateIncidentfromServices'
});
```
# Client Script to use Glide Ajax
Let's modify our UI Page Client script to use Glide Ajax and call the script include.

```
function CreateIncident() {

	//get data from html
    var description = document.getElementById("description").value;
    var short_description = document.getElementById("short_description").value;
    var assignment_group = document.getElementById("assignment_group").value;
    //var service = document.getElementById("service").value;
    //gel is a short cut to document.getElementById
    var service = gel('service').value;
	
	//confirm input
      if(assignment_group == '' || service == ''){
      //If terms are false stop submission
      alert('Please fill out all mandatory fields.');
      return false;
   }

    //call client-callable script include
    var ga = new GlideAjax('CreateIncidentfromServices');
    ga.addParam('sysparm_name', "CreateIncidentfromServices");
    ga.addParam('sysparm_short_description', short_description);
    ga.addParam('sysparm_assignment_group', assignment_group);
    ga.addParam('sysparm_service', service);
    ga.addParam('sysparm_description', description);
    ga.getXMLAnswer(function(answer) {
        //g_form.addInfoMessage(answer);
		answerparsed = answer.evalJSON();
        var url = '/incident.do?sys_id=' + answerparsed.id;
        g_form.addInfoMessage('<a href="' + url + '">' + answerparsed.number + '</a>' + ' has been generated for ' + answerparsed.service);
    });

    GlideDialogWindow.get().destroy(); //Close the dialog window

}


```

Save and Test. You should see a noticable performance improvement. A few other online sources are listed below that helped to create this content.
* [GLIDEAJAX: RETURN MULTIPLE VALUES USING JSON](http://sn.sptrac.com/?p=23)
* [GlideAJAX that can be used by client scripts and replace existing use of GlideRecord from Client script](https://community.servicenow.com/community?id=community_article&sys_id=67cdfcd3db4c985c2be0a851ca96193e)
* [GlideRecord & GlideAjax: Client-Side Vs. Server-Side](https://snprotips.com/blog/2016/2/6/gliderecord-client-side-vs-server-side)
