+++ 
title = "Setup Git Pages"
draft = false 
comments = true 
slug = "" 
tags = ["github", "gh-pages"]
categories = []
date = "2020-01-21T00:00:00-00:00"

showpagemeta = true
showcomments = true
+++

This quick tutorial will describe the process for setting up [GitHub Pages][github-pages] with a custom domain and HTTPS
enabled.

### Step 1. Uploading content

There are various methods to get GitHub Pages published. Using the `gh-pages` branch keeps the `master` branch
free from the clutter of build artifacts. This works really well for CI as demonstrated on part 2 of this journey.

First step was to create the `gh-pages` branch and push it:

```shell script
git checkout -b --orphan gh-pages
git push -u origin gh-pages
```

To verify github has detected the branch you may check under GitHub Pages at:
```text
https://github.com/{USER}/{REPO}/settings
```

{{< figure src="../img/gh-pages-branch.png" >}}

### Step 2. Custom Domain

To setup a custom domain, you must first configure it at:
```text
https://github.com/{USER}/{REPO}/settings
```

{{< figure src="../img/custom-domain-field.png" >}}

Then go to your domain registrar and configure a `CNAME` to your custom domain.

{{< figure src="../img/cname.png" >}}

Verify that the `CNAME` has taken effect. Depending on your DNS server and settings this could take a few
hours to propagate.

```shell script
$ dig why.jromero.codes CNAME +nostats +nocomments +nocmd

; <<>> DiG 9.10.6 <<>> why.jromero.codes CNAME +nostats +nocomments +nocmd
;; global options: +cmd
;why.jromero.codes.		IN	CNAME
why.jromero.codes.	1800	IN	CNAME	jromero.github.io.
```

Last part is to verify that GitHub is serving your website by double-checking the headers. 
Specifically the `server:` header.

```shell script
$ curl -I http://why.jromero.codes

HTTP/2 200 
server: GitHub.com
...
```

### Step 3. Enable HTTPS

To enable HTTPS GitHub must be able to issue a certificate for your domain via [Let's Encrypt](letsencrypt.org).

With the use of this [online tool](https://sslmate.com/caa/) a set of `CAA` records can be generated that enables
Let's Encrypt to issue certificates.

_NOTICE: All the changes are made to the top-level domain and not the subdomain._

The configuration will end up something like this:

```text
Name            Type        Value
jromero.codes.  CAA         0 issue ";"
                            0 issuewild "letsencrypt.org"
                            0 iodef "mailto:root@jromero.codes"
```

Again, you'll go to the domain registrar and configure 3 `CAA` records to your custom domain.

{{< figure src="../img/caa.png" >}}

After applying the changes and waiting for propagation, a quick `dig` should show your changes:

```shell script
$ dig jromero.codes CAA +nostats +nocomments +nocmd
  
; <<>> DiG 9.10.6 <<>> jromero.codes CAA +nostats +nocomments +nocmd
;; global options: +cmd
;jromero.codes.                 IN      CAA
jromero.codes.          1799    IN      CAA     0 iodef "mailto:root@jromero.codes"
jromero.codes.          1799    IN      CAA     0 issue ";"
jromero.codes.          1799    IN      CAA     0 issuewild "letsencrypt.org"
```

Additionally, you should see an update to the `Enforce HTTPS` option:

{{< figure src="../img/enforce-https-24hr.png" >}}

Eventually, this will change to:

{{< figure src="../img/enforce-https-active.png" >}}

You can verify by pulling up your website in chrome and checking the connection:

{{< figure src="../img/cert.png" position="center" style="width: 300px" >}}

#### Troubleshooting

If you run into any problems, a few helpful commands:

##### View connection handshake
```shell script
curl -I -v https://why.jromero.codes
```

##### View certificate information
```shell script
echo | openssl s_client -showcerts -servername why.jromero.codes -connect why.jromero.codes:443 2>/dev/null | openssl x509 -inform pem -noout -text
```

### Done

You should now have a statically hosted site with no storage, traffic, or certificate cost!

{{< figure src="https://media.giphy.com/media/SEre9eirTBgdO/source.gif" >}}


[github-pages]: https://pages.github.com/