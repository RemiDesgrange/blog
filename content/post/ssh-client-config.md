---
title: "Ssh Client Config Tips"
date: 2021-12-08T20:31:02+01:00
draft: false
tags: ["ssh", "sysadmin", "system"]
category: ["ssh", "sysadmin"]
image: ""
description: "Some tips to help you in your day to day life with the ssh command line"
---

This post concat some tips I learned along the way about the SSH which makes me more productive.

## Put generic `Host` config at the end

The config file is read sequentially, and the first rule that matched will be taken into account. If you put your `Host *` [^1] at the top of your `~/.ssh/config` then further rules won't be applied.

[^1]: This `Host *` rule will match all hosts.

```
Host foo
    Username bar
    Hostname foo.baz

# this rule will apply to all connection. and appened to the "toto" host definition.
Host *
    AddKeysToAgent yes
    IdentityFile ~/.ssh/ed25519
```

## Split your config files

By default, ssh client, will read his config from `/etc/ssh/ssh_config` and `~/.ssh/config`. The the later one, I like to add this line of config:

```
Include ~/.ssh/config.d/*
```

This "quite obvious" line will read all ssh config files in `~/.ssh/config.d` folder. This helps me keeping the size of each config file quite limited and understandable by a human.


## Use `ProxyJump`

To use bastion with ssh, there use to be an awkward command to type in the config or in the command line to get there: `ProxyCommand ssh -A ${USER}@bastion.mycorp.com nc the.host.behind.bastion.com 22`. This command is hard to remember. With OpenSSH <version> we now have a `ProxyJump` which simplify everything.

```
Host bastion
    Hostname bastion.mycorp.com
    Username me
    Port 12345

Host foo
    Hostname bar.internal
    Username lol
    ProxyJump bastion
```

Note here I am using another host definition to make things a bit "smoother".

## Decrease the keep alive to make it work on unstable network

```

Host *
    ServerAliveInterval 30
    ServerAliveCountMax 2
```

This way I don't experience any problem with long running script (10/15Minutes), nor with unstable network like cellular connections.

## Share ssh connection for the same host

```
Host *
    ControlMaster auto
    ControlPath /tmp/ssh-%r@%h:%p
    ControlPersist yes
```

This config will speed up multiple connection on the same host. I know some of you use tmux in the server. This will change nothing 😅.

Be aware that this config can beat you hard if something goes wrong and the file in `/tmp` still exists.

## Sum up everything

```
# ~/.ssh/config
Include ~/.ssh/config.d/*

Host my-corp-gateway
    Hostname gw.mycorp.com
    Port 12345
    User me

Host *
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_ed25519
    # some person still lives in 2004
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    # short keep alive
    ServerAliveInterval 30
    ServerAliveCountMax 2
    # share socket for multiple connection
    ControlMaster auto
    ControlPath /tmp/ssh-%r@%h:%p
    ControlPersist yes
```

```
# ~/.ssh/config.d/client1
Host client1-front
    Username apprunner
    Hostname front.client1.com
    ProxyJump my-corp-gateway

```
