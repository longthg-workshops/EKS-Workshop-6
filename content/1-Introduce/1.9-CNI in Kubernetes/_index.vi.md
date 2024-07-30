---
title: "CNI trong Kubernetes"
date: "`r Sys.Date()`"
weight: 9
chapter: false
pre: "<b> 1.9 </b>"
---

#### CNI trong Kubernetes

Trong phần này, chúng ta sẽ xem xét **Giao diện Mạng Container (CNI) trong Kubernetes**

#### Cấu hình CNI

![EKS](/images/0004/00010.png?featherlight=false&width=90pc)


- Kiểm tra trạng thái của Dịch vụ Kubelet

```shell
$ systemctl status kubelet.service
```

#### Xem Các Tuỳ chọn Kubelet

```shell
$ ps -aux | grep kubelet
```

#### Kiểm tra Các Plugin Hỗ trợ

- Để kiểm tra tất cả các plugin hỗ trợ có sẵn trong thư mục `/opt/cni/bin`.

```shell
$ ls /opt/cni/bin
```

#### Kiểm tra Các Plugin CNI

- Để kiểm tra các plugin cni mà kubelet cần sử dụng.

```shell
ls /etc/cni/net.d
```

#### Định dạng của Tập tin Cấu hình

![EKS](/images/0004/00011.png?featherlight=false&width=90pc)


#### Tài liệu Tham khảo

- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/