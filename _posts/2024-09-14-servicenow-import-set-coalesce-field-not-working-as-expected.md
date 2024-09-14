---
title: ServiceNow Import Set Coalesce Field not working as expected?
---

# Summary
Recently I ran into an issue with a ServiceNow Import Set that was configured to coalesce on a single field. The target system contained an Incident record with a correlation_id set. I then sent a payload to the target system with the correlation_id set in the payload, however the target system created a new record instead of matching on the existing record.

# The Cause
Security; however, with a twist! In ServiceNow you can control access to records with both ACL's and Query Business Rules. In this case the customer had a query business rule with very specific field requirements that prevented the API user account from seeing the Incident record. As a result it created a new record instead of matching on the existing record.

> In short, if your Coalesce is not working, make sure your API user account has read/write access to the target record. If it doesn't have read, it  doesn't know the record is there and will create a new one.
>
