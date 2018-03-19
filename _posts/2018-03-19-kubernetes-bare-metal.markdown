---
layout: post
category: kubernetes
title: "Super-simple bare-metal Kubernetes deployment"
date: 2018-03-19 13:17
---

Steps:
 1. Deploy using [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) (hint: There's some awesome [Ansible playbooks](https://github.com/kairen/kubeadm-ansible) to use if you're running CentOS / Ubuntu)
 2. Setup your Ingress controller, the best-one IMO is [Traefik](https://docs.traefik.io/user-guide/kubernetes/)
 3. Optionally: Create your dynamic volume provisioner, I setup a [Docker NFS Server](https://github.com/cdrage/dockerfiles/tree/master/nfs-server) and used [nfs-client](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client) from the Kubernetes external-storage incubator project.

TODO:

I'll eventually write a tutorial here. These are just my simply notes :)
