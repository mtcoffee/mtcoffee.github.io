---
title: Use REST action POST with ServiceNow Flow Designer
---

# Summary 
A quick example of using the Flow Designer action REST step to POST a message. This is great for sending information to remote systems, for example a catalog item that needs to create a new user in a remote service that can be updated via REST API. In this example we can create a POST action that will connect to our own instance and POST to the ServiceNow Incident API to create an incident.

# Create A Flow Designer Action

1. Create a new Flow Designer Action
2. Set an input of "short_description" with type string
3. Add a rest step
    *  Create a basic credential alias with username/password of a user with sn_incident_write role
    *  Set base url: https://instancename.service-now.com
    *  Resource Path: /api/now/table/incident
    *  Method: POST
    *  Header1 - Content-Type: application/json
    *  Header2 - Accept: application/json
4. Use Pill Picker to test Request Body  {"short_description":"short_description"}

Example:
![]({{ 'assets/images/fd_rest_post.PNG' | relative_url }})

That's it! Click test and if all is well, you'll have a new incident created.
