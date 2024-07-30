---
title: "CNI"
date: "`r Sys.Date()`"
weight: 6
chapter: false
pre: "<b> 1.6 </b>"
---


Trong phần này, chúng ta sẽ xem xét **Giao diện Mạng Container tiên quyết** (**Pre-requisite Container Network Interface(CNI)**).

![EKS](/images/0004/0006.png?featherlight=false&width=90pc)


#### Các Nhà Cung Cấp Plugin Mạng Bên Thứ Ba

- [Weave](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation)
- [Calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)
- [Flannel](https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md)
- [Cilium](https://github.com/cilium/cilium)

#### Để xem các Plugin Mạng CNI

- CNI đi kèm với bộ các plugin mạng được hỗ trợ.

```bash
$ ls /opt/cni/bin/
bridge  dhcp  flannel  host-device  host-local  ipvlan  loopback  macvlan  portmap  ptp  sample  tuning  vlan
```

#### Tài liệu Tham Khảo

- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/