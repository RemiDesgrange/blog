---
title: "Reducing Python Docker Images"
date: 2019-04-19T14:30:36+02:00
draft: true
---

I'm using docker a lot to build  and deploy the software that I (try to) write. I'm also writing a
lot of python. And one of the things that really anoy me is the size of docker images. Especialy
python. I often laugh about the size of a hello world in Go. But in Go you can deploy your
application in docker with no extra cost. So, compiling this hello world  in go :

```go
//test.go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```
leads to a 2MB binary :
```bash
╭─remi@laptop ~/test/go
╰─$ ls -alih
total 2.0M
8259338 drwxr-xr-x  2 remi remi 4.0K Apr 18 10:15 .
7471105 drwx------ 56 remi remi 4.0K Apr 19 14:40 ..
8258966 -rw-r--r--  1 remi remi   61 Apr 16 08:44 Dockerfile
8258953 -rwxr-xr-x  1 remi remi 2.0M Apr 16 08:43 test
8258965 -rw-r--r--  1 remi remi   72 Apr 16 08:43 test.go
```

And the corresponding `Dockerfile` is quite straight forward :
```Dockerfile
FROM scratch
COPY test /
USER 1325
ENTRYPOINT [ "/test" ]
```
PS : Don't bother about `USER`.
```bash
╭─remi@laptop ~/test  
╰─$ docker images            
REPOSITORY                    TAG                 IMAGE ID            CREATED              SIZE
test-go                       latest              d9663a505be5        8 seconds ago        2MB
```
Ok, 2MB, Not bad finaly. A Real world app that follow this principle, traefik, is 71MB large. And
it's just the binary. And one layer.

So now. I'll build an hello world in python  :

```python
print("Hello, World!")
```

Ok, then docker:

```Dockerfile
FROM python:3.7
COPY . /
ENTRYPOINT ["python3", "/test.py"]
```

and build it then :

```bash
╭─remi@laptop ~/test/go
╰─$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
test-python                        latest              d85038701a7e        10 seconds ago      929MB
```

Yeah 929MB. That's large. And we didn't install anything from pip. You'll tell me, "Yeah, use
alpine". Sure but when I use python alpine, I end up using :

* A not well supported libc (musl)
* We have to build most of package that have binary (like `psycopg2`) and I really don't like
  building python extension, it's kind of fragile. So I need a python that is build and linked
  against glibc (sorry musl and uclibc

# Idea phase
So I had this idea :

> Python is C right ? But Python is linked dynamicaly. Can we build python staticaly ?

The answers is [yes](https://wiki.python.org/moin/BuildStatically). But there are trade off. Large
trade off, like no crypto. I wanted to build a really small image of python in order to build uppon
it, not to build ultra embeddeable stuff.

## A note about the python image

Have you ever looked at the python image buid uppon debian/ubuntu. This scarry. On the `python:3.7` based on debian stretch, you have python2.7 embed in the debian image and also python3.5 because dev dependancies download to build python3.7 require python3, so it download python3.5 from debian apt repo. I can continue like this for a long time. But you get the point. This image is unnecerely heavy. So let's build a small python image.

# First Attempt : using `minideb` image

I thought that I could build on the `bitnami/minideb` image. But I end up with a super large image
without wanting it.

# Second attempt : building it myself

I end up with this `Dockerfile` :

```Dockerfile
FROM debian:stretch as builder

RUN set -ex; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		autoconf \
		automake \
		bzip2 \
		dpkg-dev \
		file \
		g++ \
		gcc \
		imagemagick \
		libbz2-dev \
		libc6-dev \
		libcurl4-openssl-dev \
		libdb-dev \
		libevent-dev \
		libffi-dev \
		libgdbm-dev \
		libgeoip-dev \
		libglib2.0-dev \
		libgmp-dev \
		libjpeg-dev \
		libkrb5-dev \
		liblzma-dev \
		libmagickcore-dev \
		libmagickwand-dev \
		libncurses5-dev \
		libncursesw5-dev \
		libpng-dev \
		libpq-dev \
		libreadline-dev \
		libsqlite3-dev \
		libssl-dev \
		libtool \
		libwebp-dev \
		libxml2-dev \
		libxslt-dev \
		libyaml-dev \
		make \
		patch \
		unzip \
		xz-utils \
		zlib1g-dev \
		\
# https://lists.debian.org/debian-devel-announce/2016/09/msg00000.html
		$( \
# if we use just "apt-cache show" here, it returns zero because "Can't select versions from package 'libmysqlclient-dev' as it is purely virtual", hence the pipe to grep
			if apt-cache show 'default-libmysqlclient-dev' 2>/dev/null | grep -q '^Version:'; then \
				echo 'default-libmysqlclient-dev'; \
			else \
				echo 'libmysqlclient-dev'; \
			fi \
		) \
	; \
rm -rf /var/lib/apt/lists/*

ENV LANG C.UTF-8

# extra dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
		tk-dev \
		uuid-dev \
		git \
        ca-certificates \
&& rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN git clone  -b 3.7 --depth=1 https://github.com/python/cpython.git --single-branch
WORKDIR /opt/cpython
RUN ./configure
RUN make -j8
RUN make install -j8

FROM busybox:glibc

COPY --from=builder /usr/local/lib/python3.7 /usr/local/lib/python3.7
COPY --from=builder /usr/local/lib/libpython3.7m.a /usr/local/lib/
COPY --from=builder /usr/local/bin /usr/local/bin

COPY --from=builder /lib/x86_64-linux-gnu/*.so.* /lib/
COPY --from=builder /usr/lib/*.so.* /usr/lib/

CMD ["/usr/local/bin/python3.7"]
```
The beginning of [this
Dockerfile](https://github.com/docker-library/python/blob/master/3.7/stretch/Dockerfile)

For the reader that knows how python is build, in the official docker image, `libpython` is linked
as a dynamic lib (`libpython3.7m.so`) and with my version it's linked staticaly (`libpython3.7m.a`,the lib, not the
python exe!). This was not always the case, it has been changed to be embeddable [issue
here](https://github.com/docker-library/python/issues/21).

Ok So it work. the image is 218 MB large. But we have to build python. So can we rely on the
already build binary ?

# Third Attempt : just copy it dude

```Dockerfile
FROM python:3.7-stretch as base

FROM busybox:glibc

COPY --from=builder /usr/local/lib/python3.7 /usr/local/lib/python3.7
COPY --from=builder /usr/local/lib/libpython3.7m.a /usr/local/lib/
COPY --from=builder /usr/local/bin /usr/local/bin

COPY --from=builder /lib/x86_64-linux-gnu/*.so.* /lib/
COPY --from=builder /usr/lib/*.so.* /usr/lib/

ENV PYTHONHOME /usr/local
ENV LD_LIBRARY_PATH /usr/local/lib

CMD ["/usr/local/bin/python3.7"]
```

I added `LD_LIBRARY_PATH` because in busybox, the `.so` file are located in `/lib` and `lddconfig` or `ldconfig` are not available.

We have now a 114MB image. We can do better by only copying the required `*.so` library files !

# Fourth Attempt

Run `python3` in docker :

```bash
╭─remi@laptop test/
╰─$ docker run -ti --rm python:3.7 bash
root@aba27e7938c4:/# ldd /usr/local/bin/python3.7
	linux-vdso.so.1 (0x00007ffc41337000)
	libpython3.7m.so.1.0 => /usr/local/lib/libpython3.7m.so.1.0 (0x00007f8208fc6000)
	libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1 (0x00007f8208d8e000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8208b71000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f820896d000)
	libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f820876a000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f8208466000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f82080c7000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f8209722000)
```
Ok so, python binary need this lib, in order to work. The problem is, it will work, until you
import, let see, pip, which use ssl. And ssl will need `libcrypto` which is not included here...

```Dockerfile
FROM python:3.7-stretch as base

FROM busybox:glibc

COPY --from=builder /usr/local/lib/python3.7 /usr/local/lib/python3.7
COPY --from=builder /usr/local/lib/libpython3.7m.a /usr/local/lib/
COPY --from=builder /usr/local/bin /usr/local/bin

COPY --from=builder /lib/x86_64-linux-gnu/*.so.* /lib/
COPY --from=builder /usr/lib/*.so.* /usr/lib/

CMD ["/usr/local/bin/python3.7"]
```


