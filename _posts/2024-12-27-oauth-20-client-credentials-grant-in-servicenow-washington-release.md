---
title: OAuth 2.0 Client Credentials Grant in ServiceNow (Washington Release)
---

# OAuth 2.0 Client Credentials Grant in ServiceNow (Washington Release)

In the **ServiceNow Washington release**, the platform introduces support for the **Client Credentials Grant** in OAuth 2.0, marking a significant advancement in security and usability. This grant type provides several key benefits:

1. **No Need for Sys_User Credentials**: Unlike other OAuth 2.0 flows, the remote party no longer needs to log in with **sys_user credentials** before authentication.
2. **Enhanced Security**: The Client Credentials Grant aligns with the OAuth 2.0 security specifications, eliminating the need for the less secure **Password Grant**. In fact, the **Password Grant** should be avoided entirely as per OAuth guidelines: [OAuth 2.0 Password Grant](https://oauth.net/2/grant-types/password/).

For more details on setting up OAuth Client Credentials in ServiceNow, refer to the official documentation: [KB1645212](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1645212).

## Example PowerShell Script Setup

Here’s a simplified process for configuring OAuth Client Credentials in ServiceNow:

1. **Enable Client Credentials Grant**:
   - Set the property `glide.oauth.inbound.client.credential.grant_type.enabled` to **true**.

2. **Create OAuth API Endpoint**:
   - Navigate to **System OAuth > Application Registry**.
   - Click **New**, and select **Create an OAuth API endpoint for external clients**.
   - Set the **Name** and **Comment** for the OAuth application, then save the record (the **Client Secret** will be auto-generated).

3. **Configure OAuth Application Settings**:
   - Open the **oauth_entity.list** and add the following fields:
     - **Default Grant Type**
     - **OAuth Application User**
   - Set **Default Grant Type** to **Client Credentials**.
   - Set **OAuth Application User** to any user with **ITIL** access.

4. **Retrieve Client Secret**:
   - Open the OAuth application record and click the **lock icon** to retrieve the **Client Secret**.

<script src="https://gist.github.com/mtcoffee/b43cf995a7dcd05e55d3010bd2900b09.js"></script>
