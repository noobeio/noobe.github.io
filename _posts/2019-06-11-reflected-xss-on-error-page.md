---
layout: post
title: Reflected XSS on Error Page
excerpt: "Sometimes to exploit an XSS (specifically Reflected XSS), we are focused on finding input pages such as Search Columns and etc to to find out is that form has an XSS vulnerability or not."
tags: [bug hunting, hacking, xss]
categories: [bug hunting]
comments: true
image:
  feature: 1602ba3c7472d6e3ebdd237341f8754d-f.png
---

Sometimes to exploit an XSS (specifically Reflected XSS), we are focused on finding input pages such as **Search Columns** and etc to to find out is that form has an XSS vulnerability or not.

Not infrequently a developer is only focused on doing sanitation and filters on these attacks on pages that are commonly accessed by visitors. Does not rule out the possibility of XSS attacks can affected on other pages, including an **Error Pages**.

When doing some Private Bug Hunting on Bugcrowd, I found a feature for Uploading and Downloading File. After the file is being uploaded successfully, to download the file, the user will be directed to the URL like this:

```
https://b15.[redacted.com]/file.php?spaceid=user@mail.com&file=filename.jpg
```

At first, I thought the URL had an `LFI` or `LFD` vulnerability, but after trying to change the file parameters with another file, it didn’t work and gave an error message.

```
https://b15.[redacted.com]/file.php?spaceid=&file=../../../../etc/passwd
```

![LFI Failed](/assets/1602ba3c7472d6e3ebdd237341f8754d-1.png)

But if you pay attention, the contents of the file parameter are reflected on the error page. Then I tried to insert an HTML tag to test whether there is a filter or not in the parameters of the file.

And sure enough, `HTML` tags were successfully rendered on that page.

```
https://b15.[redacted.com]/file.php?spaceid=&file=<h1>asu
```

![HTML on Error Page](/assets/1602ba3c7472d6e3ebdd237341f8754d-2.png)

Without waiting a long time, I immediately tried an XSS payload on the page and XSS was executed!

```
https://b15.[redacted.com]/file.php?spaceid=&file=<img src=x onmouseover=alert(1)>
```

![XSS on Error Page](/assets/1602ba3c7472d6e3ebdd237341f8754d-3.png)

Some tips for hunting Reflected XSS is to test various parameters contained in an endpoint. Either on the Front End Page, or even on the Error Page like the example above.

So this article was written, hopefully it will be useful for us all.