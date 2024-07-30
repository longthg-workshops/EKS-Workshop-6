---
title: "Amazon VPC CNI"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 6.1 </b>"
---

Trước khi chúng ta bắt đầu, hãy xác nhận rằng VPC CNI đã được cài đặt và đang chạy:

```bash
$ kubectl get pods --selector=k8s-app=aws-node -n kube-system
NAME             READY   STATUS    RESTARTS   AGE
aws-node-btst2   1/1     Running   0          107m
aws-node-xwkf2   1/1     Running   0          107m
aws-node-zd5rg   1/1     Running   0          107m
```

Xác nhận phiên bản CNI bằng 1.9.0 hoặc mới hơn:

```bash
$ kubectl describe daemonset aws-node --namespace kube-system | grep Image | cut -d "/" -f 2
amazon-k8s-cni-init:v1.12.0-eksbuild.1
amazon-k8s-cni:v1.12.0-eksbuild.1
```

Xác nhận xem VPC CNI đã được cấu hình để chạy ở chế độ tiền tố không. Giá trị `ENABLE_PREFIX_DELEGATION` phải được đặt thành "true":

```bash
$ kubectl get ds aws-node -o yaml -n kube-system | yq '.spec.template.spec.containers[].env'
[...]
- name: ENABLE_PREFIX_DELEGATION
  value: "true"
[...]
```

Do việc ủy quyền tiền tố được bật (điều này đã được thực hiện khi tạo cụm cho hội thảo này), chúng ta nên thấy tiền tố được gán cho các giao diện mạng của các nút worker. Bạn nên thấy đầu ra tương tự như dưới đây.

```bash
$ aws ec2 describe-instances --filters "Name=tag-key,Values=eks:cluster-name" \
  "Name=tag-value,Values=${EKS_CLUSTER_NAME}" \
  --query 'Reservations[*].Instances[].{InstanceId: InstanceId, Prefixes: NetworkInterfaces[].Ipv4Prefixes[]}'

 [
    {
        "InstanceId": "i-0d1f7c060cf3ad0f4",
        "Prefixes": [
            {
                "Ipv4Prefix": "10.42.10.192/28"
            },
            {
                "Ipv4Prefix": "10.42.10.80/28"
            }
        ]
    },
    {
        "InstanceId": "i-0b47d3070af05c8b1",
        "Prefixes": [
            {
                "Ipv4Prefix": "10.42.10.16/28"
            },
            {
                "Ipv4Prefix": "10.42.10.160/28"
            }
        ]
    },
    {
        "InstanceId": "i-081b2a4d4e5f27991",
        "Prefixes": [
            {
                "Ipv4Prefix": "10.42.12.128/28"
            },
            {
                "Ipv4Prefix": "10.42.12.208/28"
            }
        ]
    }
]
```

Như chúng ta có thể thấy, hiện có các tiền tố được gán cho các nút worker của chúng ta. Việc ủy quyền tiền tố đã thành công!