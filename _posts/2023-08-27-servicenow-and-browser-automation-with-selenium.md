---
title: ServiceNow and Browser Automation with Selenium (updated)
---

I created a similar version of this post a few years ago but it stopped working, it is time for an update.
# The Challenge
While ServiceNow does have the ATF, sometimes you need an external browser automation tool. Enter Selenium! I wanted something to automate the process of logging in and creating test data. I also didn't want to create and hold a testing environment, I just want to run a script and make this happen. (Yes, REST API is an also an option, but I wanted something as flexible as what can be done in the UI).

How can we make this easy? Docker! With this "one liner" we will:
1. Call the my github hossted alpine docker image that will contain all the pre-requesites
2. Execute the Python Selenium script to login to our ServiceNow instance and create an Incident.

You can also run the Python script directly by copying the needed contents
# The Solution
Here's the shell script. Just set your instance url, user and password.
<script src="https://gist.github.com/mtcoffee/f42c52690915a4e31a20272b2c5c0093.js"></script>

I like to combine jobs like this with [Rundeck](https://www.rundeck.com/open-source), so that I can generate test data in my Personal Development Instance with a single button click!

# For the Service Portal record producer
The question was asked, can this be done in the Service Portal? The answer is yes, however is it worth pointing out that it is a bit more challenging since ServiceNow can change DOM elements names between upgrades. [Discussed here](https://community.servicenow.com/community?id=community_question&sys_id=39fa29e3db9fc0901cd8a345ca9619ab).

<script src="https://gist.github.com/mtcoffee/4bc898982be62b7e56206e9a176e4e35.js"></script>
