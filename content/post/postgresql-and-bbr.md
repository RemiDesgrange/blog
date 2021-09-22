---
title: "Postgresql and BBR"
date: 2021-09-22T13:27:14+02:00
draft: true
tags: ["postgresql", "database", "tcp"]
category: ["database"]
image: "images/PostgreSQL-Logo.png"
description: "Testing postgresql peformance with bbr(v2)"
---

# Goal

The goal of this little blog post is to validate that BBR (v2) have an impact on postgresql querying performance. Postgresql is kind of sensitive to latency. Though, remember that there is three kinds of lie:

* lie
* ~~Manuel Valls~~ damn lie
* benchmark


## Infrastructure for testing

* 2 virtual machines
    * run on proxmox. With virtio-single-scsi
    * 1 for postgresql client
    * 1 for postgresql server
    * both machines are in the same network
    * both machines are on different physical machine (I own these machines, I know where they are).
* postgresql 13
* ubuntu 21.04


# Setup

I used the command below to setup 2 machines on proxmox. (I could have use terraform etc... but this was just simple enough).

```bash
wget https://cloud-images.ubuntu.com/hirsute/current/hirsute-server-cloudimg-amd64.img
qm create 1000 -name ubuntu-21.04-postgresql-bbr-test -memory 16384 -net0 virtio,bridge=vmbr1 -cores 8 -sockets 1 -cpu cputype=kvm64 -kvm 1 -numa 1
qm importdisk 1000 hirsute-server-cloudimg-amd64.img  local-lvm
qm set 1000 -scsihw virtio-scsi-single -virtio0 local-lvm:vm-1000-disk-0,discard=on,iothread=true
# Ajout lecteur CD pour cloudinit
qm set 1000 -ide0 local-lvm:cloudinit
qm set 1000 -serial0 socket
# Boot sur disque principal
qm set 1000 -boot c -bootdisk virtio0
qm set 1000 -agent 1
qm set 1000 -hotplug disk,network,usb,memory,cpu
qm resize 1000 virtio0 50G
```

When prompt is available: install the package `qemu-guest-agent` and `postgresql`.

```
sudo apt update && sudo apt upgrade -y && sudo apt install -y qemu-guest-agent curl ca-certificates gnupg
# from https://wiki.postgresql.org/wiki/Apt
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update && sudo apt install -y postgresql-13
```

And add this to the `postgresql.conf` on the server side (from pgtune, I did not tried to be clever on this).

```
max_connections = 100
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 5242kB
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
max_parallel_maintenance_workers = 4
listen_addresses = '*'
```

Then use bbr on both server and client:

Put this values in `/etc/sysctl.conf`
```
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

Then apply

```
sudo sysctl -p
```

## setup db

```
sudo -u postgres createdb test
sudo -u postgres pgbench -i test
```

Result bbr
```
PGPASSWORD=postgres pgbench -c 90 -h 192.168.141.211 -j 8 -U postgres -T 30 test
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 1
duration: 20 s
number of transactions actually processed: 325709
latency average = 0.614 ms
tps = 16284.277453 (including connections establishing)
tps = 16300.051127 (excluding connections establishing)
```

Result cubic
```
PGPASSWORD=postgres pgbench -c 90 -h 192.168.141.211 -j 8 -U postgres -T 30 test
```


Resources:
* https://www.linuxbabe.com/ubuntu/enable-google-tcp-bbr-ubuntu
* https://dropbox.tech/infrastructure/evaluating-bbrv2-on-the-dropbox-edge-network

