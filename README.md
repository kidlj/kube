### Introduction

Single master, single etcd, multiple nodes Kubernetes cluster setup, using Vagrant, Virtualbox and kubeadm.

Kubernetes version could be specified.

One-click setup (almost).

### Inventory

Define your nodes in Vagrantfile `servers` block.

### Usage

    $ vagrant up master
    $ vagrant up node1
    $ vagrant up node2
    ...


or 

    $ vagrant up master node1 node2

### Bugs

1. Auto swap off permanently doesn't work, you need to do it manually after each node provisiong, or kubelet won't start after rebooting.
2. Master node ip for node joining is hard-coded, you need to modify it carefully in Vagrantfile if necessary.

### Thanks

This Vagrantfile is heavily base on: https://medium.com/@wso2tech/multi-node-kubernetes-cluster-with-vagrant-virtualbox-and-kubeadm-9d3eaac28b98

Thanks :)
