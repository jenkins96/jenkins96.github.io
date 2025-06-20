---
layout: post
title: ZPHP/SmartApeSG Malware Traffic Analysis
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [book review, hacking, security]
comments: true
---

## Fake Browser Update Analysis - SmartApeSG

* ZPHP/SmartApeSG is a malware campaign that uses fake browser updates on legitimate sites to trick users into downloading and installing a Remote Access Trojan (RAT).
  
* It uses Net Support Manager software tool as a RAT, allowing remote control to adversaries.
	* NetSupport Manager is a legitimate tool for remote desktop control.
		* [NetSupport Manager](https://www.netsupportmanager.com)

## Infection Process

* High level:

1. User visits a compromise website.
2. Website displays a message saying that the Browser's version is out of date and must be updated.
3. User clicks the button to download the update.
4. User gets infected by RAT.

* A bit more detailed:

1. **User visits a compromised website (Stage 1 domain).**
	1. This is a legitimate site that has been injected with malicious JavaScript.
2. **The malicious script makes an asynchronous request to a second domain (Stage 2).**
	1. "/cdn/wds.min.php".
	2. "/cdn-js/wds.min.php".
3. **The Stage 2 server responds with heavily obfuscated JavaScript**, which:
	1. Evaluates whether the visitor is a valid target (based on geolocation, browser, etc.).
	2. If yes, it creates a hidden \<iframe> that requests:
		1. "/zwewmrqqgqnaww.php?reqtime=\<epoch>"
		2. The server behind ""/zwewmrqqgqnaww.php" responds not with visible content in the iframe, but with JavaScript that is executed in the parent page context.
4. **The iframe loads a fake browser update prompt.**
	1. This JavaScript replaces or overlays the page with the fake browser update with a prompt like: "Your browser is out of date. Please update now."
5. **User clicks the “update” button.**
	1. A malicious Base64 encoded ZIP file is downloaded, containing:
		1. A .js file (NetSupport RAT loader)	  
6. **The malware executes and installs a payload.**
	1.  NetSupport Manager is silently installed as a Remote Access Trojan (RAT).
	2.  Attacker gains persistent access to the system.

* (Don't be so hard with my drawing):

![](/assets/img/articles/SmartApeSg-Network-Analysis/img1.png)

* **Stage 1**: A compromised legitimate website that has been injected with malicious JavaScript. This site is what the user visits unknowingly.
* **Stage 2**: A malicious or attacker-controlled server that receives traffic from Stage 1 and delivers additional JavaScript or the actual malware payload.

## MITRE ATT&CK

* High level from the MITRE ATT&CK.

| **Tactic**               | **Technique**                                                          | **ID**        | **Explanation**                                                                 |
|--------------------------|------------------------------------------------------------------------|---------------|---------------------------------------------------------------------------------|
| Initial Access           | [Drive-by Compromise](https://attack.mitre.org/techniques/T1189)       | T1189         | User visits a legitimate but compromised website with malicious JS.            
| Execution                | [User Execution](https://attack.mitre.org/techniques/T1204)            | T1204         | User manually executes a downloaded ZIP, JS, or EXE file.                      
| Persistence              | [Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547) | T1547     | RAT is configured to auto-start at system boot.                     |
| Defense Evasion          | [Obfuscated Files or Information](https://attack.mitre.org/techniques/T1027) | T1027         | JavaScript and payloads are obfuscated to evade detection.                     
| Command and Control      | [Application Layer Protocol: Web Protocols](https://attack.mitre.org/techniques/T1071) | T1071     | NetSupport RAT communicate over HTTP/S.  
| Exfiltration (if present)| [Exfiltration Over C2 Channel](https://attack.mitre.org/techniques/T1041) | T1041         | Exfiltration via the same C2 channel.              |

## Network Analysis

* This site "https://www.malware-traffic-analysis.net/2024/11/26/index.html" is doing a fantastic job for guys like me that want to analyze malware infections.
* In above link there is a pcap file that the author capture in a control environment and asks us to complete some tasks.
	* **Analyzing these traces must be done in the correct way. Read authors page!**
* Given details:
	* **LAN segment range**:  10.11.26\[.]0/24 (10.11.26\[.]0 through 10.11.26\[.]255)
	* **Domain**:  nemotodes\[.]health
	* **Active Directory (AD) domain controller**:  10.11.26\[.]3 - NEMOTODES-DC
	* **AD environment name**:  NEMOTODES
	* **LAN segment gateway**:  10.11.26\[.]1
	* **LAN segment broadcast address**:  10.11.26\[.]255
* **Task**:
	* Write an incident report based on malicious network activity from the pcap and from the alerts.
	* The incident report should contains 3 sections:
		* **Executive Summary**: State in simple, direct terms what happened (when, who, what).
		* **Victim Details**: Details of the victim (hostname, IP address, MAC address, Windows user account name).
		* **Indicators of Compromise (IOCs)**: IP addresses, domains and URLs associated with the activity.  SHA256 hashes if any malware binaries can be extracted from the pcap.

### Interesting Alerts In The Given Files

* **ET CURRENT_EVENTS ZPHP Domain in DNS Lookup (modandcrackedapk.com)**
	* 10.11.26.183 -> 10.11.26.3
	* sport=52957 -> dport=53
* **ET POLICY HTTP Request on Unusual Port Possibly Hostile**
	* 10.11.26.183 -> 194.180.191.64
	* sport=53362 -> dport=443
* **ET POLICY HTTP POST on unusual Port Possibly Hostile**
	* 10.11.26.183 -> 194.180.191.64
	* sport=53362 -> dport=443
* **ETPRO TROJAN NetSupport RAT CnC Activity**
	* 10.11.26.183 -> 194.180.191.64
	* sport=53362 -> dport=443
* **ET POLICY HTTP traffic on port 443 (POST)**
	* 10.11.26.183 -> 194.180.191.64
	* sport=53362 -> dport=443
 
* These alerts are here because the IDS recognized these type of activities/traffic.

### WireShark Analysis

* Domain name "modandcrackedapk.com" is flagged as "ZPHP" in the alerts. Let's find out to which IP address resolves to.
* From below image we can also find out who  the victim likely is.

```
dns.qry.name eq modandcrackedapk.com

modandcrackedapk.com => 193.42.38.139
```

![](/assets/img/articles/SmartApeSg-Network-Analysis/img2.png)

* The rest of the alerts involves the same addresses. What I can tell from below image:
	* Victim's details:
		* **IP Address**: 10.11.26.183
		* **MAC Address**: d0:57:7b:ce:fc:8b
	* This is **HTTP traffic, but the destination port is 443**. In general, you can use any port for whatever traffic you want. Just as you can have an HTTP site over port 80, you can also use port 8080. So, yes you can use port 443 for HTTP traffic, but this will definitely attract some attention. Standard port for HTTPS traffic is 443, period, you just don't use it for anything else; you may publish your HTTPS site on another port, but you would not use port 443 for another purpose other than HTTPS (at least I don't see any other use for this port, otherwise it will just cause confusion).This is very odd and probably it was used to disguise the packets. Even WireShark alerts us to pay attention to this.
	* Client is **POST**ing data to a server.
	* The **User-Agent is the NetSupport Manager** that got installed, acting as a RAT. If your organization does not use NetSupport Manager, this is a clear red flag.
	* There are a bunch of messages between victim and C&C and they are exchanging lots of information.
	* Hostile C&C IP Address: 194.180.191.64
	
```
ip.addr eq 10.11.26.183 and ip.addr eq 194.180.191.64 and http
```

![](/assets/img/articles/SmartApeSg-Network-Analysis/img3.png)

* To find out the Windows user account we can filter for Kerberos traffic and look for the Authentication Request (AS-REQ)

```
kerberos.as_req_element
```

* NEMOTODES\oboomwald

![](/assets/img/articles/SmartApeSg-Network-Analysis/img4.png)

* To find out the actual name of the person we can find this in the LDAP protocol. In the pcap LDAP is being used instead of LDAPS, good for us. We can search for packets that contain common attributes like "givenName".
	* Oliver Q Boomwald

```
ldap.AttributeDescription eq givenName
```

![](/assets/img/articles/SmartApeSg-Network-Analysis/img5.png)

* As for how to find out which was likely the site that got compromised there is no way for me to objectively determine from within just the pcap. Maybe something I missed?? Sure, I see some funny domain names in the pcap, but I simply cannot conclude that "X" site was the site that got compromised.
	* From outside research you may find that one of these site is/was "classicgrand.com" and that would be the answer for this exercise.
 
* Traffic to "modandcrackedapk.com" and "classicgrand.com" is using TLSv1.3. There is nothing for me to see there other than the IP Address and maybe some potentially interesting TLS extension, but feels more like a dead end.

```
tls.handshake.extensions_server_name == "modandcrackedapk.com" || tls.handshake.extensions_server_name == "classicgrand.com"

# or
tls.handshake.type eq 1 # For Client Hello
```

![](/assets/img/articles/SmartApeSg-Network-Analysis/img6.png)

### Incident Report
#### Executive Summary

* On 2024-11-26 around 04:50 UTC a Windows host from one of the NEMATODES domain employee got infected with NetSupport RAT after visiting a compromised site that led to downloading the malware from a stage 2 domain (likely modandcrackedapk.com).

#### Victim Details

* **IP Address**: 10.11.26.183
* **MAC Address**: d0:57:7b:ce:fc:8b
* **Windows Account**: NEMOTODES\oboomwald
* **Name**: Oliver Q Boomwald

#### Indicators of Compromise (IOCs)

* HTTP traffic over port 443.
* Multiple POST to a IP flagged as NetSupport traffic.
	* POST http://194.180.191.64/fakeurl.htm
* ZPHP domain flagged.
	* modandcrackedapk.com
 
## Defenses:

* Train users to avoid browser update and in general, promote a "don't click blindly" culture. Updates should always come from IT Administration.
* Use DNS filtering, this can help you blocked known bad domain names. Attackers are always comming up with new domains, but it is worth using DNS filtering. 
* Monitor hidden iframes to unkown URL?
* Deploy EDR solutions.
* Block remote management from tools that your organization does not use.
* Consider Application Allow Listing.
* Prevent ZIP execution in user directories.
* Block file downloads (.zip, .js, .exe) from unknown sources.
* Monitor and block for URL specific patterns like "/cdn/wds.min.php",etc.
*  IDS/IPS.
  
## Resources

* [LDAP Attributes](https://www.ibm.com/docs/en/cip?topic=api-user-identity-attributes)
* [TRAFFIC ANALYSIS EXERCISE: NEMOTODES](https://www.malware-traffic-analysis.net/2024/11/26/index.html)
* [Are You Sure Your Browser is Up to Date? ](https://www.proofpoint.com/us/blog/threat-insight/are-you-sure-your-browser-date-current-landscape-fake-browser-updates#870527997)
* [Fake Browser Updates Leading to NetSupport RAT](https://www.trellix.com/blogs/research/new-techniques-of-fake-browser-updates/)
* [NetSupport Manager RAT from a Malicious Installer](https://forensicitguy.github.io/netsupport-manager-malicious-installer/)
* [InfoSec Detected #SmartApeSG ](https://infosec.exchange/@monitorsg/113544465582850811)
* [SmartApeSG walkthrough](https://www.threatdown.com/blog/smartapesg-06-11-2024/)
* [NetSupport RAT (Trojan) – Malware](https://cybermaterial.com/netsupport-rat-trojan-malware/)
* [MalwareBazaar Database](https://bazaar.abuse.ch/sample/8ccff473270017f72b0910ea0404d670cc6c0ebee16977accc7cbcf137ba168b/)
