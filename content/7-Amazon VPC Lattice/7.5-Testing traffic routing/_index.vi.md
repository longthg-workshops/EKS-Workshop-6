---
title: "Testing traffic routing"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 7.5 </b>"
---

#### Testing traffic routing

Trong thực tế, triển khai canary thường được sử dụng để phát hành một tính năng cho một phần nhỏ người dùng. Trong tình huống này, chúng ta đang định tuyến nhân tạo 75% lưu lượng sang phiên bản mới của dịch vụ thanh toán. Hoàn thành quy trình thanh toán nhiều lần với các đối tượng khác nhau trong giỏ hàng sẽ hiển thị phiên bản 2 của ứng dụng cho người dùng.

Đầu tiên, chúng ta sẽ sử dụng `exec` của Kubernetes để kiểm tra rằng URL dịch vụ Lattice hoạt động từ pod UI. Chúng ta sẽ nhận điều này từ một chú thích trên tài nguyên `HTTPRoute`:

```bash
$ export CHECKOUT_ROUTE_DNS="http://$(kubectl get httproute checkoutroute -n checkout -o json | jq -r '.metadata.annotations["application-networking.k8s.aws/lattice-assigned-domain-name"]')"
$ echo "DNS của Lattice Checkout là $CHECKOUT_ROUTE_DNS"
$ POD_NAME=$(kubectl -n ui get pods -o jsonpath='{.items[0].metadata.name}')
$ kubectl exec $POD_NAME -n ui -- curl -s $CHECKOUT_ROUTE_DNS/health
{"status":"ok","info":{},"error":{},"details":{}}
```

Bây giờ chúng ta phải chỉ định dịch vụ UI đến điểm cuối dịch vụ Lattice của VPC bằng cách vá `ConfigMap` cho thành phần UI:

```kustomization
modules/networking/vpc-lattice/ui/configmap.yaml
ConfigMap/ui
```

Thực hiện thay đổi cấu hình này:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/networking/vpc-lattice/ui/ \
  | envsubst | kubectl apply -f -
```

Hãy đảm bảo rằng các pod UI được khởi động lại và sau đó chuyển tiếp cổng tới phiên bản xem trước của ứng dụng của bạn với Cloud9.

```bash
$ kubectl rollout restart deployment/ui -n ui
$ kubectl rollout status deployment/ui -n ui
```

Hãy thử truy cập ứng dụng của chúng ta bằng trình duyệt. Một dịch vụ loại `LoadBalancer` có tên `ui-nlb` được cung cấp trong namespace `ui` từ đó giao diện người dùng của ứng dụng có thể được truy cập.

```bash
$ kubectl get service -n ui ui-nlb -o jsonpath='{.status.loadBalancer.ingress[*].hostname}{"\n"}'
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

Truy cập này trong trình duyệt của bạn và thử thanh toán nhiều lần (với các mặt hàng khác nhau trong giỏ hàng):

![Ví dụ Thanh toán](assets/examplecheckout.png)

Bạn sẽ nhận thấy rằng quá trình thanh toán bây giờ sử dụng các pod "Lattice checkout" khoảng 75% thời gian:

![Lattice Checkout](assets/latticecheckout.png)