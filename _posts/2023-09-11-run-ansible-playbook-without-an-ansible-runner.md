---
title: Run Ansible Playbook without an Ansible Runner
---

# Summary
The other day I wanted to try out running an Ansible Playbook against a Windows host. To quickly do this in an ephemeral way we can leverage docker. 
# Solution
From a linux host Just set your Windows administrative username/password and the container will connect and update the Windows machine. Handy in a pinch!

```
docker run -i --rm -v ~/.ssh:/root/.ssh -v $(pwd):/apps -w /apps alpine/ansible /bin/bash -c  \
"export ANSIBLE_HOST_KEY_CHECKING=False; \
apk add --update --no-cache openssh sshpass; \
python3 -m pip install --user --ignore-installed pywinrm; \
ansible-galaxy collection install ansible.windows; \

cat << 'EOF' > /tmp/playbook.yml
---
# Ansible playbook to run Windows Update and restart, if required
#
# http://docs.ansible.com/ansible/win_updates_module.html
# https://docs.ansible.com/ansible/win_reboot_module.html

- name:  Windows Update
  hosts: all
  gather_facts: false
  tasks:
    - name: Running Security/Critical Windows Updates
      win_updates:
        category_names:
        - SecurityUpdates
        - CriticalUpdates
        reboot: yes
      register: result
    # output results
    - debug: var=result
EOF


ansible-playbook -vv /tmp/playbook.yml -i <myWindowsMachine>, \
--extra-vars 'ansible_connection=winrm ansible_port=5985 ansible_user=administrator ansible_password=<ansiblepass>'"
```
