---
layout: post
title: Web Deploy To Migrate Sites
subtitle: Raw Commands
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [iis, web deploy, msdeploy, migration]
comments: true
---
## Table of Contents
1. [What is Web Deploy](#what-is-web-deploy)
2. [Basic Terminology](#basic-terminology)
3. [Type of Packages](#type-of-packages)
4. [Basic Workflow](#basic-workflow)
5. [What is Web Deploy and MsDeploy](#what-is-web-deploy-and-msdeploy)
6. [Where is MsDeploy Installed?](#where-is-msdeploy-installed)
7. [What does Web Deploy backup by default?](#what-does-web-deploy-backup-by-default)
8. [Best Practices](#best-practices)
9. [Raw Example](#raw-example)
10. [Resources](#resources)

## What is Web Deploy
> Web Deploy (msdeploy) simplifies deployment of Web applications and Web sites to IIS servers. Administrators can use Web Deploy to synchronize IIS servers or to migrate to newer versions of IIS. Web Deploy Tool also enables administrators and delegated users to use IIS Manager to deploy ASP.NET and PHP applications to an IIS server.

So, Web Deploy can be use for several purposes.

As for now, we will use it to migrate either an individual site hosted on IIS, as well a Server migration for all IIS' sites.

## Basic Terminology
* **Package**: zip package containing all IIS configuration + site content.
* **Source/Origin Server**: server from which the package was created. Where you have your original sites that you want to migrate to other server (destination server).
* **Destination/Target Server**: server where you want to restore the package from the origin server.

## Type of Packages:
* **Site**: package that contains IIS configuration + the content of the site.
* **Server**: package that contains IIS configuration + the content of ALL sites in IIS.

## Basic Workflow
Origin Server ==> Create package of site(s).  
Move the package to Destination Server.  
Destinatin Server ==> Restore package.

## What is Web Deploy and MsDeploy?
Do not give it to much thought, it is the same thing.

## Where is MsDeploy installed?
"C:\Program Files\IIS\Microsoft Web Deploy V3"

## What does Web Deploy backup by default?
* IIS Configuration
* Site content.

Web Deploy DOES NOT backup Application Pools nor SSL/TLS certificates.

**Altough you can change some of these behavior with disableLink/enableLink parameters:**
```
#Disable/Enable Application Pool Configuration
  -disableLink:AppPoolExtension
  -enableLink:AppPoolExtension 
 

#Disable/Enable site content
 -disableLink:ContentExtension
 -enableLink:ContentExtension
```

[Web Deploy Behaviour](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages)

## Best Practices
* You need to know OS version of Source and Destination Servers! **Commands may differ if IIS is < 7.5**.
* Before even using Web Deploy, please make sure you have a backup of the IIS configuration and your actual content!. DO THIS ON BOTH, SOURCE AND DESTINATION SERVERS. "Appcmd" tool comes with IIS. This DOES NOT INCLUDE YOUR SITE CONTENT, ONLY IIS CONFIGURATION.
   * Creating backup
   ```
   %windir%\system32\inetsrv\appcmd add backup “PreWebDeploy”
   ```
   * Viewing backups:
   ```
   %windir%\System32\inetsrv\appcmd.exe list backup
   ```

  * Restoring backups:
   ```
   %windir%\System32\inetsrv\appcmd.exe restore backup “MyBackup”
   ```
   * Use msdeploy.exe to backup both, IIS configuration + content of whole IIS server sites.
     ```
     msdeploy -verb:sync -source:webServer -dest:package=c:\_WebDeploy\PreWebDeploy.zip
     ```
* Install SAME VERSION and bitness of Web Deploy in both Source and Destination Servers. Use version at least 3.6.
    * [Download Web Deploy 3.6](https://www.iis.net/downloads/microsoft/web-deploy).

* Destination server modules MUST MATCH Origin Server modules. If Origin server had "X,Y,Z" modules installed, then Destination server NEEDS to have "X,Y,Z" modules installed as well! I do not care if you sites do not use one particular module, but as you had it when creating the package, then, the destination Server needs it; you later can remove it if you would like.
    * Source Server:
    ```
    msdeploy -verb:getDependencies -source:webServer > c:\_Webdeploy\SourceServerDepends.txt
    ```
    * Destination Server:
    ```
    msdeploy -verb:getDependencies -source:webServer > c:\_Webdeploy\TargetServerDepends.txt
    ```
    * Again, Destination Server's modules needs to match Origin Server's modules for a successful deployment.


## Raw example
Suppose you want to migrate sites from Windows Server 2016 to Windows Server 2022.
**I will put commands for both, site and server packages. Use according to your needs**.

### Creating Backup in Origin and Destination Servers.  

**Creating IIS configuration backup**
```
# Origin
%windir%\system32\inetsrv\appcmd add backup “OriginPreWebDeploy”

# Destination
%windir%\system32\inetsrv\appcmd add backup “DestinationPreWebDeploy”

```

**Creating IIS configuration + actual sites content**
```
# Origin
msdeploy -verb:sync -source:webServer -dest:package=c:\_WebDeploy\OriginPreWebDeploy.zip

# Destination
msdeploy -verb:sync -source:webServer -dest:package=c:\_WebDeploy\DestinationPreWebDeploy.zip
```

### Modules Matching
Match Destination server's modules to Origin server's modules.

```
# Origin
msdeploy -verb:getDependencies -source:webServer > c:\_Webdeploy\OriginServerDepends.txt

# Destination
msdeploy -verb:getDependencies -source:webServer > c:\_Webdeploy\DestinationServerDepends.txt
```
**MAKE THEM MATCH!**

### Download Web Deploy
As both OS' are x64, I will proceed to download Web Deploy 3.6 in both, Origin and Destination servers.

[Download Web Deploy 3.6](https://www.iis.net/downloads/microsoft/web-deploy).

### Creating Package
Create the package in Origin server.
MsDeploy.exe has a parameter "-whatif". With this parameter it will tell you what would happen wihthout making any actual changes.

**ALL BELOW COMMANDS HAVE -WHATIF PARAMETER. REMOVE IT WHEN YOU ARE SURE EVERYTHING IS OK.**

The "-enableLink:AppPoolExtension" throws the Application Pool into the package!

```
# Server Package
msdeploy -verb:sync -source:webServer -dest:package=c:\_WebDeploy\PreWebDeploy.zip -enableLink:AppPoolExtension  -verbose -whatif

# Site Package
msdeploy -verb:sync -source:apphostconfig="NAMEOFSITE" -dest:package=c:\site1.zip  -enableLink:AppPoolExtension -whatif
```

 *Note: If asked for encryption then add "encryptPassword=PASSWORD" after the destination. Like so: "-dest:package=c:\_WebDeploy\PreWebDeploy.zip,encryptPassword=123"*

### Move the package to the destination server.
Transfer package from Source to Target server.

### Restore the package
Restore package in Destination server.

MsDeploy.exe has a parameter "-whatif". With this parameter it will tell you what would happen wihthout making any actual changes.

**ALL BELOW COMMANDS HAVE -WHATIF PARAMETER. REMOVE IT WHEN YOU ARE SURE EVERYTHING IS OK.**

```
# Server Package
msdeploy -verb:sync -source:package=C:\_Webdeploy\PreWebDeploy.zip -dest:webServer -whatif

# Site Package

msdeploy -verb:sync -source:package=c:\site1.zip -dest:apphostconfig="NAMEOFSITE"  -whatif 
```

 *Note: If you created package with encryption, then you need to use that password to restore the package. Add "encryptPassword=PASSWORD" after the destination. Like so: "-dest:package=c:\_WebDeploy\PreWebDeploy.zip,encryptPassword=123"*


## Resources
* [Web Deploy 3.6](https://www.iis.net/downloads/microsoft/web-deploy)
* [Deploying Web Packages](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages)
* [How to migrate a website using Web Deploy](https://techcommunity.microsoft.com/t5/iis-support-blog/how-to-migrate-a-website-using-web-deploy/ba-p/852244)
* [Web Deploy Providers](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/dd569040(v=ws.10))
* [Web Deploy webServer Provider](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/dd569021(v=ws.10))
* [Server Level Migration with Web Deploy](https://learn.microsoft.com/en-us/archive/blogs/ericparvin/server-level-migration-with-web-deploy)
* [Web Deploy Command Line Syntax](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/dd569106(v=ws.10))
* [Web Deploy Behaviour](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages)