---
title: "CNI Weave"
date: "`r Sys.Date()`"
weight: 10
chapter: false
pre: "<b> 1.10 </b>"
---

#### CNI weave

Trong phần này, chúng ta sẽ tìm hiểu về "CNI Weave trong Cụm Kubernetes"

#### Triển khai Weave

- Cài đặt [weave net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) vào cụm Kubernetes bằng một lệnh duy nhất.

```shell
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

#### Weave Peers

```shell
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS             RESTARTS   AGE
coredns-66bff467f8-894jf                  1/1     Running            0          52m
coredns-66bff467f8-nck5f                  1/1     Running            0          52m
etcd-controlplane                         1/1     Running            0          52m
kube-apiserver-controlplane               1/1     Running            0          52m
kube-controller-manager-controlplane      1/1     Running            0          52m
kube-keepalived-vip-mbr7d                 1/1     Running            0          52m
kube-proxy-p2mld                          1/1     Running            0          52m
kube-proxy-vjcwp                          1/1     Running            0          52m
kube-scheduler-controlplane               1/1     Running            0          52m
weave-net-jgr8x                           2/2     Running            0          45m
weave-net-tb9tz                           2/2     Running            0          45m
```

#### Xem nhật ký của Pod Weave

```shell
$ kubectl logs weave-net-tb9tz weave -n kube-system 
```

#### Xem đường đi mạng mặc định trong Pod

```shell
$ kubectl run test --image=busybox --command -- sleep 4500
pod/test created

$ kubectl exec test -- ip route
default via 10.244.1.1 dev eth0
```

#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/cluster-administration/addons/
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/