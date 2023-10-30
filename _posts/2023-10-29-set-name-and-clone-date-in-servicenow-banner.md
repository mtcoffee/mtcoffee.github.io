---
title: Set Name and Clone Date in ServiceNow Banner
---

# Challenge
A while ago I [shared a script](https://mtcoffee.github.io/servicenow-clone-automation/) for setting the instancename and clone details in the header string. This still works but doesn't present as clearly in Polaris as it did in UI16. To get around this, I found a way to generate a ServiceNow banner logo from script using svg. 

# Solution
Use the background script as a clone script in your clone profile or just run it manually as needed.

<script src="https://gist.github.com/mtcoffee/d26b96832e31cd65656d8df4da2d9d75.js"></script>

Sample Result  
![]({{ 'assets/images/acmedev.PNG' | relative_url }})

SVG's are quite powerful! You can even make dynamic images that follow your CSS! [See this talk from GF.](https://youtu.be/iBfpgcusda4?si=oi22UCVgTLO_QvdZ&t=217)
