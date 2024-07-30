---
title: "DNS trong Kubernetes"
date: "`r Sys.Date()`"
weight: 13
chapter: false
pre: "<b> 1.13 </b>"
---


#### DNS trong Kubernetes

  - Dẫn tôi đến [Bài giảng](https://kodekloud.com/topic/dns-in-kubernetes/)

Trong phần này, chúng ta sẽ xem xét **DNS trong Cụm Kubernetes**

#### Bản ghi DNS của Pod

- Quá trình giải quyết DNS sau:

```plaintext
<ĐỊA-CHỈ-IP-POD>.<tên-namespace>.pod.cluster.local
```
> Ví dụ
```plaintext
# Pod nằm trong một namespace mặc định

10-244-1-10.default.pod.cluster.local
```

```plaintext
# Để tạo một namespace
$ kubectl create ns apps

# Để tạo một Pod
$ kubectl run nginx --image=nginx --namespace apps

# Để lấy thông tin bổ sung của Pod trong namespace "apps"
$ kubectl get po -n apps -owide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          99s   10.244.1.3   node01   <none>           <none>

# Để lấy bản ghi dns của Pod nginx từ namespace mặc định
$ kubectl run -it test --image=busybox:1.28 --rm --restart=Never -- nslookup 10-244-1-3.apps.pod.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      10-244-1-3.apps.pod.cluster.local
Address 1: 10.244.1.3
pod "test" deleted

# Truy cập với lệnh curl
$ kubectl run -it nginx-test --image=nginx --rm --restart=Never -- curl -Is http://10-244-1-3.apps.pod.cluster.local
HTTP/1.1 200 OK
Server: nginx/1.19.2

```

#### Bản ghi DNS của Service

- Quá trình giải quyết DNS sau:

```plaintext
<tên-dịch-vụ>.<tên-namespace>.svc.cluster.local
```
> Ví dụ
```plaintext
# Dịch vụ nằm trong một namespace mặc định

web-service.default.svc.cluster.local
```
- Pod, Service nằm trong namespace `apps`

```plaintext
# Tiếp cận Pod nginx
$ kubectl expose pod nginx --name=nginx-service --port 80 --namespace apps
service/nginx-service exposed

# Lấy nginx-service trong namespace "apps"
$ kubectl get svc -n apps
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.96.120.174   <none>        80/TCP    6s

# Để lấy bản ghi dns của nginx-service từ namespace mặc định
$ kubectl run -it test --image=busybox:1.28 --rm --restart=Never -- nslookup nginx-service.apps.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx-service.apps.svc.cluster.local
Address 1: 10.96.120.174 nginx-service.apps.svc.cluster.local
pod "test" deleted

# Truy cập với lệnh curl
$ kubectl run -it nginx-test --image=nginx --rm --restart=Never -- curl -Is http://nginx-service.apps.svc.cluster.local
HTTP/1.1 200 OK
Server: nginx/1.19.2

```

#### Tài liệu Tham khảo

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
- https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/