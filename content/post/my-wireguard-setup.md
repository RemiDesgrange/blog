---
title: "My WireGuard Setup"
date: 2020-04-22T15:13:58+02:00
draft: false
tags: ["VPN", "WireGuard", "Linux", "System"]
category: ["VPN", "WireGuard", "Linux", "System"]
---

We have a new VPN at [work](https://twitter.com/camptocamp) which works with [WireGuard](https://www.wireguard.com/).

There are a lot of guides on the web like :

* [Wireguard VPN : Typical Setup : The poetry of (in)security](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup)
* [Getting Started with WireGuard](https://miguelmota.com/blog/getting-started-with-wireguard/)

I'm going to present 2 cases:

* Home need: I need a VPN access for my phone and laptop in order to access block stuff in some
  situation. All the traffic goes throught the VPN. It's the simplest case
* Work need: I need to access some ip or ip ranges but not all the traffic goes throught the VPN.

## Home

### Server

* Install of WireGuard. I'm on debian 10
    * create a `/etc/apt/sources.list.d/backport.list`
    ```
    deb http://deb.debian.org/debian buster-backports main
    ```
    * `apt update && apt install wireguard`
    * `reboot`. WireGuard consist of a kernel module which need to be loaded by the kernel. And
      unstable will upgrade you kernel to 4.19. that's why. On ArchLinux for example you don't need
      to reboot.
* Generate the key pair

```
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

* Fill the the file `/etc/wireguard/wg0.conf`. `wg0` is the name of your interface it can be
  everything like `wg10` or even `thatsit10`

```
[Interface]
Address = 10.10.10.1/24
ListenPort = 51820
PrivateKey = <my private key>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client pub key>
AllowedIPs = 10.10.10.2/32
```

`Interface` means that you listen here. And a `Peer` means a distant... Peer at the end of the
tunnel. You have to declare every peer in your VPN. Which mean that if you deploy wireguard as a VPN
concentrator for you company you will need some automation here !


The `PostUp/PostDown` are here to NAT the private address to the address where the traffic goes
(genera lly, the internet). But in case you just want a tunnel to access internal stuff. You don't
need NAT.

* Start the service : `wg-quick up wg0`.
* Once it works, enable the service in systemd, this way the interface get up when you start the
  system: `systemctl enable wg-quik@wg0`

### Client

I'm on Arch, so I just have to do `pacman -S wireguard`. Note that, Since kernel 5.6 (at this time
Arch runs on 5.6) integrate WireGuard. So pacman only download the tooling 😏. Pretty neat.

* Generate a key pair. Same way than on the server :

```
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

Then fill the `/etc/wireguard/wg0.conf`

```
[Interface]
PrivateKey = <Client Private Key>
Address = 10.10.10.2/32
DNS=10.10.10.1 # if you have a dns on the server !! if not, server of the machine will be the same as before.

[Peer]
PublicKey = <Client Public Key>
AllowedIPs = 0.0.0.0/0 # this specific address means all the traffic.
Endpoint=1.2.3.4/32 # The ip address of your server.
```

You can now launch !: `wg-quick up wg0`

And see status : `wg show`.

## Corp

At [Camptocamp](https://www.camptocamp.com/) I needed a [split
tunneling](https://en.wikipedia.org/wiki/Split_tunneling). A VPN where only the traffic for our
clients goes throught the tunnel, not the whole traffic of my machine, it allow us to have a pretty
small machine and bandwidth compared to what it should if all the traffic of all employees goes
throught it.


### Server

The server config is the same. as above

### Client

What we want is to route throught some route or ip range. The list of IP addresses I want is on
github but it does not matter, it could anywhere.

Since `AllowedIPs` cannot be set via shell script in the `wgX.conf`, we will need to use
`PostUp/PreDown` to:

1. Add allowed ips to the peer.
2. add the route in the routing table of the system. Wireguard doesn't do this for you.

Set this script where you want, you'll have to prive a `$VPN_ROUTES` yourself ! This need
modifications.

```shell
#/bin/sh

set -e

UP_OR_DOWN=$1
WG_INTERFACE=$2
WG_PEERPUBKEY=$3
# this script expose a variable $VPN_ROUTES with all the endpoint that need
# to be added to route and AllowedIPs. You can curl your IPAM, maintain a list curl a gist on
# github. It's shell you can do whatever you want 😀 !
VPN_ROUTES="1.2.3.4/24 10.0.0.0/24"
# input like "join_by , A B C " output "A,B,C"
function join_by { local IFS="$1"; shift; echo "$*"; }

function add_routes() {
    for route in $VPN_ROUTES; do
        echo "ip route add $route"
        ip ro add $route dev $WG_INTERFACE
    done
}

function up_peer() {
    wg set $WG_INTERFACE peer $WG_PEERPUBKEY allowed-ips $(join_by , $VPN_ROUTES)
    add_routes
}

```


Then in `/etc/wireguard/wg1.conf`

```
[Peer]
PublicKey = <server pub key>
Endpoint = wg.example.com:12345

[Interface]
PrivateKey = <my private key>
Address = 10.2.3.4
PostUp = /path/to/my/wireguard/setup/route.sh up wg0 "<server pub key>"
```

 > Why does it need to set up the route ? If I set `AllowedIPs` to `0.0.0.0/0` it works !

 It works because `wg-quick` will read the `AllowedIPs` and create the route accordingly. Here we are setting up route via `wg` which does _only_ takes care of 
