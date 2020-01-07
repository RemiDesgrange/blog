---
title: "My Dns-Over-HTTP (DoH) Setup"
date: 2020-01-07T13:08:52+01:00
draft: false
---

I recently setup a Dns over HTTP server. I use the excellent article from [Stéphane
Bortzmeyer](https://www.bortzmeyer.org) : [Documentation technique de mon résolveur
DoH](https://www.bortzmeyer.org/doh-mon-resolveur.html)[FR].

The setup use `dnsdist` in 1.4 version, which support DoH.

Here is a docker composition to run the setup : https://github.com/RemiDesgrange/dnsdist-config/

It run for 2 months on one of my machine without any problem. I didn't put any TLS config on
`dnsdist`, instead a `nginx` config takes care of it. It's pretty standard nginx reverse proxy
config from

```nginx
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name doh.desgran.ge;


	ssl on;
	ssl_certificate fullchain.pem;
	ssl_certificate_key privkey.pem;
	ssl_session_timeout 1d;
	ssl_session_tickets off;
	ssl_protocols TLSv1.3;
	ssl_prefer_server_ciphers off;

	# HSTS (ngx_http_headers_module is required) (63072000 seconds)
	add_header Strict-Transport-Security "max-age=63072000" always;

	# OCSP stapling
	ssl_stapling on;
	ssl_stapling_verify on;
	location / {
		proxy_pass http://localhost:8053;
		proxy_redirect off;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

}
```

Since it's for my personal use, I don't bother supporting TLS prior to 1.3.

Once done you can switch your resolver in the Firefox settings. On Chrome it's not (yet) that
straight forward.
