---
title: Updating ServiceNow from Ansible or AWX
---

# Summary
Ansible has a [built-in collection](https://github.com/ansible-collections/servicenow.itsm) that allows you to connect to a ServiceNow instance and do things like:
* Create or Update an Incident/Change/Problem
* Submit a request
* Query the CMDB and return a list of records
* Update the CMDB
* Create an Ansible Inventory from your CMDB Server records
* Trigger a ServiceNow Incident from an AWX workflow if a job fails

You can do this in Ansible directly or with Ansible AWX/Tower. In this example we will use AWX. 

# Create a custom ServiceNow Credential Type
In AWX create a [custom credential type](https://docs.ansible.com/ansible-tower/latest/html/userguide/credential_types.html).  
For the name use **ServiceNow Credential**  

For the Input configuration use this yaml.
```
fields:
  - id: username
    type: string
    label: User
    secret: false
  - id: password
    type: string
    label: Password
    secret: true
required:
  - username
  - password
```

For the Injector configuration use this yaml.
```
{% raw %}extra_vars:
sn_pass: '{{ password }}'
sn_user: '{{ username }}'{% endraw %}
```

# Create a Playbook
Here is a sample playbook to use with AWX. It will use the SN Table API.
```{% raw %}
- name: Create ServiceNow Incident
  hosts: all
  tasks:
    - name: Create incident #https://github.com/ansible-collections/servicenow.itsm
      servicenow.itsm.incident:
        instance:
           host: "https://{{ sn_instance }}.service-now.com" 
           username: "{{ sn_user }}" 
           password:  "{{ sn_pass }}" 

        state: new
        caller: abel.tuter
        short_description: Test Incident from Ansible
        description: User has been unable to receive email for the past 15 minutes
        impact: low
        urgency: low
      register: result

    - name: Print incident number
      debug:
        msg: 
         Incident Number: "{{result.record.number}}"
         Link: "https://{{ sn_instance }}.service-now.com/incident.do?sys_id={{result.record.sys_id}}"
         Raw:  "{{result}}"
      when: not ansible_check_mode{% endraw %}
```
When using this playbook be sure to include the variable for the ServiceNow instance to connect to. Use the extra_vars in the job template

# Create AWX Job Template
Now we can Configure the Job template to use our playbook. We need to make sure that the ServiceNow Credential is included in the job template. This is how AWX will provide the credentials to login to the ServiceNow Table API.  Again, be sure to use the extra_vars in the job template for sn_intance
![]({{ 'assets/images/awx_sn.png' | relative_url }})


That's all there is to it. When the job template runs it will create a new ServiceNow Incident. Make sure to review the Github Repo for the [Servicenow.itsm collection.](https://github.com/ansible-collections/servicenow.itsm)
