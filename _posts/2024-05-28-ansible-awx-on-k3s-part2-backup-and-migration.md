---
title: Ansible AWX on K3s Part2 - Backup and Migration
---

# Summary
Continuing from the last post, now that we have a new AWX instance on the latest release how do we move our old instance data over. It's actually quite easy.

# Solution
1. Backup the AWX secret key
2. Backup the postgres database
3. Drop existing DB in existing or new AWX instance and restore the key and db, then bring online. 

This will also complete the upgrade process. For light upgrades you'll want to stick with helm/operator updates but this works well for major upgrades or infrastructure changes. There is also a playbook for backup that I may investigate next. After that, we'll investigate integrating AWX with ServiceNow!

<script src="https://gist.github.com/mtcoffee/2bd5f6f5e3fbee8289c6c0d3f1b02a40.js"></script>
