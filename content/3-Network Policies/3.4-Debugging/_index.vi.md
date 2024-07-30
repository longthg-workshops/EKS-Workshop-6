---
title: "Debugging"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 3.4 </b>"
---

#### Debugging

## Debug Chính sách Mạng bằng Amazon VPC CNI Logs

Đến nay, chúng ta đã có thể áp dụng các chính sách mạng mà không gặp vấn đề hoặc lỗi. Nhưng điều gì sẽ xảy ra nếu có lỗi hoặc vấn đề? Làm thế nào để chúng ta có thể gỡ lỗi những vấn đề này?

Amazon VPC CNI cung cấp các bản ghi nhật ký có thể được sử dụng để gỡ lỗi vấn đề trong quá trình triển khai các chính sách mạng. Ngoài ra, bạn có thể giám sát các bản ghi này thông qua các dịch vụ như Amazon CloudWatch, nơi bạn có thể tận dụng CloudWatch Container Insights để cung cấp thông tin về việc sử dụng của bạn liên quan đến NetworkPolicy.

Bây giờ, hãy thử triển khai một chính sách mạng ingress sẽ hạn chế quyền truy cập vào thành phần dịch vụ 'orders' từ thành phần 'ui' chỉ, tương tự như chúng ta đã làm trước đó với thành phần dịch vụ 'catalog'.

```file
manifests/modules/networking/network-policies/apply-network-policies/allow-order-ingress-fail-debug.yaml
```

Hãy áp dụng chính sách này:

```bash wait=30
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/allow-order-ingress-fail-debug.yaml
```

Và kiểm tra:

```bash expectError=true
$ kubectl exec deployment/ui -n ui -- curl -v orders.orders/orders --connect-timeout 5
*   Trying XXX.XXX.XXX.XXX:80...
* ipv4 connect timeout after 4999ms, move on!
* Failed to connect to orders.orders port 80 after 5000 ms: Timeout was reached
* Closing connection 0
curl: (28) Failed to connect to orders.orders port 80 after 5000 ms: Timeout was reached
...
```

Như bạn có thể thấy từ các đầu ra, có gì đó sai ở đây. Cuộc gọi từ thành phần 'ui' nên đã thành công, nhưng thay vào đó, nó đã thất bại. Để gỡ lỗi điều này, chúng ta có thể tận dụng các nhật ký của agent chính sách mạng để xem vấn đề ở đâu.

Nhật ký của agent chính sách mạng có sẵn trong tập tin `/var/log/aws-routed-eni/network-policy-agent.log` trên mỗi nút worker. Hãy xem liệu có bất kỳ câu lệnh `DENY` nào được đăng nhập trong tập tin đó không:

```bash test=false
$ POD_HOSTIP_1=$(kubectl get po --selector app.kubernetes.io/component=service -n orders -o json | jq -r '.items[0].spec.nodeName')
$ kubectl debug node/$POD_HOSTIP_1 -it --image=ubuntu
# Chạy các lệnh này bên trong pod
$ grep DENY /host/var/log/aws-routed-eni/network-policy-agent.log | tail -5
{"level":"info","timestamp":"2023-11-03T23:02:17.916Z","logger":"ebpf-client","msg":"Flow Info:  ","Src IP":"10.42.190.65","Src Port":55986,"Dest IP":"10.42.117.209","Dest Port":8080,"Proto":"TCP","Verdict":"DENY"}
{"level":"info","timestamp":"2023-11-03T23:02:18.920Z","logger":"ebpf-client","msg":"Flow Info:  ","Src IP":"10.42.190.65","Src Port":55986,"Dest IP":"10.42.117.209","Dest Port":8080,"Proto":"TCP","Verdict":"DENY"}
{"level":"info","timestamp":"2023-11-03T23:02:20.936Z","logger":"ebpf-client","msg":"Flow Info:  ","Src IP":"10.42.190.65","Src Port":55986,"Dest IP":"10.42.117.209","Dest Port":8080,"Proto":"TCP","Verdict":"DENY"}
$ exit
```

Như bạn có thể thấy từ các đầu ra, các cuộc gọi từ thành phần 'ui' đã bị từ chối. Trên phân tích tiếp theo, chúng ta có thể thấy rằng trong chính sách mạng của chúng ta, trong phần ingress, chúng ta chỉ có podSelector và không có namespaceSelector. Khi namespaceSelector rỗng, nó sẽ mặc định là không gian tên của chính sách mạng, đó là 'orders'. Do đó, chính sách sẽ được hiểu là cho phép các pod khớp với nhãn 'app.kubernetes.io/name: ui' từ không gian tên 'orders', dẫn đến việc từ chối lưu lượng từ thành phần 'ui'.

Hãy sửa chính sách mạng và thử lại.

```bash wait=30
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/allow-order-ingress-success-debug.yaml
```

Bây giờ kiểm tra xem 'ui' có thể kết nối không:

```bash
$ kubectl exec deployment/ui -n ui -- curl -v orders.orders/orders --connect-timeout 5
*   Trying XXX.XXX.XXX.XXX:80...
* Connected to orders.orders (172.20.248.36) port 80 (#0)
> GET /orders HTTP/1.1
> Host: orders.orders
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 
...
```

Như bạn có thể thấy từ các đầu ra, bây giờ thành phần 'ui' có thể gọi thành phần dịch vụ 'orders', và vấn đề đã được giải quyết.