---
layout: post
title: Edge Cipher Suites
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [ssl, tls cipher suites, edge, browsers]
comments: true
---

This is a quick dirty example of what I think proves Edge uses different cipher suites than OS offers.
 
Quick recap.

To establish a TLS channel, the very first thing that needs to happen is the SSL/TLS handshake. During this process, both client and server agree on supported cipher suites to use, the server authenticates itself to the client among other things. In Windows OS we have the Secure Channel, SCHANNEL, that implements SSL/TLS standard and is responsible for negotiating the handshake between the client and the server.

So, SCHANNEL is the Windows provider that facilitates the use of SSL/TLS communications.
 
A cipher suite is just a combination of algorithms that are going to be used for authentication, symmetric encryption, hashing, and key exchange. Think of it like a specific language, the server and the client must establish a common language for conversation to continue.

The purpose of SSL/TLS is to provide confidentiality, integrity and, authentication. This is accomplished by a set of algorithms. The client and the server must establish which algorithms are they going to be using to establish a TLS conversation.
 
Take this cipher suite as an example:
 
```
TLS_DHE_RSA_WITH_AES_256_CBC_SHA
```

* Key exchange: DHE
* Authentication: RSA
* Encryption: AES 256 CBC
* Hashing: SHA
 
Each Operating System has a pre-defined list of available cipher suites. The ciphers that are enabled and available are the ones that SCHANNEL can use. 
 
Let's take as an example my Windows Server 2022. We can use Powershell "Get-TlsCipherSuite" to get an ordered collection of cipher suites for a computer that Transport Layer Security (TLS) can use.
 
``` 
Name: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
Name: TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
Name: TLS_AES_256_GCM_SHA384
Name: TLS_AES_128_GCM_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
Name :TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
Name: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Name: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
Name: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Name: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
```

So,  this OS currently has 13 enabled and available ciphers that SCHANNEL can use.
 
Although you might expect Browsers to use these cipher suites, this is not the case.
 
To understand what ciphers each Browser supports we can either capture a Wireshark/network trace and look at the Client Hello, or we can use this website to see what ciphers is our Browser using:
 
* [SSL/TLS Capabilities of Your Browser](https://clienttest.ssllabs.com:8443/ssltest/viewMyClient.html)
 
My Edge version is 123.0.2420.81. Here is the result:
 
Browser supports 15 cipher suites. So, we know there are differences between what Edge Browser supports and what our OS/SCHANNEL can use.

| Cipher Suites (in order of preference)                               	|
|--------------------------------------------------------------------------|
| TLS_AES_128_GCM_SHA256 (0x1301)   Forward Secrecy                    	|
| TLS_AES_256_GCM_SHA384 (0x1302)   Forward Secrecy                    	|
| TLS_CHACHA20_POLY1305_SHA256 (0x1303)   Forward Secrecy              	|
| TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b)   Forward Secrecy   	|
| TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   Forward Secrecy     	|
| TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c)   Forward Secrecy   	|
| TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   Forward Secrecy     	|
| TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca9)   Forward Secrecy |
| TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)   Forward Secrecy   |
| TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)  WEAK                    	|
| TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)  WEAK                    	|
| TLS_RSA_WITH_AES_128_GCM_SHA256 (0x9c)  WEAK                         	|
| TLS_RSA_WITH_AES_256_GCM_SHA384 (0x9d)  WEAK                         	|
| TLS_RSA_WITH_AES_128_CBC_SHA (0x2f)  WEAK                            	|
| TLS_RSA_WITH_AES_256_CBC_SHA (0x35)  WEAK                            	|

Here is a table summarizing what we have at the moment:

| COMPUTER AND EDGE                       | COMPUTER/SCHANNEL ONLY                  | EDGE ONLY                                     |
|-----------------------------------------|-----------------------------------------|-----------------------------------------------|
| TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 | TLS_DHE_RSA_WITH_AES_256_GCM_SHA384     | TLS_CHACHA20_POLY1305_SHA256                  |
| TLS_AES_256_GCM_SHA384                  | TLS_DHE_RSA_WITH_AES_128_GCM_SHA256     | TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 |
| TLS_AES_128_GCM_SHA256                  | TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 | TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256   |
| TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 | TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 | TLS_RSA_WITH_AES_128_GCM_SHA256               |
| TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384   | TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384   | TLS_RSA_WITH_AES_256_GCM_SHA384               |
| TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256   | TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256   | TLS_RSA_WITH_AES_128_CBC_SHA                  |
|                                         | TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA    | TLS_RSA_WITH_AES_256_CBC_SHA                  |
|                                         |                                         | TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA            |
|                                         |                                         | TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA            |

I used IIS Crypto as a tool to quickly disable cipher suite: "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256".
 
This cipher suite is a common cipher suite between Edge and Computer.
 
I would expect that, if removed, both Edge and the computer/SCHANNEL cannot use this cipher suite.
 
* [IIS Crypto Tool](https://www.nartac.com/Products/IISCrypto/Download)
 
 
Rebooted machine and used PowerShell command "Get-TlsCipherSuite":

```
Name: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
Name : TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
Name: TLS_AES_256_GCM_SHA384
Name: TLS_AES_128_GCM_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
Name: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
Name: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Name: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
Name: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Name: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
Name: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
 
```
 
The cipher suite "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256" is no longer available for the OS. As expected.
 
Ran again the tool to understand what cipher suite does Edge offer and to my surprise, the cipher suite "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256" IS STILL being offered. Now, although it is still being offered, I am not sure how this would play out if TLS channel depended on this cipher suite exclusively.

![](/assets/img/articles/Edge-Ciphers/img1.png)

In IIS Crypto I selected "Best Practices". This enables a bunch of cipher suites and TLS.

![](/assets/img/articles/Edge-Ciphers/img2.png)

Rebooted the machine.
 
Tested with this tool to understand what the SERVER supports:
 
* [SSL Server Test ](https://www.ssllabs.com/ssltest/analyze.html)

![](/assets/img/articles/Edge-Ciphers/img3.png)
 
Ran the "Get-TlsCipherSuite" and we got 20 ciphers. Seems to be the same 20 cipher suites showed in above image (do not want to take time and compare them).
 
```
Name:  TLS_AES_256_GCM_SHA384
Name:  TLS_AES_128_GCM_SHA256
Name:  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
Name:  TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
Name:  TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Name:  TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
Name:  TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
Name:  TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
Name:  TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
Name:  TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
Name:  TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Name:  TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
Name:  TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
Name:  TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
Name:  TLS_RSA_WITH_AES_256_GCM_SHA384
Name:  TLS_RSA_WITH_AES_128_GCM_SHA256
Name:  TLS_RSA_WITH_AES_256_CBC_SHA256
Name:  TLS_RSA_WITH_AES_128_CBC_SHA256
Name:  TLS_RSA_WITH_AES_256_CBC_SHA
Name:  TLS_RSA_WITH_AES_128_CBC_SHA
 
```
 
However, Edge is still showing 15 cipher suites:

![](/assets/img/articles/Edge-Ciphers/img4.png)

Chrome shows 15 cipher suites:

![](/assets/img/articles/Edge-Ciphers/img5.png)

Firefox shows 17 cipher suites:

![](/assets/img/articles/Edge-Ciphers/img6.png)

Internet Explorer shows 20 cipher suites:

![](/assets/img/articles/Edge-Ciphers/img7.png)


As a side note, on my personal laptop, that has Ubuntu 22.04 with Edge 123.0.2420.97, it says Edge supports 15 cipher suites as well.

![](/assets/img/articles/Edge-Ciphers/img8.png)

It seems to be the Browsers use different cipher suites than SCHANNEL, except for Internet Explorer.

IE seems to be aligned with what the OS offers as cipher suites.

This article might have the answer, it says Edge no longer uses SCHANNEL, but instead uses BoringSSL implementation for HTTPS connections. So, that explains why disabling/enabling ciphers at OS/SCHANNEL level has no impact to Edge. Edge uses its implementation called BoringSSL and seems there is no way to add ciphers to Edge/Chromium. We can use a policy to disable ciphers for Edge, but we cannot add.
 
* [“Can I… in the new Edge?” (Un-FAQ)](https://textslashplain.com/2020/02/26/can-i-in-the-new-edge/#:~:text=The%20new%20Edge%2C%20like%20all%20Chromium-based%20browsers%2C%20uses,settings%20have%20any%20effect%20on%20the%20new%20Edge)

![](/assets/img/articles/Edge-Ciphers/img9.png)

The article mentions that IE uses SCHANNEL, which explains why IE cipher suites align with OS cipher suites.
  
Anyways, as far as Firefox I have no documentation, but it should be a similar story, where instead of using SCHANNEL, they use their  implementation and that is why cipher suites differ from what OS has.
 
## In summary

* Windows uses SCHANNEL provider to establish SSL/TLS communication.

* Browsers may use different cipher suites than OS offers because they might use their own implementation instead of SCHANNEL.

* Always be careful when disabling ciphers, you might leave some clients or other services without a way to connect to  your server.

* Seems to be that, offered cipher suites is dependent on Browser's version. However, OS version might limit this as well in the sense that, a Browser version might not be available for a specific OS version.

* You can review the cipher suites your Browser supports by capturing the Client Hello or by using the tool mentioned in this article.

* Chromium-based browsers use BoringSSL implementation for HTTPS instead of SCHANNEL.

* There is no way to add cipher suites for Edge/Chrome. You can disable ciphers with a browser's policy, but you cannot add ciphers to it.
 
 
## Resources

* [Get-TlsCipherSuite](https://learn.microsoft.com/en-us/powershell/module/tls/get-tlsciphersuite?view=windowsserver2022-ps)

* [IIS Crypto Tool](https://www.nartac.com/Products/IISCrypto/Download)

* [“Can I… in the new Edge?” (Un-FAQ)](https://textslashplain.com/2020/02/26/can-i-in-the-new-edge/#:~:text=The%20new%20Edge%2C%20like%20all%20Chromium-based%20browsers%2C%20uses,settings%20have%20any%20effect%20on%20the%20new%20Edge.)

* [TLS/SSL overview (Schannel SSP)](https://learn.microsoft.com/en-us/windows-server/security/tls/tls-ssl-schannel-ssp-overview)

* [Demystifying Schannel](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/demystifying-schannel/ba-p/259233)

* [SSL/TLS Capabilities of Your Browser](https://clienttest.ssllabs.com:8443/ssltest/viewMyClient.html)

* [SSL Server Test ](https://www.ssllabs.com/ssltest/analyze.html)

 

 
 


