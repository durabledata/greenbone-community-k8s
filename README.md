# greenbone-community-k8s

This repository provides a Kubernetes deployment of the Greenbone Community Edition.

Created as a k8s alternative for deploying [Greenbone Community
Containers](https://greenbone.github.io/docs/latest/22.4/container/index.html)

Tested with K3s on Rasberry Pi 4 8GB on Ubuntu and Raspian, modify resources as needed

## Basic Deployment

On a Rasberry Pi 4 with Raspian

Disable IPV6
https://www.howtoraspberry.com/2020/04/disable-ipv6-on-raspberry-pi/

Add add `cgroup_memory=1 cgroup_enable=memory`` to the end of my cat
/boot/cmdline.txt file, reboot

```sh
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none" sh -
kubectl apply -f [`gvm-2204-basic.yaml`](./gvm-2204-basic.yaml)
```

## Remote Scanner Deployment

- Kilo: https://kilo.squat.ai/docs/introduction

Two k3s nodes used for example, aws (ec2) and edge (rpi4, behind nat with no
port mapping) comments indicate which node each command is run on/for

Bring up ec2 instance with EIP, allow tcp/6443 (k8s) and udp/51820 (kilo)
inbound, then:

```sh
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none" sh - # aws no flannel

# enable wg logging if desired
modprobe wireguard && echo module wireguard +p > /sys/kernel/debug/dynamic_debug/control # enable logs
dmesg | grep wireguard # check logs

# install wg tools
apt install wireguard-tools # aws

# update aws k3s node to have awareness of its EIP, if this isn't done, it will tell the other nodes to update to its private IP which they can't route to
mkdir -p /etc/systemd/system/k3s.service.d # aws
vim /etc/systemd/system/k3s.service.d/override.conf # aws
[Service]
ExecStart=
ExecStart=/usr/local/bin/k3s server --flannel-backend=none --tls-san [EIP] --advertise-address [EIP]

# install kilo on cluster
kubectl apply -f https://raw.githubusercontent.com/squat/kilo/main/manifests/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/squat/kilo/main/manifests/kilo-k3s.yaml

kubectl annotate node [k3s node name of aws node] kilo.squat.ai/location="aws" # aws
kubectl annotate node [k3s node name of aws node] kilo.squat.ai/persistent-keepalive="10" # aws
kubectl annotate node [k3s node name of aws node] kilo.squat.ai/force-endpoint="[EIP]:51820" # aws
kubectl label nodes [k3s node name of aws node] type=aws # aws

cat /var/lib/rancher/k3s/server/node-token # aws

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --server https://[EIP]:6443 --token [token]" sh -s - # edge, rpi4 as example

kubectl label nodes [k3s node name of edge node] type=edge # edge
kubectl annotate node [k3s node name of edge node] kilo.squat.ai/location="edge" # edge
kubectl annotate node [k3s node name of edge node] kilo.squat.ai/persistent-keepalive="10" # edge

# deploy greenbone
# update ingress domain name, default example.durabledata.io
kubectl apply -f [`gvm-2204-remote-scanner-kilo.yaml`](./gvm-2204-remote-scanner-kilo.yaml)
# note that greenbone will need to spend some time, over an hour, to pull feeds, populate database, and sync between gvmd and ospd-openvas before functioning properly
```
