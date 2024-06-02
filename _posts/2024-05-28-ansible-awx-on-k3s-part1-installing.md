---
title: Ansible AWX on K3s Part1 - Installing
---

# Summary
I had a need to setup Ansible AWX and discovered that recently the dev team has moved from Docker to a Kubernetes Operator as the preferred way to install AWX. In this post series we will cover installing and then migrating from an older AWX release on Docker to the current release on K3s.

# Solution

Steps are:
1. Install Platform Preqrequisites
2. Install K3s
3. Load the AWX Kubernetes Operator
4. Create new AWX Instance from Operator
5. Verify Installation

I have put together a bash script available below to auto install on most Debian or Redhat Linux variants. Just run this on your fresh Linux host and after a few minutes you should have K3s loaded with AWX deployed into the single node cluster.
```
curl https://gist.githubusercontent.com/mtcoffee/79744090a1c4ce1e0ac04d64df510f3b/raw/4faf912c57facce4033f908484a75565a90e33be/InstallAWX.sh  > install-awx.sh
chmod u+x install-awx.sh
sudo ./install-awx.sh
```

Full script below:
<script src="https://gist.github.com/mtcoffee/79744090a1c4ce1e0ac04d64df510f3b.js"></script>
