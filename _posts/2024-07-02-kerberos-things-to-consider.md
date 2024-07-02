---
layout: post
title: Kerberos Things To Consider
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [iis, ntlm authentication, kerberos authentication]
comments: true
---
This is just a like of things that I have discovered regarding Kerberos that you may find useful at some point.

Do please feel free to comment things that you may consider important or to challenge anything described here.

<hr>


* There are two self-contained pages to troubleshoot Kerberos issues. You can use "whoami.aspx" to understand what authentication is taking place and details, and you can also use "scrappertest.aspx" for double hop scenarios:

[Kerberos Troubleshooting Pages](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/www-authentication-authorization/diagnostics-pages-windows-integrated-authentication)
 
 
* Windows Authentication is not supported with HTTP/2.0.
 
[HTTP/2 on IIS](https://learn.microsoft.com/en-us/iis/get-started/whats-new-in-iis-10/http2-on-iis)

* When setting up a server, like IIS, for Windows Authentication, we must choose our providers. We typically set "Negotiate" followed by "NTLM". Take into consideration that we are enabling two options for authentication schemes: Negotiate and NTLM. However, "Negotiate" is a container that will try to negotiate between Kerberos or NTLM. So, there are three possible options here:
 
1. Use Kerberos inside Negotiate container.
2. Use NTLM inside Negotiate container.
3. Use NTLM.
 
* By default Windows will not attempt Kerberos authentication for a host if the hostname is an IP address. It will fall back to other enabled authentication protocols like NTLM. A registry change is needed on client machines that needs to access Kerberos-protected resources by IP address. Also, SPN must be registered to correct IP.
 
[Kerberos For IP](https://learn.microsoft.com/en-us/windows-server/security/kerberos/configuring-kerberos-over-ip)


* Purging Kerberos Tickets
 
```
Klist # displays current tickets for current user
Klist purge # purges kerb tickets for current user
```

You can see tickets for any username but you need to supply its LUID. You can use the following tool to get ID.

[logonsessions](https://learn.microsoft.com/en-us/sysinternals/downloads/logonsessions)

## Regarding IIS Settings

### Anonymous Authentication
Anonymous Authentication must be disabled when using Windows Authentication. Otherwise you are allowing access to a resource that you meant to have authentication.

The only exception that you would want Anonymous Authentication enabled is when you have an Application, which you want Windows Authentication enabled, but this lives inside a parent Site. In this case Parent site must have Anonymous and Windows Authentication enabled, in order for request to actualy hit the Application, and then the Application must only have Windows Authentication enabled.
 
### useKernelMode and useAppPoolCredentials settings

These settings are located in IIS Manager > Site > Configuration Editor > section:system.webServer/security/authentication/windowsAuthentication

These might be tough to understand, so read it as many times as you want. 

[Kerberos Authentication IIS](https://techcommunity.microsoft.com/t5/iis-support-blog/setting-up-kerberos-authentication-for-a-website-in-iis/ba-p/347882)

[WindowsAuthentication](https://learn.microsoft.com/en-us/iis/configuration/system.webServer/security/authentication/windowsAuthentication/)
 
| useKernelMode | useAppPoolCredentials | Result                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|---------------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FALSE         | FALSE                 | Authentication takes place in user-mode (w3wp.exe). As Kernel authentication is disabled, then authentication must happen in user-mode (inside w3wp.exe),  which means it does not matter what "useAppPoolCredentials is set to" (logically it's like setting it to true, but its value does not matter since the app pool credentials run the w3wp). So, as w3wp.exe runs unders the Application Pool's Identity, then these are the credentials that will be used to decrypt Kerberos traffic.  |
| TRUE          | FALSE                 | Authentication takes place in Kernel-mode. HTTP.SYS will use the Machine account to decrypt Kerberos traffic.                                                                                                                                                                                                                                                                                                                                |
| TRUE          | TRUE                  | Authentication takes place in Kernel-mode. HTTP.SYS will used the credentials supplied by the IIS. The Application Pool's Identity credentials will be used to decrypt Kerberos traffic.                                                                                                                                                                                                                                                     |
| FALSE         | TRUE                  | Authentication takes place in user-mode (w3wp.exe). As Kernel authentication is disabled, then authentication must happen in user-mode (inside w3wp.exe),  which means it does not matter what "useAppPoolCredentials is set to" (logically it's like setting it to true, but its value does not matter since the app pool credentials run the w3wp). So, Application Pool's Identity credentials will be used to decrypt Kerberos traffic.  |
|               |                       |                                                                                                                                                                                                                                                                                                                                                                                                                                              |


So,
If SPN is registered to machine account, then "userKernelMode = TRUE" and "useAppPoolCredentials = FALSE".


If SPN is registered to domain account, "useAppPoolCredentials = TRUE".
 
Take note that, when modifying these settings, "iisreset /noforce" is always needed to notify HTTP.SYS of changes.

### authPersistNonNTLM
 
Customize Kerberos to behave like a session or request based protocol.

[Windows Authentication authPersistnonNTLM](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/authentication/windowsauthentication/)

### Extended Protection

This is not exactly an IIS setting, it is an AD setting. However it can be configured through IIS Manager.
 
Honestly, if you do not needed, do not use it. 
 
[Extended Protection](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/authentication/windowsauthentication/extendedprotection/)
 


## Regarding Edge Policies

* Remember you can download templates through this link and then add them to your SYSVOL o you can copy and paste local PolicyDefinitions folder into SYSVOL.
Then you need to actually download Edge Templates.

[Administrative Templates (.admx) for Windows 10 October 2018 Update (1809)](https://www.microsoft.com/en-us/download/details.aspx?id=57576)

[Download and configure Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download?form=MA13FJ)

* Edge assigns a value to each authentication scheme. It will select the one with the highest value (more security).

[HTTP Authentication](https://www.chromium.org/developers/design-documents/http-authentication/) 


* **AuthNegotiateDelegateAllowlist**: Edge works with constrained delegation by default. So, if you are using unconstrained delegation, you must activate this policy and specify which server(s) can delegate to. For example, if a "serverA.com" needs to delegate to "serverB.com", this policy must be defined with a value of "serverA.com".

[AuthNegotiateDelegateAllowlist](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#authnegotiatedelegateallowlist)
 
[Kerberos Edge](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/www-authentication-authorization/kerberos-double-hop-authentication-edge-chromium)
 
* **AuthSchemes**: This policy can alter which authentication protocol will the Browser use. For example, if you would to exclude "negotiate", then Kerberos will never be used.

[AuthSchemes](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#authschemes)
 
* **AuthServerAllowlist**: The Browser will only reply to Windows Integrated Authentication (WIA) requests from a proxy of server that is in the Local Intranet or Trusted sites zones. Internet zones is a concept of Internet Explorer. However, Edge still uses zones for some specific settings, like this one for example. Read carefully, IF this policy IS NOT enabled, then Edge tries to detect if a server is on the intranet/trusted zone, only then will it respond to IWA requests. IF this policy IS enabled, then Edge will use this list to determine or not if it will reply  to IWA requests.
 
So, if you don't want to enable this policy, then make sure the server is added in either Local Intranet or Trusted Site zones. If you enabled this policy, then make sure you add server to this list.

[AuthServerAllowlist](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#authserverallowlist)
 
* **DisableAuthNegotiateCnameLookup**: This is an interesting one. As you know for Kerberos to work, SPN must be registered correctly, now it is job of the Browser to construct the SPN based on the information it has. If you disable this policy or don't configure it, the canonical name of the server is used to construct the SPN.

``` 
# Policy Not Set
Example.com => 10.0.0.5
SPN == HTTP/example.com
 
Test.com => 10.0.0.5
SPN ==HTTP/test.com
 
Prod.com => [CNAME] example.com
SPN == HTTP/example.com
 
Prod.com => [CNAME] gg.com => [CNAME] example.com
SPN == HTTP/example.com
 
# Policy Set
Example.com => 10.0.0.5
SPN == HTTP/example.com
 
Test.com => 10.0.0.5
SPN ==HTTP/test.com
 
Prod.com => [CNAME] example.com
SPN == HTTP/prod.com
 
Prod.com => [CNAME] gg.com => [CNAME] example.com
SPN == HTTP/prod.com
```
 
[HTTP Authentication](https://www.chromium.org/developers/design-documents/http-authentication/)
 
* **EnableAuthNegotiatePort**: If you enable this policy, and a user includes a non-standard port (a port other than 80 or 443) in a URL, that port is included in the generated Kerberos SPN.
 
```
# Policy Not Set
Test.com:8080
SPN == HTTP/test.com
 
# Policy Set
Test.com:8080
SPN == test.com:8080
```

[EnableAuthNegotiatePort](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#enableauthnegotiateport)
 
 