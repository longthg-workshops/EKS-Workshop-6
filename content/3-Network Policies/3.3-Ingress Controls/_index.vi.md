---
title: "Cài đặt các Ingress Control"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.3 </b>"
---

#### Cài đặt các Ingress Control

![EKS](/images/0004/00015.png?featherlight=false&width=60pc)

Như đã thể hiện trong sơ đồ kiến trúc, namespace 'catalog' chỉ nhận lưu lượng từ namespace 'ui' và không nhận lưu lượng từ bất kỳ namespace nào khác. Ngoài ra, thành phần cơ sở dữ liệu 'catalog' chỉ có thể nhận lưu lượng từ thành phần dịch vụ 'catalog'.

Chúng ta có thể bắt đầu triển khai các quy tắc mạng trên bằng cách sử dụng một chính sách mạng ingress sẽ kiểm soát lưu lượng đến namespace 'catalog'.

Trước khi áp dụng chính sách, dịch vụ 'catalog' có thể truy cập được từ cả thành phần 'ui':

```bash
$ kubectl exec deployment/ui -n ui -- curl -v catalog.catalog/health --connect-timeout 5
   Trying XXX.XXX.XXX.XXX:80...
* Connected to catalog.catalog (XXX.XXX.XXX.XXX) port 80 (#0)
> GET /catalogue HTTP/1.1
> Host: catalog.catalog
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
...
```

Cũng như từ thành phần 'orders':

```bash
$ kubectl exec deployment/orders -n orders -- curl -v catalog.catalog/health --connect-timeout 5
   Trying XXX.XXX.XXX.XXX:80...
* Connected to catalog.catalog (XXX.XXX.XXX.XXX) port 80 (#0)
> GET /catalogue HTTP/1.1
> Host: catalog.catalog
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
...
```

Bây giờ, chúng ta sẽ định nghĩa một chính sách mạng sẽ cho phép lưu lượng đến thành phần dịch vụ 'catalog' chỉ từ thành phần 'ui':

```file
manifests/modules/networking/network-policies/apply-network-policies/allow-catalog-ingress-webservice.yaml
```

Áp dụng chính sách:

```bash wait=30
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/allow-catalog-ingress-webservice.yaml
```

Bây giờ, chúng ta có thể xác minh chính sách bằng cách xác nhận rằng chúng ta vẫn có thể truy cập vào thành phần 'catalog' từ 'ui':

```bash
$ kubectl exec deployment/ui -n ui -- curl -v catalog.catalog/health --connect-timeout 5
  Trying XXX.XXX.XXX.XXX:80...
* Connected to catalog.catalog (XXX.XXX.XXX.XXX) port 80 (#0)
> GET /catalogue HTTP/1.1
> Host: catalog.catalog
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
...
```

Nhưng không thể từ thành phần 'orders':

```bash expectError=true
$ kubectl exec deployment/orders -n orders -- curl -v catalog.catalog/health --connect-timeout 5
*   Trying XXX.XXX.XXX.XXX:80...
* ipv4 connect timeout after 4999ms, move on!
* Failed to connect to catalog.catalog port 80 after 5001 ms: Timeout was reached
* Closing connection 0
curl: (28) Failed to connect to catalog.catalog port 80 after 5001 ms: Timeout was reached
...
```

Như bạn đã thấy từ các đầu ra trên, chỉ có thành phần 'ui' có thể giao tiếp với thành phần dịch vụ 'catalog', và thành phần dịch vụ 'orders' không thể.

Nhưng điều này vẫn để mở cửa với mọi truy cập vào thành phần cơ sở dữ liệu 'catalog', vì vậy hãy triển khai một chính sách mạng để đảm bảo chỉ có thành phần dịch vụ 'catalog' mới có thể giao tiếp với thành phần cơ sở dữ liệu 'catalog'.

```file
manifests/modules/networking/network-policies/apply-network-policies/allow-catalog-ingress-db.yaml
```

Áp dụng chính sách:

```bash wait=30
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/allow-catalog-ingress-db.yaml
```

Hãy xác minh chính sách mạng bằng cách xác nhận rằng chúng ta không thể kết nối với cơ sở dữ liệu 'catalog' từ thành phần 'orders':

```bash expectError=true
$ kubectl exec deployment/orders -n orders -- curl -v telnet://catalog-mysql.catalog:3306 --connect-timeout 5
*   Trying XXX.XXX.XXX.XXX:3306...
* ipv4 connect timeout after 4999ms, move on!
* Failed to connect to catalog-mysql.catalog port 3306 after 5001 ms: Timeout was reached
* Closing connection 0
curl: (28) Failed to connect to catalog-mysql.catalog port 3306 after 5001 ms: Timeout was reached
command terminated with exit code 28
...
```

Nhưng nếu chúng ta khởi động lại pod 'catalog' thì vẫn có thể kết nối:

```bash
$ kubectl rollout restart deployment/catalog -n catalog
$ kubectl rollout status deployment/catalog -n catalog --timeout=2m
```

Như bạn đã thấy từ các đầu ra trên, chỉ có thành phần dịch vụ 'catalog' mới có thể giao tiếp với thành phần cơ sở dữ liệu 'catalog'.

Bây giờ chúng ta đã triển khai một chính sách ingress hiệu quả cho namespace 'catalog', chúng ta mở rộng cùng một logic cho các namespace và thành phần khác trong ứng dụng mẫu, từ đó giảm thiểu đáng kể bề mặt tấn công cho ứng dụng mẫu và tăng cường bảo mật mạng.