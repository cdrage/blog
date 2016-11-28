---
layout: post
category: docker
title: Kubernetes local development cluster up in one command
date: 2016-03-23 12:45
---

UPDATE 11/28/2016: I'd recommend using `minikube` or `oc cluster up` instead.

If you'd still like access to this script, It's most likely still in my [dotfiles](https://github.com/cdrage/dotfiles)

Want a Kubernetes / k8s local development cluster without having to follow an installation guide, grab the latest containers, etc?

Simply add this to your __.bashrc__ (or __source foobar.sh__ it, whatever you'd like) and run __dev_k8s up__. Need to take it down? __dev_k8s down__.

Oh, the only things you need is  __docker__ and the __kubectl__ binary installed, but I assume you already have that :)


__Usage:__

```
▶ dev_k8s
Kubernetes dev environment

Usage: 
 dev_k8s {up|down|restart|clean|gui|dns|pv}

Methods: 
 up
 down
 restart
 clean - returns k8s env to a clean slate
 gui - ui for k8s at localhost:9090
 dns - deployment of skydns / name resolution
 pv - creates a 20Gb persistent volume named foobar at /tmp/foobar
```

__Script:__

```sh
REMOVED
```
