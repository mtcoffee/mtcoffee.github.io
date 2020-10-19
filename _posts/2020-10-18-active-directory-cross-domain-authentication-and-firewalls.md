---
title: Active Directory, Cross Domain Authentication and Firewalls
toc_sticky: true
tags:
- kerberos
- ActiveDirectory
- SSSD
---

# The Challenge
Recently a co-worker asked if I could help join a Linux host to an Active Directory domain so that users could SSH in with their AD credentials. No problem, [SSSD](https://sssd.io/docs/users/ad_provider.html) has made this an easy task for a while now. 

However, there was a special addition. The Linux host needed to be joined to a Domain A and authenticate users in trusted Domain B. This presented a challenge. Out of the box this will work assuming the client (Linux host) has direct connectivity to a domain controller in Domain B. However, in this case they were relying on the Domain A controller to handle the request on behalf of the client and did not permit client connectivity to a domain controller in Domain B. THIS WILL NOT WORK WITH SSSD or KERBEROS. To this they responded:

"But it it works with the Windows clients!"

Yes after a timeout delay, Windows clients by default will fall back to NTLM authentication, where only the DCs need to communicate each other. But this is no longer a good thing. This requires an explanation.

# Explanation
From Microsoft, [How does Authentication Work Cross Domain](https://docs.microsoft.com/en-us/archive/blogs/anthonw/how-does-authentication-work-cross-domain)? – In short, there are 2 authentication protocols in play when logging into a workstation. Kerberos(preferred) and NTLM (legacy). In order for Kerberos to work, the connecting computer (workstation) needs to be able to see the AD DC server on all the domains it plans to connect to; but the connected computer (server) does not need to be able to see the other AD DC server for the computers which connect to it. If this fails it will switch to NTLM authentication, where only the DCs need to see each other However, also per Microsoft [NTLM and NTLMv2 authentication is vulnerable](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) to a variety of malicious attacks. 

# Solution
To resolve this issue you have a few options:
1. Add an RODC to the remote site so that Kerberos will work.
2. Add a site to site vpn/firewall exception from the remote site
3. Accept the NTLM risk and use Linux's Winbind in place of SSSD which supports NTLM
4. The Linux SSSD client does not support NTLM, although it does support LDAP. If the site in Domain A has a global catalog in the AD forest, it can proxy the request, so it is possible to make this work by using the id_provider/auth_provider "ldap". 

Even so, it is a good idea to [disable NTLM ](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain)and require all clients Windows or Linux to use Kerberos.
