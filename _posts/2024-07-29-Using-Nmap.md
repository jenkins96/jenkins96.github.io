---
layout: post
title: Using Nmap
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [cybersecurity, scanning, nmap]
comments: true
---

## What Is Nmap?

Nmap is a Network Mapper. It is an open-source Linux tool that supports TCP, UDP, ICMP, and SCTP protocols. This tool allows us to do :
  
* Port Scanning: probing host/server to enumerate running services on each port.

* Network Scanning: the process of identifying active hosts in a network as well as opened ports and their services.

So, this tool can help us scan ports, services, networks, and applications and even detect potential vulnerabilities that might be exploited.


## What Is A Port?

A software socket that gives access to the device, and then an application can support that port and we can interact with this application through that port.

It is a software abstraction, used to distinguish between communication channels.

So, ports allow us to differentiate between different services. We know traffic coming on port 80 is HTTP traffic, while port 69 is for TFPT traffic.

The range of available TCP and UDP ports is from 0 to 65535. This range is divided into:

* **Well-Known/System Ports**: From 0 to 1023. These ports are used by the system processes to provide different types of services.

* **Registered Ports**: From 1024 to 49151. These ports are assigned by IANA for a specific service.

* **Dynamic/Ephemeral Ports**: Typically, from 49152-65535. These ports cannot be registered with IANA. However, ephemeral port definitions are defined per OS's version.

## Three-way Handshake

HTTP is based on TCP. Before the very first HTTP message can be sent, we must establish a TCP connection, during this process what is known as the "three-way handshake" must happen before sending an HTTP message.

In short, the sender sends a SYNchronize sequence number. Then, the server ACKnowledges that it received the previous message and also sends its own SYNchronize sequence number. Finally, the client ACKnowledges it received the previous message.

So:

1. Client: SYN.
2. Server: SYN, ACK.
3. Client: ACK.

Of course, there are many interesting details regarding the three-way handshake that we are leaving out here. If you want to understand these messages, please research this topic.

## What Can I Scan?

Anything that you have permission for and under the scope that it is allowed.

For example, we can scan "scanme.nmap.org" under the following scope:

> Hello, and welcome to Scanme.Nmap.Org, a service provided by the Nmap Security Scanner Project.
We set up this machine to help folks learn about Nmap and also to test and make sure that their Nmap installation (or Internet connection) is working properly. You are authorized to scan this machine with Nmap or other port scanners. Try not to hammer on the server too hard. A few scans in a day is fine, but don't scan 100 times a day or use this site to test your ssh brute-force password cracking tool. 

[scanme.nmap.org](http://scanme.nmap.org/)

## Installing Nmap
  
* [Download nmap](https://nmap.org/download)

## Nmap Help

The "man" pages or the "--help" option should be the very first place to look for help. Of course, the Internet is another great as well!
    
What I like to do is, say I need to understand what "-sn" option means, so I would type: 

```
nmap --help | grep "\-sn"
```
  
```
nmap --help | grep "\-sn"
  -sn: Ping Scan - disable port scan
  nmap -v -sn 192.168.0.0/16 10.0.0.0/8
```

Another example:

```
nmap --help | grep "UDP"
  -PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
  -sU: UDP Scan
  --badsum: Send packets with a bogus TCP/UDP/SCTP checksum
```

## Nmap Port States 

Here are the three by just typing `nmap`` TARGET IP`plication is actively accepting connections. We sent SYN and received a SYN-ACK back.

* **Closed**: A closed port is accessible (it receives and responds to Nmap probe packets), but there is no application listening on it.  Because closed ports are reachable, it may be worth scanning later in case some open up. Administrators may want to consider blocking such ports with a firewall. We sent SYN and received RST back.

* **Filtered**: Nmap cannot determine whether the port is open because packet filtering prevents its probes from reaching the port. The filtering could be from a dedicated firewall device, router rules, or host-based firewall software. We sent SYN but we did not receive SYN ACK or RST. So it might be that the port is being filtered or there is a Firewall in place.

In reality, nmap has three port states:

* **Unfiltered**: The unfiltered state means that a port is accessible, but Nmap is unable to determine whether it is open or closed.

* **Open\|filtered**: Nmap places ports in this state when it is unable to determine whether a port is open or filtered. This occurs for scan types in which open ports give no response. The lack of response could also mean that a packet filter dropped the probe or any response it elicited. So Nmap does not know for sure whether the port is open or being filtered. 

* **Close-filtered**: This state is used when Nmap is unable to determine whether a port is closed or filtered. It is only used for the IP ID idle scan.

## Using Nmap

We can get started by just typing ``` nmap TARGET IP ``` . This scans 1000 TCP ports and makes a full connect scan, meaning it completes the three-way handshake connection. Nmap asks the underlying operating system to establish a connection with the target machine and port by issuing the connect system call. 

Nmap will make a full connect scan (the equivalent of -sT option) when SYN scan is not an option. When the user does not have raw packet privileges (admin rights), Nmap will use the same system call that a Browser does.

So, when a SYN scan is not possible, a full connect scan is used.

Take for example this result:

```
$ nmap scanme.nmap.org
Starting Nmap 7.80 ( https://nmap.org ) at 2024-07-26 08:19 CST
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.11s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 993 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
25/tcp    filtered smtp
80/tcp    open     http
2000/tcp  open     cisco-sccp
5060/tcp  open     sip
9929/tcp  open     nping-echo
31337/tcp open     Elite

Nmap done: 1 IP address (1 host up) scanned in 8.99 seconds
```

* 1000 ports were scanned.
* 993 ports are closed.
* 6 ports are opened 
* 1 port is filtered.
* We can see the services running on each port.

Let's dive deeper and see what messages were exchanged for port 2000.

The result tells us port 2000 is open and the service running on this port is "cisco-sccp". But how does Nmap determine if it is open or closed? Well, when you do not have admin rights, the default Nmap scan is a full connection scan. This means that it does the full three-way handshake.

1. The client sends a SYN message.
2. The server sends back a SYN and ACK messages.
3. The client sends an ACK message and immediately closes the connection with a RST message.

As the server responded with a SYN ACK, I know the port is open.

![](/assets/img/articles/nmap/img0.png)

In contrast to port 25, where we could not determine the state of the port because we sent several SYN messages and never got any type of response back. So, Nmap cannot be sure what is the state of this port, meaning a firewall could potentially be in place.

![](/assets/img/articles/nmap/img1.png)

If a port is closed, the server will immediately close the connection by sending a RST flag. We can see this behavior for port 53, where Nmap sends SYN flag and the server closes the connection by sending RST flag.

![](/assets/img/articles/nmap/img2.png)

Up until now, no response from the server might mean there could be a firewall in place, therefore, Nmap cannot determine the state of the port. If the server replies with RST flag, nmap knows the port is closed. 

So:

* When a SYN scan cannot be done, such as when the user does not have admin rights, the default scan is the "sT", which completes the three-way handshake.

* When there is no firewall/device in place, the normal behavior of the Server when a port is closed is to send RST.

## Scanning Range Of Hosts-Subnet-IP Range

We can also scan a range of IPs. All the following are valid:

```
# One IP
nmap 10.106.0.169

# Two IPs
nmap 10.106.0.1 10.106.0.41 10.106.0.3 

# IP range
nmap 10.106.0.1/24
nmap 10.106.0.*
nmap 10.106.0.1-10

# Scan where decimal last octet is 1, 2, 3, and 4
nmap 10.106.0.1,2,3,4

# Scan two or more subnets
10.10.10.0/24 192.168.4.0/24

# You can also create a list of hosts and pass it to nmap
nmap -iL <listOfHosts>
```
  
## Most Common Scans

### Ping Scan (-sn)

If non-privilege mode, Nmap does not send ICMP. Ping is a TCP scan

> This option tells Nmap not to do a port scan after host discovery, and only print out the available hosts that responded to the host discovery probes. This is often known as a “ping scan”. It allows light reconnaissance of a target network without attracting much attention. Knowing how many hosts are up is more valuable to attackers than the list provided by list scan of every single IP and host name. The default host discovery done with -sn consists of an ICMP echo request, TCP SYN to port 443, TCP ACK to port 80, and an ICMP timestamp request by default. When executed by an unprivileged user, only SYN packets are sent (using a connect call) to ports 80 and 443 on the target. When a privileged user tries to scan targets on a local ethernet network, ARP requests are used unless --send-ip was specified.

Let's make our very first ping scan.

```
$ nmap -sn scanme.nmap.org
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-27 21:21 CST
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.20s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Nmap done: 1 IP address (1 host up) scanned in 2.49 seconds
```

Nmap sent two SYN packets, one to port 80 and the other one to port 443. We then received a SYN-ACK from the server from port 80, nmap knows it is open and proceeds to RST connection in frame 16. However, for port 443, the server sent back RST in frame 15, so Nmap knows port 443 is closed.

![](/assets/img/articles/nmap/img3.png)

If you pay attention to the image, you would have noticed that there are no ping requests (IMCP). This happens because the scan was not made with root privileges.

Re-running the scan with sudo privileges we do see the ping requests:

![](/assets/img/articles/nmap/img4.png)

Now we do see the ICMP messages. However, take note that the "ping scan" also sends more than just ICMP traffic. It is also sending a TCP-SYN scan for port 443 and a TCP-ACK scan for port 80. We will discuss these options later on. As for now, just be mindful that option "-sn" does not send just ICMP traffic and that behavior changes depending if the user has admin rights or not.

* If you want to scan a network that is not local to you and you want to send ICMP ping request, make sure to use administrative mode. If not, it will only do TCP protocol.

### Top N Ports (--top-ports)

We can use the "--top-ports <NUMBER>" to scan top "20" ports, for example. This sends TCP-SYN packets to all those 20 ports.

```
$ nmap --top-ports 20 192.168.18.25
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-27 21:36 CST
Nmap scan report for 192.168.18.25
Host is up (0.00030s latency).

PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
23/tcp   open   telnet
25/tcp   open   smtp
53/tcp   open   domain
80/tcp   open   http
110/tcp  closed pop3
111/tcp  open   rpcbind
135/tcp  closed msrpc
139/tcp  open   netbios-ssn
143/tcp  closed imap
443/tcp  closed https
445/tcp  open   microsoft-ds
993/tcp  closed imaps
995/tcp  closed pop3s
1723/tcp closed pptp
3306/tcp open   mysql
3389/tcp closed ms-wbt-server
5900/tcp open   vnc
8080/tcp closed http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.16 seconds

```

### OS Version Scan (-O)

> One of Nmap's best-known features is remote OS detection using TCP/IP stack fingerprinting. Nmap sends a series of TCP and UDP packets to the remote host and examines practically every bit in the responses. After performing dozens of tests such as TCP ISN sampling, TCP options support and ordering, IP ID sampling, and the initial window size check, Nmap compares the results to its nmap-os-db database of more than 2,600 known OS fingerprints and prints out the OS details if there is a match. 

Nmap sends a series of TCP and UDP packets to the remote host and examines the response. Nmap then compares the response to nmap-os-db database of more than 2,600 known OS fingerprints and prints out the OS details if there is a match.

```
$ sudo nmap -O 192.168.18.26
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 13:55 CST
Nmap scan report for 192.168.18.26
Host is up (0.00044s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 08:00:27:E8:06:77 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Microsoft Windows XP
OS CPE: cpe:/o:microsoft:windows_xp::sp2 cpe:/o:microsoft:windows_xp::sp3
OS details: Microsoft Windows XP SP2 or SP3
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.78 seconds
```

![](/assets/img/articles/nmap/img5.png)

### Aggresive Scan (-A)

> This option enables additional advanced and aggressive options. Presently this enables OS detection (-O), version scanning (-sV), script scanning (-sC) and traceroute (--traceroute). However, because script scanning with the default set is considered intrusive, you should not use -A against target networks without permission. Options which require privileges (e.g. root access) such as OS detection and traceroute will only be enabled if those privileges are available. 

This option enables additional advanced and aggressive options. Presently this enables OS detection (-O), version scanning (-sV), script scanning (-sC) and traceroute (--traceroute). However, because script scanning with the default set is considered intrusive, you should not use -A against target networks without permission. Options that require privileges (e.g. root access) such as OS detection and traceroute will only be enabled if those privileges are available.

```
$ sudo nmap -A 192.168.18.25
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 14:09 CST
Nmap scan report for 192.168.18.25
Host is up (0.00022s latency).
Not shown: 977 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.18.12
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
|_ssl-date: 2024-04-28T20:10:21+00:00; 0s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
53/tcp   open  domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 8
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, SupportsTransactions, SwitchToSSLAfterHandshake, ConnectWithDatabase, LongColumnFlag, Speaks41ProtocolNew, SupportsCompression
|   Status: Autocommit
|_  Salt: k~[,hGke:.w/#a?UnIa0
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
|_ssl-date: 2024-04-28T20:10:21+00:00; 0s from scanner time.
5900/tcp open  vnc         VNC (protocol 3.3)
| vnc-info: 
|   Protocol version: 3.3
|   Security types: 
|_    VNC Authentication (2)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
| irc-info: 
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN 
|   uptime: 0 days, 0:21:46
|   source ident: nmap
|   source host: A24FFFC4.E33C28C1.FFFA6D49.IP
|_  error: Closing Link: tbqssbbkk[192.168.18.12] (Quit: tbqssbbkk)
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/5.5
MAC Address: 08:00:27:6B:33:B3 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
|_smb-security-mode: ERROR: Script execution failed (use -d to debug)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE
HOP RTT     ADDRESS
1   0.22 ms 192.168.18.25

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.81 seconds
```

### Port Scanning (-p)

It is always best to just scan the port or ports that you are interested in rather than scanning all the ports. More ports equals more time and more chance of getting detected.

```
# Scan port 80
nmap -p 80 <IP>

# Scan port 80 through 443
nmap -p 80-443 <IP>

# Scan ports 80, 22, and 443
nmap -p 80,22,443 <IP>
```

### Stealth Scan (-sS)

> SYN scan is the default and most popular scan option for good reason. It can be performed quickly, scanning thousands of ports per second on a fast network not hampered by intrusive firewalls. SYN scan is relatively unobtrusive and stealthy, since it never completes TCP connections. It also works against any compliant TCP stack. It also allows clear, reliable differentiation between open, closed, and filtered states. SYN scan may be requested by passing the -sS option to Nmap. It requires raw-packet privileges, and is the default TCP scan when they are available. So when running Nmap as root or Administrator, -sS is usually omitted. 


In this scan nmap will send a SYN packet, the server will reply with the SYN-ACK, and then, nmap will RST the connection, without completing the three-way handshake. This is a half-open connection.

The idea behind this technique is that, as the three-way handshake never completes, it might not be logged on the device.

This is the default scan when you have Administrator privileges.

```
$ sudo nmap -sS scanme.nmap.org
[sudo] password for user: 
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 16:33 CST
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.13s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 993 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
25/tcp    filtered smtp
80/tcp    open     http
2000/tcp  open     cisco-sccp
5060/tcp  open     sip
9929/tcp  open     nping-echo
31337/tcp open     Elite

Nmap done: 1 IP address (1 host up) scanned in 19.41 seconds
```

Here is what it would look like in WireShark:

![](/assets/img/articles/nmap/img7.png)

However, these types of scans are not that difficult to catch because when Nmaps crafts these packages, it sets a Window size of 1024. The Window size is simply an advertisement of how much data (in bytes) the receiving device is willing to receive before sending an ACK back. A window size of 1024 is oddly small and suspicious.

Using a display filter to get all packets with TCP window_size_value==1024, we can get all Nmap packets:

![](/assets/img/articles/nmap/img8.png)
![](/assets/img/articles/nmap/img8.1.png)

The Maximum Segment Size (MSS) is the largest TCP segment that can be transported in a single TCP packet. So, I can handle 1460 bytes of data, in a single, unfragmented, packet, but my recieve buffer is only 1024 bytes. This is rather odd, typically we see values such as MSS=1460 and Window Size=64240 that make more sense.

### TCP Connect Scan (-sT)

> TCP connect scan is the default TCP scan type when SYN scan is not an option. This is the case when a user does not have raw packet privileges or is scanning IPv6 networks. Instead of writing raw packets as most other scan types do, Nmap asks the underlying operating system to establish a connection with the target machine and port by issuing the connect system call. When SYN scan is available, it is usually a better choice. Nmap has less control over the high level connect call than with raw packets, making it less efficient. The system call completes connections to open target ports rather than performing the half-open reset that SYN scan does. Not only does this take longer and require more packets to obtain the same information, but target machines are more likely to log the connection. 

NMmap uses the TCP stack to send the message. This is how, for example, a Browser would establish a connection.

This completes the three-way handshake and then we immediately send RST packet.

However, we do risk the device logging this connection.

![](/assets/img/articles/nmap/img9.png)

This is the default scan when a SYN scan (-sS) cannot be done.

### TCP NULL, FIN & Xmas Scans (-sN -sF -sX)

> These three scan types (even more are possible with the --scanflags option described in the next section) exploit a subtle loophole in the TCP RFC to differentiate between open and closed ports. Page 65 of RFC 793 says that “if the [destination] port state is CLOSED .... an incoming segment not containing a RST causes a RST to be sent in response.” Then the next page discusses packets sent to open ports without the SYN, RST, or ACK bits set, stating that: “you are unlikely to get here, but if you do, drop the segment, and return.”
https://www.rfc-editor.org/rfc/rfc793.txt


> These three scan types are exactly the same in behavior except for the TCP flags set in probe packets. If a RST packet is received, the port is considered closed, while no response means it is open\|filtered. The port is marked filtered if an ICMP unreachable error (type 3, code 0, 1, 2, 3, 9, 10, or 13) is received.

The key advantage of these scan types is that they can sneak through certain non-stateful firewalls and packet-filtering routers. Another advantage is that these scan types are a little more stealthy than even a SYN scan. Don't count on this though—most modern IDS products can be configured to detect them. The big downside is that not all systems follow RFC 793 to the letter. Several systems send RST responses to the probes regardless of whether the port is open or not. This causes all of the ports to be labeled closed.

> When scanning systems compliant with this RFC text, any packet not containing SYN, RST, or ACK bits will result in a returned RST if the port is closed and no response at all if the port is open. As long as none of those three bits are included, any combination of the other three (FIN, PSH, and URG) are OK. Nmap exploits this with three scan types:

#### Null Scan (-sN)

It does not set any bits (TCP flag header is 0). It has no flags set on.

The idea is that if I send you a TCP packet with no flags and server RST connection, then Nmaps knows is closed.

```
$ sudo nmap -sN 192.168.18.26
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 19:47 CST
Nmap scan report for 192.168.18.26
Host is up (0.0027s latency).
All 1000 scanned ports on 192.168.18.26 are closed
MAC Address: 08:00:27:E8:06:77 (Oracle VirtualBox virtual NIC)
```

![](/assets/img/articles/nmap/img10.png)

#### X Scan(-sX)

Sets the FIN, PSH, and URG flags, lighting the packet up like a Christmas tree.

Flags URG, PSH, FIN will be set.

```
$ sudo nmap -sX 192.168.18.26
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 19:52 CST
Nmap scan report for 192.168.18.26
Host is up (0.0022s latency).
All 1000 scanned ports on 192.168.18.26 are closed
MAC Address: 08:00:27:E8:06:77 (Oracle VirtualBox virtual NIC)

```

![](/assets/img/articles/nmap/img11.png)

#### Fin Scan (-sF)

Sets just the TCP FIN bit. Some stacks do not respond at all, so that would be considered an opened port.

```
$ sudo nmap -sF 192.168.18.26
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 19:56 CST
Nmap scan report for 192.168.18.26
Host is up (0.00084s latency).
All 1000 scanned ports on 192.168.18.26 are closed
MAC Address: 08:00:27:E8:06:77 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 1.57 seconds
```

![](/assets/img/articles/nmap/img12.png)

However, many TCP stacks these days will reply with an RST if they receive something they do not understand.

Also, stateful firewall will typically drop the package if they don't know how to respond to it.

### UDP Scan (-sU)

> UDP scan works by sending a UDP packet to every targeted port. For most ports, this packet will be empty (no payload), but for a few of the more common ports a protocol-specific payload will be sent. Based on the response, or lack thereof, the port is assigned to one of four states,

| Probe Response | Assigned State       |
|----------------|----------------------|
| Any UDP response from target port (unusual) | open |
| No response received (even after retransmissions) | open\|filtered |
| ICMP port unreachable error (type 3, code 3) | closed |
| Other ICMP unreachable errors (type 3, code 1, 2, 9, 10, or 13) | filtered |


> The most curious element of this table may be the open\|filtered state. It is a symptom of the biggest challenges with UDP scanning: open ports rarely respond to empty probes. Those ports for which Nmap has a protocol-specific payload are more likely to get a response and be marked open, but for the rest, the target TCP/IP stack simply passes the empty packet up to a listening application, which usually discards it immediately as invalid. If ports in all other states would respond, then open ports could all be deduced by elimination. Unfortunately, firewalls and filtering devices are also known to drop packets without responding. So when Nmap receives no response after several attempts, it cannot determine whether the port is open or filtered. 

> UDP scan works by sending a UDP packet to every targeted port. For most ports, this packet will be empty (no payload), but for a few of the more common ports, a protocol-specific payload will be sent. Based on the response, or lack thereof, the port is assigned to one of four states.

**open\|filtered** = got no response. we sent an empty packet, the application received it and did nothing as is empty OR it got filtered by a Firewall.

![](/assets/img/articles/nmap/img14.png)

**open** = nmap generates a legitimate NTP request and the server responds. The actual service is open and available.

![](/assets/img/articles/nmap/img13.png)

**closed** = ICMP unreachable. Several UDP messages, no response back and then ICMP Destination unreachable.

![](/assets/img/articles/nmap/img15.png)

```
$ sudo nmap -sU -F -T3 scanme.nmap.org
Starting Nmap 7.80 ( https://nmap.org ) at 2024-04-28 20:28 CST
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.19s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 95 closed ports
PORT     STATE         SERVICE
67/udp   open|filtered dhcps
68/udp   open|filtered dhcpc
123/udp  open          ntp
520/udp  open|filtered route
5060/udp open|filtered sip

Nmap done: 1 IP address (1 host up) scanned in 104.58 seconds
```

### Version Discovery Scan (-sV)

The version of a service that is running on a service.

The idea first is to do a TCP scan to determine if the port is open or not. If it is open, Nmap will either make a request or wait for a response from the server and inspect any indication regarding the version of the service.

Port 80
![](/assets/img/articles/nmap/img16.png)

Port 22:
![](/assets/img/articles/nmap/img17.png)


## Exporting Nmap Result

```
-oN <fileName> # text
-oX <fileName> # xml
-oS <fileName> # script kiddie
-oG <fileName> # grepable format
```

## Scan Timing & Performance

This determines how aggressive we want that scan to be. Do we the scan to be as quick as possible or slow and gentle?

Slower might give us a better chance of not being detected, but can potentially take a long time to complete.

T3 is the default

* Paranoid (T0)
* Sneaky (T1)
* Polite (T2)
* Normal (T3)
* Aggressive (T4)
* Insane (T5)

Here you can take a look at the chart [Timing Templates (-T)](https://nmap.org/book/performance-timing-templates.html)

## Nmap Scripting Engine (NSE)

This allows us to write and share our own scripts (Lua language).

Every Nmap installation comes prepacked with a built-in script database located at: "/usr/share/nmap/scripts".

```
$ ls  /usr/share/nmap/scripts | head
acarsd-info.nse
address-info.nse
afp-brute.nse
afp-ls.nse
afp-path-vuln.nse
afp-serverinfo.nse
afp-showmount.nse
ajp-auth.nse
ajp-brute.nse
ajp-headers.nse
```
Here is the list of all scripts available in Nmap [NSE Scripts](https://nmap.org/nsedoc/scripts/)
We can run several scripts against a target.

Check if we have the latest database:

```
sudo nmap --script-updatedb
```
To run a script you start with your basic nmap command and add "--script <SCRIPTNAME>" option.

We can run several scripts at once. Suppose you want to run all scripts that start with "ftp-", then:

```
nmap --script ftp-* <TARGETIP>
```

**External Scripts**

You can also use external scripts. Take for example
[vulscan script](https://github.com/scipag/vulscan).

Clone the repository and then you can pass the location of the script into the "--script" option.

```
nmap --script vulscan <TARGETIP>
```

## Firewall/IDS Evasion & Spoofing

### IP Fragmentation

The idea is to fragment the packet so that the firewall cannot reassemble the packet. However, nowadays they typically reassemble packets. Use this as a learning exercise.

When using the option "-f" it will fragment into 8 bytes each. This requires admin privileges.

Notice the data is 8 bytes and the packet itself is saying "more fragments are coming".

```
$ sudo nmap -p 80 -f scanme.nmap.org
```

![](/assets/img/articles/nmap/img18.png)

### Spoof IP Address

Take into consideration that the response will get back to the spoof IP Address, not yours. So, this tends to be a one-way communication.

You also need to provide the name of your interface.

```
sudo nmap -sS -p 80 -S <spoof IP> -e <interface> scanme.nmap.org

# Example
sudo nmap -sS -p 80 -S 192.168.44.44 -e en0  scanme.nmap.org
```

### Decoys

When using decoys we are hiding our real address is a bunch of fake addresses. For this, we can use the "-D" option. We can pass the IP's that we want to use or we can tell Nmap to randomly generate IP's addresses.

Nmap already knows our real IP, so it is not necessary to specify this in the spoofed IP Addresses.

**Using one fake IP Address**
```

sudo nmap -sS -p 80 -D 45.45.45.45 -e <int> scanme.nmap.org

```
![](/assets/img/articles/nmap/img19.png)

**Using multiple fake IP Addresses**

```
sudo nmap -sS -p 80 -D 45.45.45.45,20.20.20.20,53.53.53.53 -e <int>  scanme.nmap.org
```

![](/assets/img/articles/nmap/img19.png)

**Using random fake IP Address**

Use "-D RND:<DesiredNumberOfIPs>".

```
sudo nmap -sS -p 80 -D RND:5 -e <int>  scanme.nmap.org
```

![](/assets/img/articles/nmap/img21.png)

### Spoofing MAC Address

```
sudo nmap -p 80 --spoof-mac 11:22:33:44:55:66 -Pn scanme.nmap.org
```

![](/assets/img/articles/nmap/img21.png)

### Change Source Port

There could potentially be a situation where we know that the firewall is only allowing traffic from certain source ports.

We can tell Nmap to use a specific source port with the option "--source-port <PORT>"

```
sudo nmap -p 80 --source-port 49001 scanme.nmap.org
```

![](/assets/img/articles/nmap/img23.png)

## Best Practices

* Your scan needs to be as specific as possible.
* Don't use port scan if not need it. Stick with a ping scan if it suffices.
* Avoid "-A" and "-O" options if not needed.
* Specify the port to be more specific.
* Take into consideration what you are putting on the wire when running the scan.
* Keep in mind and read the documentation regarding what the scan will send depending if the user has admin rights or not.
* Scan timing, fragmentation, decoys, etc are good for evasion. But keep in mind nowadays firewalls can identify these types of tactics.
* Do not scan if you do not have explicit permission from the owner.
* Read the Nmap port state documentation.
* If no firewall is in place, the normal behavior from the Server when a port is closed is to send RST the connection.
* Nmap has an in-built collection of scripts that you can use.
* Nmap allows for the use of external scripts.
* Capture network traces to understand what is actually being sent.
* You can use levels of verbosity: -v or -vv for double verbosity.

### Nmap Options (Not Exhaustive)

| Option | Description |
| --- | --- |
| **-sn** | Ping scan. Tells Nmap not to run a port scan after host discovery. |
| **-sS** | SYN scan. The fastest way to scan ports due to its nature. SYN scan is relatively unobtrusive and stealthy, as it never completes TCP connections. It's stealthier than a connect scan, but it can be less accurate. This technique is often referred to as half-open scanning because you don't open a full TCP connection. You send a SYN packet, as if you are going to open a real connection, and then wait for a response. A SYN/ACK indicates the port is listening (open), while a RST (reset) is indicative of a non-listener. If no response is received after several retransmissions, the port is marked as filtered. The port is also marked filtered if an ICMP unreachable error (type 3, code 0, 1, 2, 3, 9, 10, or 13) is received. The port is also considered open if a SYN packet (without the ACK flag) is received in response. This can be due to an extremely rare TCP feature known as a simultaneous open or split handshake connection. |
| **-sV** | Probe open ports to determine service/version info. |
| **-A** | Aggressive scan. Enables OS detection, version detection, script scanning, and traceroute. |
| **-p** | Specifies a specific port or range of ports. For example: "-p 80"; "-p 0-65535" or "-p-". |
| **-iL** | Input filename containing a list of hosts/networks to scan. |
| **-v** | First level of verbosity. |
| **-vv** | Second level of verbosity. |
| **-oN [file]** | Write output in Nmap's normal format. |
| **-oG [file]** | Write output in Nmap's so-called grepable format. |
| **-oX [file]** | Output in XML format. |
| **-oA [basename]** | Store scan results in normal, XML, and grepable formats at once. They are stored in `<basename>.nmap`, `<basename>.xml`, and `<basename>.gnmap`. |
| **-F** | Fast mode. Not actually faster, it just scans 100 well-known ports instead of the default scan that scans 1000 ports. |
| **-sA** | ACK scan. Sends an ACK packet to trick the target. |
| **-sT** | Connect() scan. This completes the 3-way handshake. Nmap actually calls the HTTP stack of the machine, which then initiates a complete request. |
| **-O** | Enables OS detection. |
| **-Pn** | Treat all hosts as online -- skip host discovery. |
| **-sU** | UDP scan. |
| **-PE** | ICMP echo. |
| **--data-length** | Append random data to sent packets. |
| **--spoof-mac <MAC>** | Spoof MAC address. |
| **--source-port** | Specify a source port. |
| **-S** | Spoof source address. |
| **-D** | Cloak a scan with decoys. |
| **-6** | Enable IPv6 scanning. |
| **--script [script]** | Specify an Nmap script to run. |

## Resources

* [What is Nmap and How to Use it – A Tutorial for the Greatest Scanning Tool of All Time](https://www.freecodecamp.org/news/what-is-nmap-and-how-to-use-it-a-tutorial-for-the-greatest-scanning-tool-of-all-time/)
* [Nmap Tutorial](https://hackertarget.com/nmap-tutorial/)
* [Nmap Commands - List of Top Nmap Command](https://intellipaat.com/blog/nmap-commands/)
* [NMAP basics Tutorial](https://linuxhint.com/nmap_basics_tutorial/)
* [Stealth Scans With Nmap](https://linuxhint.com/stealth_scans_nmap/)
* [Nmap Flags and What They Do](https://linuxhint.com/nmap_flags/)
* [NSE Scripts](https://nmap.org/nsedoc/scripts/)
* [How Nmap really works](https://www.youtube.com/watch?v=F2PXe_o7KqM)
* [Nmap Tutorial ](https://www.youtube.com/watch?v=5MTZdN9TEO4&list=PLBf0hzazHTGM8V_3OEKhvCM9Xah3qDdIx)
* [How Nmap really works](https://www.youtube.com/watch?v=ws0MeqfuA58&list=PLBf0hzazHTGM8V_3OEKhvCM9Xah3qDdIx&index=17)
* [Download nmap](https://nmap.org/download)
* [Options Summary](https://nmap.org/book/man-briefoptions.html)
* [Command-line Flags](https://nmap.org/book/port-scanning-options.html)
* [TCP Analysis](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvTCPAnalysis.html)
* [Host Discovery](https://nmap.org/book/man-host-discovery.html)
* [TCP SYN (Stealth) Scan (-sS)](https://nmap.org/book/synscan.html)
* [The Phases of an Nmap Scan](https://nmap.org/book/nmap-phases.html)
* [Timing Templates (-T)](https://nmap.org/book/performance-timing-templates.html)
* [Nmap Scripting Engine (NSE)](https://nmap.org/book/man-nse.html)
* [Firewall/IDS Evasion and Spoofing](https://nmap.org/book/man-bypass-firewalls-ids.html)
* [TCP Connect Scan (-sT)](https://nmap.org/book/scan-methods-connect-scan.html)
* [Service and Version Detection](https://nmap.org/book/man-version-detection.html)
* [Bypassing Firewall Rules](https://nmap.org/book/firewall-subversion.html)
* [Specifying Target Hosts and Networks](https://nmap.org/book/host-discovery-specify-targets.html)
* [TCP FIN, NULL, and Xmas Scans (-sF, -sN, -sX)](https://nmap.org/book/scan-methods-null-fin-xmas-scan.html)