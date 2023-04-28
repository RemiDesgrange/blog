---
title: "My Python Setup"
date: 2019-11-18T20:35:21+01:00
draft: false
tags: ["Python", "Dev", ]
categories: ["English", "Python"]
---

The Python programming language is, I think, a fantastic tool. But there is two *fucking* thing that really makes me sad: packaging and dependency management.

Today, my Linux distribution, Arch, decide that it was OK to migrate python 3.8. And it broke *tons* of shit on my machine. So I took a moment to re-install and harden stuff so that it doesn't happen again.

## Pyenv To Have Multiple Version

I use [Pyenv](https://github.com/pyenv/pyenv) with the plugin to handle [virtualenv](https://github.com/pyenv/pyenv-virtualenv). I use to rely on `pew`. But pew rely on the system Python. So today I couldn't use my virtualenv... `pew` is a great tool but I don't know why it use a fork of `pythonz` to handle version management. Latest version that you can install with `pew` is 3.5. 3.5 [was release in 2015](https://www.python.org/downloads/release/python-350/)


## Pipx for binary

For tools that are written in python but used as a program, I use [pipx](https://github.com/pipxproject/pipx) instead of the distribution packages. Example of packages I install with `pipx`: `ansible`, `docker-compose`, `pgcli`.

## Package management

I'm still using good old `requirements.txt` to handle my dependencies. I tried `pipenv` but was not convinced by it. I think `Pipfile` is a huge step forward for dep mgmt in `python` but I can't get use to it. I didn't tried `poetry` maybe it's good.
If you have opinion on this you can reach me on twitter at [@DesgrangeRemi](https://twitter.com/DesgrangeRemi)





