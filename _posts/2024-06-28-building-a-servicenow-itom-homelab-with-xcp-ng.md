---
title: Building a ServiceNow ITOM Homelab with XCP-NG
---

# Summary
For anyone interested in ServiceNow ITOM, there's no better way to learn than on your own infrastructure that you can tear down and rebuild as needed. This necessitates a "homelab." While [AWS free tier](/instant-mid-server-on-aws-free-tier-with-terraform/) is an option, it's limited and can get expensive beyond the basics. To delve deeper into homelab building, visit [https://reddit.com/r/homelab ](https://reddit.com/r/homelab)for abundant resources.
# Homelab Components
To get started you'll need a PDI and some hardware/software.
## ServiceNow PDI
To begin, you'll need a PDI ([Personal Development Instance](https://developer.servicenow.com/)). Fortunately, these are free, and currently, you can install Discovery and connect a MID Server to your PDI.
## Hardware
Choose a Mini-PC suitable for running as a headless server and install XCP-ng on it. I am a fan of [Intel NUC](https://www.amazon.ca/intel-nuc/s?k=intel%20nuc) series machines, however other options exist. XCP-ng boasts excellent hardware support, making it compatible with most devices.
## Hypervisor
[XCP-ng](https://xcp-ng.org/) is a free and open-source hypervisor, allowing you to run multiple virtual machines for diverse learning and testing purposes. While it may pose a challenge for beginners, comprehensive tutorials and a growing XCP-ng community offer substantial support. Start with the [official documentation](https://docs.xcp-ng.org/installation/install-xcp-ng/) and join the [XCP-ng community](https://xcp-ng.org/forum/) for help.

An added advantage is the ability to perform discovery against your XCP-ng host(s) using the Citrix Xen HyperV Pattern to identify all your VMs. Refer to [ServiceNow Documentation](https://docs.servicenow.com/csh?topicname=citrix-xen-hyper-v-discovery.html&version=latest) for details.
## MID Server
Once your Hypervisor is operational, the first VM to create should be a Windows VM to function as your MID Server. Unfortunately, at the time of this writing, Windows Discovery cannot run from a Linux MID Server, so you will probably need a Window host. Refer to my [MID Server automated install script](/automated-mid-server-install) for assistance.
## Automation
Explore Ansible, Packer, and Terraform. While it's not a one-day task, once implemented, you can fully automate and deconstruct your setups. For instance, I utilize Packer/Terraform to swiftly build new Linux/Windows VMs and dismantle them even faster. Some examples are available in a [repo here](https://github.com/mtcoffee/xcp-ng-packer-examples) and [here](https://github.com/mtcoffee/xcp-ng-terraform-examples). With Ansible, you can automate the installation of applications such as IIS, Wordpress, Docker, Kubernetes, etc., followed by running discovery on those applications!
