---
layout: post
title: Exploring Certificate Revocation Process
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [ssl, tls, certificates, openssl]
comments: true
---

This article aims to explore the three ways used to check if a given certificate is revoked or not. 

Some of the reasons a certificate might have been revoked are:
* keyCompromise.
* affilitationChange.
* superseded.
* privilegeWithdrawn.

I am going to use the certificate from "www.baccredomatic.com" as an example.

## Certificate Revocation List (CRL)

This is a list, maintained by the Certificate Authority (CA), which holds information regarding which certificates have been revoked. They are typically, in DER format (binary). It holds the serial number of all revoked certificates.

General process:

* The client connects to the Server.

* The server replies with the certificate for the website.

* The client locates and downloads CRL info inside the provided certificate.

* The client takes the serial number of the provided certificate and searches this value in the CRL list.

  * If there is a match, then the certificate has been revoked.

  * If there is no match, then the certificate is good to use.

Example:

1. Acquiring certificate

```
openssl s_client -connect www.baccredomatic.com:443
```

Extracted the value of the public key and saved it into a file called "www.baccredomatic.com.cert"

```
-----BEGIN CERTIFICATE-----
MIILZDCCCkygAwIBAgIQBMQb2m6ecKki9dZ9XAYNozANBgkqhkiG9w0BAQsFADB1
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMTQwMgYDVQQDEytEaWdpQ2VydCBTSEEyIEV4dGVuZGVk
IFZhbGlkYXRpb24gU2VydmVyIENBMB4XDTI0MDMxODAwMDAwMFoXDTI1MDMxODIz
NTk1OVowga8xEzARBgsrBgEEAYI3PAIBAxMCUEExHTAbBgNVBA8MFFByaXZhdGUg
T3JnYW5pemF0aW9uMQ8wDQYDVQQFEwYzMDYwMTcxCzAJBgNVBAYTAlBBMRQwEgYD
VQQHEwtQYW5hbWEgQ2l0eTElMCMGA1UEChMcQkFDIEludGVybmF0aW9uYWwgQmFu
aywgSW5jLjEeMBwGA1UEAxMVd3d3LmJhY2NyZWRvbWF0aWMuY29tMFkwEwYHKoZI
zj0CAQYIKoZIzj0DAQcDQgAEu1/D8VipfQo61Y2pXez46oRQ9bS0XlZKMUqdhSgP
dsDUNgSz2wMXNKPuxhxDIB8EHMmA0tn7DJ9OFN3B/60b/aOCCH4wggh6MB8GA1Ud
IwQYMBaAFD3TUKXWoK3u80pgCmXTIdT4+NYPMB0GA1UdDgQWBBQ35P3/+EzWMYjV
CC9Barvabbg07jCCBSkGA1UdEQSCBSAwggUcghV3d3cuYmFjY3JlZG9tYXRpYy5j
b22CF2F5dWRhLmJhY2NyZWRvbWF0aWMuY29tghxiYWNjbG1wcm9kLmJhY2NyZWRv
bWF0aWMuY29tghFiYWNjcmVkb21hdGljLmNvbYIWYmx1ZS5iYWNjcmVkb21hdGlj
LmNvbYIgY2FyZ29zYXV0b21hdGljb3MuY3JlZG9tYXRpYy5jb22CGmNvbWVyY2lv
LmJhY2NyZWRvbWF0aWMuY29tgh1jb250LW1vYmlsZS5iYWNjcmVkb21hdGljLmNv
bYIZY29udGVudC5iYWNjcmVkb21hdGljLmNvbYIYY3IuY3NwLmJhY2NyZWRvbWF0
aWMuY29tgiBkZXZlbG9wZXItdGVzdC5iYWNjcmVkb21hdGljLmNvbYIcZGV2ZWxv
cGVycy5iYWNjcmVkb21hdGljLmNvbYIYZWNvbW1lcmNlLmNyZWRvbWF0aWMuY29t
ghZlZGl0LmJhY2NyZWRvbWF0aWMuY29tgh9lbmN1ZXN0YS1jaGF0LmJhY2NyZWRv
bWF0aWMuY29tghhmbG90YXMuYmFjY3JlZG9tYXRpYy5jb22CGGd0LmNzcC5iYWNj
cmVkb21hdGljLmNvbYIYaG4uY3NwLmJhY2NyZWRvbWF0aWMuY29tghhtb2JpbGUu
YmFjY3JlZG9tYXRpYy5jb22CGG5pLmNzcC5iYWNjcmVkb21hdGljLmNvbYIYcGEu
Y3NwLmJhY2NyZWRvbWF0aWMuY29tghhwYXliYWMuYmFjY3JlZG9tYXRpYy5jb22C
CXBheWJhYy5jcoIRcGF5YmFjLnBhLmJhYy5uZXSCFXBmbS5iYWNjcmVkb21hdGlj
LmNvbYIZcmVnLmNzcC5iYWNjcmVkb21hdGljLmNvbYIZc2FuZGJveC5iYWNjcmVk
b21hdGljLmNvbYIdc29saWNpdHVkZXMuYmFjY3JlZG9tYXRpYy5jb22CI3NvbGlj
aXR1ZGVzLnN1Y3Vyc2FsZWxlY3Ryb25pY2EuY29tgh5zdGF0aWMuc3VjdXJzYWxl
bGVjdHJvbmljYS5jb22CGXN0ZXByb3ZlZWRvcmVzLnBhLmJhYy5uZXSCGHN2LmNz
cC5iYWNjcmVkb21hdGljLmNvbYIOd3d3LjJtb3ZpbC5jb22CC3d3dy5iYWMubmV0
ghB3d3cuYmFjYmFtZXIubmV0ghV3d3cuYmFjZWxzYWx2YWRvci5jb22CFHd3dy5i
YWNndWF0ZW1hbGEuY29tghN3d3cuYmFjaG9uZHVyYXMuY29tghR3d3cuYmFjbmlj
YXJhZ3VhLmNvbYIRd3d3LmJhY29ubGluZS5jb22CEXd3dy5iYWNwYW5hbWEuY29t
ghd3d3cuYmFuY29yZWZvcm1hZG9yLmNvbYISd3d3LmNyZWRvbWF0aWMuY29tgg13
d3cuZS1iYWMubmV0ghR3d3cuZm9uZG9wcmVtaWVyLmNvbYINd3d3LnBheWJhYy5j
coIOd3d3LnJlZGJhYy5jb22CG3d3dy5zdWN1cnNhbGVsZWN0cm9uaWNhLmNvbYIV
d3d3LnN1Y3Vyc2FsbW92aWwuY29tghx3d3cxLnN1Y3Vyc2FsZWxlY3Ryb25pY2Eu
Y29tghZ3d3cyLmJhY2NyZWRvbWF0aWMuY29tghx3d3cyLnN1Y3Vyc2FsZWxlY3Ry
b25pY2EuY29tghx3d3c2LnN1Y3Vyc2FsZWxlY3Ryb25pY2EuY29tMEoGA1UdIARD
MEEwCwYJYIZIAYb9bAIBMDIGBWeBDAEBMCkwJwYIKwYBBQUHAgEWG2h0dHA6Ly93
d3cuZGlnaWNlcnQuY29tL0NQUzAOBgNVHQ8BAf8EBAMCA4gwHQYDVR0lBBYwFAYI
KwYBBQUHAwEGCCsGAQUFBwMCMHUGA1UdHwRuMGwwNKAyoDCGLmh0dHA6Ly9jcmwz
LmRpZ2ljZXJ0LmNvbS9zaGEyLWV2LXNlcnZlci1nMy5jcmwwNKAyoDCGLmh0dHA6
Ly9jcmw0LmRpZ2ljZXJ0LmNvbS9zaGEyLWV2LXNlcnZlci1nMy5jcmwwgYgGCCsG
AQUFBwEBBHwwejAkBggrBgEFBQcwAYYYaHR0cDovL29jc3AuZGlnaWNlcnQuY29t
MFIGCCsGAQUFBzAChkZodHRwOi8vY2FjZXJ0cy5kaWdpY2VydC5jb20vRGlnaUNl
cnRTSEEyRXh0ZW5kZWRWYWxpZGF0aW9uU2VydmVyQ0EuY3J0MAwGA1UdEwEB/wQC
MAAwggF+BgorBgEEAdZ5AgQCBIIBbgSCAWoBaAB2AE51oydcmhDDOFts1N8/Uusd
8OCOG41pwLH6ZLFimjnfAAABjlMWOi8AAAQDAEcwRQIhAObVJxsNSZQwULWym0bY
nB07WRp+v9AfoL0bNT8kdkSgAiBbF7JbKiqIrWVs/InVBiKI1+EaID1EWjXeVt9o
Cz/MIAB3AObSMWNAd4zBEEEG13G5zsHSQPaWhIb7uocyHf0eN45QAAABjlMWOkkA
AAQDAEgwRgIhAL89Oyh8JTJm4CfDrQzLbvNsD19rXhiME1Ze4krYi9c5AiEAjwhr
2pcovUepA80HYNRZH3VbJzykY/04bT95VXOf/roAdQDPEVbu1S58r/OHW9lpLpvp
GnFnSrAX7KwB0lt3zsw7CAAAAY5TFjprAAAEAwBGMEQCIA6QFLDSxrwtAFGLG1mf
aIixkY4o2km3i/9HlzOi5vLbAiAwt8dL2Ju1/JArDz6pYvNAM4Qz1cOddUCtk/Ar
jDnYtDANBgkqhkiG9w0BAQsFAAOCAQEAVpzmHfwj4XK1myCCI4KfI0LYqBD57S3i
d+/PrTf7bzFMsgaY0zmbDUH03NwsShNFEyj5vLbpjG1uKe503WciycR2li1s1fYC
AaTQd93a2GAzYEaIA5PGb5LYQ3w5af1jLG0gtrc4zyEBCa8KDjowMcLDvNR2ixS4
dH4RTa7Y27CG08HXkMsWuBwmPWgeVG67YE4AlH1vVRiVNzPFfajoYfpZytTNAMgq
6dD+/gbF+dA3QzY7Hiywk77EgFk42Sb6XvP7EP7T5ZflR6FjKVSAkd0iodIKLbXt
ITPxUOPXTSXPUVaM8mdH97B9cpx2s1t/vpCyfEL3EVNo/xWMFJar3w==
-----END CERTIFICATE-----
```

2. Extract CRL Distribution Points information

This comes inside x509v3 extensions.

```
# To extract directly you can use this command:

$ openssl x509 -in www.baccredomatic.com.cert -noout -ext crlDistributionPoints
X509v3 CRL Distribution Points: 
    Full Name:
      URI:http://crl3.digicert.com/sha2-ev-server-g3.crl
    Full Name:
      URI:http://crl4.digicert.com/sha2-ev-server-g3.crl

# Or you can dump whole certificate in text format and locate it:

$ openssl x509 -in www.baccredomatic.com.cert -noout -text
```

3. Download the CRL list.

This list comes in DER (binary) format.

```
$ wget http://crl3.digicert.com/sha2-ev-server-g3.crl
--2024-04-05 16:35:18--  http://crl3.digicert.com/sha2-ev-server-g3.crl
Resolving crl3.digicert.com (crl3.digicert.com)... 192.229.211.108
Connecting to crl3.digicert.com (crl3.digicert.com)|192.229.211.108|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 202399 (198K) [application/pkix-crl]
Saving to: ‘sha2-ev-server-g3.crl’

sha2-ev-server-g3.crl     100%[====================================>] 197,66K   955KB/s    in 0,2s    

2024-04-05 16:35:18 (955 KB/s) - ‘sha2-ev-server-g3.crl’ saved [202399/202399]

```
4. Transform CRL DER format into text format

Extracted the value of CRL and saved it into a file called "crl-www.baccredomatic.com" in text format:

```
$ openssl crl -in sha2-ev-server-g3.crl -inform DER -noout -text > crl-www.baccredomatic.com

```

Let's do a quick view of what is on this list

```
# head
$ head -30 crl-www.baccredomatic.com 
Certificate Revocation List (CRL):
        Version 2 (0x1)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert SHA2 Extended Validation Server CA
        Last Update: Apr  5 03:07:25 2024 GMT
        Next Update: Apr 12 03:07:25 2024 GMT
        CRL extensions:
            X509v3 Authority Key Identifier: 
                3D:D3:50:A5:D6:A0:AD:EE:F3:4A:60:0A:65:D3:21:D4:F8:F8:D6:0F
            X509v3 CRL Number: 
                1289
            X509v3 Issuing Distribution Point: critical
                Full Name:
                  URI:http://crl3.digicert.com/sha2-ev-server-g3.crl
                  URI:http://crl4.digicert.com/sha2-ev-server-g3.crl
Revoked Certificates:
    Serial Number: 03A44998D804DE49042B935B0F96CF0D
        Revocation Date: Feb 27 07:19:12 2023 GMT
        CRL entry extensions:
            X509v3 CRL Reason Code: 
                Superseded


# tail
$ tail -30 crl-www.baccredomatic.com 
    Serial Number: 0935BFA9F9FC9F4D9328CA9E0F5EC443
        Revocation Date: Apr  4 08:55:30 2024 GMT
    Serial Number: 048AA6775D93E41455C4982669EB893F
        Revocation Date: Apr  4 08:56:54 2024 GMT
    Serial Number: 06F7F34A149E0B02EAE81C02ED063653
        Revocation Date: Apr  4 08:56:55 2024 GMT
    Serial Number: 0264624E2A3AE90C098FDC81A19D1DC4
        Revocation Date: Apr  4 11:17:25 2024 GMT
    Serial Number: 09E3DF5F63B397BA2AF65358072D067D
        Revocation Date: Apr  5 00:20:57 2024 GMT
        CRL entry extensions:
            X509v3 CRL Reason Code: 
                Superseded
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        d1:a8:8e:be:16:e8:b2:48:8c:ab:ac:e0:8e:0a:81:08:71:0b:
        7c:0b:c9:3f:27:a8:33:db:e9:5e:cb:56:d9:2a:d8:39:e5:cf:
        90:fc:1c:a3:ac:e7:fd:64:d5:99:a3:a8:ed:8d:99:e7:ab:9c:
        5b:6d:99:a7:2f:0c:29:37:e5:16:01:96:09:0b:90:99:d4:fe:
        26:87:f3:d7:22:ba:cf:c7:6a:44:91:d6:43:ea:1a:96:bb:ff:
        e9:74:42:9c:78:97:17:c4:84:1f:50:eb:13:c9:e3:22:ec:33:
        e8:0b:c4:9b:c1:7e:98:97:70:df:2b:27:98:f3:8d:bf:bc:ed:
        38:5f:56:65:a2:5a:08:bd:0d:d0:61:53:07:5f:06:ca:59:d6:
        6d:7e:2b:06:a0:b2:30:8e:6f:c7:53:b2:a9:4f:af:42:e1:14:
        e1:b8:70:8d:cb:80:11:eb:38:f3:a9:cd:f3:a9:0a:d9:8c:21:
        2e:fa:5a:ad:65:f6:b7:17:22:63:16:01:1d:c4:50:68:cc:7c:
        bb:e7:41:26:aa:f2:19:98:59:41:58:81:87:e5:81:26:e0:a2:
        5b:0c:df:43:d5:78:ad:d3:5c:47:20:f1:82:e8:54:27:12:05:
        7e:d7:19:91:b1:34:30:ae:18:d3:8a:11:16:4f:2b:66:5e:c4:
        3a:53:fb:22

```

It contains the serial numbers of revoked certificates, revocation date, and revocation reason (optional).

Notice that the CRL list has a digital signature!


5- Get the serial number of the certificate

We will use this value in the next step and search for it in the CRL list.

```
$ openssl x509 -in www.baccredomatic.com.cert -noout -serial
serial=04C41BDA6E9E70A922F5D67D5C060DA3
```

6- Is the serial number in the CRL List?

We had no result, so the certificate is not revoked!

```
$ cat crl-www.baccredomatic.com | grep 04C41BDA6E9E70A922F5D67D5C060DA3 -C6
adrian@vivobook:~/Desktop/PracticalTLSCourse/LAB4.2-CertificateRevocation/www.baccredomatic.com$ 
```

## Online Certificate Status Protocol (OCSP)

The CA maintains an OCSP Responder that provides real-time certificate status.

The client takes the serial number of the certificate and makes an OCSP request.

OCSP information is embedded in each certificate.

General Process:

* The client connects to the website and acquires the server certificate.

* The client locates OCSP information from the certificate.

* The client requests the status of the certificate from the OCSP Responder using the serial number of the certificate.

* OCSP Responder replies with either: good, revoked or, unknown.

Example:

1. Get the certificate

We already have the certificate for www.baccredomatic.com.

2. Get OCSP information from the certificate

This is located inside the "Authority Information Access" field.

```
$ openssl x509 -in www.baccredomatic.com.cert -noout -text

            Authority Information Access: 
                OCSP - URI:http://ocsp.digicert.com
                CA Issuers - URI:http://cacerts.digicert.com/DigiCertSHA2ExtendedValidationServerCA.crt

```

Before making an OCSP request, we need to understand that the reply from the OCSP Responder is digitally signed by that CA, so we need to have CA's public key to verify the digital signature.

   2.1 Getting CA's certificate

This is located in "CA Issuers" inside the "Authority Information Access".

```
$ wget http://cacerts.digicert.com/DigiCertSHA2ExtendedValidationServerCA.crt
--2024-04-05 19:23:17--  http://cacerts.digicert.com/DigiCertSHA2ExtendedValidationServerCA.crt
Resolving cacerts.digicert.com (cacerts.digicert.com)... 192.229.211.108
Connecting to cacerts.digicert.com (cacerts.digicert.com)|192.229.211.108|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1210 (1,2K) [application/pkix-cert]
Saving to: ‘DigiCertSHA2ExtendedValidationServerCA.crt’

DigiCertSHA2ExtendedValida 100%[======================================>]   1,18K  --.-KB/s    in 0s      

2024-04-05 19:23:17 (166 MB/s) - ‘DigiCertSHA2ExtendedValidationServerCA.crt’ saved [1210/1210]

```

3. Make OCSP Request

We need to provide the following information.

* The OCSP Responder location.
* The certificate that we want to request status for.
* The CA's certificate because OCSP response will be digitally signed by this CA.

```
$ openssl ocsp -url http://ocsp.digicert.com -issuer 'DigiCertSHA2ExtendedValidationServerCA.crt' -cert www.baccredomatic.com.cert 
WARNING: no nonce in response
Response verify OK
www.baccredomatic.com.cert: good
	This Update: Apr  5 11:45:02 2024 GMT
	Next Update: Apr 12 10:45:02 2024 GMT

```

The certificate status is good!

## OCSP Stapling

The server periodically requests its own status to OCSP Responder. The response is time-stamped and signed by the CA. The server has to support this method, if you receive something like "OCSP response: no response", it means the server does not support OCSP stapling.

This requires TLS 1.2+ and support from the client and the server as well.

General process:

* The client requests the certificate and its status to the server itself.

* The server provides both the certificate and the status.

Example:

```
$ openssl s_client -connect www.baccredomatic.com:443 -status
CONNECTED(00000003)
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert High Assurance EV Root CA
verify return:1
depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert SHA2 Extended Validation Server CA
verify return:1
depth=0 jurisdictionC = PA, businessCategory = Private Organization, serialNumber = 306017, C = PA, L = Panama City, O = "BAC International Bank, Inc.", CN = www.baccredomatic.com
verify return:1
OCSP response: 
======================================
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: 3DD350A5D6A0ADEEF34A600A65D321D4F8F8D60F
    Produced At: Apr  3 12:00:30 2024 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: 49F4BD8A18BF760698C5DE402D683B716AE4E686
      Issuer Key Hash: 3DD350A5D6A0ADEEF34A600A65D321D4F8F8D60F
      Serial Number: 04C41BDA6E9E70A922F5D67D5C060DA3
    Cert Status: good
    This Update: Apr  3 11:45:02 2024 GMT
    Next Update: Apr 10 10:45:02 2024 GMT

    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        6f:88:4a:9f:99:34:b0:8d:c3:95:7e:51:27:70:f0:8d:bc:43:
        4e:de:f9:41:3e:99:67:e6:6c:df:62:f1:18:00:c3:45:f1:d9:
        2b:0e:97:6c:f0:ef:3b:a2:fa:41:a6:4d:b0:bc:c2:eb:7e:2b:
        7c:42:13:6a:24:a3:b8:29:d7:23:e7:b5:16:fd:43:be:0c:c0:
        d4:a4:5e:28:21:d4:70:db:3e:ee:1a:9d:24:e6:4a:35:26:fa:
        d3:d1:1c:64:40:39:b0:a3:75:5b:14:d4:3c:59:6f:ec:86:e6:
        61:1d:49:ca:7d:6f:c5:86:db:b7:99:10:94:3c:21:57:60:0c:
        19:a8:f8:fa:ee:b1:be:e3:17:82:43:5c:c9:98:4b:05:2a:70:
        a9:1f:53:b2:94:27:60:ea:9c:62:76:4e:66:78:c0:77:7c:97:
        45:f9:f3:8c:da:11:da:2e:45:13:6d:e1:c9:40:2c:27:1b:68:
        c4:63:b4:2f:3c:f6:4c:16:28:a6:c4:7b:ba:93:5c:9e:fd:83:
        64:b8:f0:20:9f:5f:bc:72:c6:2b:2f:8d:f5:bb:a1:b3:38:ec:
        01:12:03:82:d6:b5:f1:8a:b8:9a:ec:62:f0:02:ef:2c:5e:c6:
        22:ef:b3:3f:15:35:cf:e7:f2:b5:db:91:3f:df:31:76:5d:c9:
        a0:75:f2:00
======================================
```

## In Summary

* Several methods have been created because each of these methods has its limitations and criticisms.

* CRL is a list of revoked certificates maintained by the CA.

* OCSP is a method where the CA maintains an OCSP Responder that is responsible for replying to OCSP requests with either good, revoked or, unknown. The client makes an OCSP request using the certificate's serial number.

* OCSP Stapling is a method where the server itself periodically queries its own certificate status with its OCSP Responder. The server and client need to support this method. In addition to this, TLS 1.2+ is required.

* All responses are digitally signed by the CA.
