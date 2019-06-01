---
layout: post
title: Disclosing TOR users' real IP address through 301 HTTP Redirect Cache Poisoning 
---

This blog post describes a practical application of the '[HTTP 301 Cache Poisoning](https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/)" attack that can be used by a malicious TOR exit node to disclose real IP address of chosen  clients. 

Persistency and possibility to automate HTTP 301 cache poisoning for chosen non-TLS URLS is the key element here.

### PoC Video

- Client: Chrome Canary (76.0.3796.0) 
- Client real IP address: 5.60.164.177 
- Client tracking parameter: 6b48c94a-cf58-452c-bc50-96bace981b27 
- TOR exit node IP address: 51.38.150.126
- Transparent Reverse Proxy: tor.modlishka.io ([Modlishka](https://github.com/drk1wi/Modlishka) - updated code to be released.)

Note: In this scenario Chrome was configured, through SOCKS5 settings, to use the TOR network. Tor circuit was set to a particular TOR test exit node: '51.38.150.126'. This is also a proof-of-concept and many things can be further optimized...

On the malicious TOR exit node all of the traffic is being redirect to Modlishka proxy:
```javascript
iptables -A OUTPUT -p tcp -m tcp --dport 80 -j DNAT --to-destination ip_address:80
iptables -A FORWARD -j ACCEPT
````

[![Watch the video](https://i.vimeocdn.com/video/787654981.jpg)](https://vimeo.com/339586722?autoplay=1&quality=1080p)
[https://vimeo.com/339586722](https://vimeo.com/339586722)


### Example Attack Scenario Description

Assumptions:
- Browser-based application (in this case a standard browser) that will connect through the TOR network and, finally, through a malicious TOR exit node.
- Malicious TOR exit node that is intercepting and HTTP 301 cache poisoning all of the non-tls HTTP traffic.

![Disclosing TOR client IP address](https://raw.githubusercontent.com/drk1wi/assets/master/tor_ip.png)

Lets consider the following attack scenario steps:

1. User connects to the Internet through the TOR network either by setting up browsers' settings to use the TOR SOCKS5 or system wide, where the whole OS traffic is being routed through the TOR network.
2. User begins his typical browsing session with his favorite browser, where usually a lot of non-TLS HTTP traffic is being sent through the TOR tunnel. 
3. Evil TOR exit node intercepts those non-TLS requests and responds with a HTTP 301 permanent redirect to each of them. These redirects will be cached permanently by the browser and will point to a tracking URL with an assigned TOR client identifier. 
The tracking URL can be created in the following way: http://user-identifier.evil.tld. Where 'evil.tld' will collect all source IP information and redirect clients to the originally requested hosts ... or, as an alternative, to a transparent reverse proxy that will try to intercept all of the clients subsequent HTTP traffic flow.
Furthermore, since it is also possible to carry out an automated cache pollution for the most popular domains (as described in the previous [post](https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/)), e.g. TOP Alexa 100 , an attacker can maximize his chances of disclosing the real IP address.
4. User, after closing the TOR session, will switch back to his usual network.
5. As soon user types into the URL address bar one of the previously poisoned entries, e.g. "google.com," browser will use the cache and internally redirect to the tracking URL with an exit-node context identifier.
6. Exit node will now be able to correlate previously intercepted HTTP request and users' real IP address through the information gathered on the external host that used tracking URL with user identifier. The _evil.tld_ host will have information about all of the IP addresses that were used to access that tracking URL.

Obviously, this gives a possibility to effectively correlate chosen HTTP requests with the client IP address by the TOR exit node. This is because the previously generated tracking URL will be requested by the client through the TOR tunell and later, after connecting through a standard ISP connection, again. This is because of the poisoned cache entries. 

Another approach, might rely on injecting modified JavaScript with embeded tracking URLS into the relevant non-TLS responses and setting up the right Cache control headers (e.g. to 'Cache-Control: max-age=31536000'). However, this approach wouldn't be very effective.

Tracking users through standard cookies, by different web applications is also possible, but it's not easy to force the client to visit the same, attacker-controlled, domain twice: once while it's connecting through the TOR exit node and later after it switched back to the standard ISP connection. 


### Conclusions
The fact that it is possible to achieve certain persistency in browsers cache, by injecting poisoned entries, can be abused by an attacker to disclose real IP address of the TOR users that send non-TLS HTTP traffic through malicious exit nodes. Furthermore, poisoning a significant number of popular domain names will increase the likelihood of recieving a callback HTTP request (with assigned user identifier), that will allow to disclose users real IP.
An attempt can be also made to 'domain hook' some of the browser-based clients and hope that a mistyped domain name will not be noticed by the user or will not be displayed (e.g. mobile application WebViews).


Possible mitigation:
- When connecting through the TOR network ensure that all non-TLS traffic is disabled. Example browser plugins that can be used: ["Firefox"](https://addons.mozilla.org/en-US/firefox/addon/force-https/), ["Chrome"](https://chrome.google.com/webstore/detail/dpipdndjcofdfhknlfloeokjiooiojoo/). 
- Additionally, always use browser 'private' mode for browsing through TOR.
- Do not route all of your OS traffic through TOR without ensuring that there TLS traffic only...
- Use latest version of the TOR browser whenever possible for browsing web pages.

### References
- [https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/](https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/) - "HTTP 301 Cache Poisoning Attack"
- [https://www.torproject.org/download/](https://www.torproject.org/download/) - "TOR Browser"
