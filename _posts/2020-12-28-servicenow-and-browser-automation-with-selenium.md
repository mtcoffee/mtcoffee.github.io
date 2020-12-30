---
title: ServiceNow and Browser Automation with Selenium
---

# The Challenge
While ServiceNow does have the ATF, sometimes you need an external browser automation tool. Enter Selenium! I wanted something to automate the process of logging in and creating test data. I also didn't want to create and hold a testing environment, I just want to run a script and make this happen. (Yes, REST API is an also an option, but I wanted something as flexible as what can be done in the UI).

How can we make this easy? Docker! With this "one liner" we will:
1. Call the official python:alpine docker image from docker hub
2. Install Chrome (chromium) and Selenium on the fly
3. Execute the Python Selenium script to login to our ServiceNow instance and create an Incident.

# The Solution
Here's the shell script. Just set your instance url, user and password.
```
docker run -i --rm --user root --name pythoncontainer python:alpine /bin/sh -c \
'pip3 install selenium webdriver_manager && \
apk update && \
apk add chromium chromium-chromedriver && \
echo ###start of python script#### && \
python' <<EOF 
print('start of login')
import selenium	
from selenium import webdriver

from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.select import Select
from selenium.webdriver.support.ui import Select

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--headless')
import time
sninstance = "https://dev12345.service-now.com"
sninstanceuser = "ITILBOB"
sninstancepwd = "ITILBOBSPASSWD"
browser = webdriver.Chrome(options=chrome_options)
browser.get(str(sninstance) + "/login.do") 
time.sleep(5)

username = browser.find_element_by_id("user_name")
password = browser.find_element_by_id("user_password")
username.send_keys(str(sninstanceuser))
password.send_keys(str(sninstancepwd))
login_attempt = browser.find_element_by_id("sysverb_login")
login_attempt.click()
print('end of login')

#create incident
print('logged in, now creating incident')
time.sleep(5)
browser.get(str(sninstance) + "/incident.do")
assignmentGroup = browser.find_element_by_id("sys_display.incident.assignment_group")
assignmentGroup.send_keys('Help Desk',Keys.RETURN)
callerId = browser.find_element_by_id("sys_display.incident.caller_id")
callerId.send_keys('Abel Tuter',Keys.RETURN)
contactType = Select(browser.find_element_by_id("incident.contact_type"))
contactType.select_by_visible_text('Walk-in')
shortDescription = browser.find_element_by_id("incident.short_description")
shortDescription.send_keys("Testing Incident creation with Selenium")

browser.find_element_by_id("sysverb_insert").click() #insert record
print('completed incident creation')
EOF
```
I like to combine jobs like this with [Rundeck](https://www.rundeck.com/open-source), so that I can generate test data in my Personal Development Instance with a single button click!

# Edit: By Request
The question was asked, can this be done in the Service Portal? The answer is yes, however is it worth pointing out that it is a bit more challenging since ServiceNow can change DOM elements names between upgrades. [Discussed here](https://community.servicenow.com/community?id=community_question&sys_id=39fa29e3db9fc0901cd8a345ca9619ab).

**Creating an Incident in the Service Portal Catalog Item**
```
docker run -i --rm --user root --name pythoncontainer python:alpine /bin/sh -c \
'pip3 install selenium webdriver_manager && \
apk update && \
apk add chromium chromium-chromedriver && \
echo ###start of python script#### && \
python' <<EOF 
print('start of login')
import selenium	
from selenium import webdriver

from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.select import Select
from selenium.webdriver.support.ui import Select

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--headless')
import time
sninstance = "https://dev12345.service-now.com"
sninstanceuser = "ITILBOB"
sninstancepwd = "ITILBOBSPASSWD"
browser = webdriver.Chrome(options=chrome_options)
browser.get(str(sninstance) + "/login.do") 
time.sleep(5)

username = browser.find_element_by_id("user_name")
password = browser.find_element_by_id("user_password")
username.send_keys(str(sninstanceuser))
password.send_keys(str(sninstancepwd))
login_attempt = browser.find_element_by_id("sysverb_login")
login_attempt.click()

#confirm login
time.sleep(5)
print('current url after login is ' + browser.current_url)
expected = 'nav_to.do'
if expected in browser.current_url:
    print ('login success!')
else:
    print ('login failed')
    exit(1)
print('end of login')

#create incident
print('logged in, now submitting incident')
time.sleep(5)
browser.get(sninstance + "/sp?id=sc_cat_item&sys_id=3f1dd0320a0a0b99000a53f7604a2ef9")
time.sleep(3)
print('current url is ' + browser.current_url)

print('setting urgency')
urgency = browser.find_element_by_id('s2id_autogen1')
urgency.send_keys('3',Keys.RETURN)
print('setting description')
description = browser.find_element_by_id("sp_formfield_comments")
description.send_keys("Testing Incident submission with Selenium")

browser.find_element_by_xpath('//button[text()="Submit"]').click() #submit
print('completed incident submission')

#confirm we are now on ticket page
time.sleep(5)
print('current url after login is ' + browser.current_url)
expected = 'id=ticket'
if expected in browser.current_url:
    print ('submission success!')
else:
    print ('submission failed')
    exit(1)
print('end of login')
EOF
```
