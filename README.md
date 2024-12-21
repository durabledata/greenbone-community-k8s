

# Greenbone Community Edition Kubernetes Deployment

This repository provides a Kubernetes deployment of the Greenbone Community Edition.

Created as a k8s alternative for deploying [Greenbone Community Containers](https://greenbone.github.io/docs/latest/22.4/container/index.html).

Tested with K3s on Raspberry Pi 4 8GB running Ubuntu and Raspbian. Modify resources as needed.

## Basic Deployment

### On a Raspberry Pi 4 with Raspbian

#### Prerequisites

- **Disable IPv6**: Follow the instructions [here](https://www.howtoraspberry.com/2020/04/disable-ipv6-on-raspberry-pi/) to disable IPv6.



##### Edit `/boot/cmdline.txt`

Edit `/boot/cmdline.txt` file and add the line `cgroup_memory=1 cgroup_enable=memory`, then reboot.

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none" sh -
kubectl apply -f [gvm-2204-basic.yaml](gvm-2204-basic.yaml)
```

## Remote Scanner Deployment

### Kilo: [Kilo Documentation](https://kilo.squat.ai/docs/introduction)

Two k3s nodes used for example, aws (ec2) and edge (rpi4, behind NAT with no port mapping). Comments indicate which node each command is run on/for.

#### Bring up EC2 instance
- Assign an EIP.
- Allow TCP/6443 (k8s) and UDP/51820 (kilo) inbound.


