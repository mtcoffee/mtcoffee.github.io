---
title: ServiceNow Clone Automation
---

# Summary
These are a few scripts I like to include with ServiceNow instance clones so that there is no need to reconfigure anything after a clone is completed.

## Set name and clone date on instance header
This one will capture the instance name and clone date for quick display to easily identify which instance we're in.

1. In your production instance, create a clone profile. This is the one we will alter and use for cloning.
2. Create a new clone script. - “Set name clone date” and add the script below:

```

//set instancename in banner with last clone date
	var instanceName = gs.getProperty('instance_name');
	var tabStr = '';
	var date = new GlideDate().getByFormat(gs.getProperty('glide.sys.date_format'));
	gs.setProperty('glide.product.name', tabStr + ' ' + instanceName);
	gs.setProperty('glide.product.description', instanceName + ' - Last Cloned: ' + date );

```

Now whenever you clone, it will automatically set the instance name and  date in the banner. You can run this manually in your PDI to see what it looks like.  
![]({{ 'assets/images/cloneA.PNG' | relative_url }})

## Set theme per instance
It is nice to have a unique default theme per instance so that users know immediately they are not in production.  Let's set their theme based on the instance name. Create another clone script in our profile “Set theme per instance”. This will change the theme for all users based on the instance name.
```
//fetch instance name
var instanceName = gs.getProperty('instance_name');

//set theme variable
if (instanceName === 'mycompanyqa') {
            gs.log(instanceName);
            var theme = '1450b416b70023003f5c21c8ee11a954';
            }
if (instanceName === 'mycompanytest') {
            gs.log(instanceName);
            var theme = '7642a610ff200200096fffffffffff43';
            }
if (instanceName === 'mycompanydev') {
            gs.log(instanceName);
            var theme = '1db5f185ff200200096fffffffffff1d';
            }

//Delete all user theme preferences
var grUserPreference = new GlideRecord('sys_user_preference');
grUserPreference.addQuery('name','glide.css.theme.ui16');
grUserPreference.addQuery('system',false);
grUserPreference.deleteMultiple();

//Set system wide user preference
grUserPreference = new GlideRecord('sys_user_preference');
grUserPreference.addQuery('name','glide.css.theme.ui16');
grUserPreference.addQuery('system',true);
grUserPreference.query();
if( grUserPreference.next() ) {
       grUserPreference.value = theme;
       grUserPreference.update();
}

```

## Send email to collector mailbox
The clone default is to disable sending and receiving email, but this makes development/troubleshooting a challenge. Instead we can send email to a development mailbox and our work notes/email logs will still show the message sent. This also allows receiving mail.
```
//set email test address to black hole
	gr = new GlideRecord("sys_properties");
	gr.addEncodedQuery("name=glide.email.test.user");
	gr.query();
	while (gr.next()) {
		gr.value = "sntest-noreply@mydomain.com";
		gr.update();
	}

//enable email sending and receiving as cloning will disable it. We want to see email in work notes
	gr = new GlideRecord("sys_properties");
	gr.addEncodedQuery("name=glide.email.read.active^ORname=glide.email.smtp.active");
	gr.query();
	while (gr.next()) {
		gr.value = "true";
		gr.update();
	}

```
## Enable ATF
Always a must in your testing instances.
```
//enable ATF 
var properties = 'sn_atf.runner.enabled,sn_atf.schedule.enabled';
var target = new GlideRecord('sys_properties'); 
target.addQuery('name', 'IN', properties);
target.query(); // Issue the query to the database to get relevant records 
while (target.next()) { 
  // add code here to process the incident record
   target.value = "true";
  gs.info(target.name + ' ' + target.value);
  target.update(); 
}


```

## Disable select user accounts
You may not want everyone logging into sub production. Set your criteria as desired.
```
//disable all active users who are not type employee
var target = new GlideRecord('sys_user'); 
target.addQuery('u_user_type','NOT IN','Employee'); // user type is not employee,
target.addQuery('active',true); // is active,

//optionally exclude specific users
target.addQuery('user_name','NOT IN','admin,midserver.dev,midserver.test,midserver.qa');
target.query(); // Issue the query to the database to get relevant records
while (target.next()) { 
  // add code here to process the user record 
  target.active = false;
  //gs.info('user ' + target.user_name + ' updated');
  target.update(); 
  }
```

The nice thing about clone profiles is you can keep layering new scripts into it!
