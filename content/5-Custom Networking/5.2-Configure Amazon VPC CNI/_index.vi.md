---
title: "Cấu hình Amazon VPC CNI"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 5.2 </b>"
---

#### Cấu hình Amazon VPC CNI

Chúng ta sẽ bắt đầu bằng việc cấu hình Amazon VPC CNI. VPC của chúng ta đã được cấu hình lại với sự thêm vào của một CIDR phụ với phạm vi `100.64.0.0/16`:

```bash wait=30
$ aws ec2 describe-vpcs --vpc-ids $VPC_ID | jq '.Vpcs[0].CidrBlockAssociationSet'
[
  {
    "AssociationId": "vpc-cidr-assoc-0ef3fae4a0abc4a42",
    "CidrBlock": "10.42.0.0/16",
    "CidrBlockState": {
      "State": "associated"
    }
  },
  {
    "AssociationId": "vpc-cidr-assoc-0a6577e1404081aef",
    "CidrBlock": "100.64.0.0/16",
    "CidrBlockState": {
      "State": "associated"
    }
  }
]
```

Điều này có nghĩa là chúng ta hiện có một dải CIDR riêng biệt mà chúng ta có thể sử dụng bổ sung vào dải CIDR mặc định, trong đầu ra trên là `10.42.0.0/16`. Từ dải CIDR mới này, chúng ta đã thêm 3 mạng con mới vào VPC sẽ được sử dụng để chạy các pods của chúng ta:

```bash wait=30
$ echo "Mạng con phụ trong AZ $SUBNET_AZ_1 là $SECONDARY_SUBNET_1"
$ echo "Mạng con phụ trong AZ $SUBNET_AZ_2 là $SECONDARY_SUBNET_2"
$ echo "Mạng con phụ trong AZ $SUBNET_AZ_3 là $SECONDARY_SUBNET_3"
```

Để kích hoạt mạng tùy chỉnh, chúng ta phải thiết lập biến môi trường `AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG` thành *true* trong aws-node DaemonSet.

```bash wait=30
$ kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true
```

Sau đó, chúng ta sẽ tạo một tài nguyên tùy chỉnh `ENIConfig` cho mỗi mạng con mà các pod sẽ được triển khai vào:

```file
manifests/modules/networking/custom-networking/provision/eniconfigs.yaml
```

Hãy áp dụng chúng vào cụm của chúng ta:

```bash wait=30
$ kubectl kustomize ~/environment/eks-workshop/modules/networking/custom-networking/provision \
  | envsubst | kubectl apply -f-
```

Xác nhận rằng các đối tượng `ENIConfig` đã được tạo:

```bash wait=30
$ kubectl get ENIConfigs
```

Cuối cùng, chúng ta sẽ cập nhật aws-node DaemonSet để tự động áp dụng `ENIConfig` cho một Zone khả dụng cho bất kỳ node Amazon EC2 mới nào được tạo trong cụm EKS.

```bash wait=30
$ kubectl set env daemonset aws-node -n kube-system ENI_CONFIG_LABEL_DEF=topology.kubernetes.io/zone
```

