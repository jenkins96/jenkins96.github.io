

## What Is Client Certificate Authentication

It is the concept of the end user sending an SSL certificate for authentication purposes.

## Renegotiation Concept

When a new TLS handshake negotiation happens inside of an existing secure session. It is the process that allows the client of the server to initiate a new handshake within an existing TLS session.

## Normal Flow Of Client Certificate Authentication In IIS With TLS 1.2

Browsed to my site "https://localhost", got prompted for a certificate, selected one and got access to my site.

I took a Wireshark and looks as follows (leaving some parts out of the picture):

```
Client Hello

Server Hello, Certificate, Server Key Exchange, Server Hello Done

Application Data
```

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img001.png)

This works well and I got access to my site without any issue.

However, you might have expected to see message "Certificate Request" in the "Server Hello" as, it is the Server requesting a certificate to the client for authentication purposes.

Well, rest assured that this is indeed happening, but the reason that we are not explicitly seeing this message is because this happens in a renegotiation. 

Inside this already encrypted TLS tunnel, a TLS renegotiation takes place, and in this renegotiation is where the "Certificate Request" is being sent.

**Why is this happening?**

Every HTTP(s) request is first handled by a Windows Networking subsystem called HTTP.SYS. If everything works well, then HTTP.SYS forwards the request to IIS.

So, HTTP.SYS receives the message and establishes the TLS handshake, however, HTTP.SYS is not configured to request a certificate for authentication. Therefore, we do not see the message in Wireshark. This is the first negotiation that we see in the trace. Notice that IIS is not in the picture yet. After this succeeds, the TLS tunnel gets created and now the client sends the HTTP request that hits IIS and IIS is configured to request a certificate for authentication and now, a renegotiation of TLS parameters happens during this same encrypted tunnel. This triggers HTTP.SYS to send again "TLS Hello Request" to the client. The client responds and now the Server sends "TLS Server Hello + Certificate Request". Again, this is all happening INSIDE the ALREADY TLS encrypted session.

In short:

```
-Client -> TCP handshake with server
-TLS Client Hello
-TLS Server Hello
[Finish TLS handshake]
[Everything below is encrypted]
-HTTP Request
-IIS checks for client certificate -> triggers HTTP.SYS to send TLS Hello Request to client
-TLS Client Hello
-TLS Server Hello + Certificate Request
-TLS Client Certificate + other stuff
[finish TLS handshake]
[further processing]
```

## Behavior With TLS 1.3

When we do the same experiment that we did above but we change the TLS to 1.3, then we have a different behavior.

The Server is sending a TCP RST message.

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img002.png)

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img003.png)

This happens because TLS 1.3 does not allow renegotiation.

**4.6.2. Post-Handshake Authentication**

> When the client has sent the "post_handshake_auth"    extension (see Section 4.2.6), a server MAY request client authentication at any time after the handshake has completed by sending a CertificateRequest message. The client MUST respond with the appropriate Authentication messages (see Section 4.4). If the client chooses to authenticate, it MUST send Certificate, CertificateVerify, and Finished. If it declines, it MUST send a Certificate message containing no certificates followed by Finished. All of the client's messages for a given response MUST appear consecutively on the wire with no intervening  messages of other types.
A client that receives a CertificateRequest message without having sent the "post_handshake_auth" extension MUST send an "unexpected_message" fatal alert.
Note: Because client authentication could involve prompting the user, servers MUST be prepared for some delay, including receiving an arbitrary number of other messages between sending the CertificateRequest and receiving a response. In addition, clients which receive multiple CertificateRequests in close succession MAY respond to them in a different order than they were received (the certificate_request_context value allows the server to disambiguate the responses).

**4.2.6.  Post-Handshake Client Authentication**

> The "post_handshake_auth" extension is used to indicate that a clientis willing to perform post-handshake authentication (Section 4.6.2). Servers MUST NOT send a post-handshake CertificateRequest to clients which do not offer this extension.  Servers MUST NOT send this
extension.

## With Firefox
![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img004.png)

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img005.png)

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img006.png)

## Workarounds

### Disable TLS 1.3

Go to the Registry and disable for specific certificate. May vary depending to where certificate is attached (IP or hostname):
"Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters\SslBindingInfo\0.0.0.0:443"

```
net stop http
```

"DefaultFlags" DWORD32 = 40

![](/assets/img/articles/Client-Certificate-Authenticaiton-TLS1.3/img007.png)

```
net start http
net start w3svc
```


We can now see  that TLS 1.3 has now been disabled for this interface:

```
C:\Users\sysadmin>netsh http show sslcert

SSL Certificate bindings:
-------------------------
    IP:port                      : 0.0.0.0:443
    Certificate Hash             : 2ace368a90d2117be7a0065dfe259b2e8c94f538
    Application ID               : {4dc3e181-e14b-4a21-b022-59fc669b0914}
    Certificate Store Name       : My
    Verify Client Certificate Revocation : Enabled
    Verify Revocation Using Cached Client Certificate Only : Disabled
    Usage Check                  : Enabled
    Revocation Freshness Time    : 0
    URL Retrieval Timeout        : 0
    Ctl Identifier               : (null)
    Ctl Store Name               : (null)
    DS Mapper Usage              : Disabled
    Negotiate Client Certificate : Disabled
    Reject Connections           : Disabled
    Disable HTTP2                : Not Set
    Disable QUIC                 : Not Set
    Disable TLS1.2               : Not Set
    Disable TLS1.3               : Set

```

Site can now be accessed because it is using TLS 1.2


### Set HTTP.SYS To Require Cert
Since TLS 1.3 does not allow renegotiation, we can set HTTP.SYS to ask for a certificate to end user. In this case, we will be able to see "Certificate Request" in the "Server Hello" as this will be happening in this first an only TLS negotiattion.

However, take into consideration, based in my understaind, that doing this is not compliant with TLS 1.3 RFC. A "Certificate Request" message should not be sent in the main handshake, UNLESS the client sent the "post_handshake_auth" extension in the "Client Hello".

4.3.2.  Certificate Request
> Servers which are authenticating with a PSK MUST NOT send the CertificateRequest message in the main handshake, though they MAY send it in post-handshake authentication (see Section 4.6.2) provided that the client has sent the "post_handshake_auth" extension (see Section 4.2.6).

#### Certificate Bound To IP Address



#### Certificate Bound To Hostname

#### Certificate Using Central Certificate Store(CCS)

## Resources
* [Windows Server 2022 IIS web site TLS 1.3 does not work with client certificate authentication](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-server-2022-iis-web-site-tls-1-3-does-not-work-with/ba-p/4129738)

* [RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446)

* [Why is open OpenSSL client certificate not working for me with TLSv1.3?
](https://stackoverflow.com/questions/70925286/why-is-open-openssl-client-certificate-not-working-for-me-with-tlsv1-3)