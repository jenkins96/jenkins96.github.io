---
layout: post
title: Change ApplicationHost.config In App Service
[comment]: <> (subtitle: Each post also has a subtitle)
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [app service, iis]
comments: true
---

This article will explain three methods on how to change the “applicationHost.config” file in App Service.  
Imagine a scenario where you want to change a particular setting, however, this cannot be changed at the “web.config” level.  
For this scenario I would like to allow four Server Variables to be rewritten. By default, this can only be done at “applicationHost.config” level.  
In a Virtual Machine this setting would look like this in the “applicationHost.config” file:  

```xml
 <system.webServer>
  <rewrite>
   <allowedServerVariables>
    <add name="HTTP_X_UNPROXIED_URL" />
    <add name="HTTP_X_ORIGINAL_ACCEPT_ENCODING" />
    <add name="HTTP_X_ORIGINAL_HOST" />
    <add name="HTTP_ACCEPT_ENCODING" />
    </allowedServerVariables>
  </rewrite>
 </system.webServer>

```

The issue is that in an App Service the “applicationHost.config” file is read-only.

File is located at “C:\local\Config\applicationHost.config”.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img1.png)

What we can do is to create a transform file to, you guessed it, transform the actual “applicationHost.config” file.

There are three ways to do this:

## Private Extensions
Go to “C:\home\SiteExtensions” (crete /SiteExtensions folder if not present) and create a folder within this directory with the desire name of your extension.

In my case I will create a folder name “/MyPrivateExtension” inside “/SiteExtensions”.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img2.png)

Navigate to “/MyPrivateExtension” and create/upload a file called “applicationHost.xdt”. You can use “touch” command.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img3.png)

Past the desire property to transform and with the appropriate syntax.

For our purposes the correct syntax would be the following:
```xml
<?xml version="1.0"?> 
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform"> 
  <system.webServer> 
     <rewrite xdt:Transform="InsertIfMissing">
        <allowedServerVariables xdt:Transform="InsertIfMissing">
            <add name="HTTP_X_UNPROXIED_URL" xdt:Locator="Match(name)" xdt:Transform="InsertIfMissing" />
            <add name="HTTP_X_ORIGINAL_ACCEPT_ENCODING" xdt:Locator="Match(name)" xdt:Transform="InsertIfMissing" />
            <add name="HTTP_X_ORIGINAL_HOST" xdt:Locator="Match(name)" xdt:Transform="InsertIfMissing" />
            <add name="HTTP_ACCEPT_ENCODING" xdt:Locator="Match(name)" xdt:Transform="InsertIfMissing" />
        </allowedServerVariables>
    </rewrite>
  </system.webServer> 
</configuration>
```

Note: At the end of this article, I will leave some sample for you to understand syntax.
![](/assets/img/articles/changeAppHostInAppServiceIMG/img4.png)

Save it and restart your App Service

This file should have transformed the actual “applicationHost.config”.

Let us check:
![](/assets/img/articles/changeAppHostInAppServiceIMG/img5.png)

## Top Level Private Extension
<Removed previous “applicationHost.xdt” and restarted App Service>

Basically, we will create the exact same file (“applicationHost.xdt”), but within “C:\home\site” folder.
![](/assets/img/articles/changeAppHostInAppServiceIMG/img6.png)

Paste content and save file.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img7.png)

Restart App Service.

This should have transformed the actual “applicationHost.config”.

## IIS Manager Extension
In Kudu go to Site Extension > Gallery and search for “iis manager” and install it.

Here the repository for this iis manager extension.
https://github.com/shibayan/IISManager

![](/assets/img/articles/changeAppHostInAppServiceIMG/img8.png)

Restart site.

Then add “/iismanager” to your Kudu URL. “https://<YOURAPPSERVICE.scm.azurewebsites.net/iismanager”

Make changes, as you normally would in the “applicationHost.config”, save and restart it.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img9.png)

This should have created a top level “applicationHost.xdt” that will modify actual “applicationHost.config”.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img10.png)

Changes in “applicationHost.config”:
![](/assets/img/articles/changeAppHostInAppServiceIMG/img11.png)

## Troubleshooting
If you restarted the App Service but still not seeing changes in “applicationHost.config” file, I would suggest restarting App Service again just to be sure. The other thing that you could do is to check transform logs located in “C:\home\LogFiles\Transform”. These logs can tell you why changes did not apply.

![](/assets/img/articles/changeAppHostInAppServiceIMG/img12.png)

## Resources
https://www.thebestcsharpprogrammerintheworld.com/2015/01/19/making-changes-to-the-applicationhost-config-on-azure-app-service/
https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions#finding-your-applicationhostconfig
https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples
https://github.com/shibayan/IISManager
