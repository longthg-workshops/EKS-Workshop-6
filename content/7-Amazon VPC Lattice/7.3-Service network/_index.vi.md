---
title: "Service network"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 7.3 </b>"
---

#### Service network

# Tạo mạng dịch vụ VPC Lattice và liên kết mạng dịch vụ Kubernetes với nó tự động

Bộ điều khiển API Cổng đã được cấu hình để tạo một mạng dịch vụ Lattice VPC và tự động liên kết một cụm VPC Kubernetes với nó. Một mạng dịch vụ là ranh giới logic được sử dụng để tự động triển khai khám phá dịch vụ và kết nối cũng như áp dụng các chính sách truy cập và quan sát cho một bộ sưu tập dịch vụ. Nó cung cấp kết nối giữa các ứng dụng qua các giao thức HTTP, HTTPS và gRPC trong một VPC. Hiện tại, bộ điều khiển chỉ hỗ trợ HTTP và HTTPS.

Trước khi tạo một `Gateway`, chúng ta cần định hình các loại triển khai cân bằng tải có sẵn thông qua mô hình tài nguyên Kubernetes với một [GatewayClass](https://gateway-api.sigs.k8s.io/concepts/api-overview/#gatewayclass). Bộ điều khiển lắng nghe API Cổng phụ thuộc vào một tài nguyên `GatewayClass` liên quan mà người dùng có thể tham chiếu từ `Gateway` của họ:

```file
manifests/modules/networking/vpc-lattice/controller/gatewayclass.yaml
```

Hãy tạo `GatewayClass`:

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/vpc-lattice/controller/gatewayclass.yaml
```

YAML sau sẽ tạo một tài nguyên `Gateway` Kubernetes được liên kết với một **Mạng Dịch Vụ** VPC Lattice.

```file
manifests/modules/networking/vpc-lattice/controller/eks-workshop-gw.yaml
```

Áp dụng nó bằng lệnh sau:

```bash
$ cat ~/environment/eks-workshop/modules/networking/vpc-lattice/controller/eks-workshop-gw.yaml \
  | envsubst | kubectl apply -f -
```

Xác minh rằng `eks-workshop` gateway đã được tạo:

```bash
$ kubectl get gateway -n checkout
NAME                CLASS                ADDRESS   PROGRAMMED   AGE
eks-workshop        amazon-vpc-lattice             True         29s
```

Sau khi gateway được tạo, tìm mạng dịch vụ VPC Lattice. Chờ đến khi trạng thái là `Reconciled` (có thể mất khoảng năm phút).

```bash
$ kubectl describe gateway ${EKS_CLUSTER_NAME} -n checkout
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
status:
   conditions:
      message: 'aws-gateway-arn: arn:aws:vpc-lattice:us-west-2:1234567890:servicenetwork/sn-03015ffef38fdc005'
      reason: Programmed
      status: "True"

$ kubectl wait --for=condition=Programmed gateway/${EKS_CLUSTER_NAME} -n checkout
```

Bây giờ bạn có thể thấy **Mạng Dịch Vụ** liên quan được tạo trong bảng điều khiển VPC dưới các tài nguyên Lattice trong [bảng điều khiển AWS](https://console.aws.amazon.com/vpc/home#ServiceNetworks).

![Mạng Dịch Vụ Checkout](assets/servicenetwork.png)