---
title: Get better descriptions on your ServiceNow records
---

# Summary
Sometimes the description field on an Incident or CSM Case can have less than ideal information supplied by the agent. To address this we can supply a pop up that will ask qualifying questions, and once completed will write to the description field, similar to a record producer. With a little bit of jquery, this is possible.

Result
![]({{ 'assets/images/UpdateDescription.gif' | relative_url }})
# Solution
We will use the Incident form for this example:
1. Make the description field read only
2. Create a new client UI action and clear the "isolate script" check box. This is required to confirm the modal has loaded.
3. Set the Onclick function to "showModal()"
4. Choose form button with your preferred title "Update Description"
5. Set the script with the script below:

<script src="https://gist.github.com/mtcoffee/f43670a63a8969405159b1855c7e4f0c.js"></script>
