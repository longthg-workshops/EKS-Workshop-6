---
title: "Configuring routes"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 7.4 </b>"
---

Trong phần này, chúng ta sẽ hướng dẫn cách sử dụng Amazon VPC Lattice cho quản lý lưu lượng tiên tiến với tuyến đường có trọng số cho việc triển khai blue/green và canary-style.

Hãy triển khai một phiên bản sửa đổi của dịch vụ nhỏ `checkout` với một tiền tố được thêm vào là *"Lattice"* trong các tùy chọn vận chuyển. Hãy triển khai phiên bản mới này trong một không gian tên mới (`checkoutv2`) bằng cách sử dụng Kustomize.

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/networking/vpc-lattice/abtesting/
$ kubectl rollout status deployment/checkout -n checkoutv2
```

Kể từ này, không gian tên `checkoutv2` bây giờ chứa một phiên bản thứ hai của ứng dụng, trong khi sử dụng cùng một phiên bản `redis` trong không gian tên `checkout`.

```bash
$ kubectl get pods -n checkoutv2
NAME                        READY   STATUS    RESTARTS   AGE
checkout-854cd7cd66-s2blp   1/1     Running   0          26s
```

Bây giờ hãy minh họa cách tuyến đường có trọng số hoạt động bằng cách tạo tài nguyên `HTTPRoute`. Đầu tiên, chúng ta sẽ tạo một `TargetGroupPolicy` cho biết cho Lattice cách thực hiện kiểm tra sức khỏe đúng đắn cho dịch vụ kiểm tra của chúng ta:

```file
manifests/modules/networking/vpc-lattice/target-group-policy/target-group-policy.yaml
```

Áp dụng tài nguyên này:

```bash wait=10
$ kubectl apply -k ~/environment/eks-workshop/modules/networking/vpc-lattice/target-group-policy
```

Bây giờ tạo tuyến đường `HTTPRoute` của Kubernetes để phân phối 75% lưu lượng đến `checkoutv2` và 25% lưu lượng còn lại đến `checkout`:

```file
manifests/modules/networking/vpc-lattice/routes/checkout-route.yaml
```

Áp dụng tài nguyên này:

```bash hook=route
$ cat ~/environment/eks-workshop/modules/networking/vpc-lattice/routes/checkout-route.yaml \
  | envsubst | kubectl apply -f -
```

Việc tạo các tài nguyên liên quan này có thể mất 2-3 phút, chạy lệnh sau để chờ đợi nó hoàn thành:

```bash wait=10 timeout=400
$ kubectl wait -n checkout --timeout=3m \
  --for=jsonpath='{.status.parents[-1:].conditions[-1:].reason}'=ResolvedRefs httproute/checkoutroute
```

Khi hoàn thành, bạn sẽ tìm thấy tên DNS của `HTTPRoute` từ trạng thái `HTTPRoute` (được làm nổi bật ở đây trên dòng `message`):

```bash
$ kubectl describe httproute checkoutroute -n checkout
Name:         checkoutroute
Namespace:    checkout
Labels:       <none>
Annotations:  application-networking.k8s.aws/lattice-assigned-domain-name:
                checkoutroute-checkout-0d8e3f4604a069e36.7d67968.vpc-lattice-svcs.us-east-2.on.aws
API Version:  gateway.networking.k8s.io/v1beta1
Kind:         HTTPRoute
...
Status:
  Parents:
    Conditions:
      Last Transition Time:  2023-06-12T16:42:08Z
      Message:               DNS Name: checkoutroute-checkout-0d8e3f4604a069e36.7d67968.vpc-lattice-svcs.us-east-2.on.aws
      Reason:                ResolvedRefs
      Status:                True
      Type:                  ResolvedRefs
...
```

Bây giờ bạn có thể thấy dịch vụ liên kết được tạo trong [bảng điều khiển VPC Lattice](https://console.aws.amazon.com/vpc/home#Services) dưới các tài nguyên Lattice.
![Dịch vụ CheckoutRoute](assets/checkoutroute.png)

:::tip Lưu lượng hiện được xử lý bởi Amazon VPC Lattice
Amazon VPC Lattice hiện có thể tự động chuyển hướng lưu lượng đến dịch vụ này từ bất kỳ nguồn nào, bao gồm các VPC khác! Bạn cũng có thể tận dụng đầy đủ các [tính năng](https://aws.amazon.com/vpc/lattice/features/) khác của VPC Lattice.
:::