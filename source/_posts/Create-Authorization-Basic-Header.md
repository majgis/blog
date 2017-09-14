---
title: Create Authorization Basic Header
date: 2017-09-13
tags:
- curl
- bash
- header 
---
The [HTTP Authorization request header][0] is sometimes required to authenticate a user agent with a server.
This post explains how to create the header on linux at command line.

## Syntax

The HTTP Authorization request header has the following syntax:
```
Authorization: <type> <credentials>
```

The type is typically ["Basic"][4], in which case the credentials are of the form `user:password` encoded as [base64][1].


Curl will generate this header for us if we use the `-u` option:
``` bash
$ curl -v -u user:password majgis.github.io     
...
> Authorization: Basic dXNlcjpwYXNzd29yZA==
...
```

Now for the real question, how do we generate this header for use with curl's `-H` option?

## Experiments
Here is the first attempt to base64 encode `user:password` that is WRONG:
``` bash
# This is WRONG!
$ echo user:password | base64 
dXNlcjpwYXNzd29yZAo=
```
Notice that `dXNlcjpwYXNzd29yZAo=` does not equal `dXNlcjpwYXNzd29yZA==`.

Let's decode to find the difference:
``` bash
$ echo dXNlcjpwYXNzd29yZAo= | base64 -d
user:password

$ echo dXNlcjpwYXNzd29yZA== | base64 -d
user:password%
```

That `%` is different.  What is that?  

The `%` symbol is [how zsh handles the end of partial lines][2].  
In other words, zsh assumes we want our prompt on a newline even if the last command didn't end with
a newline. That `%` symbol is how we know the difference between a newline the last command output
and a newline zsh graciously inserted for us.  

Let's execute this command under bash and see the difference:

``` bash
bash
$ echo dXNlcjpwYXNzd29yZAo= | base64 -d
user:password

$ echo dXNlcjpwYXNzd29yZA== | base64 -d
user:password$ 
```

The base64 encoded `user:password` that curl generated is not terminated with a newline, unlike the one we generated.

Let's see if we can get `echo` to not add that newline.

``` bash
$ man echo | cat | grep 'newline' 
       -n     do not output the trailing newline
```

The `-n` option for `echo` is what we want.
``` bash
$ echo -n user:password | base64
dXNlcjpwYXNzd29yZA==
```
We see that the output is exactly the same as what curl generated.

Putting it all together:

``` bash
$ curl -v -H "Authorization: Basic `echo -n user:password | base64`" majgis.github.io
...
> Authorization: Basic dXNlcjpwYXNzd29yZA==
...
```
## Take Away
In zsh, a `%` symbol is placed at the end of a line where zsh inserted a newline for us.

It is important to include echo's `-n` option when piping content to an encoder to avoid a
newline character being included in the encoded output.


``` bash
$ echo -n user:password | base64
```

## Experimental Setup

``` bash
$ echo && curl --version && echo && lsb_release -d -c && echo && echo $0 && zsh --version

curl 7.52.1 (x86_64-pc-linux-gnu) libcurl/7.52.1 OpenSSL/1.0.2g zlib/1.2.11 libidn2/0.16 libpsl/0.17.0 (+libidn2/0.16) librtmp/2.3
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz TLS-SRP UnixSockets HTTPS-proxy PSL 

Description:    Ubuntu 17.04
Codename:       zesty

/usr/bin/zsh
zsh 5.2 (x86_64-ubuntu-linux-gnu)

```

## References
- [MDN - HTTP/Headers/Authorization][0]
- [Wikipedia - base64][1]
- [The Z Shell Manual - Prompting][2]
- [MDN - HTTP/Authentication: Basic authentication scheme][3]
- [RFC7617 - The 'Basic' HTTP Authentication Scheme][4]

[0]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization
[1]: https://en.wikipedia.org/wiki/Base64
[2]: http://zsh.sourceforge.net/Doc/Release/Options.html#Prompting
[3]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#Basic_authentication_scheme
[4]: https://tools.ietf.org/html/rfc7617


