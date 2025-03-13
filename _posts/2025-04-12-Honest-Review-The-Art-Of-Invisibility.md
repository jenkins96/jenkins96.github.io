---
layout: post
title: An Honest Review - The Art Of  Invisibility(Kevin Mitnick & Robert Vamosi)
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [book review, hacking, security]
comments: true
---

This book is a great read for the average person. If you're in IT and at an intermediate level in your career, a summary might be enough. However, if you're just starting out in IT or work in a different field, reading the full book could be worthwhile.

I’m not trying to discourage anyone from reading this book—by all means, go for it if you're interested. My point is that if you work in IT, you’re likely already familiar with many of the concepts and examples the author covers. While you might find a few valuable insights, most of the ideas will probably be things you already know.

While I have a deep respect for Mitnick’s knowledge and contributions to the field, my review will focus on both the strengths and weaknesses of this book from my perspective, naturally. My goal is to provide an honest perspective—highlighting what works well and where I believe the book falls short. Whether you're considering reading it or just looking for different viewpoints, I hope this review helps you decide if The Art of Invisibility is the right book for you. Of course, readers are more than welcome to share their own perspectives and challenge mine.

## Premises Of The Book

The whole premises of the book are:

* It is practically inevitable to stop companies from collecting our data. But this is not the problem, the issue lies in what these companies do with our data and with whom they share it.
  
* Privacy is like an onion, every layer or security measure you can commit to, adds more security while decreasing convenience. It is up to the reader up to which layer you are willing to commit to maintain your anonymity. As you will see, to become anonymous (again there are levels of anonymity) you really need to follow a strict set of rules, and one tiny error can give you away.

## Where The Book Falls Short

The book is full of stories that makes the reader aware of the problem that companies collect and share our data with third parties. Mitnick also shares some stories where the law enforcement managed to captured the "bad guy" because of an informatic error that led the police to him.

Personally, I am not really interested in these kind of stories, and some outcomes are kind of obvious. These stories I just feel that makes the reading experience a bit boring.

Mitnick  describes several security measures that we may take to better defend ourselves; security measures like deleting browsing history, installing JS plug-in blocker, using  Tor browser, etc. I never understood why he wastes time explaining the process of "minor" security measures like installing a given plug-in or deleting the browser's history, but no clear explanation on how to use Tor Browser which is far more interesting and more complex to  set-up to the average Joe, My issue here is "where  do you draw the line?", how are you deciding which security measures to further explain and which not?

The author suggests to use "HTTPS Everywhere" plug-in extension. What this does is to make sure that every site you visit is over HTTPS rather than HTTP. Nowdays, I believe every modern browser offer native support for HTTPS only mode that you can activate. [HTTPS Everywhere](https://www.eff.org/https-everywhere). Also, from my understanding, HTTPS Everywhere only accepts cipher suites considered as secured. This sounds great, but what if you or your company has a legitimate site that is only available over HTTP and there is no intention of moving this site over HTTPS because there is no reason to? Or even, you might have a site over HTTPS but the server uses a cipher that HTTPS Everywhere does not consider it as secure, then you won'e be able to access the site.

In general, I am not a fan of prescribing extensions like "HTTPS Everywhere" of extensions that block JS to regular people. They might have installed these extensions and now they may be wondering why they can't access a site. Sure, there is a balance between security and usability, but I feel we need to be more careful when addressing the general population.

Another recommendations is to delete browser's cookies. It is true that he says to determine the faith of a cookie on a case-by-case basis, but there is no clear method on how to determine which cookies to delete. 

My general issue here is that you are addressing the general population, I do not think that half solutions, like recommending to delete cookies, but not offering a method or a link where the reader can go and learn to determine if it is safe to delete a cookie.

## Notable Insights & Takeaways

* The author makes us question why do we accept, by default, that everything that we do online can be seen by other. You may no be aware if this, but are still implicitly accepting this.
* The big problem lies not in the fact that information is being collected, but what is done with the data once it is collected and with whom is being shared.
* You should use a strong password, but quick reminder that EVERY password is crackable. Our goal is to make the password so difficult so that the attacker moves on to an easier target.
* Reminder that there is the site [Have I Been Pawned](https://www.haveibeenpwned.com) to check if your email has been compromise in a data breach.
* If I understand it correctly, the author suggest a strong password should be at least 20 characters. Of course this would be amazing, but I think this is a bit excessive. I would say anything between 12-16 characters long is good enough provided is made up of a combination of alphanumeric characters + special characters and that the password is not a word, just random characters. Example: "nL2!U7!kr&t44Np".
* Use a Password Manager. A Password Manager saves all your passwords for all your sites. You only need to know the password for your Password Manager and then you can access all of your passwords.
* Never use the same password for two different accounts.
* Enable 2-FA /MFA. Hopefully, not through SMS, you are better off with an Authenticator Application.
* When dealing with chat/video applications look for a service with end-to-end encryption. This means that the data gets encrypted when you send it and it is decrypted by the receiver. So, as it travels through the network, it is encrypted.
	* Look for services that does not store the key, or that the key stays on your device. If the service provider does the encryption, it has the key that can decrypt the message. Remember the provider will give Law Enforcement the key if they ask them.
	* Look for Perfect Forward Secrecy(PFS). This creates a key, encrypts data, and then it discards the key. So, there is no way to recover the key and this means your data cannot be decrypted.
	* Well, remember everything can be decrypted. But it may take years for the adversary to decrypt it.
* Use a firewall and a Anti-Virus.
* Your email provider probably has a copy of every sent and received email.
* Although many email services use encryption for email in-transit, encryption may not be use for Mail Transfer Agent(MTA). Encrypt your messages with PGP, OpenPGP, GPG, etc.
	* Even using encryption the emails metadata is unencrypted. This contains information like the email's subject, sender and recipient addresses, data, message size, etc.
	* You can use an anonymous remailer, this is a service that sends emails anonymously by hiding the sender's identity. It masks your IP address, changes the email address of the sender before sending the message to its intended recipient. Do your research as type I and II  do not allow you to respond to emails. Type III or Mixminion does allow send/respond among other features.
* "You are only as secure as the weakest link".
* In general, open-source and non-profit organizations provide the most secure software.
* What information can be obtained by sharing location to the Browser [Browser geolocation test](https://benwerd.com/lab/geo.php)
* Search Engines ties your IP Address with every request you make. You can use [startpage](https://www.startpage.com/ ) or [https://html.duckduckgo.com/html/](duckduckgo).
* Some sites can create a profile of you by the [referer HTTP header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer). This header shows the last URL you were on before requesting the actual page. To avoid this you can commit to alway start fresh from the Browser or to always start every request from [startpage](https://www.startpage.com/ ) or [https://html.duckduckgo.com/html/](duckduckgo) or some page like that.
* How secure is your browser? [Panopticlick](https://coveryourtracks.eff.org/)
* Try not to use OAuth when registering to a service. Create an actual account with that service. OAuth gives up a lot of privacy for the sake of convenience.
* HTML5 has enabled new tracking technology, by accident. The Canvas fingerprinting, uses HTML5 canvas element to draw a simple image. The idea is that hardware and software will render the image uniquely. This number is then matched to previous instances of that number seen on other websites  around the world. This starts to create a profile whenever you return to this site.  You can use CanvasBlocker plugin.
	* [canvas](https://browserleaks.com/canvas)
	* [canvas-fingerprinting](https://fingerprint.com/blog/canvas-fingerprinting/)
* Update your network device firmware and change the default password. Use WPA/WPA2. Disable WPS and you might consider whitelisting (only allow certain MAC Addresses).
* "In general do not respond to any requests for personal information, even if they seem trustworthy."
* Photos have metadata. Exchangeable Image File (EXIF). This can include your location. You can either disable this feature when taking a photo, of extract metadata later on.
* "[...] once you post something to a social network, it's owned by that network and out of your hands."
* When pairing your phone with a car, it typically also copies all  your contacts.
* IoT devices are a open vector. "This is a computer that the user can't put an antivirus on". This can lead the attacker to other devices on your network.
* Any hard drive you want to sell/donate it needs to be securely wiped.
* Try to change the way you write online. This, along with the speed in which you type, can also be use to create a profile of you. This is called "keystroke analysis". There are keyboard privacy plug-ins that randomizes the rate at which characters reach the DOM.
* Don't use public Wi-Fi. You are better off using your cellular connection from your phone or tethering to your laptop. Also, use a VPN service that does not store logs. A VPN may hide your IP from the Internet, but the VPN does know your originating IP Address. Better to use a VPN service that does not store logs.
* As for Browser selection, better to use [Tor](https://www.torproject.org/download/). Tor is an excellent option for browsing the web because when you make a request to a site, this requests passes through several nodes before reaching the site's IP Address.
	* From what I understand the author says VPN then Tor. But you can also do TOR then VPN, each has its pros and cons.
	* Also, is it really worth it to use a VPN service if you are already using Tor? I do not know, maybe maybe not.
* If you have to connect to a public Wireless network, change your MAC Address and then connect to your VPN provider. Remember that your MAC address will be stored in the wireless device. Change to a new MAC address every time you connect to a public Wi-Fi. After a reboot the original MAC address returns.
* If you upload anything to the Cloud, you first need to encrypt it. Don't trust the Cloud.Or make sure you Cloud provider has no knowledge of passwords and data. If using AES use 256-bit key.
* Encrypt you hard drive with PIN or external USB. If using Bitlocker don't ask Microsoft to save your key.
* Cell phones continually send out tiny beacons to the tower or towers physically closes to them. This will identify you. You cannot avoid this.
* With a burner phone you can still be traceable given patterns of usage. If you always use them from the same spot or if then you call with your registered phone at the same spot that is suspicious.
* If going to a Hotel, make sure that they wipe the card  after you check-out. These cards may have: customer name, customer home address, hotel room number, check-in date and check-out data, customer credit card number and expiration date! Author suggests to destroy them, but well, in my country you do have to pay if the card is missing lol.

## Things I Can Commit To...

This is a list of things that, either I currently do, or can commit to do in the near future. I would say that this is a list that most people can commit to:

* Use a VPN (when outside home).
* Use a Password Manager.
* Don't use OAuth.
* Strong passwords.
* Consider using a specific Anti-Virus and Firewall.
* Use the Keyboard privacy plug-in.
* Don't use public Wi-Fi. If I really need it, VPN.
* Change MAC Address if I use public Wi-Fi.
* Always different passwords for each account.
* Use 2FA/FA in every service that I can.
* Consider using other search engines.
* Always starting from a certain site when starting a request.
* Consider encrypting before storing information in the Cloud.
* Router with WPA2, strong passwords and WPS disabled.
* Encrypt my hard drive.

## Te Become Invisible

I have summarized it as well as I could:

1. Buy a laptop, with cash. Ask somebody. This laptop's solely purpose is for anonymous activities. **Never** turn on this laptop near your home, never access personal stuff with this laptop, never connect this laptop to your  home network.
2. Install Tail as Operating System (OS).
3. Buy prepaid gift cards.
4. Find an email provider that allows sign up without SMS validation. Use Tor to mask you IP location when you sign up.
	1. Use generic names when creating email. Nothing that identifies you...
5. Use this account, using Tor of course, to sign up for a Bitcoin wallet, pay with the prepaid card that you bought.
	1. This wallet will be in charge of sending bitcoins to the laundering service.
6. Setup a second anonymous email address and a new second Bitcoin wallet that will receive the bitcoin from the laundering service.
	1. This needs to be done by establishing a new Tor circuit, to prevent any association with the first email account and wallet. Close Tor and the open it again to get a new circuit.
7. Use a bitcoin laundering service.
8. Sign up for a VPN service that accept Bitcoins. Pay with the laundered money. Use VPN that does not store logs.
9. Obtain a burner portable hotspot device. Pay someone to buy it, cash.
	1. Never turn on at home.
10. To access the Internet, use the burner hotspot device AWAY from home/work and your other cellular devices.
	1. When using anonymous hotspot, turn off any of personal devices that use celular signal to prevent the pattern of your personal devices registering in the same place as the anonymous device.
11. Power up your laptop and connect to the VPN through the burner hotspot device.
12. Use Tor to browser the internet.

**One tiny error can give your identity away.**

If you really do not have the need for it, then this is simply not necessary. The author really mentions this: **what is your purpose for being anonymous? And then act appropriately** It is not a one-size-fits all solution. **Your individual requirements will dicate the necesasry steps you need to take to maintain your desire level of anonymity.**

**Separation is really important!**

To wrap up, The Art of Invisibility offers valuable insights into protecting your privacy in a world where data is constantly being collected. While some of its concepts may seem familiar to those in the IT field, it still provides useful tips for anyone looking to enhance their digital security. Ultimately, the book serves as a reminder that achieving true anonymity requires commitment, discipline, and a constant awareness of the digital world around us.




