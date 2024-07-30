---
title: "Switching - Routing - Gateways"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

#### Switching Routing Gateways

Trong phần này, chúng ta sẽ xem xét **Chuyển mạch, Định tuyến và Cổng kết nối** (Switching, Routing, and Gateways)

#### Chuyển mạch (Switching)

- Để xem giao diện trên hệ thống máy chủ

```
$ ip link
```
- Để xem địa chỉ IP của các giao diện.

```
$ ip addr
```

![EKS](/images/0004/0001.png?featherlight=false&width=90pc)

#### Định tuyến (Routing)

- Để xem bảng định tuyến hiện tại trên hệ thống máy chủ.

```
$ route
```
```
$ ip route show
hoặc
$ ip route list
```

- Để thêm các mục vào bảng định tuyến.

```
$ ip route add 192.168.1.0/24 via 192.168.2.1
```

![EKS](/images/0004/0002.png?featherlight=false&width=90pc)

#### Cổng kết nối (Gateways)

- Để thêm một đường đi mặc định.
```
$ ip route add default via 192.168.2.1
```

- Để kiểm tra xem chuyển tiếp IPv4 đã được bật trên máy chủ chưa.
```
$ cat /proc/sys/net/ipv4/ip_forward
0

$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

- Bật chuyển tiếp gói tin cho IPv4.
```
$ cat /etc/sysctl.conf

# Bỏ dấu chú thích trong dòng
net.ipv4.ip_forward=1
```

- Để xem các biến sysctl.
```
$ sysctl -a 
```

- Để tải lại cấu hình sysctl.
```
$ sysctl --system
```

