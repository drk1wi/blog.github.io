---
layout: post
title: Hijacking browser TLS traffic through Client Domain Hooking
---


I am releasing a paper that describes a new variation of a man-in-the-middle (MITM) technique which, under certain circumstances, allows to permanently hijack browsers encrypted HTTP communication channel flow and compromise its confidentiality and integrity. 

The technique has been named as "Client Domain Hooking", since it relies on a particular way of achieving client-side communication endpoint persistency by forcing an application to communicate only through a chosen attacker-controlled domain through a single intercepted HTTP request and without breaking applications functionality. This technique was originally implemented in the ['Modlishka' ](https://blog.duszynski.eu/phishing-ng-bypassing-2fa-with-modlishka/) reverse proxy.

The described approach, although very similar to the previously published techniques, has few extra benefits:
-  HTTP flow does not have to be actively redirected to the proxy through a MITM attack (a single intercepted non-TLS HTTP request will be sufficient to hijack current browsing session).
-  reverse proxy server can be located in an arbitrary location (both Internet and Intranet)
-  TLS layer will be trusted by a client without a requirement of installing any additional CA certificate.

Limitations:
- Intercepted TLS connections rely on 'bogus' domain names, which can be spotted by the user. 
- Handling obfuscated JavaScript code is a bit of a challenge and can break the applications functionality.

This paper also contains conclusions from a review of the current security posture of browser-based (desktop and mobile) applications and Top 1000 Alexa web applications from the HTTP Strict Transport Security (HSTS) security mechanism perspective. 

As it appeared, **80%** of the reviewed web applications did not use the HSTS mechanism.


You can find the paper here: 

['Hijacking browser TLS traffic through Client Domain Hooking - Piotr Duszynski.pdf'](https://github.com/drk1wi/assets/raw/master/Hijacking%20browser%20TLS%20traffic%20through%20Client%20Domain%20Hooking%20-%20Piotr%20Duszynski.pdf)

_SHA256:c2d9bf2062b310b92cab5971d5c4454e8bc3e288720cf3241da17f885c7ec9ce_

Along with the paper, I have released an updated version of the ['Modlishka'](https://github.com/drk1wi/Modlishka) tool was released, with capabilities to diagnose browser-based client applications from the described attack perspective. Developers should find this tool helpful in pin pointing and fixing relevant security issues in their applications. 

You can also find example attack scenarios in the following ["post"](https://blog.duszynski.eu/client-domain-hooking-in-practice/).

