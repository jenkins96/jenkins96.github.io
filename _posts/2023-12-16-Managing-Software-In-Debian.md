---
layout: post
title: Managing Software In Debian-based Distros
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [linux, debian, ubuntu, software, apt-get]
comments: true
---
## Introduction
  
This guide aims to provide a quick reference guide on how to manage software in Debian-based distributions.
  
We will start with a brief overview of basics concepts follow by dpkg command and then we will take a closer look at the apt-get command.
  
## Debian Package
  
A Debian package/software is identified by a ".deb" file extension. A package is a compress version of the software. As you know, we are constantly installing, updating, and removing software in our computers. However, this can be time consuming if it wasn't for the package manager.
  
## What Is A Package Manager?
  
A package manager is a software that allows us to easily install, upgrade, or remove software on the computer. A package manager deals with "packages", a collection of files than can be installed and removed as a group.
  
But just before starting to install software, surely there must be a list of all available software for our distribution right?
  
## Sources.list
  
The file "/etc/apt/sources.list" contains the repositories which the computer will use in order to install or update software onto the computer. A repository is a server that hold the software for particular distribution of Linux. The repositories that we have will determine: i) the available packages for download; ii) what versions of packages are available; iii) who packages the software.
  
Here is a quick look at how this file looks like. 
  
```
$ head -n 20  /etc/apt/sources.list
#deb cdrom:[Ubuntu 22.04.2 LTS _Jammy Jellyfish_ - Release amd64 (20230223)]/ jammy main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://archive.ubuntu.com/ubuntu/ jammy main restricted
deb-src http://archive.ubuntu.com/ubuntu/ jammy main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted
deb-src http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://archive.ubuntu.com/ubuntu/ jammy universe
deb-src http://archive.ubuntu.com/ubuntu/ jammy universe
deb http://archive.ubuntu.com/ubuntu/ jammy-updates universe
deb-src http://archive.ubuntu.com/ubuntu/ jammy-updates universe
```
  
So, links here points to Servers that have the information which our computer will use in order to install or upgrade software.
  
## Updating & Upgrading
  
A good idea before installing a package is to **update** our repository list (sources.list). Updating updates the list of  packages avaialble for downlaod from the repository. 
  
**Upgrading** will upgrade **the already installed packages** to the latest version in the repository.
  
**Update = updates sources.list.**
  
**Upgrade = actually upgrades installed packages to the latest version in repository.**

Updating the list of packages availablr for download from the repository.
  
```
$ sudo apt-get update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease                   
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]  
Get:3 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Hit:4 http://packages.microsoft.com/repos/code stable InRelease          
Hit:5 https://download.docker.com/linux/ubuntu jammy InRelease           
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease         
Hit:7 https://repo.protonvpn.com/debian stable InRelease
Get:8 http://archive.ubuntu.com/ubuntu jammy-updates/main i386 Packages [547 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1 263 kB]
Fetched 2 039 kB in 3s (738 kB/s) 
Reading package lists... Done
```
  
Upgrading existing packages in my system:
  
```
$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```
    
You can also use **"sudo apt-get dist-upgrade"** since this handle situations where a package upgrade requires the installation of new dependencies or the removal of conflicting packages
sudodist-upgrade
  
## dpkg
  
The lowest-level tool for managing these files is the dpkg command. Dpkg is the Debian package management system.

List all available packages:
  
```
dpkg -l
```
  
We can use "grep" command to seach for a package:
  
```
dpkg -l | grep name_of_package
```
  
Installing a package:
  
```
sudo dpkg -i name_of_package.deb
```
  
Removing a package:
  
```
sudo dpkg -r name_of_package.deb
```
  
Use the "-P" option to remove everything including configuration files.
  
## apt-get
  
The Advanced Package Tool, apt-get is the front-end to the dpkg tool. It makes management of packages easier. You might have seen "apt" and "apt-get", there are a few differences in terms of options and what they do, but they are very similar.
  
Searching for a firewall software called "ufw":
  
```
$ apt-cache search ufw
ufw - program for managing a Netfilter firewall
gufw - graphical user interface for ufw
plasma-firewall - Plasma configuration module for firewalls
```
  
Installing "gufw" package:
  
```
$ sudo apt-get install gufw
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  gufw
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 954 kB of archives.
After this operation, 3 673 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/universe amd64 gufw all 22.04.0-0ubuntu1 [954 kB]
Fetched 954 kB in 3s (325 kB/s)
Selecting previously unselected package gufw.
(Reading database ... 272351 files and directories currently installed.)
Preparing to unpack .../gufw_22.04.0-0ubuntu1_all.deb ...
Unpacking gufw (22.04.0-0ubuntu1) ...
Setting up gufw (22.04.0-0ubuntu1) ...
Processing triggers for mailcap (3.70+nmu1ubuntu1) ...
Processing triggers for desktop-file-utils (0.26-1ubuntu3) ...
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for gnome-menus (3.36.0-1ubuntu3) ...
Processing triggers for man-db (2.10.2-1) ...
```
  
Removing software:

This does not remove the configuration files. 
  
```
$ sudo apt-get remove gufw
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be REMOVED:
  gufw
0 upgraded, 0 newly installed, 1 to remove and 3 not upgraded.
After this operation, 3 673 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 272784 files and directories currently installed.)
Removing gufw (22.04.0-0ubuntu1) ...
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for gnome-menus (3.36.0-1ubuntu3) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for mailcap (3.70+nmu1ubuntu1) ...
Processing triggers for desktop-file-utils (0.26-1ubuntu3) ...

```
  
Use "purge" to uninstall and remove configuration files:
  
```
apt-get purge gufw
```
  
Removing dependencies:
  
```
apt autoremove gufw
```
  
We can also combine purge and autoremove:
  
```
$ sudo apt-get purge --auto-remove gufw 
```
  
Some other useful commands:
  
```
apt-cache stats #shows statistic
apt-cache unmet #shows unmet dependencies
apt-cache depends apache2 #shows what packages depends on apache2
apt-cache rdepends apache2 #shows what packages does apache2 depends on.
```
  
## In Summary:
  
* A Debian package ends in ".deb" file extension.
* A package is a compress version of the software.
* A package manager is a software that allows us to easily install, upgrade, or remove software on the computer.
* The file "/etc/apt/sources.list" contains the repositories which the computer will use in order to install or update software onto the computer.
* A repository is a server that hold the software for particular distribution of Linux.
* Update = updates sources.list.
* Upgrade = actually upgrades installed packages to the latest version in repository.
* It is not mandatory, however it is a good idea to update our repository before installing a package.
* Dpkg is the Debian package management system.
* apt-get is the front-end to the dpkg tool
* If you are looking for a GUI package manager you can take a look at [Synaptic](https://wiki.debian.org/Synaptic) and [Gdebi](https://packages.debian.org/search?keywords=gdebi).

## Resoureces
* [Package Manager](https://www.debian.org/doc/manuals/aptitude/pr01s02.en.html)
* [Difference Between APT and DPKG in Ubuntu](https://www.geeksforgeeks.org/difference-between-apt-and-dpkg-in-ubuntu/)
* [Wiki Sources.list](https://wiki.debian.org/SourcesList)
* [Debian Sources.list](https://linuxhint.com/debian_sources-list/)
* [Difference Between APT and DPKG in Ubuntu](https://www.geeksforgeeks.org/difference-between-apt-and-dpkg-in-ubuntu/)
* [Debian Software Package Management(dpkg) in Linux](https://www.geeksforgeeks.org/debian-software-package-managementdpkg-in-linux/)
* [The Debian package management tools](https://www.debian.org/doc/manuals/debian-faq/pkgtools.en.html)




