---
title: SharePoint API and Postman
---

I had a need to post data from a ServiceNow Request Item to a SharePoint List. To accomplish this I needed to understand the REST API options for SharePoint. There are a couple of options:

* [Microsoft Graph API](https://docs.microsoft.com/en-us/sharepoint/dev/apis/sharepoint-rest-graph)
* [SharePoint REST API](https://docs.microsoft.com/en-us/sharepoint/dev/sp-add-ins/complete-basic-operations-using-sharepoint-rest-endpoints)

While Microsoft does steer you in the direction of graph, it seems to have a few limitations still. Most notably, you cannot set read/write permission on the site or list level. It's all or nothing! For that reason we'll stick with the tried and true API until Microsoft is able to achive parity. Lets's get started.
# Create a new client id and key, aka Register Add-In
*Alternatively this can be done in Azure Portal.*
1. Open the the appregnew.aspx link on you SharePoint site for example:
https:/mycompany.sharepoint.com/teams/myteam/_layouts/15/appregnew.aspx
2. Generate new id/secret and click create  
![]({{ 'assets/images/sharepoint1.PNG' | relative_url }})
3. Record the values and save for later

# Grant permissions to new client id, aka Add-In # 
1. Open the the appinv.aspx link on you SharePoint site for example:
https:/mycompany.sharepoint.com/teams/myteam/_layouts/15/appinv.aspx
2. Copy in your Client ID that was registered and click lookup
3. Add in the following xml to grant full permission to this Client ID


```
<AppPermissionRequests AllowAppOnlyPolicy="true">  
   <AppPermissionRequest Scope="http://sharepoint/content/sitecollection" 
    Right="FullControl" />
</AppPermissionRequests>
```

**Example:**  
![]({{ 'assets/images/sharepoint2.PNG' | relative_url }})

To review these at any time  
https:/mycompany.sharepoint.com/teams/myteam/_layouts/15/appprincipals.aspx

For more info from Microsoft see [Granting access using SharePoint App-Only](https://docs.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-apponly-azureacs) and [Add-in permissions in SharePoint](https://docs.microsoft.com/en-us/sharepoint/dev/sp-add-ins/add-in-permissions-in-sharepoint)
# Configure Postman Authentication #
Configure Postman to connect using the new client id and key. Before you begin you'll need the following information.

| Variable: | Value | 
| -------- | -------- | 
| Client Id:     |  (client secret)      | 
| Client Secret:     | (client secret)     | 
| tenantId:     | (azure tennant id)     | 
| resource:     | 00000003-0000-0ff1-ce00-000000000000/mydomain.sharepoint.com@tennantid     | 

1. In post click new and create a new "Collection"
2. Fill in the variables. It should look similar to this. (don't worry, these aren't real values)  
![]({{ 'assets/images/postmansp1.PNG' | relative_url }})  
3. In the pre-request script tab use the script below:  
```
pm.sendRequest({
    url: 'https://accounts.accesscontrol.windows.net/' + pm.variables.get("tenantId") + '/tokens/OAuth/2',
    method: 'POST',
    header: 'Content-Type: application/x-www-form-urlencoded',
    body: {
        mode: 'urlencoded',
        urlencoded: [ 
            {key: "grant_type", value: "client_credentials", disabled: false},
            {key: "client_id", value: pm.variables.get("clientId") + '@' + pm.variables.get("tenantId"), disabled: false},
            {key: "client_secret", value: pm.variables.get("clientSecret"), disabled: false},
            {key: "resource", value: pm.variables.get("resource"), disabled: false}
        ]
    }
}, function (err, res) {
    pm.globals.set("bearerToken", res.json().access_token);
});
```

# Query all lists
1. Right click the header of the collection and choose "Add request"
2. Add the following header values under the headers tab  


	| HeaderKey: | Value | 
	| -------- | -------- | 
	| Authorization     |  Bearer {{bearerToken}}    | 
	| Accept    |  application/json;odata=verbose   | 

3. Set the type to get and the value to https://mycompany.sharepoint.com/teams/myteam/_api/web/lists/
4. Click send. Sample result:  
![]({{ 'assets/images/postmansp2.PNG' | relative_url }})

# Posting to a list
It's tricky to go in to detail since every list is different but here is a primer.
* I have a list titled "Asset Manager"
* I want to add a new item to this list

1. Right click the header of the collection and choose "Add request"
2. Add the following header values under the headers tab  


	| HeaderKey: | Value | 
	| -------- | -------- | 
	| Authorization     |  Bearer {{bearerToken}}    | 
	| Accept    |  application/json;odata=verbose   | 
	| Content-Type    |  application/json;odata=verbose   | 
	
3. Set the type to POST and the value to *https://mycompany.sharepoint.com/teams/myteam/api/web/lists/GetByTitle('Asset manager')/items*
4. Set the body to  the code snippet below
```
{
  "__metadata": {
    "type": "SP.Data.Asset_x0020_managerListItem"
  },
  "Title": "my test item"
}
```
5. Click send. Sample result:  
![]({{ 'assets/images/postmansp3.PNG' | relative_url }})  

As expected, the new item is in the SharePoint List!
