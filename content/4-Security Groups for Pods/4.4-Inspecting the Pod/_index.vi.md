---
title: "Inspecting the Pod"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4.4 </b>"
---

#### Inspecting the Pod

Bây giờ khi các pod `catalog` đang chạy và sử dụng cơ sở dữ liệu Amazon RDS của chúng ta một cách thành công, hãy xem xét kỹ hơn để xem các tín hiệu liên quan đến SG cho các Pods.

Điều đầu tiên chúng ta có thể làm là kiểm tra các chú thích của Pod:

```bash
$ kubectl get pod -n catalog -l app.kubernetes.io/component=service -o yaml \
  | yq '.items[0].metadata.annotations'
kubernetes.io/psp: eks.privileged
prometheus.io/path: /metrics
prometheus.io/port: "8080"
prometheus.io/scrape: "true"
vpc.amazonaws.com/pod-eni: '[{"eniId":"eni-0eb4769ea066fa90c","ifAddress":"02:23:a2:af:a2:1f","privateIp":"10.42.10.154","vlanId":2,"subnetCidr":"10.42.10.0/24"}]'
```

Chú thích `vpc.amazonaws.com/pod-eni` hiển thị dữ liệu về các thứ như ENI nhánh đã được sử dụng cho Pod này, địa chỉ IP riêng của nó, v.v...

Các sự kiện Kubernetes cũng sẽ hiển thị bộ điều khiển tài nguyên VPC thực hiện hành động phản hồi các cấu hình mà chúng ta đã thêm:

```bash
$ kubectl get events -n catalog | grep SecurityGroupRequested
5m         Normal    SecurityGroupRequested   pod/catalog-6ccc6b5575-w2fvm    Pod sẽ nhận các Nhóm Bảo mật sau [sg-037ec36e968f1f5e7]
```

Cuối cùng, bạn có thể xem các ENI được quản lý bởi bộ điều khiển tài nguyên VPC trong bảng điều khiển:

[Console AWS EC2](https://console.aws.amazon.com/ec2/home#NIC:v=3;tag:eks:eni:owner=eks-vpc-resource-controller;tag:vpcresources.k8s.aws/trunk-eni-id=:eni)

Điều này sẽ cho phép bạn xem thông tin về ENI nhánh như security group đã chỉ định.
