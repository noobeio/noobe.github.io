---
layout: post
title: How I accidentally found Bug in Google Search Console
excerpt: "It started when I wanted to add my website to Google Search Colsole. I found a form to enter a domain name to add to the application."
tags: [bug hunting, hacking, google, broken access control]
categories: [bug hunting]
comments: true
image:
  feature: 40b89ba586da5e300fe220d4af958c52-f.png
---


بسم الله الرحمن الرحيم

In this simple writeup, I would like to tell how I found an Access Control bug in the `Google Search Console` application, where I can get information related to the domain that I added to the application even though the domain was not successfully verified by me.

It started when I wanted to add my website to Google Search Console. I found a form to enter a domain name to add to the application. So I try to add `google.com` as my domain.

![Add Domain](/assets/40b89ba586da5e300fe220d4af958c52-1.png)


After submiting the domain name, then a popup box appears to continue the domain verification process. There are several ways to verify the domain, such uploading an HTML file, and adding a meta tag to the website header.

In this process, I tried a several techniques to bypass the process, such as `Tampering the Request`, `Polluting the Parameter`, and `Manipulate Responses`, but everything is fail.


![Domain Verification](/assets/40b89ba586da5e300fe220d4af958c52-2.png)


I didn't try more things for this, because at first I just wanted to add my website to Google Search Console, not to find bugs.

A few days later, I got an unusual email in my inbox. The email informs me about an update to the domain that registered in my Google Search Console. But the information sent is an update from the `google.com`!


![Domain Update Notification](/assets/40b89ba586da5e300fe220d4af958c52-3.png)


Of course this is a bug, because at first I was unable to verify the `google.com` domain. It seems that the system cannot validate whether my account has successfully verified the domain or not, so when there is an update on that domain, the information will be sent to my email.

So I reported the bug through Google VRP, and a few days later I got `Nice Catch!`.

![Nice Catch!](/assets/40b89ba586da5e300fe220d4af958c52-4.png)

Through this bug, someone can get information related to a domain registered in Google Search Console. Only by adding the domain in his account, so every time there is an update, he will get that information. For this bug, Google awarded me a reward of `$1337`.

![Reward](/assets/40b89ba586da5e300fe220d4af958c52-5.png)