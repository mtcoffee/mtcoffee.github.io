---
title: ServiceNow Customize Reference Field Pop Up
toc_sticky: false
toc: false
---

Adding new reference fields to the pop up view is something that comes up from time and is a much easier that you might expect to configre. A simple form layout!

1. Go to the sys_user table and open a record
2. Configure -> Form Layout
3. Under View New name, create a new view "sys_popup"
4. Add/Remove the fields you want from this view

Now when you go to record the caller reference will contain different view than the default.    
![]({{ 'assets/images/sys_popup.PNG' | relative_url }})

Also see this community post!   
[How to customize fields displayed on the reference field pop up form using a sys_popup view.](https://community.servicenow.com/community?id=community_blog&sys_id=f81f859d1b837b08ada243f6fe4bcb92)
