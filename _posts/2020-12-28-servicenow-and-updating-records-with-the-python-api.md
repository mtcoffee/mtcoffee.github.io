---
title: ServiceNow and updating records with the Python API
---

# The Challenge
Sometimes its nice to update or close records in large sets. For example, in testing I like to be able to cleanup and start fresh by closing all incidents of specific criteria. Enter the ServiceNow Python API! I needed a way to close all incidents based on a query and Python will do just that.

Once again, how can we make this easy? Docker! With this “one liner” we will:

1. Call the official python:alpine docker image from docker hub
2. Install the python requests module on the fly
3. Execute the Python script to login to our ServiceNow instance and close all incidents that match our filter.

# The Solution
Here’s the shell script. Just set your instance url, user and password.

```
docker run -i --rm --user root --name pythoncontainer python:alpine /bin/sh -c \
'pip3 install requests && \
echo ###start of python script#### && \
python' <<EOF 
# testing
print('start of python job')

#Need to import requests package for python
import requests
import xml.etree.ElementTree as ET

# Set the request parameters
sninstance = 'https://dev12345.service-now.com'
sninstanceuser = 'ITILBOB'
sninstancepwd = 'ITILBOBSPASSWD'
tableurl = sninstance + '/api/now/table/incident'
filter = '?sysparm_query=active=true^short_descriptionCONTAINStest'
url = tableurl + filter

# Set proper headers
headers = {"Accept":"application/xml"}

# Do the HTTP request
response = requests.get(url, auth=(sninstanceuser, sninstancepwd), headers=headers)

# Check for HTTP codes other than 200
if response.status_code != 200: 
    print('Status:', response.status_code, 'Headers:', response.headers, 'Error Response:', response.content)
    exit()

# Decode the XML response into a dictionary and use the data
#print(response.content)

####for each incident found, close####
root = ET.fromstring(response.content)
for sys_id in root.iter('sys_id'):
    print(sys_id.tag, sys_id.text)
    sys_id = sys_id.text
    tableurl = sninstance + '/api/now/table/incident/'
    updateurl = tableurl + sys_id
    print('Attempting to close incident: ' + updateurl + '\n')
    
    # Set proper headers
    headers = {"Content-Type":"application/xml","Accept":"application/xml"}
    response = requests.put(updateurl, auth=(sninstanceuser, sninstancepwd), headers=headers, data="<request><entry><close_code>Closed/Resolved By Caller</close_code><state>7</state><work_notes>Closing</work_notes><close_notes>Closing</close_notes></entry></request>")
    print(response.content)
EOF
```
I like to combine jobs like this with [Rundeck](https://www.rundeck.com/open-source), so that I can do this cleanup in my Personal Development Instance with a single button click!
