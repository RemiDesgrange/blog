---
title: "Fast DB Branching With BTRFS Subvolumes as Docker Volumes"
date: 2025-05-08T00:00:00+01:00
draft: true
tags: ["btrfs", "docker", "filesystem"]
category: ["docker", "container", "btrfs"]
image: ""
description: ""
lang: "en"
---

I encounter a "rather" PITA problem that I've had for years: How do I load production DB into a test/integration/you-name-it environment without waiting half a day for it.

Ok maybe you have a nice set of fixtures for your app, which eliminiates this need. Still, this need is pretty common.

I previsouly stumbled upon [this doc](https://docs.docker.com/storage/storagedriver/btrfs-driver/) of BTRFS/ZFS as a driver for docker storage.
And then I remembered this filesystems had CoW[^1] and snapshot capabilities.

[^1]: Copy-On-Write.


So I thought:

> Ok why not replace the OverlayFS with BTRFS and use the volume/snapshot capabilities to have a "primary" DB, which would be loaded every day,
then snapshot this volume everytime I need the DB for testing purpose, then trash the volume when the experiment/test is over


## First Try: TL; DR

At first I did not read extensively the [docker doc](https://docs.docker.com/storage/storagedriver/btrfs-driver/). So I created an empty disk, which I attached to my VM.

I then switched the `storage-driver` in `/etc/docker/daemon.json`.

I then realised that the list of features for docker named volume is pretty low. The only thing you can do is pretty much create/list/delete a volume.

So I rolled back the changes and went back to the OverlayFS for storing images. Plus after some month, even with `docker system prune` and all that stuff, docker doesn't seem to reclaim (or delete) old images subvolumes[^2]

[^2]: [Each layer is a subvolume](https://docs.docker.com/engine/storage/drivers/btrfs-driver/#image-and-container-layers-on-disk)

## Second Try: It Works

So I got back to docker bind mount, which is far from ideal, but hey, it works 🤷‍♂️.

So given a BTRFS disk (created with `mkfs.btrfs /dev/vdb`), mounted at `/mnt/data`, and a subvolume `btrfs subvolume create /mnt/data/mariadb-primary`. I created a simple `docker-compose.yml` for the db

```
services:
  db:
    image: mariadb
    ports:
      - "3306:3306"
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: mydb
    volumes:
      - "/mnt/data/mariadb-primary:/var/lib/mysql"
```

`mysql -h localhost -p 3306 -p < dump.sql`

Once the dump has been loaded, you can `docker compose down`.

And then launch a new DB
```
btrfs subvolume snapshot /mnt/data/mariadb-primary /mnt/data/test-db
docker run -d --rm -v /mnt/data/test-db:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=root -e MARIADB_DATABASE=mydb --name my-test-db mariadb
docker stop my-test-db
btrfs subvolume delete /mnt/data/test-db
```

I know there are plenty of solution like PlanetScale. But I was looking at something that work in-house.

Big shoutout to [Nathan](https://www.linkedin.com/in/nr-memento-cloud/) for helping me understand how this file system shenigan works.
