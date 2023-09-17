---
title: GlideDate and GlideDateTime Reference
toc_sticky:
  New field 1: ''
---

Just a GlideDate and GlideDateTime reference

# Some Basics

```javascript
//Get current time in UTC
var gdt = new GlideDateTime()
gs.info(gdt)

//Get current time in the timezone of your instance
var gdt = new GlideDateTime().getDisplayValue()
gs.info(gdt)

//Get time in another timezone, method A
var gdt = new GlideDateTime()
var gsdt = new GlideScheduleDateTime(gdt);
gsdt.setTimeZone("US/Pacific");
gs.info(gsdt)

//Get time in another timezone, method B
var gdt = new GlideDateTime()
gdt.setTimeZone("US/Pacific");
gs.info(gdt.getDisplayValue())

//get date time in UTC 12 hour format
var gdt = new GlideDate().getByFormat('yyyy-MM-dd hh:mm:ss a');
gs.info(gdt);

//get date time in Local Time 12 hour format
var DateTimeUTC = new GlideDateTime();
var DateLocal = DateTimeUTC.getLocalDate();
var TimeLocal = DateTimeUTC.getLocalTime();
gs.info(DateLocal + ' ' + TimeLocal.getByFormat("hh:mm:ss a"));

//adding or subtracting hours
var currentDateTime = new GlideDateTime()
gs.info(currentDateTime);

currentDateTime.addSeconds(-86400)
gs.info(currentDateTime + ' is 24 hours ago'); //24 hours ago

var currentDateTime = new GlideDateTime();
currentDateTime.addSeconds(86400);
gs.info(currentDateTime + ' is 24 hours from now'); //24 hours from now
```

# Mixing GlideDate and GlideDateTime
Something useful to know is you can mix GlideDate and GlideDateTime.  GlideDate is a wrapper for GlideDateTime so the value on the back end is the same. Just be careful about same-day comparisons because GlideDate strips the time and sets it to 00:00:00.

```javascript
var gdt= new GlideDateTime();
gdt.addDaysUTC(-1)
var gd = new GlideDate();

gs.info('Is ' + gdt + ' Greater than ' + gd )
gs.info(gdt > gd)

gdt.addDaysUTC(2)
gs.info('Is ' + gdt + ' Greater than ' + gd )
gs.info(gdt > gd)
```
