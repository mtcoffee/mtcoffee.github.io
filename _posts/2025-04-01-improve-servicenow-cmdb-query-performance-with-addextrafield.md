---
title: Improve ServiceNow CMDB Query Performance with addExtraField()
---

# Summary
When building a CMDB integration or a complex report, you often need to pull in additional reference data using "dotwalking" to transform it into a CSV or another flat format. While dotwalking gets the job done, it can be costly in terms of performance.

A better approach? Use the new `addExtraField()` function.

[Robert Fedoruk has already covered](https://www.servicenow.com/community/developer-blog/quot-addextrafield-quot-amazing-performance-improvement-to/ba-p/3127640) this well, but we're putting an ITOM spin on it by querying some CMDB tables. The results speak for themselves—query times drop by 50% to 75%!
# Example
Run this sample background script and see if first hand!
<script src="https://gist.github.com/mtcoffee/7e1231e83f29e506e1d8086427d9e6b6.js"></script>
