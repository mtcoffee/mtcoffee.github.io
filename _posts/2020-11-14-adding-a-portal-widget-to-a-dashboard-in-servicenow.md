---
title: Adding a portal widget to a dashboard in ServiceNow
---

In my previous post we created an RSS widget for use in the Service Portal. Rather than developing another widget for the platform ui Dashboards, can't we just use portal widget in the Dashboard? Yes we can!

I discovered the solution on the SN community [here](https://community.servicenow.com/community?id=community_article&sys_id=c48db33ddb356700107d5583ca9619a5) . Here is my version of the steps:
# Create a portal page and add your widget to it
1. Navigate to Service Portal --> Pages
2. Click New to create a new page  
```
		Title:BlogRSS 
		ID:blogrss
		Short description:BlogRSS  
```  

4. Go to the Hamburger Menu and Click save
5. Under Related Links "Open in Desiger" and add a #12 container
6. Find your widget and add it to the page

# Create a UI Page to contain the portal page
1. Navigate to System UI --> UI Pages
2. Click New to create a new page. 
3. Name =  BlogRSS
4. Paste the script below into the HTML field

```javascript
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
 <style>
        #preview{
            text-align: center;
        }
 </style>

     <div id="preview">     
       <iframe height="500" width="850" scrolling="no" src="/$sp?id=BlogRSS" />
    </div>
</j:jelly>
```
# Create the Widget and use the UI Page as the source for the Widget
1. Navigate to System UI --> Widgets
2. Click New to create a new widget and set name field to "BlogRSS"
4. Paste the script below into the HTML field

```javascript
function sections() {
    return {
		'BlogRSS': {'name': 'BlogRSS'}	
    };
}

function render() {
    var name = renderer.getPreferences().get("name");
    return renderer.getRenderedPage(name);
}

function getEditLink() {
    return "sys_ui_page.do?sysparm_query=name=" + renderer.getPreferences().get("name");
}
```

Now just find the "BlogRSS" Widget in the Widget picker for your Dashboard. Result:
![]({{ 'assets/images/blogrss.PNG' | relative_url }})
