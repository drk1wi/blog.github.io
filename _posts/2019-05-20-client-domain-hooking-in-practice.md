---
layout: post
title: Client Domain Hooking - Example Attack
---

In my last blog [post](https://blog.duszynski.eu/hijacking-browser-tls-traffic-through-client-domain-hooking/) I have released a paper that described all relevant technical aspects of the '_Client Domain Hooking_' attack, along with a HTTP Strict Transport Security (HSTS) survey made for the TOP 1000 Alexa websites.  

In this post I will show how to run a simulated, '_Client domain Hooking_' attack which will be used to evaluate an example browser-based application resistance to this form of an attack.

### "Prerequisites" 

In order to execute this attack, you will need the following set-up:
- any active and registered domain name (we will be using '*cdh.modlishka.io*' as an example).
- a valid wildcard TLS certificate for the domain that we own (it can be either bought or obtained from the "[Let's Encrypt](https://letsencrypt.org/)" service).
- latest version of [**Modlishka**](https://github.com/drk1wi/Modlishka) v.1.1 reverse proxy.
- target applications, such as the latest version of Chrome browser and example vulnerable WebView based [mobile applications](https://github.com/drk1wi/WebViewApps).

I will not go through the basic steps of configuring and deploying all of the above mentioned prerequisites. You can find all of the relevant information about it [here](https://github.com/drk1wi/Modlishka/wiki/How-to-use). 
The remaining part of this post will assume that the whole infrastructure and tools have been properly configured.

### A general overview of the approach

![Client Domain Hooking](https://raw.githubusercontent.com/drk1wi/assets/master/client_domain_hooking.png)

Interception of the HTTP flow through a single non-TLS HTTP request for one of the domains, will indirectly result in a compromise of all future requests in the current browsing session context.

### "Hijacking HTTP traffic flow"

Lets consider a standard attack vector, when the target domain name has been hijacked through a 'DNS Cache Poisoning':

- target domain: google.com
- browser-based client: Chrome (76.0.3796.0), iOS and Android vulnerable WebView mobile app.

#### 1. Simulate a poisoned DNS entry

The recommended approach, is to use the [_dnsmasq_](https://wiki.debian.org/HowTo/dnsmasq), which allows to easily spoof chosen domain names.

This can be also achieve by simply  modifying your ‘/etc/hosts’ entries and appending the following entry: 
```bash
127.0.0.1 google.pl
```
where 127.0.0.1, should be of course replaced with your proxy IP address. 
Just bear in mind, that not all browsers respect this file.

![Spoofed DNS record](https://raw.githubusercontent.com/drk1wi/assets/master/ping_google.png)

_Note_: google.com has been chosen for a particular reason. It is one of many domains that do not return HSTS headers (as highlighted in original paper).

 #### 2. Run Modlishka in 'Dynamic Mode'

Enable the following settings in the JSON configuration file:
```bash
  "dynamicMode": true,
  "plugins": "hijack"
```

Execute the tool with your prepared configuration file:

```bash
 ./dist/proxy -config domain_client_hooking.json 
```
![Modlishka running in dynamic mode](https://raw.githubusercontent.com/drk1wi/assets/master/modlishka_hijack_run.png)

 #### 3. Test the browser

Open the browser and type in example 'google.com':


![alt text](https://raw.githubusercontent.com/drk1wi/assets/master/req1.png)

![alt text](https://raw.githubusercontent.com/drk1wi/assets/master/req2.png)

Modlishka diagnose log output:
![alt text](https://raw.githubusercontent.com/drk1wi/assets/master/hijacked.png)


[![Watch the video](https://i.vimeocdn.com/video/783692472.jpg)](https://vimeo.com/336760747?autoplay=1&quality=1080p)


 #### 4. iOS mobile app
 
Project for this simple mobile application can be found [here](https://github.com/drk1wi/WebViewApps) for the reference.


Running this iOS application will have the following effect:

[![Watch the video](https://i.vimeocdn.com/video/783694263.jpg)](https://vimeo.com/336762373?autoplay=1&quality=1080p)

 
 #### 5. Android mobile app
 
Project for this simple mobile application can be found [here](https://github.com/drk1wi/WebViewApps) for the reference.


Running this Android application will have the following effect:

[![Watch the video](https://i.vimeocdn.com/video/783706297.jpg)](https://vimeo.com/336771078?autoplay=1&quality=1080p)



#### 6. Hijacking and diagnosing client HTTP Traffic at scale
Hijacking HTTP traffic flow through a single domain is definitely an interesting proof-of-concept of the potential consequences.
However, it's a bit impractical from a usable diagnosis perspective.

In order to improve and automate this process it is recommended to redirect all of the HTTP related traffic to the IP address of our proxy domain 'evil.tld* .

Example _IPTABLES_ rule, that can be used on your router gateway:

```bash
iptables -t nat -I PREROUTING -p tcp -s client_ip_address --dport 80 -j DNAT --to proxy_server_address:80
```

Then you can easily inspect if any of your applications are sending non-TLS HTTP requests that can be potentially hijacked:

![Client Domain Hooking](https://raw.githubusercontent.com/drk1wi/assets/master/diagnose.png)



#### 6.Conclusions

This form of an attack can lead, in certain circumstances , to hijack of both TLS and non-TLS HTTP traffic of browser-based applications without a requirement of constantly running an active network layer attack. 

Fortunately, there are several things that can be done about this:

There are several mitigation methods that can be used to prevent this form of an attack:
- For mobile application, remember to forbid all non-TLS traffic without any exceptions.
- For browsers, you should ensure that HSTS is always used for all of your domain names, preferably an an HSTS 'preload' option registered.
- Furthermore, you can use Modlishka with the above described set-up to check if your application cannot be easily domain hooked.
- End users should consider disabling non-TLS traffic through the following example plugins: ["Firefox"](https://addons.mozilla.org/en-US/firefox/addon/force-https/), "Chrome" (still waiting for approval to the Chrome Store...)
- More possible solutions are described in the released paper.
 
 _Note: Details and potential risks related to this attack have been reported to all major browser vendors for their consideration._
