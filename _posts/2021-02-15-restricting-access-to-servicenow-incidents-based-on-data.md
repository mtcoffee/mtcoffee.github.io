---
title: Restricting Access to ServiceNow Incidents based on data
---

# The Challenge
I was trying to understand options for segmenting access in Servicenow. For example: 
* Department B should not see Department A's Incidents
* Incidents of a particular sensitivity should only be visible by specific roles

# Background
There are 3 common methods for access control in Servicenow
1. ACL
2. Query Business Rule
3. Domain Separation 

First of all stay away from Domain Separation unless you are an MSP or you have very strong case for it.   
[Domain Separation, part 1: Do I need it? What are the alternative options?](https://community.servicenow.com/community?id=community_article&sys_id=bc90fa39db2f48145ed4a851ca9619f7)  
# ACL's vs Query Business Rule
Now that, that is out of the way, lets talk about ACL's vs Query Business Rules. First See [jace bensons](https://jace.pro/post/2019-11-30-qbr-vs-acl/) thoughts.

The noteable differences are:
* If you use ACL's users will see "n records removed by security" when viewing their lists.
* If you use ACL's, your instance has to decide per record what a user can see after fetching them from the database
* Query business rules are typically better for performance since it only evaluates once.
* However, if your not careful you can hinder performance with Query Business Rules.
* If you need field level restrictions, the ACL's are required

Takeway
* If your requirement is looking to limit results to a role based on a single attribute - Query Business Rule
* If you need field level restrictions - ACL
* You might also need a combination of these  

Regardless of option, make sure you document!!!

# Solution
1. Create a Role. Navigate to User Administration > Roles and click New. Create a new role called top_secret
2. Add a new true/false field "top secret" to the incident table. [Reference](https://docs.servicenow.com/bundle/paris-platform-administration/page/administer/field-administration/task/t_CreatingNewFields.html)
3. Open an Incident and enable "Top Secret".
4. Create a before query business rule and set the condition as described in the script below
The example from the [ServiceNow Community](https://community.servicenow.com/community?id=community_article&sys_id=bc90fa39db2f48145ed4a851ca9619f7) proved useful so I made a version of it.
In this case we're saying, if user does not have role top_secret, add to the query filter to exclude incidents with tag "Top Secret". Use the script below in the rule

```
(function executeRule(current, previous /*null when async*/) {

	// role validation (!gs.hasRole("top_secret")) is part of the Business Rule Conditions
	//add to query filter to only return incidents with top secret false
	var extraQuery = "u_top_secret=false";
	
	if(current.getEncodedQuery() == ""){
		current.addEncodedQuery(extraQuery);
	}
	else{
		current.addEncodedQuery("^EQ^" + extraQuery);
		// ^EQ^ is needed to handle ^NQ (big OR) conditions
	}						
	
})(current, previous);
```
# Result
Users without the "top_secret" role should no longer be able to see the incidents with this value set to true.
