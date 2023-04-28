---
title: "TLS Debugging Nightmare With Macos and Nginx"
date: 2022-05-15T20:11:21+02:00
draft: false
tags: ["MacOS", "nginx"]
category: ["MacOS"]
image: ""
description: ""
---
I have a server running Nginx for years [^1], and it just work super nice.

On thing that I do is watching video with VLC, streaming the video from the server.

It worked well on Linux and Windows but not on MacOS.

## But Why ?

The server was TLS 1.3 only. I'm the only one using it so it has a very narrowed cipher suite and just TLSv1.3.

It's working on every OS. Except MacOS. Because VLC uses the "[SecureTransport](https://developer.apple.com/documentation/security/secure_transport) 
API from MacOS, and it doesn't support TLS 1.3. 

MacOS ships with LibreSSL instead of openssl, and I could replicate the problem nicely in the CLI:

```
openssl s_client -connect my.server.com:443
CONNECTED(00000005)
4300588588:error:1400442E:SSL routines:CONNECT_CR_SRVR_HELLO:tlsv1 alert protocol version:/AppleInternal/Library/BuildRoots/66382bca-8bca-11ec-aade-6613bcf0e2ee/Library/Caches/com.apple.xbs/Sources/libressl/libressl-2.8/ssl/ssl_pkt.c:1200:SSL alert number 70
4300588588:error:140040E5:SSL routines:CONNECT_CR_SRVR_HELLO:ssl handshake failure:/AppleInternal/Library/BuildRoots/66382bca-8bca-11ec-aade-6613bcf0e2ee/Library/Caches/com.apple.xbs/Sources/libressl/libressl-2.8/ssl/ssl_pkt.c:585:
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
    Protocol  : TLSv1.2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Start Time: 1652638832
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
```

I quickly jump into google, with post on SO etc... saying that MacOS doesn't support TLS1.3 and I should activate TLS1.2. I did it didn't worked.

## Solution

The solution is to add a TLS1.2 into your nginx config + `default_server` as following:

```
server {
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    ssl_protocols TLSv1.2 TLSv1.3;
...redacted
}
```


Thanks Apple for letting me loose 2 hours.




[^1]: I updated it !
