---
layout: post
title: FlareVM Pre-Requisites And Installation
subtitle: FlareVM Pre-Requisites
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [malware, digital forensics, security]
comments: true
---

## What Is FlareVM

FlareVM is a Windows-based virtual machine (VM) pre-configured for malware analysis, reverse engineering, and cybersecurity research. Essentially this is a collection of scripts that downloads specialized software for malware analysis and reverse engineering. 

## Requirements & Pre-Requisites

* This is meant to be installed on a Virtual Machine.
* Windows >= 10.
* At least 60GB and 2GB RAM.
* Disable Tamper Protection.
* Disable any kind of AntiVirus(e.g., Windows Defender).
* Disable Windows Firewall.
* Disable Windows Updates.

## Pre-Requisites Process

* Here is a quick drawing:

![](../assets\img\articles\Flare-VM\flarevm-pre.png)


* Disable Tamper Protection
    * Defender > Virus & Threat Protection > Viruts & Threat Protection Settings > Manage Settings and disable all options.

![](../assets\img\articles\Flare-VM\img1.png)

* Disable any kind of AntiVirus(e.g., Windows Defender).
    * Administrative Templates > Windows Components > Microsoft Defender Antivirus > "Turn off Microsoft Defender Antivirus" **(Enable this)**

![](../assets\img\articles\Flare-VM\img2.png)

* Disable Windows Firewall.
    * Administrative Templates > Network > Network Connections >Windows Defender Firewall
        * Domain Profile > Windows Defender Firewall: Protect all network connections = **Disabled**
        * Standard Profile > Windows Defender Firewall: Protect all network connections = **Disabled**

![](../assets\img\articles\Flare-VM\img3.png)
![](../assets\img\articles\Flare-VM\img4.png)

* Disable Windows Updates.
    * Administrative Templates > Windows Components > Windows Update > Configure Automatic Updates = Disabled

![](../assets\img\articles\Flare-VM\img5.png)

* Highly recommended to take a SnapShot of a "FlareVM-Pre-Installation".

## Installation

* Open PowerShell in Admin mode.

```PS
# Download the script
(New-Object net.webclient).DownloadFile('https://raw.githubusercontent.com/mandiant/flare-vm/main/install.ps1',"$([Environment]::GetFolderPath("Desktop"))\install.ps1")

# Change Directory to your Desktop.
# Unblock File
Unblock-File .\install.ps1

# ExecutionPolicy
Set-ExecutionPolicy Unrestricted

# (If, when running the script you get an error complaining about "System.Drawing", then run this before the script)
Add-Type -Assembly System.Drawing

# Run it
.\install.ps1 

```

* **This will take a really long time and it will reboot several times.**

* When ready, again it is recommended to take another SnapShot after successfull FlareVM installation.

![](../assets\img\articles\Flare-VM\img6.png)


