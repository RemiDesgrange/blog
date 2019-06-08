---
title: "Idea For A Multitenant Postgres"
date: 2019-06-07T12:43:29+02:00
draft: false
---

I recently migrate a postgres DB from an ubuntu server VM to Azure Postgres. If you use this
DBaaS before, you may have noticed you need to add `@<my_instance_name>` after your login. For
exemple if you create a `test_azure_db`, your url for this postgres instance is
`test_azure_db.postgres.database.azure.com`. And imagine the name of your user is `remi`, to login
you need to do `psql -h test_azure_db.postgres.database.azure.com -U remi@test_azure`.

I don't know what sort of magic Azure is doing, but it gave me an idea. Imagine that you are a 
software editor company. Every company now have a SaaS product. But then you have this client,
that have special needs, so you create a db for him. And then, Sales sell access to the db
(or a replica) "for BI purpose" they say. But, hey, in postgres there are no way to limit the db
a user can see. Plus this client want to be superuser, you know, reasons.

> What if you could have multiple db, completly separate from each other ?

# What postgresql call a cluster

A cluster in postgres is _really_ different from any other software I know. A cluster in postgres is a disk space initialized properly in order for a postgres process to run. see [the doc](https://www.postgresql.org/docs/current/creating-cluster.html) for a better explanation 😅.

# On cluster per user

So now that you know what a cluster is, here is the idea : **you create a new cluster for every clients**
This strategy is not sustainable if you have many clients. But in some case it might work.

I'll explain the why after. Once's you have done the `initdb`, you can run your postgres. You might say :

> But what about ip and port ? only one process can listen to port 5432. We don't want to open 1 
port for each client !!.

And you're right 🤗. You will have to do some proxing. Proxing create a SPOF, but it also allow you to make change in the cluster with minimal downtime (blue/green deployment). And remember that the proxy will be stateless, so you can spawn an instance if it crash without any concern. Also note that the proxy could do TLS endpoint. I said endpint, not termination. It's 2019, please use valid certificate, between you're app and the proxy and between the proxy and the db.
I don't believe in fortress, middle age is over.

## Modus operandi for a new cluster

Every step I mention here should be automatic through a tool like ansible, puppet, salt, \<whatever tool you like\>

* Start provising a big VM. with minimal root fs (10GB ?)
* Create a harddrive (EBS, blockstorage, whatever) for the postgres cluster. Attach it, correctly labeled. For those who don't know, you can label hard drive in linux like this : `mkfs.ext4 -L this_super_client /dev/sda1`. And you can access in `/dev/disk/by-label/this_super_client`
* Mount this harddrive in a known location, like `/mnt/db/this_super_client`
* Init the cluster (`init_db`)
* Create or copy configuration files (`postgresql.conf`, `pg_hba.conf`, etc...)
* Install extension needed (Postgis, etc...)
* Run the postgres process. You should (maybe) use a systemd unit. This postgres will run on a random port
* Check that postgres run (important !)
* Register postgres in the proxy config (that's another topic !)
* enjoy.

## Pro

With this setup you'll have a shared server, with completely separate postgres instance. The advantage I see are numerous :

* You can fine tune the postgres process for the client use case
* You have one partition per client. Which mean that a full partition for one client won't affect others, unless it's the root partition, but c'mon, you monitor it right ?
* You can test easily
* One VM, many clients
* You don't need docker
* easy upgrade
* easy blue/green (feature flag, whatever you call it)
* you can have cluster in multiple versions of postgresql.

## Con

There are drawback too in this approach

* What if a postgres client takes all the ressources ? I mean ok, there are linux cgroups, but it's not enough. I guess this approach does not fit for some heavy load use cases.
* If a client or an attacker got access to the server, you're fucked. But in case you're using one db per client on a shared postgres, that's the same stuff. If the threat model is that high, create separate VM per client.
* You'll provision the replica on the same server than the primary. You'll do it at some point 😉.
* You'll run out of file descriptor.
* If the VM fail, it will take more time to spawn a new VM and re-init everything. But you have a read-replica and a proxy right ?

# A word about the proxy

Maybe we could do it with haproxy or some general TCP proxy. But I can't see how.  Instead you can use a specific proxy like [pgproxy](https://github.com/wgliang/pgproxy) and write our own piece of code to route the query. To store the state of such an app, we could use etcd, consul, zookeeper or any other software that can keep track of config. Maybe hacking pg-pool can be done too.

This proxy could also mirror the trafic. For exemple if you want to see what is happening in a production db. Like we do in web (nginx [mirror](http://nginx.org/en/docs/http/ngx_http_mirror_module.html) stuff)

I have no idea what are the performance of such a proxy. I tend to think that under heavy load Go will perform poorly (GC pause and that sort of things). But for this use case, it can actually be okay.

# A word about docker

In this scenario I don't see any value in docker. Docker will not simply _any_ steps here. Maybe gathering postgres binary will be simpler but the config will be the same. You will just add another layer of complexity if you add docker in this stack, and **no** it is **not** more secure.

# Use cases

* Non-prod env, like dev etc... it's easy to spin up a new instance (no VM start, easily scriptable).
* Prod application that needs special treatment (custom setup, some weird extensions.)

# Conclusion

I have _no clue_ if this idea is realistic or not, even sustainable, technicaly and financialy, but I just wanted to write what I had in mind and expose my thought to the world.