---
layout: post
title: Configure ODBC Logging For IIS (High Level Overview)
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [iis, odbc, odbc logging]
comments: true
---

What this accomplishes is to log requests into a Database.

## Step 1 - Create Database & Table
For the table, you can use the standard sql script located at: "%windir%\System32\inetsrv\logtemp.sql".

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img1.png)

## Step 2 - Create System DSN
 
Create System DSN with appropriate settings to connect to the DB.

Open the ODBC Data Source Administrator and add a new data source.

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img2.png)

## Step 3 - Install IIS Needed Features
 
Install ODBC Logging & Custom Logging modules.

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img3.png)

## Step 4 - Settings In "system.applicationHost/sites"
 
Go to "Configuration Editor > system.applicationHost/sites > Select site that you want to configure" and change these two properties:
 
* **CustomLogPluginClsid**: {FF16065B-DE82-11CF-BC0A-00AA006111E0}
 
* **LogFormat**: Custom

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img4.png)

## Step 5 - Configure system.webServer/odbcLogging
 
Go to site level  "Configuration Editor > system.webServer/odbcLogging" and configure the following properties:

* **DataSource**: name of ODBC data source in step 2.
* **password**:
* **tableName**: name of table in DB
* **userName**: username (needs to have access to DB)

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img5.png)

## Step 6 - Application Pool Identity
 
The Application Pool should be using an Identity that has read/write access to DB.

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img6.png)

## Step 7 - Test

Made some requests and then made a query to DB.

![](/assets/img/articles/Configure-ODBC-Logging-For-IIS/img7.png)



