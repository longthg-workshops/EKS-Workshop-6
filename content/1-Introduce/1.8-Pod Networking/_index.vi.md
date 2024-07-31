---
title: "Xây dựng mạng Pod"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b> 1.8 </b>"
---

#### Xây dựng mạng Pod

Trong phần này, chúng ta sẽ xem xét **Mạng Pod**

- Để thêm mạng bridge trên mỗi nút

> node01
```
$ ip link add v-net-0 type bridge
```
> node02
```
$ ip link add v-net-0 type bridge
```

> node03
```
$ ip link add v-net-0 type bridge
```

- Hiện tại nó đang tắt, hãy bật nó lên.

> node01
```
$ ip link set dev v-net-0 up
```

> node02
```
$ ip link set dev v-net-0 up
```

> node03
```
$ ip link set dev v-net-0 up
```

- Đặt địa chỉ IP cho giao diện bridge

> node01
```
$ ip addr add 10.244.1.1/24 dev v-net-0
```

> node02
```
$ ip addr add 10.244.2.1/24 dev v-net-0
```

> node03
```
$ ip addr add 10.244.3.1/24 dev v-net-0
```

![EKS](/EKS-Workshop-6/images/0004/0007.png?featherlight=false&width=90pc)

- Kiểm tra khả năng kết nối

```
$ ping 10.244.2.2
Connect: Network is unreachable
```

- Thêm route vào bảng định tuyến
```
$ ip route add 10.244.2.2 via 192.168.1.12
```

> node01
```
$ ip route add 10.244.2.2 via 192.168.1.12

$ ip route add 10.244.3.2 via 192.168.1.13
```

> node02
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.3.2 via 192.168.1.13

```

> node03
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.2.2 via 192.168.1.12
```

- Thêm một mạng lớn duy nhất

![EKS](/EKS-Workshop-6/images/0004/0008.png?featherlight=false&width=90pc)


## Giao diện Mạng Container

![EKS](/EKS-Workshop-6/images/0004/0009.png?featherlight=false&width=90pc)

#### Tài liệu Tham khảo

