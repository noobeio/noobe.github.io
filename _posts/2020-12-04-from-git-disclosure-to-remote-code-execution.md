---
layout: post
title: From Git Folder Disclosure to Remote Code Execution
excerpt: "A few moments ago I did Bug Hunting activities in one of the Private Programs at Bugcrowd. As usual, the hunting process begins with Recon and Enumeration. The hunting process is carried out on this target in Blackbox. No credentials are provided, and the app's front page is just a login page."
tags: [bug hunting, hacking, git folder disclosure, file upload, php]
categories: [bug hunting]
comments: true
image:
  feature: f2c8ef1e2f6eb0783abcc526779783f1.png
---

بسم الله الرحمن الرحيم

A few moments ago I did Bug Hunting activities in one of the Private Programs at Bugcrowd. As usual, the hunting process begins with Recon and Enumeration. The hunting process is carried out on this target in Blackbox. No credentials are provided, and the app's front page is just a login page.

**I. GIT Folder Disclosure**

During the Enumeration process, I found a `.git` directory that was exposed to the public.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120301.png"/>

By using <a href="https://github.com/arthaud/git-dumper">this tool</a>, I was able to download the Application's Source Code from the `.git` directory.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120302.png"/>

**II. Finding The Credentials**

Even though I got the source code, I don't get any credentials on it. Also, this application is not vulnerable to SQL Injection attacks so there's no way to bypass the Login Page.

After checking a few folders, I found a `database` directory and an SQL file called `structure.sql`.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120303.png"/>

However, in those SQL file, there's only one default user which md5 hashed password that can't be cracked.

Fortunately, in the application, the **Directory Listing** is **Enabled**. When I open the `database` directory, it shows a few database files, not just one file like in the `.git` directory.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120304.png"/>

I quickly grab the latest file and check it's content. There are a few users on it with multiple roles. Unfortunately, there are only 2 users with the **admin role** and the password can't be cracked.

Then I move to the user with a **non-admin role**, and I was able to crack some **non-admin** users and finally, I can log in to the application.

**III. Bypassing Restricted File Upload**

The Application has a feature called `Create Avatar`. Through that feature, user can create a custom avatar by choosing several options on it.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120305.png"/>

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120306.png"/>

After choosing the image option, the browser will send a request to server with 2 parameters, `imgdata` and `filename`. The `imgdata` parameter is containing **Base64 Encoded** image file that we generated from **Create Avatar Feature**, and the `filename` is the file name that will be stored in the server.

There are restrictions that have been implemented by the application to prevent users from uploading malicious files:

1. The server only accepts files with `.png`,` .jpg`, and `.gif` extensions
2. The server only accepts files with the image data type

If we try to upload a file with `.php` extension, the server will returning an `error` message.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120307.png"/>

However, after checking the Source Code that I obtained before, this filter can be easily bypassed by using double extension, for example: `filename.png.php`.

The second filter is checking the content of `imgdata` and must be containing `data:image/png`, `data:image/jpg`, or `data:image/gif`. This is not an issue since we still able to execute the PHP file even though the Content Type was set to Image file.

For the initial test, I try to upload PHP Info function:

```
data%3Aimage%2Fpng%3Bbase64%2CPD9waHAgcGhwaW5mbygpOyA/Pg%3d%3d
```

The `PD9waHAgcGhwaW5mbygpOyA/Pg` is a Base64 Encoded for `<?php phpinfo(); ?>`

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120308.png"/>

And the file was successfully uploaded!

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120309.png"/>

By using the same way, I was able to Upload PHP Shell and successfully execute an OS command.

<img width="800px" alt="GIT Folder Disclosure" src="https://noobe.io/img/2020120310.png"/>


