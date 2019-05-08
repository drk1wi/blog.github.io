---
layout: post
title: Hijacking browser TLS traffic through Client Domain Hooking
---


I am releasing a paper that describes a new variation of a man-in-the-middle (MITM) technique which, under certain circumstances, allows to permanently hijack browsers encrypted HTTP communication channel flow and compromise its confidentiality and integrity. 

The technique has been named as "Client Domain Hooking", since it relies on a particular way of achieving client-side communication endpoint persistency by forcing an application to communicate only through a chosen attacker-controlled domain through a single intercepted HTTP request and without breaking applications functionality. 

The presented technique was originally implemented in the ['Modlishka' ](https://blog.duszynski.eu/phishing-ng-bypassing-2fa-with-modlishka/) reverse proxy.

This paper also contains conclusions from a review of the current security posture of browser-based (desktop and mobile) applications and Top 1000 Alexa web applications from the HTTP Strict Transport Security (HSTS) security mechanism perspective. 

As it appeared, **80%** of the reviewed web applications did not use the HSTS mechanism.


You can find the paper here: 

['Hijacking browser TLS traffic through Client Domain Hooking - Piotr Duszynski.pdf'](https://github.com/drk1wi/assets/raw/master/Hijacking%20browser%20TLS%20traffic%20through%20Client%20Domain%20Hooking%20-%20Piotr%20Duszynski.pdf)

_SHA256:c2d9bf2062b310b92cab5971d5c4454e8bc3e288720cf3241da17f885c7ec9ce_

Along with the paper, I will shortly release an updated version of the 'Modlishka' tool, with capabilities to diagnose browser-based client applications from the described attack perspective. Developers should find this tool helpful in pin pointing and fixing relevant security issues in their applications.

