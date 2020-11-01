---
title: Creating an RSS Portal Widget in ServiceNow
---

Surprisingly, there is no built in Portal Widget for RSS feeds in ServiceNow. This is particuarly useful if you want to display information on the Service Portal from external sources, like the company website or an HR Services. Fortunately this is an easy one to create. In this example we will use a well known ServiceNow Blog as the source.

# Querying RSS feeds
To pull content from an RSS feed into ServiceNow, we use Outbound Rest Messages. *Ideally, this should be configured under System Web Services -> Outbound -> REST message, but for this purpose we'll add to our glide script. If the target required authentication, then this would make more sense*

In our [personal dev instance](https://developer.servicenow.com/dev.do#!/guides/orlando/now-platform/pdi-guide/personal-developer-instance-guide-introduction), lets go to System Defintion -> Scripts - Background and run the script below.

```javascript
 var r = new sn_ws.RESTMessageV2();
 r.setHttpMethod('GET');
 r.setEndpoint('https://developer.servicenow.com/blog.do?p=/authors/chuck-tomasi/index.xml');

 var response = r.execute();
 var responseBody = response.getBody();
 var httpStatus = response.getStatusCode();

//gs.info (responseBody)

var xmldoc = new XMLDocument(responseBody);
var title = xmldoc.getNodeText("//title");
gs.info (title);

var helper = new XMLHelper(responseBody);
var obj = helper.toObject();

//set tree level you want to loop over
var mytree = obj['channel'].item;

//put each data item on its own row with all keys
data = mytree
var keys = Object.keys(data); 
keys.forEach( function(key) {
  var values = data[key]
  // do stuff with "values"
  gs.info(JSON.stringify(values))
})
```

This will fetch and convert the XML into objects, making it much easier to parse. It should give us an output with a list of posts from the feed. Now we're ready to create a widget.
# Create a New Portal Widget
1. Open the widget editor and create a new blank widget - Instructions [here](https://docs.servicenow.com/bundle/paris-servicenow-platform/page/build/service-portal/task/create-new-widget.html)  
2. Check off the boxes for HTML Template and Server Script 
3. In the top right corner, click the "hamburger menu" and enable preview and public widget
4. Under HTML Template paste the code below    

```javascript
<div class="panel panel-primary" >
  <div class="panel-heading">
          {% raw %}<h2 class="h4 panel-title ng-binding">{{c.data.title}}</h2>{% endraw %}
  </div>
      <div class="list-group">
          <div ng-repeat="item in data.items | limitTo: 5" class="list-group-item">
              {% raw %}<a ng-href={{item.link}} target="_blank"><h5>{{item.title }} </h5></a>
              <!-- <div class="item-content">{{ item.description }}</div> --> {% endraw %}
          </div>
      </div>
  </div>
```  

{:start="5"}
5. Under Server Script paste the code below 

```javascript
(function() {
try {

 var r = new sn_ws.RESTMessageV2();
 r.setHttpMethod('GET');
 r.setEndpoint('https://developer.servicenow.com/blog.do?p=/authors/chuck-tomasi/index.xml');

 var response = r.execute();
 var responseBody = response.getBody();
 var httpStatus = response.getStatusCode();

var xmldoc = new XMLDocument(responseBody);
var title = xmldoc.getNodeText("//title");
data.title = title

var helper = new XMLHelper(responseBody);
var obj = helper.toObject();

var items = obj.channel.item;
data.items = items
	
}
catch(ex) {
        var message = ex.getMessage();
}
})();
```

Click "Save" and observe what your widget will appear like in the preview.

# Add the Widget to your Portal
1. From the top of the Widget editor page, click "Designer"
2. Find your service portal page, usually "Service Portal index" and open it
3. Drag and drop your widget from the left pane.

Sample result:  
![]({{ 'assets/images/rss.PNG' | relative_url }})

For more detail on Widgets, see the [Tutorial: Build a custom widget](https://docs.servicenow.com/bundle/paris-servicenow-platform/page/build/service-portal/concept/adv-widget-tutorial.html).
