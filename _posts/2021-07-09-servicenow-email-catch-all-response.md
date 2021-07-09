---
title: ServiceNow Email Catch All Response
---

# The Problem
Out of the box, emails replies sent to Request or Service Catalog items, are ignored by the system with no automatic response to the user, resulting in frustrated users/poor service.

# The Solution
To address this we can create a "catch all" response to notify users that their message was not processed.

1. Navigate to System Notification -> Email -> Notifications
2. Click New
3. Set name and table
    * NName = MY - Email Catchall and AutoReply on ignored
    * Table = Email [sys_email]
4. When to send tab
    * "Record inserted or updated" (check both boxes)
    * "Mailbox is Received"
    * "State changes to Ignored"
    * "Error string does not contain sys_email" (This will prevent "Infinite Email" Loops)
5. Who will receive tab
    * Users/Groups in fields = "User"
6. What will it contain
    * Subject: RE: ${subject}
    * Message HTML:

```
Unfortunately, your email message "${subject}" to the ServiceNow system could not be processed. Please login to the Service Portal to follow up on your active requests.  

---Previous Message---

${body_text}
```
All ignored email should now receive an an automatic response.
