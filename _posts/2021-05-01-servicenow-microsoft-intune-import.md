---
title: ServiceNow Microsoft Intune Import
---

This post will cover a  method used to import Microsoft Intune data in the CMDB. In this example, we will use the [Microsoft Graph API](https://docs.microsoft.com/en-us/mem/intune/developer/intune-graph-apis "Microsoft Graph API"). The official discovery plugin is [mapped here](https://docs.servicenow.com/bundle/quebec-servicenow-platform/page/product/configuration-management/concept/cmdb-integration-intune.html "mapped here"), which we will reference.

Shout out to Maverick Embry who has a [great post](https://mavembry.info/post/intune-servicenow-integration/) on this as well!
# Configure Intune API Access
To be begin, connect with the Azure admin to setup an application inside Azure to gain access to the API. They will need to use [setup a role assignment](https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles "setup a custom role assignment") to register a client application that can accesses the Azure Resource Manager REST API. 

**Important note for Intune:**

Make sure the app has the `DeviceManagementManagedDevices.Read.All` application & delegated permission.

The admin should provide you with:

**clientId**  
**clientSecret**  
**tenantId**  
**subscriptionId**
## Testing with Postman
This is optional, but to confirm access to the REST API, its best to [configure Postman](https://blog.jongallant.com/2019/04/azure-rest-apis-postman-in-no-time-flat/ "configure Postman"). This is also useful for troubleshooting and reviewing the data to be imported.

1.  Install [Postman](https://www.postman.com/downloads/ "Postman")
2.  Once installed, click "New Collection" and title it Azure
3.  Under the "pre-request scripts tab" paste the following code block
    
        pm.sendRequest({
            url: 'https://login.microsoftonline.com/' + pm.variables.get("tenantId") + '/oauth2/token',
            method: 'POST',
            header: 'Content-Type: application/x-www-form-urlencoded',
            body: {
                mode: 'urlencoded',
                urlencoded: [ 
                    {key: "grant_type", value: "client_credentials", disabled: false},
                    {key: "client_id", value: pm.variables.get("clientId"), disabled: false},
                    {key: "client_secret", value: pm.variables.get("clientSecret"), disabled: false},
                    {key: "resource", value: pm.variables.get("resource"), disabled: false}
                ]
            }
        }, function (err, res) {
            pm.globals.set("bearerToken", res.json().access_token);
        });
    
4.  Under the variable tab set the  
    clientId  
    clientSecret  
    tenantId  
    subscriptionId  
    resource: [https://graph.microsoft.com/](https://graph.microsoft.com/)  
5.  Save and close the collection
6.  Add an intuneDevices request - right click the collection and chose "Add Request"  
    * a.  Title "**intuneDevices**" and click save
    * b.  On the request set the url to "[https://graph.microsoft.com/v1.0/deviceManagement/managedDevices](https://graph.microsoft.com/v1.0/deviceManagement/managedDevices)"
    * c. On the Headers tab, create an **Authorization** header and set value to "**Bearer {{bearerToken}}**"
    * d.  Click send to test querying the subscription. This should provide a list of intune devices in JSON format

## Configure OAuth2.0 Profile in the Application Registry
1.  Navigate to System OAuth -> Application Registry
2.  Click New -> Connect to a third party OAuth Provider  
    * a.  Name: Microsoft Intune
    * b. Client ID: Provided by Azure Admin
    * c.  Client Secret: Provided by Azure Admin
    * d..  Default Grant type: client credentials
    * e.  Authorization URL:  
         `https://login.microsoftonline.com/---tenantId---/oauth2/v2.0/authorize`
        
        where tenantId is the tenant ID that we saved in the Azure configuration.
    * f. Token URL: Set this to  
    `https://login.microsoftonline.com/---tenantId---/oauth2/v2.0/token`
          
        where tenantId is the tenant ID that we saved in the Azure configuration
3.  Go to the Hamburger Menu and click save.
4.  Scroll down to "OAuth Entity Scopes" tab
5.  Click on insert a new role and set both "name" and "Oauth scope" to  
    `https://graph.microsoft.com/.default`
6.  Switch to the "OAuth Entity Profiles" tab
7.  Click on the default profile.
8.  Insert a new row and set "OAuth Entity Scope" to
		 `https://graph.microsoft.com/.default`
9.   Click update.

## Test with Glide script
1.  Navigate to System Definition-> Scripts - Background and paste in the following test script  
    
        var oAuthClient = new sn_auth.GlideOAuthClient();
        var OAuthProvider = "Microsoft Intune"
        var params = {grant_type:"client_credentials",scope:"https://graph.microsoft.com/.default"};
        var tokenResponse = oAuthClient.requestToken(OAuthProvider,global.JSON.stringify(params));
        var token = tokenResponse.getToken();
        var r = new sn_ws.RESTMessageV2();
        r.setRequestHeader('Authorization','Bearer ' + token.getAccessToken());
        gs.log("AccessToken:" + token.getAccessToken());
        gs.log("AccessTokenExpiresIn:" + token.getExpiresIn());
    
2.  If successful, the script should return an access token and expiry   

## Configure Outbound REST messages
1.  Navigate to System Web Services -> Outbound -> REST Message
2.  Click New  
    1.  Name: Microsoft Intune
    2.  EndPoint:  
        
            https://graph.microsoft.com
        
3.  Authentication type: OAuth 2.0
4.  OAuth profile: Microsoft Intune default\_profile
5.  Go to the Hamburger Menu and click save.
6.  Under "related links" click "Get OAuth Token" and confirm that a window pops up with "OAuth token flow completed successfully"  .  *If you get an error about "./default" check the syntax of the OAuth Entity Scope. i.e. type it out*
7.  Under HTTP Methods, click New  
    1.  Name: GetDevices
    2.  Endpoint:  
       
            https://graph.microsoft.com/v1.0/deviceManagement/managedDevices
    
8.  Go to the Hamburger Menu and click save.
9.  Under "related links" click "Test" and confirm that you get a JSON response with a list of Intune Devices

## Test with Glide script
1. Navigate to System Definition-> Scripts - Background and paste in the following test script  
    
         try { 
         var r = new sn_ws.RESTMessageV2('Microsoft Intune', 'GetDevices');
        
         var response = r.execute();
         var responseBody = response.getBody();
         var httpStatus = response.getStatusCode();
         gs.info('Device List Unparsed: ' + responseBody)
         
         //parse json
         var parsedResponse = JSON.parse(responseBody);
         var obj = parsedResponse.value
         //gs.info(parsedResponse.value);
         for (var device in obj) {
         gs.info('Device Name: ' + obj[device].deviceName + ' DeviceOwnerShip: ' + obj[device].managedDeviceOwnerType) ;
         }
        }
        catch(ex) {
         var message = ex.message;
        }
    
      
    
2.  If successful, the script should return an list of devices.  
   
You may notice in the output that the return count is capped at 1000. Microsoft Azure does this by design. [See Paging](https://docs.microsoft.com/en-us/graph/paging "See Paging")** To address this we can tell the script to fetch each page until there are no more.
## Test with Glide script -  full results

First we can add a block to see if we're getting paged results.

     try { 
     var r = new sn_ws.RESTMessageV2('Microsoft Intune', 'GetDevices');
    
     var response = r.execute();
     var responseBody = response.getBody();
     var httpStatus = response.getStatusCode();
     gs.info('Device List Unparsed: ' + responseBody)
     
     //parse json
     var parsedResponse = JSON.parse(responseBody);
     var obj = parsedResponse.value
    
    //check if we are paging
     if (parsedResponse["@odata.nextLink"]) { // if it has paged results
               gs.info("yes it does have paged results");
               var ojb2 = parsedResponse["@odata.nextLink"]
               gs.info(ojb2);
            }
    
    
     //gs.info(parsedResponse.value);
     for (var device in obj) {
     gs.info('Device Name: ' + obj[device].deviceName + ' DeviceOwnerShip: ' + obj[device].managedDeviceOwnerType) ;
     }
    }
    catch(ex) {
     var message = ex.message;
    }​

Next instruct the job to loop through the page results and return them all. **(careful, this might take a while!)**

    try {
    
        var endpoint = null;
        var loop = true;
    
        while (loop) {
    
            var r = new sn_ws.RESTMessageV2('Microsoft Intune', 'GetDevices');
    
            if (endpoint !== null) {
                r.setEndpoint(endpoint);
            }
    
            var response = r.execute();
            var responseBody = response.getBody();
            var httpStatus = response.getStatusCode();
            gs.info('Device List Unparsed: ' + responseBody)
    
            //parse json
            var parsedResponse = JSON.parse(responseBody);
            var obj = parsedResponse.value
    
    
            //gs.info(parsedResponse.value);
            for (var device in obj) {
                gs.info('Device Name: ' + obj[device].deviceName + ' DeviceOwnerShip: ' + obj[device].managedDeviceOwnerType);
            }
    
            if (parsedResponse["@odata.nextLink"]) {
                // if it has next link
                endpoint = parsedResponse["@odata.nextLink"];
            } else {
                loop = false;
            }
        }
    } catch (ex) {
        var message = ex.message;
    }

# Create a data source and load data into staging tables

To load the intune data from a REST script:

1.  Navigate to **System Import Sets -> Data Sources**
2.  **Click New**  
    1.  Name: IntuneDevices
    2.  Import set table label: IntuneDevices
    3.  Type: Custom (Load by Script)
3.  Paste in the script below   
    
        (function loadData(import_set_table) {
        
            // Add your code here to insert data to import_set_table
            var endpoint = null;
            var loop = true;
        
            while (loop) {
        
                var r = new sn_ws.RESTMessageV2('Microsoft Intune', 'GetDevices');
        
                if (endpoint !== null) {
                    r.setEndpoint(endpoint);
                }
        
                var response = r.execute();
                var responseBody = response.getBody();
                var httpStatus = response.getStatusCode();
        
                //parse json
                var parsedResponse = JSON.parse(responseBody);
                var obj = parsedResponse.value;
        
        
                //gs.info(parsedResponse.value);
                for (var device in obj) {
        		if (obj[device].managedDeviceOwnerType == 'company'){
                    var map = {
                        u_name: obj[device].deviceName,
                        u_managedDeviceOwnerType: obj[device].managedDeviceOwnerType,
                        u_userdisplayname: obj[device].userdisplayname,
                        u_userPrincipalName: obj[device].userPrincipalName,
                        u_deviceEnrollmentType: obj[device].deviceEnrollmentType,
                        u_complianceState: obj[device].complianceState,
                        u_operatingSystem: obj[device].operatingSystem,
                        u_osVersion: obj[device].osVersion,
                        u_serialnumber: obj[device].serialNumber,
                        u_model: obj[device].model,
                        u_manufacturer: obj[device].manufacturer,
                        u_wiFiMacAddress: obj[device].wiFiMacAddress,
                        u_deviceRegistrationState: obj[device].deviceRegistrationState,	
                        u_id: obj[device].id,
                        u_imei: obj[device].imei
                    };
                    import_set_table.insert(map);
        			}
                }
        
                if (parsedResponse["@odata.nextLink"]) {
                    // if it has next link
                    endpoint = parsedResponse["@odata.nextLink"];
                } else {
                    loop = false;
                }
            }
        
        })(import_set_table);
    
      
4.  Go to the Hamburger Menu and click save.
5.  Under related links, click "Test load 20 records" to test the import.
6.  If successful, you should receive a message like this and staging data is now loaded in "IntuneDevices" temp table.  
    ![](https://queensu.service-now.com/sys_attachment.do?sys_id=cb397945dbaf24504921690913961969)

# Create transforms for each source and target table
Once we have successfully loaded some sample data into our staging table we can create a transform map. This is a mapping of fields from our source to our target table. Transforms can also use scripts, to determine what to write to a target field. This can be useful for data conversion or writing predefined data to a field.
## Computers Transform
1.  Navigate to **System Import Sets -> Run Transform -> Administration -> Transform Maps** 
2.  Set name to something relevant like **IntuneComputerImport**
3.  Set source table to our staging table 
4.  Set target to **cmdb\_ci\_computer**
5.  Go to Hamburger menu and click Save.
6.  Start with "**Auto Map Matching Fields**" to quickly create field maps for name and manufacturer.
7.  Go to the source map **u\_name**/**name** and set the **coalesce** field to true. This checks for an existing record in the target table that has the same value in the Target field as the import set row Source field. If an existing record with a matching value in the target table is found, that record is updated. If no matching record is found, then a new record is created in the target table.
8.  Now we need to create our field map. Click on mapping assist under related links. Create the field map and use this table as the reference.  
    
|Source Field|Target Field|Coalesce|
|--- |--- |--- |
|name|name|Yes|
|manufacturer|manufacturer|No|
|wifimacaddress|mac_address|No|
|os_version|os_version|No|
|model|model_id|No|
|operatingsystem|os|No|
|id|object_id|No|
|userprincipalname  (choice action = ignore)|assigned_to|No|
|serialnumber|serial_number|No|
|script|discovery_source|No|
|script|last_discovered|No|
|compliancestate|intune_compliance (custom field)|No|
|deviceregistrationstate|intune_deviceregistrationstate (custom field)|No|
|deviceenrollmenttype|intune_deviceenrollmenttype (custom field)|No|
      
    Script for Discovery Source
    
        //set choice action to create
        answer = "IntuneImport"; 
    
    **Script for last\_discovered aka Most recent discovery  
    **
    
        answer = gs.nowDateTime();
    
1.  Now we need to add add 3 transform scripts:  
    a.  Create an **OnBefore** script and add the snippet below. This script will ensure we only import the listed OS's and only of the intune ID is not null.  
        
                // Add your code here
                //criteria for import into computer table
                if (((source.u_operatingsystem != 'macOS') && (source.u_operatingsystem != 'Windows')) || source.u_id.nil()) {
                    ignore = true;
                }​
        
          
        
    b.  Create an **onComplete** script and add the snippet below. This will ensure that CI's are marked as retired after they fall out of the feed. It will check with the last\_discovered field, and if it is older than an hour, set operational status as retired  
        
            //Find all records in the computers table not updated by import set in X hours and set to retired	
            var gr = new GlideRecord('cmdb_ci_computer');
            //add any filters here to limit what you want to retire
            gr.addQuery('last_discovered', "<", gs.minutesAgo(60));
            gr.addQuery('discovery_source', '=', 'IntuneImport');
            gr.query();
            while (gr.next()) {
                //gs.print(gr.name + ' will be retired'); 
                gr.operational_status = 6;
                gr.update();
            }​
        
    c.  Now we should also create an **onComplete** script to restore the CI to operational, incase it returns to feed  
        
            //Find all records in the computers table not updated by import set in X hours and set to retired	
            var gr = new GlideRecord('cmdb_ci_computer');
            //add any filters here to limit what you want to retire
            gr.addQuery('last_discovered', ">", gs.minutesAgo(60));
            gr.addQuery('discovery_source', '=', 'IntuneImport');
            gr.query();
            while (gr.next()) {
                //gs.print(gr.name + ' will be retired'); 
                gr.operational_status = 1;
                gr.update();
            }
        
          
        

Now run your first data load.

1.  Navigate to Data Sources -> IntuneDevices
2.  Under related records click "Load All Records"
3.  Upon successful completion, click "run transform"
4.  Run the default map and click transform
5.  Data should now be loaded into the **cmdb\_ci\_computer** table. Logs are available under "Import Log" and results can be viewed on the table.

## Mobile Device Transform

1.  Navigate to **System Import Sets -> Run Transform -> Administration -> Transform Maps** 
2.  Set name to something relevant like **IntuneMobileImport**
3.  Set source table to our staging table 
4.  Set target to **cmdb\_ci\_handheld\_computing**
5.  Go to Hamburger menu and click Save.
6.  Start with "**Auto Map Matching Fields**" to quickly create field maps for name and object\_id.
7.  Go to the source map **u\_name**/**name** and set the **coalesce** field to true. This checks for an existing record in the target table that has the same value in the Target field as the import set row Source field. If an existing record with a matching value in the target table is found, that record is updated. If no matching record is found, then a new record is created in the target table.  
    
8.  Now we need to create our field map. Click on mapping assist under related links. Create the field map and use this table as the reference.  
    
|Source Field|Target Field|Coalesce|
|--- |--- |--- |
|name|name|Yes|
|manufacturer|manufacturer|No|
|wifimacaddress|mac_address|No|
|os_version|os_version|No|
|model|model_id|No|
|operatingsystem|os|No|
|id|object_id|No|
|userprincipalname (choice action = ignore)|assigned_to|No|
|serialnumber|serial_number|No|
|imei|imei|No|
|script|discovery_source|No|
|script|last_discovered|No|
|compliancestate|intune_compliance (custom field)|No|
|deviceregistrationstate|intune_deviceregistrationstate (custom field)|No|
|deviceenrollmenttype|intune_deviceenrollmenttype (custom field)|No|
    
      
    **Script for Discovery Source**  
    
        //set choice action to create
        answer = "IntuneImport"; ​
    
    **Script for last\_discovered aka Most recent discovery  
    **
    
        answer = gs.nowDateTime();
    
1.  Now we need to add add 3 transform scripts:  
    a.  Create an **OnBefore** script and add the snippet below. This script will ensure we only import the listed OS's and only of the intune ID is not null.  
        
            // Add your code here
                //do not import into import non mobile devices into mobile table
                if (((source.u_operatingsystem != 'iOS') && (source.u_operatingsystem != 'Android')) || source.u_id.nil()) {
                    ignore = true;
                }
        
       
        
    b.  Create an **onComplete** script and add the snippet below. This will ensure that CI's are marked as retired after they fall out of the feed. It will check with the last\_discovered field, and if it is older than an hour, set operational status as retired  
        
            //Find all records in the mobile table not updated by import set in X hours and set to retired	
            var gr = new GlideRecord('cmdb_ci_handheld_computing');
            //add any filters here to limit what you want to retire
            gr.addQuery('last_discovered', "<", gs.minutesAgo(60));
            gr.addQuery('discovery_source', '=', 'IntuneImport');
            gr.query();
            while (gr.next()) {
                //gs.print(gr.name + ' will be retired'); 
                gr.operational_status = 6;
                gr.update();
            }
        
    c.  Now we should also create an **onComplete** script to restore the CI to operational, incase it returns to feed  
        
            //Find all records in the mobile table not updated by import set in X hours and set to retired	
            var gr = new GlideRecord('cmdb_ci_handheld_computing');
            //add any filters here to limit what you want to retire
            gr.addQuery('last_discovered', ">", gs.minutesAgo(60));
            gr.addQuery('discovery_source', '=', 'IntuneImport');
            gr.query();
            while (gr.next()) {
                //gs.print(gr.name + ' will be retired'); 
                gr.operational_status = 1;
                gr.update();
            }​​
        

Now run your first data load.

1.  Navigate to Data Sources -> IntuneDevices
2.  Under related records click "Load All Records"
3.  Upon successful completion, click "run transform"
4.  Run the default map and click transform
5.  Data should now be loaded into the **cmdb\_ci\_handheld\_computing** table. Logs are available under "Import Log" and results can be viewed on the table.

# Schedule Each Import

Imports from these sources can be scheduled to regularly fetch and import the data.

1.  Navigate to **System Import Sets -> Administration -> Scheduled Imports**
2.  Set name to something relevant like **IntuneDevices**
3.  Set data source to our our existing source created earlier **IntuneDevices**
4.  Set Run as to "system administrator" or whichever user you want to show has the updated\_by
5.  Set scheduled (default is midnight daily)
6.  Go to Hamburger menu and click Save.

Repeat this for each data source and optionally run "Execute Now" to test the result.  To review your imported data go to System Import Sets -> Import Set Tables -> IntuneDevices.

*Also, be sure that the necessary monitoring is in place to notify the SN Admin team in the event an error on import.*
## Update Form Layouts and Related lists
To improve the visibility of the imported data, we will add additional fields to the form for the **cmdb\_ci\_computer** table. Open Form Layout/Form Designer and use the result capture further down as a reference.
# UI Action to Open In Intune

We can also add a UI Action button to view the device in Microsoft Intune, using the Object ID from the current record.

1.  Right click the header on any Computer record and choose Configure --> UI Action
2.  Set the following values  
    1.  Name = Open in Intune
    2.  Client = True
    3.  Form Button = True
    4.  Form Style = Primary
    5.  Onclick = openRecordInIntune()
    6.  Condition = current.object\_id.hasValue()  
        
3.  Set the script to this block:  
    
        function openRecordInIntune() {
            var url = 'https://endpoint.microsoft.com/#blade/Microsoft_Intune_Devices/DeviceSettingsBlade/overview/mdmDeviceId/' + g_form.getValue('object_id');
            g_navigation.openPopup(url);
        }
    
The button should only display when the object_id field has a value.

# Result
![]({{ 'assets/images/intune.PNG' | relative_url }})
