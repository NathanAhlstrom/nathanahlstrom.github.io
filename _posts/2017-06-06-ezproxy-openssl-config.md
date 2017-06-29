---
layout: post
title: SSLCipherSuite for ezProxy
date: 20170606
type: post
published: true
categories:
- System Administration
tags: [library, ezproxy, openssl, security]
meta:
author: Nathan Ahlstrom
excerpt: Locking down ezProxy SSL configuration
---
# OpenSSL configuration for ezProxy - SSLCipherSuite #

Moving our ezProxy installation around on our network to increase availability required a security assessment.  Resulting in:

### [Happy Birthday to the SWEET32 BEAST](https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack "wikipedia summary of TLS attacks"){:target="_blank"} ###

"Clean those up!", my stern-faced security colleague commanded.

The ezProxy listserver and [documentation](http://www.oclc.org/support/services/ezproxy/documentation/manage/ezproxy-openssl.en.html "ezProxy OpenSSL configuration"){:target="_blank"} are very helpful, however the vulnerability scanning remediation guide had the most help, recommending:

<code>
ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSAAES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSAAES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK
</code>

After adding this complex line the scan came back essentially the same!  I still had TLS 1.0 enabled.

A few rounds of scan and tweak with my security friend, then I learned about [openssl s_client](https://www.openssl.org/docs/man1.0.2/apps/s_client.html "openssl s_client manpage"){:target="_blank"}.

<code>
	$ openssl s_client -servername 10.1.1.1 -connect 10.1.1.1:443 -tls1 -cipher 3DES < /dev/null
</code>

This snippet connects to server 10.1.1.1 using TLS 1.0 and the 3DES cipher, both of which have problems, so I'm looking for this to fail!

As you can see it did fail:

<pre>
CONNECTED(00000003)
140219233040064:error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure:s3_pkt.c:1487:SSL alert number 40
140219233040064:error:1409E0E5:SSL routines:ssl3_write_bytes:ssl handshake failure:s3_pkt.c:656:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 7 bytes and written 0 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1
    Cipher    : 0000
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: 
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1496751399
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
</pre>

Ended up running this command with about a dozen different combinations of -tls1, -tls1_1, -tls1_2, and -ssl3 (just to be sure) and ciphers: 3DES, RC4 until it failed on all the weak encryption standards.

This is the final outcome in my ezProxy config.txt file:

<code>
SSLHonorCipherOrder On
</code>

<code>
SSLCipherSuite  ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSAAES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK
</code>

Make sure this SSLCipherSuite line does not contain any newline characters in your config.txt file.

## UPDATE 6/29/2017 ##

Some colleagues on the ezproxy listserv (extremely helpful resource for ezproxy administrators, btw) mentioned this resource:

[https://wiki.mozilla.org/Security/Server_Side_TLS](https://wiki.mozilla.org/Security/Server_Side_TLS){:target="_blank"}

Based on this resource, he recommended removing the kEDH+AESGCM cipher option.

Thanks Dom!

Here is the updated SSLCipherSuite line minus kEDH+AESGCM:

<code>
SSLCipherSuite  ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSAAES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK
</code>

Another colleague chimed in recommending this post for keeping up to date on web server cipher options:

[https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/](https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/){:target="_blank"}

Thanks Dwight!
