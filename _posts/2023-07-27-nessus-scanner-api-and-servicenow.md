---
title: Nessus Scanner API and ServiceNow
---

While doing some sandboxing with Nessus Scanner, I was curious if it could be integrated into ServiceNow using the Tenable Integration. It cannot because the Tenable API has dedicated paths for assets and vulnerabilities. Nessus Scanner API doesn't have these paths, so you have to get creative. To get started I was able to put together a background script that will print a list of hosts and associated vulnerabilities.

# Sample Script
<script src="https://gist.github.com/meatsac/f458b46ab75ba92f15bfca8fc0156b3a.js"></script>

Sample Result  
![]({{ 'assets/images/nessus.png' | relative_url }})

That's all for now.
