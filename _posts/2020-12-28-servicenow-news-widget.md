---
title: ServiceNow News Widget
---

I came across the [KB News widget](https://docs.servicenow.com/bundle/paris-servicenow-platform/page/build/service-portal/concept/kb-news-widget.html) feature and found that even though its shown, its actually deprecated. The concept is simple, it's a  UI page, that displays the title of any KB article with the category "News". A handy way for teams to share their internal announcements.

To resurect this we can create similar version.
# Create a UI Page
Create a ui page that will display the title, and date of any KB article of category "News". Navigate to System UI -> Ui Pages and create a new ui page called "**my_news**" with the html/jellyscript below.
```
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
<g:evaluate object='true' >
var kb = new GlideRecord('kb_knowledge');
kb.adddQuery('active=true');
kb.addQuery('workflow_state', 'published');
kb.addQuery('kb_category.label', 'News');
kb.query();
</g:evaluate>
<table id="statsTable" class="table table-striped table-bordered  hide-query-stats">
<tr>
<th>Title</th>
<th>Unit</th>
<th>Date</th>
</tr>
<j:while test="${kb.next()}">

<tr>
<td><a target='_blank' href="kb_view.do?sys_kb_id=${kb.sys_id}"> ${kb.short_description}</a></td>
<td>${kb.kb_knowledge_base.title}</td>
<td>${kb.published}</td>
</tr>
</j:while>
</table>
</j:jelly>
```
# Create a Widget
Create a widget and nest the UI page into. (You can display any ui page as a widget with this trick). System UI -> Widgets and create a new javascript widget called "News" with the script below.
```
function sections() {
    return {
        'Department News': { 'uiPageName' : 'my_news'}
    };
}

function render() {
    var uiPageName = renderer.getPreferences().get("uiPageName");
    return renderer.getRenderedPage(uiPageName);
}

function getEditLink() {
    var uiPageName = renderer.getPreference('uiPageName');
    return 'sys_ui_page.do?sysparm_query=' + encodeURIComponent('name=' + uiPageName);
}
```
# Create a KB article and add the Widget to your Dashboard
* Create a new KB article with the category News
* Find the "News" Widget in your Dashboard picker and you can choose your UI page from the list. You can add multiple UI pages to this widget for selection.

Example:
![]({{ 'assets/images/news.PNG' | relative_url }})

It's not as pretty as the original, but that's an easy fix. Perhaps in a future post. What's handy about this is, it can be used for any case where you want to display selected table data in a dashboard widget.
