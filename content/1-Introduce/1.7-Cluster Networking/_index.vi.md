---
title: "Xây dựng mạng Cụm"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b> 1.7 </b>"
---

#### Cluster Networking

Trong phần này, chúng ta sẽ xem xét **Tiền điều kiện của Mạng lưới Cụm**

- Thiết lập tên máy duy nhất.
- Lấy địa chỉ IP của hệ thống (nút chủ và nút công việc).
- Kiểm tra các cổng.

#### Địa chỉ IP và Tên máy chủ

- Để xem tên máy chủ

```
$ hostname 
```

- Để xem địa chỉ IP của hệ thống

```
$ ip a
```


#### Thiết lập tên máy chủ

```
$ hostnamectl set-hostname <tên-máy-chủ>

$ exec bash
```

#### Xem các Cổng Đang Nghe của hệ thống

```
$ netstat -nltp
```



#### Tài liệu Tham khảo

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports
- https://kubernetes.io/docs/concepts/cluster-administration/networking/