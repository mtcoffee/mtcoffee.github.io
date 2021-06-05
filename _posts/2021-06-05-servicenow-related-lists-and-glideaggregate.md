---
title: ServiceNow Related Lists and GlideAggregate
---

I have ServiceNow assessment questionnaire that is configured to generate assessments against CI's. The requirement is to show these assessments in a related list, on the CI form.
# The Problem
The problem is that the assessment instance table does not store the related source record (in this case the CI that was assessed). This information is recorded in each question that is in the assessment instance. So to find all of the related assessments we have to query the asmt_assessment_instance_question table and get the instance numbers.

# The Solution

1. Open System Definition > Relationships 
2. Create new list "**CI Assessments**"
3. Set table to **cmdb_ci_service** (in our script this will be "parent")
4. Set queries from table to **asmt_assessment_instance** (in our script this will be "current")

With GlideAggregate we can retrieve and group our instances. In this script we will find a group the questions that contain our CI by instance. Then pass the instance sysid to the asmt_assessment_instance table and retrieve our list of assessments.

```
//custom list for displaying related assessments
(function refineQuery(current, parent) {
    var sysid = '';
    var agg = new GlideAggregate('asmt_assessment_instance_question');
	//find all questions with our cmdb_ci_service.sys_id and group by Assessment Instance
    agg.addQuery('source_id', parent.sys_id);
    agg.addAggregate('COUNT');
    agg.groupBy('instance');
    agg.query();
	//for each instance found get the sys_id of the Assessment Instance
    while (agg.next()) {
        //do things on the results
        //gs.info(agg.instance.number)
        sysid += agg.instance + ',';
    }
	//Now use our list of instance id's to retrieve the list assessments related to our service
    current.addQuery('sys_id', 'IN', sysid);
})(current, parent);
```

Also good to be aware of, ServiceNow has recently introduced GlideQuery, which if you're familar with SQL may be preferrable since the groupby function is built in.

[https://developer.servicenow.com/dev.do#!/reference/api/paris/server/no-namespace/GlideQueryAPI](https://developer.servicenow.com/dev.do#!/reference/api/paris/server/no-namespace/GlideQueryAPI)
