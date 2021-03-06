---
layout: post
title: Phishing NG. Bypassing 2FA with Modlishka.
---

This blog post is an introduction to the reverse proxy "Modlishka" tool, that I have just [released](https://github.com/drk1wi/Modlishka). 
I hope that this software will reinforce the fact that social engineering is a serious threat, and cannot be treated lightly.


On the page below, I will shortly describe how it is possible tool to bypass most of the currently used 2FA authentication schemes, including OTP token interception and post authentication user session hijack.


### "Modlishka" introduction

Over many years of my penetration testing experience, I have found '_social engineering_'  the easiest and most of effective way to get a proper foothold into the internal network of my customers. I know that many APT groups think the same...
This is all because one definitely does not need to burn a _0day_ exploit/s for all of those sophisticated top-notch security defenses that are protecting the perimeter, when often just few e-mails or phone calls will do just perfectly fine to compromise internal infrastructure and companies sensitive data.  


Modlishka was written with an aim to make that second approach (ethical phishing campaigns) as effective as possible and to show that current 2FA does not protect well against this form of an attack.

This tool should be very useful to all penetration testers, that want to carry out an effective phishing campaign (also as part of their red team engagements).  


### "Bypassing 2FA"

Enough with the introductions and lets get into the 'merit'.

_Note: This will be an example set up that will run locally on your computer._ 

#### 1. Fetch the tool 

```bash
$ go get -u github.com/drk1wi/Modlishka
$ cd $GOPATH/src/github.com/drk1wi/Modlishka/
```

    
#### 2. Configure the 'autocert' plugin 
 
 This step is required if you want to serve the page over a browser trusted TLS channel:
 
```bash
$ openssl genrsa -out MyRootCA.key 2048`
$ openssl req -x509 -new -nodes -key MyRootCA.key -sha256 -days 1024 -out MyRootCA.pem
```

Replace the _const CA_CERT_ variable with the content of MyRootCA.pem file and _const CA_CERT_KEY_  with the content of MyRootCA.key in the 'plugin/autocert.go' file.

Install and set the right trust level for the 'MyRootCA' CA in your browsers certificate store and you are all done.

#### 3. Compile and launch "Modlishka" 
    
```bash
$ make
$ sudo ./dist/proxy  -config templates/google.com_gsuite.json
```

_A small disclaimer here: I am not encouraging to run campaigns against any particular company. The choice of an example service is purely based on its popularity and my believe that its really well secured. As such I am not trying to prove that is not the case (especially since most of the services can be targeted in a similar way), but to raise an awareness of the risk by using one of the most popular service as a proof of concept._

#### 4. View the web page in your browser

 Modlishka in action against an example 2FA (SMS) enabled authentication scheme:

[![Watch the video](https://i.vimeocdn.com/video/749353683.jpg)](https://vimeo.com/308709275)

[https://vimeo.com/308709275](https://vimeo.com/308709275)

The following link can be used to view your launched test page. You can notice how the '_ident_' parameter is hidden from the user on a first request: [https://loopback.modlishka.io?ident=user_tracking_param](https://loopback.modlishka.io?ident=user_tracking_param).

Collected credentials, 2FA tokens and all other requests (for all related domains) can be found in the 'log' file or in one of the included plugins (this includes session impersonation proof-of-concept - still in beta though):

![alt text](https://raw.githubusercontent.com/drk1wi/assets/master/7d0426a133a85a46a76a424574bf5a2acf99815e.png)

![alt text](https://raw.githubusercontent.com/drk1wi/assets/master/779e2185531eadb81996045fe56952860efd7c08.png)


####  5. Customize your settings

If you like the tool. You can start adjusting the configuration for your chosen domain.
Modlishka can be easily customized through a set of available command line options or JSON configuration files.

Please refer to [wiki](https://github.com/drk1wi/Modlishka/wiki)  pages for further functionality descriptions. 

### Conclusion

So the question arises... is 2FA broken? 

Not at all, but with a right reverse proxy targeting your domain over an encrypted, browser trusted, communication channel one can really have serious difficulties in noticing that something is seriously wrong.

Add to the equation different browser bugs, that allow URL bar spoofing, and the issue might be even bigger...

Include lack of user awareness, and it literally means giving away your most valuable assets to your adversaries on a silver plate. 

_At the end even the most sophisticated security defense systems can fail if there is no sufficient user awareness and vice versa for that matter._

Currently, the only way to address this issue, from a technical perspective, is to entirely rely on 2FA hardware tokens, that are based on **U2F** protocol. You can easily buy them online. However, remember, that right user awareness is as equally important.

Summing it up, you should:
- use U2F hardware tokens as your second authentication factor.
- use password managers, that will ensure that domain name in your browser is correct before pasting in the password.
- constantly raise user awarness about current social engineering techniques.

Why am I releasing this tool?
I believe that without a working proof of concept, that really proves the point, the risk is treated as theoretical, and no real measures are taken to address it properly. This status quo, and lack of right awareness about the risk, is a perfect situation for malicious actors that will happily exploit it*. 
This concerns both currently used 2FA schemes as well user awareness.

*This was the case for the past year, since the concept of using a basic reverse proxy for phishing purposes was already actively exploited (Amnesty International [report](https://www.amnesty.org/en/latest/research/2018/12/when-best-practice-is-not-good-enough/)). 



