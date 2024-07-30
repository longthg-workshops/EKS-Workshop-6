---
title: "Cung cấp node group mới"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.3 </b>"
---

#### Cung cấp node group mới


Tạo một node group được quản lý EKS:

```bash wait=30
$ aws eks create-nodegroup --region $AWS_REGION \
  --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name custom-networking \
  --instance-types t3.medium --node-role $CUSTOM_NETWORKING_NODE_ROLE \
  --subnets $PRIMARY_SUBNET_1 $PRIMARY_SUBNET_2 $PRIMARY_SUBNET_3 \
  --labels type=customnetworking \
  --scaling-config minSize=1,maxSize=1,desiredSize=1
```

Quá trình tạo node group mất vài phút. Bạn có thể chờ quá trình tạo node group hoàn thành bằng cách sử dụng lệnh này:

```bash wait=30 timeout=300
$ aws eks wait nodegroup-active --cluster-name $EKS_CLUSTER_NAME --nodegroup-name custom-networking
```

Khi quá trình này hoàn tất, chúng ta có thể thấy các node mới đã được đăng ký trong cụm EKS:

```bash wait=30
$ kubectl get nodes -L eks.amazonaws.com/nodegroup
NAME                                            STATUS   ROLES    AGE   VERSION               NODEGROUP
ip-10-42-104-242.us-west-2.compute.internal     Ready    <none>   84m   vVAR::KUBERNETES_NODE_VERSION   default
ip-10-42-110-28.us-west-2.compute.internal      Ready    <none>   61s   vVAR::KUBERNETES_NODE_VERSION   custom-networking
ip-10-42-139-60.us-west-2.compute.internal      Ready    <none>   65m   vVAR::KUBERNETES_NODE_VERSION   default
ip-10-42-180-105.us-west-2.compute.internal     Ready    <none>   65m   vVAR::KUBERNETES_NODE_VERSION   default
```

Bạn có thể thấy 1 node mới được cung cấp được gắn nhãn với tên của node group mới.