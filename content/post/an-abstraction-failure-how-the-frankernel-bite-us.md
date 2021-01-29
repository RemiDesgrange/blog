---
title: "An Abstraction Failure. How a Frankernel Bite Us"
date: 2021-01-29T18:00:36+01:00
draft: true
tags: ["kernel", "linux", "linker", "docker", "system"]
category: ["kernel", "linux", "linker", "system", "docker"]
image: "images/lib_not_found.png"
description: "How we got bite by the dynamic linker of an old kernel."
---

Last week a colleague ping us on `#container-support` slack channel to report us a weird bug. A
[container](https://hub.docker.com/r/camptocamp/qgis-server) worked ok on his machine, but not on our
OpenShift cluster. He said that the dynamic linker where not able to resolve the `libQt5Core.so.5`. This was new since it worked perfectly for qgis 3.10 but not in 3.16.
Result of `ldd /usr/local/bin/qgis_mapserv.fcgi|grep Core` where:

```
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
        libQt5Core.so.5 => not found
```

1. What the hell happened on OpenShift that did not happened on the dev host (which is a ubuntu
   host)?
2. Why in hell do we have lots of not found line?


## First Assumption

We first (by we I means colleagues) tought of:

* SELinux/AppArmor getting in the way.
* A dynamic linker cache problem.


## New section

Qgis 3.16 use a new version of Qt. Qt 5.12, which, when compile use a 32 bytes long section
"ABI-tag". This section was not recognized by the dynamic linker which cause failure on our
OpenShift Cluster.

## Reproducing

```
Vagrantfile with centos/7
```

```dockerfile
FROM ubuntu:18.04
RUN apt update && apt install -y build-essential wget
RUN wget https://download.qt.io/official_releases/qt/5.12/5.12.2/qt-opensource-linux-x64-5.12.2.run
RUN chmod +x qt-opensource-linux-x64-5.12.2.run
RUN ./qt-opensource-linux-x64-5.12.2.run

```

```cpp
#include <QObject>
int main(void) {
    QObject  * foo = NULL;
    printf("Hello, World\n");
    return 0;
}
```

```bash
g++ -Wall -fPIC -o hello hello.cpp `pkg-config --libs --cflags Qt5Core`
```

### Quick Fix

Strip the section using the `strip` command: `strip --remove-section=.note.ABI-tag /usr/lib/libQt5Core.so.5.12.8`

So this might bite us since we have a binary compiled with a newer incompatible ABI. But the hack
worked for us.

So the problem was the dynamic linker of our openshift node, _not_ the dynamic linker in the
container. This breaks a bit the promises of "build once, run everywhere" of docker. Our OpenShift cluster run a 3.11 kernel. Which was released in 2013. S👏E👏V👏E👏N Y👏E👏A👏R👏S
 A👏G👏O. And still maintain. This is what I call Frankernel[^2]. A Kernel with so old with so much backported patch that it become dangerous.

## Conclusion

Update your kernel.

[^1](note: https://refspecs.linuxfoundation.org/LSB_1.2.0/gLSB/noteabitag.html)
[^2]: I'm not the first calling a kernel like that I think I first heard of it on twitter by [Jess Frazelle](https://twitter.com/jessfraz) but not sure.
