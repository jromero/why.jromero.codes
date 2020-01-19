+++ 
title = "Migrating to GitHub - Git Pages"
draft = true 
comments = true 
slug = "" 
tags = ["ci"]
categories = []

showpagemeta = true
showcomments = true
+++

### History

Back before [Microsoft acquired GitHub][ms-acquires], I dealt with an internal struggle between GitHub and
GitLab. They were both great in their own way but what finally sold me on choosing GitLab as my primary
source repository was:

1. I could have as many private repositories as I'd like.
2. It provided a built-in robust CI/CD pipeline system.
 
At the time I also chose to use S3 as my hosting solution. _What a cluster that is compared to other static hosting 
solutions but I'll leave that tale for a different day_. 

Today, I'm switching over my personal site (this site) to use exclusively GitHub. That means using [GitHub Pages][github-pages]
for static file hosting and [GitHub Actions][github-actions] for CI/CD.

What I'll really be focusing on is how to setup Pages and Actions and not so much the migration process I personally
had to go through.

### Setting Up Pages

#### Uploading content

```
# create a new branch for gh-pages
git checkout -b --orphan gh-pages

# push
git push -u origin gh-pages
```

To verify github has detected your branch you may check under GitHub Pages
at:
```
https://github.com/{USER}/{REPO}/settings
```


#### Custom Domain

```
└─▪ dig why.jromero.codes CNAME

; <<>> DiG 9.10.6 <<>> why.jromero.codes CNAME
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46137
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 8192
;; QUESTION SECTION:
;why.jromero.codes.		IN	CNAME

;; ANSWER SECTION:
why.jromero.codes.	1800	IN	CNAME	jromero.github.io.
```

```
└─▪ curl -I http://why.jromero.codes
HTTP/1.1 200 OK
x-amz-id-2: IwZkv6MJib3ThZiywMvokRolo2N8Vmhmtu6IKmhpDg9oDJpVNuBZ1VxwGz/DHZVD5jfOdpb8wSY=
x-amz-request-id: 21E00A2E0BFFE5F3
Date: Sun, 19 Jan 2020 04:21:12 GMT
Last-Modified: Mon, 01 Apr 2019 18:36:22 GMT
ETag: "bc3afa84f33d41684ed62a532d3cb62d"
Content-Type: text/html
Content-Length: 3674
Server: AmazonS3
```

#### Enable HTTPS

https://sslmate.com/caa/

```
Name            Type        Value
jromero.codes.  CAA         0 issue ";"
                            0 issuewild "letsencrypt.org"
                            0 iodef "mailto:root@jromero.codes"
```


```
└─▪ dig why.jromero.codes CAA

; <<>> DiG 9.10.6 <<>> why.jromero.codes CAA
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19838
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 8192
;; QUESTION SECTION:
;why.jromero.codes.		IN	CAA

;; ANSWER SECTION:
why.jromero.codes.	1800	IN	CNAME	jromero.github.io.
jromero.github.io.	3600	IN	CAA	0 issue "digicert.com"
jromero.github.io.	3600	IN	CAA	0 issue "letsencrypt.org"
jromero.github.io.	3600	IN	CAA	0 issuewild "digicert.com"
```

### Setting Up Actions

[ms-acquires]: https://news.microsoft.com/2018/06/04/microsoft-to-acquire-github-for-7-5-billion/
[github-actions]: https://github.com/features/actions 
[github-pages]: https://pages.github.com/

