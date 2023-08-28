---
title: ServiceNow GlideModalForm to Create a record from a record
---

# Summary
GlideModalForm is an often forgotten gem to open the new record form in a Modal. For example it can be handy to create a new Incident from a Configuration Item, with just a single click.

# UI Action Example

1. Create a new UI Action on the Window Server table
2. Form button = true
3. Isolate script = false (optional)
4. Client = true
5. Onclick = "loadModal()"
6. The script is below:
<script src="https://gist.github.com/mtcoffee/1b538d885499abf97f077ef1cac4bffd.js"></script>

Sample result:
![]({{ 'assets/images/GlideModalForm.PNG' | relative_url }})
