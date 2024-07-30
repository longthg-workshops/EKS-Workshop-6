---
title: "DNS"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

#### Name Resolution

- Với sự giúp đỡ của lệnh `ping`. Kiểm tra khả năng tiếp cận của Địa chỉ IP trên Mạng.

```bash
$ ping 172.17.0.64
PING 172.17.0.64 (172.17.0.64) 56(84) bytes of data.
64 bytes from 172.17.0.64: icmp_seq=1 ttl=64 time=0.384 ms
64 bytes from 172.17.0.64: icmp_seq=2 ttl=64 time=0.415 ms

```

- Kiểm tra với tên máy chủ của họ

```bash
$ ping web
ping: unknown host web

```

- Thêm mục nhập trong tệp `/etc/hosts` để giải quyết bằng tên máy chủ của họ.

```bash
$ cat >> /etc/hosts
172.17.0.64  web


# Nhấn Ctrl + c để thoát
```

- Nó sẽ tìm trong tệp `/etc/hosts`.

```bash
$ ping web
PING web (172.17.0.64) 56(84) bytes of data.
64 bytes from web (172.17.0.64): icmp_seq=1 ttl=64 time=0.491 ms
64 bytes from web (172.17.0.64): icmp_seq=2 ttl=64 time=0.636 ms

$ ssh web

$ curl http://web
```

#### DNS

- Mỗi máy có tệp cấu hình phân giải DNS tại `/etc/resolv.conf`.

```bash
$ cat /etc/resolv.conf
nameserver 127.0.0.53
options edns0
```

- Để thay đổi thứ tự phân giải DNS, chúng ta cần thay đổi trong tệp `/etc/nsswitch.conf`.

```bash
$ cat /etc/nsswitch.conf

hosts:          files dns
networks:       files

```

- Nếu nó thất bại trong một số điều kiện.

```bash
$ ping wwww.github.com
ping: www.github.com: Temporary failure in name resolution

```

- Thêm nameserver công cộng đã biết vào tệp `/etc/resolv.conf`.

```bash
$ cat /etc/resolv.conf
nameserver   127.0.0.53
nameserver   8.8.8.8
options edns0
``` 
```bash
$ ping www.github.com
PING github.com (140.82.121.3) 56(84) bytes of data.
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=1 ttl=57 time=7.07 ms
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=2 ttl=57 time=5.42 ms

```

#### Domain Names

![EKS](/images/0004/0003.png?featherlight=false&width=90pc)


#### Search Domain

![EKS](/images/0004/0004.png?featherlight=false&width=90pc)


#### Record Types

![EKS](/images/0004/0005.png?featherlight=false&width=90pc)


#### Networking Tools

- Công cụ mạng hữu ích để kiểm tra phân giải DNS.

#### nslookup 

```bash
$ nslookup www.google.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   www.google.com
Address: 172.217.18.4
Name:   www.google.com
```

#### dig

```bash
$ dig www.google.com

; <<>> DiG 9.11.3-1 ...
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8738
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; ANSWER SECTION:
www.google.com.         63      IN      A       216.58.206.4

;; Query time: 6 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
```