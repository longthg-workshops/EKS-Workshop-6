---
title: "CoreDNS"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 1.3 </b>"
---

#### CoreDNS

#### Điều kiện tiên quyết CoreDNS


Trong phần này, chúng ta sẽ xem xét **CoreDNS**

#### Cài đặt CoreDNS

```
$ wget https://github.com/coredns/coredns/releases/download/v1.7.0/coredns_1.7.0_linux_amd64.tgz
coredns_1.7.0_linux_amd64.tgz

```

#### Giải nén tệp tar

```
$ tar -xzvf coredns_1.7.0_linux_amd64.tgz
coredns
```

#### Chạy tệp thực thi

- Chạy tệp thực thi để khởi động một máy chủ DNS. Theo mặc định, nó lắng nghe trên cổng 53, đó là cổng mặc định cho máy chủ DNS.

```
$ ./coredns

```

#### Cấu hình tệp hosts

- Thêm các mục vào tệp `/etc/hosts`.
- CoreDNS sẽ chọn các IP và tên từ tệp `/etc/hosts` trên máy chủ.

```
$ cat > /etc/hosts
192.168.1.10    web
192.168.1.11    db
192.168.1.15    web-1
192.168.1.16    db-1
192.168.1.21    web-2
192.168.1.22    db-2
```

#### Thêm vào tệp Corefile

```
$ cat > Corefile
. {
	hosts   /etc/hosts
}

```

#### Chạy tệp thực thi

```
$ ./coredns

```


#### Tài liệu tham khảo

- https://github.com/kubernetes/dns/blob/master/docs/specification.md
- https://coredns.io/plugins/kubernetes/
- https://github.com/coredns/coredns/releases
