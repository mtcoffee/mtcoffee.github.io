---
title: Installing Windows Subsystem for Linux and ServiceNow CLI
---

For years developers have longed for a Linux shell on Windows and today Microsoft has answered the call. WSL allows you to run Linux on Windows without running a resource intensive hypervisor! So for fun lets load it up and install the new ServiceNow CLI.

# Installing WSL2 on Windows 10
Reference [https://docs.microsoft.com/en-us/windows/wsl/install-win10](https://docs.microsoft.com/en-us/windows/wsl/install-win10)  

**Run the following Deployment Image Servicing and Management (DISM) commands in PowerShell As Administrator** 
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
Now reboot Windows.

**Configure WSL to use version 2, run in PowerShell As Administrator** 

```
wsl --set-default-version 2
```

In the same PowerShell window, download the latest release of Ubuntu
```
curl.exe -L -o c:\temp\wslubuntu2004.appx https://aka.ms/wslubuntu2004
Add-AppxPackage c:\temp\wslubuntu2004.appx
```

At this point you have to go the Start menu, search for Ubuntu and open the new "App". This will launch a shell window, ask you to set a username/pasword for your new ubuntu instance. That's it, you now have true a Linux shell in Windows!

# Installing Service Now CLI
Now that we have an Ubuntu environment we can install ServiceNow CLI
[https://docs.servicenow.com/bundle/quebec-application-development/page/build/servicenow-cli/task/download-cli.html#download-cli](https://docs.servicenow.com/bundle/quebec-application-development/page/build/servicenow-cli/task/download-cli.html#download-cli)

Head to the ServiceNow Store and look for ServiceNow CLI. You will download a "ServiceNow CLI.zip" file. Copy this to C:\Temp
[https://store.servicenow.com/sn_appstore_store.do#!/store/application/9085854adbb52810122156a8dc961910](https://store.servicenow.com/sn_appstore_store.do#!/store/application/9085854adbb52810122156a8dc961910)


In the Linux instance run the commands below.

```
sudo apt update
sudo apt install unzip
cp /mnt/c/Temp/ServiceNow\ CLI.zip .
unzip ServiceNow\ CLI.zip
cd ServiceNow\ CLI/
./snc-1.1.0-linux-x64-installer.run
```

The guided setup will coach you through the rest of the process. Once complete you need to [configure a json file](https://docs.servicenow.com/bundle/quebec-application-development/page/build/servicenow-cli/task/configure-profile.html#configure-profile) under ~/.snc/config.json

Finally, you can generate a new incident with this 1 liner
```
snc record create --table incident --data "{short_description: 'New Incident'}"
```

# Confusion with Now CLI aka Vercel
There is also "[Now CLI](https://developer.servicenow.com/dev.do#!/reference/now-experience/orlando/cli/cli)" which is node js instance for ServiceNow Development purposes.  This has been deprecated in favour of "vercel". I recommend [Jace Bensons's blog](https://jace.pro/post/2021-02-05-custom-domain-for-pdis-using-vercel/) for an example use case.
