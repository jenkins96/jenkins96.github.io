---
layout: post
title: Getting Docker Logs In Windows Container
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [docker, container, logs,logmonitor,servicemonitor, iis]
comments: true
---

## Table Of Contents
1. [Introduction](#introduction)
2. [Downloading Log Monitor & Service Monitor](#downloading-log-monitor--service-monitor)
3. [Basics Of Log Monitor And Service Monitor ](#basics-of-log-monitor-and-service-monitor-open-source)
4. [Initial Setup](#initial-setup)
    1. [Dockerfile](#dockerfile)
    2. [LogMonitorConfig.json](#logmonitorconfigjson)
5. [Build Image](#building-image)
6. [Create And Run Container](#create-and-run-container)
7. [Open Interactive Shell](#open-interactive-shell)
8. [Docker Logs](#getting-docker-logs)
9. [Logs From EV](#getting-logs-directly-from-event-viewer)
10. [Resources](#resources)

## Introduction
In this guide we will collect three Event Viewer Logs from a Docker Container. One of these logs is not enabled by default. So, we will need to enable it with a powershell script and add this to our Dockerfile.

We will collect the following logs:
* Event Viewer Application.
* Event Viewer System.
* Event Viewer IIS Configuration Operational(this logs whenever IIS settings are modified)

In addition to this, we will use Windows Container Log Monitor and IIS Service Monitor to collect same logs plus subscribing to additional logs.

Ok, Application and System are logged by default. However, IIS Configuration Operational is not! The very first thing we need to know is what is the actual name of this log. I know its official name is "Microsoft-IIS-Configuration/Operational".

In any case you are not sure what is the name of the log you want to activate, you can use the following Powershell command:
```PS
Get-WinEvent -ListLog *
```


## Downloading Log Monitor & Service Monitor
* [Download Log Monitor](https://github.com/microsoft/windows-container-tools/releases/tag/v2.0.1)
* [Download Service Monitor](https://github.com/Microsoft/IIS.ServiceMonitor/releases)

## Basics Of Log Monitor And Service Monitor (open source)

### Service Monitor
Service Monitor is just a tool that monitors the status of the w3svc. It will log when the status changes from SERVICE_RUNNING to either of SERVICE_STOPPED, SERVICE_STOP_PENDING, SERVICE_PAUSED or SERVICE_PAUSE_PENDING.

## Log Monitor
This tool works for both Windows and Linux Container. However, our focus is in Windows Containers.  
Log Monitor works somewhat as a publisher-subscriber design pattern. Where you **configured the "LogMonitorConfig.json" with the logs/events you want to subscribe to.**  
So, Log Monitor comprises of: LogMonitor.exe and LogMonitorConfig.json.

Supported log sources are:
* ETW.
* Event Logs.
* Log files.

Supported Images:
* Windows.
* Server Core.
* Nano.

## Initial Setup
In my root directory for this guide I have the following files:
* Dockerfile
* LogMonitor.exe
* LogMonitorConfig.json
* ServiceMonitor.exe
* Enable-IIS-Operational.ps1 

### Dockerfile
Dockerfile has the following contents:  
```Dockerfile
FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022
EXPOSE 80
#LOG MONITOR
WORKDIR /LogMonitor
COPY LogMonitorConfig.json .
COPY LogMonitor.exe .
COPY ServiceMonitor.exe .
#Enable IIS Operational
COPY Enable-IIS-Operational.ps1 .
RUN powershell .\Enable-IIS-Operational.ps1
# Set the W3SVC startup type to "demand"
RUN sc config w3svc start=demand
# Enable ETW logging for Default Web Site on IIS
RUN c:\windows\system32\inetsrv\appcmd.exe set config -section:system.applicationHost/sites "/[name='Default Web Site'].logFile.logTargetW3C:"File,ETW"" /commit:apphost
# Start LogMonitor and ServiceMonitor for W3SVC service
ENTRYPOINT ["C:\\LogMonitor\\LogMonitor.exe", "C:\\LogMonitor\\ServiceMonitor.exe", "w3svc"]
```
If you are wondering what is "ETW logging for Defautl Web Site on IIS", this lets IIS write information regarding every requests that comes into IIS to ETW. Information such as: user-client, username, referer, etc. As this is not enabled by default, we need to enable IIS ETW logging.  
### LogMonitorConfig.json
LogMonitorConfig.json has the following contents:  
Rememeber Event Viewer has different level, we need to specify not only to which log we want to subscribe, but to which level too.  
If you do not know what is the ETW provider you can identify it with this Powershell command ``` Get-ETWTraceProvider" ```
```json
{
  "LogConfig": {
    "sources": [
      {
        "type": "EventLog",
        "startAtOldestRecord": true,
        "eventFormatMultiLine": false,
        "channels": [
          {
            "name": "system",
            "level": "Information"
          },
          {
            "name": "application",
            "level": "Error"
          },
          {
            "name": "Microsoft-IIS-Configuration/Operational",
            "level": "Verbose"
          },
          {
            "name": "Microsoft-IIS-Configuration/Operational",
            "level": "Information"
          }
        ]
      },
      {
        "type": "File",
        "directory": "c:\\inetpub\\logs",
        "filter": "*.log",
        "includeSubdirectories": true
      },
      {
        "type": "ETW",
        "eventFormatMultiLine": false,
        "providers": [
          {
            "providerName": "IIS: WWW Server",
            "providerGuid": "3A2A4E84-4C21-4981-AE10-3FDA0D9B0F83",
            "level": "Information"
          },
          {
            "providerName": "Microsoft-Windows-IIS-Logging",
            "providerGuid": "7E8AD27F-B271-4EA2-A783-A47BDE29143B",
            "level": "Information"
          }
        ]
      }
    ]
  }
}

```
## Building Image
Change directory to where you have your Dockerfile
This will build the image under the name "testapp" with a tag of "v1" using the "Dockerfile" found at curernt level.
* -t = tag image.
```
docker build -t testapp:v1 .
```
Here is my output:
```
S C:\DockerLogTest\testing> docker build -t testapp:v1 .
Sending build context to Docker daemon  952.8kB
Step 1/11 : FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022
windowsservercore-ltsc2022: Pulling from windows/servercore/iis
7c76e5cf7755: Already exists
feca8e06011a: Pull complete
605c96ba0549: Pull complete
a6043d259417: Pull complete
fb40e3d76475: Pull complete
Digest: sha256:9e9c7272da9477a50bbc0642fd975f60b23605d3d38bed1c98bc582dad594012
Status: Downloaded newer image for mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022
 ---> 5c55aeaa31e7
Step 2/11 : EXPOSE 80
 ---> Running in e8031b8ea66c
Removing intermediate container e8031b8ea66c
 ---> dbf481b2eb47
Step 3/11 : WORKDIR /LogMonitor
 ---> Running in 3e73c294de64
Removing intermediate container 3e73c294de64
 ---> 21f45482613b
Step 4/11 : COPY LogMonitorConfig.json .
 ---> b491913a09b8
Step 5/11 : COPY LogMonitor.exe .
 ---> e43ebaeb3793
Step 6/11 : COPY ServiceMonitor.exe .
 ---> a28fe27a0ec3
Step 7/11 : COPY Enable-IIS-Operational.ps1 .
 ---> 396f64b48f14
Step 8/11 : RUN powershell .\Enable-IIS-Operational.ps1
 ---> Running in 2ccadb40412f
Removing intermediate container 2ccadb40412f
 ---> 8969ff428598
Step 9/11 : RUN sc config w3svc start=demand
 ---> Running in 28a6e3f5fe50
[SC] ChangeServiceConfig SUCCESS
Removing intermediate container 28a6e3f5fe50
 ---> 50815aca8b4c
Step 10/11 : RUN c:\windows\system32\inetsrv\appcmd.exe set config -section:system.applicationHost/sites "/[name='Default Web Site'].logFile.logTargetW3C:"File,ETW"" /commit:apphost
 ---> Running in a2c766065658
Applied configuration changes to section "system.applicationHost/sites" for "MACHINE/WEBROOT/APPHOST" at configuration commit path "MACHINE/WEBROOT/APPHOST"
Removing intermediate container a2c766065658
 ---> 61f382179882
Step 11/11 : ENTRYPOINT ["C:\\LogMonitor\\LogMonitor.exe", "C:\\LogMonitor\\ServiceMonitor.exe", "w3svc"]
 ---> Running in 886e1cb628e1
Removing intermediate container 886e1cb628e1
 ---> e6506dcdd6b0
Successfully built e6506dcdd6b0
Successfully tagged testapp:v1
```

## Create And Run Container
Create container under the name of "testingapp", map host port 8080 to exposed port 80 from container, use image "testapp:v1" run it in the background.
```
docker run -d -p 8080:80 --name testingapp testapp:v1
```
* -d = run container in background and print container ID.
* -p = publish a container's port to the host.
* --name = names the container.
Output:
```
PS C:\DockerLogTest\testing> docker run -d -p 8080:80 --name testingapp testapp:v1
5b2be7311d62f799b68d9c61bdb91b4a58f121f0996ae76e3f996ea23ca8f8c3
PS C:\DockerLogTest\testing>
```
## Activating IIS Worker Process
I will restart IIS and change some settings so we see these events in the logs. But before doing this we need to make sure the w3wp.exe is up and running. If it is not up, there is nothing to recycle.
Browsed to "http:localhost:8080" to activate IIS worker process. 
![](/assets/img/articles/dockerLogs/localhost8080.PNG)

## Open Interactive Shell 
Following command will open powershell from the container. We are now inside container.
``` 
docker exec -it testingapp powershell
```
* -i = kepp stdin open even if not attached.
* -t =  allocate a pseudo-TTY

IIS comes with a CLI tool that we can use for administration. This is located in "C:\Windows\System32\inetsrv". Change directory to this path and execute following commands to recycle the "DefaultAppPool", change some basic setting and the restart IIS:
  
```
.\appcmd.exe recycle apppool /apppool.name:'DefaultAppPool'
.\appcmd.exe set config -section:system.applicationHost/applicationPools /+"[name='DefaultAppPool'].recycling.periodicRestart.schedule.[value='03:00:00']" /commit:apphost
iisreset /noforce
```  
So, we should see configuration changes and recycle events in logs.

## Getting Docker Logs
Open another CMD or Powershell in your host OS, as the other one is in interactive shell, and run following command to extract logs.
Use only "docker logs <container> " to output directly into terminal. If you want to save output in a txt file, then follow format from below:  

```
docker logs testingapp >> C:\DockerLogTest\testing\dockerlogs.txt
```
Output is large, here I just pasted some events.  
We can see we have "Application", "System", "Microsoft-IIS-Configuration/Operational" and "ETW" events. We also can see configurational changes and when we did "iisreset /noforce".

**Everything is showing as expected.** 
  
```
{"Source":"File","LogEntry":{"Logline":"2023-09-26 15:32:05 172.26.128.182 GET /iisstart.png - 80 - 10.6.0.7 Mozilla/5.0+(Windows+NT+10.0;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/117.0.0.0+Safari/537.36+Edg/117.0.2045.41 http://localhost:8080/ 200 0 0 38","FileName":"LogFiles\\W3SVC1\\u_ex230926_x.log"},"SchemaVersion":"1.0.0"}
{"Source":"File","LogEntry":{"Logline":"2023-09-26 15:32:05 172.26.128.182 GET / - 80 - 10.6.0.7 Mozilla/5.0+(Windows+NT+10.0;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/117.0.0.0+Safari/537.36+Edg/117.0.2045.41 - 200 0 0 1290","FileName":"LogFiles\\W3SVC1\\u_ex230926_x.log"},"SchemaVersion":"1.0.0"}

{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:33:47.000Z","Channel": "Application","Level": "Error","EventId": 8198,"Message": "License Activation (slui.exe) failed with the following error code:\r\nhr=0x8007232B\r\nCommand-line arguments:\r\nRuleId=eeba1977-569e-4571-b639-7623d8bfecc0;Action=AutoActivate;AppId=55c92734-d682-4d71-983e-d6ec3f16059f;SkuId=ef6cfc9f-8c5d-44ac-9aad-de6a2ea0ae03;NotificationInterval=1440;Trigger=TimerEvent"}}
{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:33:51.000Z","Channel": "System","Level": "Information","EventId": 7040,"Message": "The start type of the Windows Update service was changed from disabled to demand start."}}

{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:36:50.000Z","Channel": "System","Level": "Information","EventId": 5079,"Message": "An administrator has requested a recycle of all worker processes in application pool 'DefaultAppPool'."}}
{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:37:03.000Z","Channel": "System","Level": "Information","EventId": 5080,"Message": "The worker processes serving application pool 'DefaultAppPool' are being recycled due to 1 or more configuration changes in the application pool properties which necessitate a restart of the processes."}}
{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:37:03.000Z","Channel": "Microsoft-IIS-Configuration/Operational","Level": "Verbose","EventId": 29,"Message": "Changes to '/system.applicationHost/applicationPools/add[@name=\"DefaultAppPool\"]/recycling/periodicRestart/schedule/add[@value=\"03:00:00\"]' at 'MACHINE/WEBROOT/APPHOST' have successfully been committed."}}
{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:37:03.000Z","Channel": "Microsoft-IIS-Configuration/Operational","Level": "Verbose","EventId": 29,"Message": "Changes to '/system.applicationHost/applicationPools/add[@name=\"DefaultAppPool\"]/recycling/periodicRestart/schedule/add[@value=\"03:00:00\"]/@value' at 'MACHINE/WEBROOT/APPHOST' have successfully been committed."}}
{"Source": "EventLog","LogEntry": {"Time": "2023-09-26T15:37:03.000Z","Channel": "Microsoft-IIS-Configuration/Operational","Level": "Information","EventId": 50,"Message": "Changes have successfully been committed to 'MACHINE/WEBROOT/APPHOST'."}}
```  
  
## Getting Logs Directly From Event Viewer
While in interactive shell inside container we can use "wevtutil" to backup desired logs and then transfer them to host OS.
  
```
wevtutil epl Application C:\AppLogBackup.evtx
wevtutil epl System C:\SystemBackup.evtx
wevtutil epl Microsoft-IIS-Configuration/Operational C:\ConfigurationChanges.evtx
```
  
Now we need to extract those logs from container to host OS. In order for this to work, we need to stop running container.
  
```
docker stop <container>
```
  
We can use "docker cp" to accomplish this task.
  
```
docker cp testingapp:C:\AppLogBackup.evtx C:\DockerLogTest\testing\AppLogBackup.evtx
docker cp testingapp:C:\SystemBackup.evtx C:\DockerLogTest\testing\SystemBackup.evtx
docker cp testingapp:C:\ConfigurationChanges.evtx C:\DockerLogTest\testing\ConfigurationChanges.evtx
```
  
And now you should have those files.



