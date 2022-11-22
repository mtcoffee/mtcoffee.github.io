---
title: Automated Mid Server Install
---

# Summary
Installing a MID server on Windows while easy can present challenges at times. In recent releases, you can no longer use the local system account to run the service so it can be frustrating to install. ServiceNow has a silent install PowerShell script but it was missing a few things so I created an extension of it. 

The original script from ServiceNow is embedded in a function. All you need to do is set the variables and run the script. It will download the msi, configure the service account and run the install.
# Script
<script src="https://gist.github.com/meatsac/3486ed488401ec4a8995b94a102a96eb.js"></script>
