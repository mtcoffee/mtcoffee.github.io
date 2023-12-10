---
title: Instant MID Server on AWS Free Tier with Terraform
---

# Summary
Sometimes for a demo or troubleshooting it is handy to be able to quickly launch a new ServiceNow MID Sever. Even better it is handy to have a declaritive action to allow rebuilding Windows servers on demand without having to manually install and configure. For this we can use Terraform!

# Solution
To accomplish the example below you will need:  
1. An AWS Free Tier Account -  [Link](https://aws.amazon.com/free/)
2. An AWS Access Key for your AWS account -  [Link](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
3. A Key Pair defined in your AWS EC2 console -  [Link](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/create-key-pairs.html)
4. The Terraform Binaries - [Link](https://developer.hashicorp.com/terraform/install)

To get started on your Windows workstation, create the following directory structure from these files:
* TopLevelDirectory
  - terraform.exe
  - main.tf
  - variables.tf
  - terraform.tfvars
<details>
<summary>Expand to see sample Terraform Configuration files</summary>
main.tf
<script src="https://gist.github.com/mtcoffee/325ba4fd29b4528e9e15ab61293d1118.js"></script>
	variables.tf
<script src="https://gist.github.com/mtcoffee/5ccd59eadd7c024046fab152a291d118.js"></script>
	terraform.tfvars
	<script src="https://gist.github.com/mtcoffee/34556c0afac924f1ec8f7423262829ad.js"></script>
</details>

* Set your console key and secret in the terraform.tfvars file
* Then run:
  ```
	terraform init
	terraform plan
	terraform apply
	```
	
Upon completion you should get the IP and Admin password for your instance. You can login over RDP with these details and view the instance in the AWS Console.
![]({{ 'assets/images/terraform_output.PNG' | relative_url }})

![]({{ 'assets/images/aws_terraform.PNG' | relative_url }})
	
Now see my other post [PowerShell script for MID Installation](/automated-mid-server-install), configure and run it on your new Windows Server. It will take a few minutes and then you can run discovery against your MID IP. Afterwards you may want to tighten up security on the AWS Security group for TCP 3389.
