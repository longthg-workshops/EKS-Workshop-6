---
title: "Re-deploy workload"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 5.4 </b>"
---
Để kiểm tra các cập nhật mạng tùy chỉnh mà chúng ta đã thực hiện đến lúc này, hãy cập nhật `deployment checkout` để chạy các pods trên node mới mà chúng ta đã cung cấp ở bước trước đó.

Để thực hiện thay đổi, hãy chạy lệnh sau để sửa đổi `deployment checkout` trong cụm của bạn:

```bash wait=30 timeout=240
$ kubectl apply -k ~/environment/eks-workshop/modules/networking/custom-networking/sampleapp
$ kubectl rollout status deployment/checkout -n checkout --timeout 180s
```

Lệnh này thêm một `nodeSelector` vào `deployment checkout`.

```kustomization
modules/networking/custom-networking/sampleapp/checkout.yaml
Deployment/checkout
```

Hãy xem lại các dịch vụ microservices được triển khai trong không gian tên "checkout".

```bash wait=30
$ kubectl get pods -n checkout -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP             NODE                                         NOMINATED NODE   READINESS GATES
checkout-5fbbc99bb7-brn2m         1/1     Running   0          98s   100.64.10.16   ip-10-42-10-14.us-west-2.compute.internal    <none>           <none>
checkout-redis-6cfd7d8787-8n99n   1/1     Running   0          49m   10.42.12.33    ip-10-42-12-155.us-west-2.compute.internal   <none>           <none>
```

Bạn có thể thấy rằng pod `checkout` được gán một địa chỉ IP từ khối CIDR `100.64.0.0` đã được thêm vào VPC. Các pods chưa được triển khai lại vẫn được gán địa chỉ từ khối CIDR `10.42.0.0`, vì đó là khối CIDR ban đầu duy nhất được liên kết với VPC. Trong ví dụ này, pod `checkout-redis` vẫn có một địa chỉ từ dải này.