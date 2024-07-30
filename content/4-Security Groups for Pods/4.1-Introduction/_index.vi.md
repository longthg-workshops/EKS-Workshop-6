---
title: "Giới thiệu"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

#### Sử dụng Amazon EKS cho ứng dụng Catalog của bạn

Trong kiến trúc của chúng ta, thành phần `catalog` sử dụng một cơ sở dữ liệu MySQL làm backend lưu trữ. Cách mà API catalog hiện đang triển khai sử dụng một cơ sở dữ liệu được triển khai dưới dạng một Pod trong cụm EKS.

Bạn có thể thấy điều này bằng cách chạy lệnh sau:

```bash
$ kubectl -n catalog get pod 
NAME                              READY   STATUS    RESTARTS        AGE
catalog-5d7fc9d8f-xm4hs             1/1     Running   0               14m
catalog-mysql-0                     1/1     Running   0               14m
```

Trong trường hợp trên, Pod `catalog-mysql-0` là một Pod MySQL. Chúng ta có thể xác minh ứng dụng `catalog` của chúng ta đang sử dụng pod này bằng cách kiểm tra môi trường của nó:

```bash
$ kubectl -n catalog exec deployment/catalog -- env \
  | grep DB_ENDPOINT
DB_ENDPOINT=catalog-mysql:3306
```

Chúng ta muốn di chuyển ứng dụng của mình để sử dụng dịch vụ Amazon RDS được quản lý đầy đủ để tận dụng hoàn toàn quy mô và đáng tin cậy mà nó cung cấp.