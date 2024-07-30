---
title: "Network Policies (Chính Sách Mạng)"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

#### Network Policies (Chính Sách Mạng)

Chuẩn bị môi trường của bạn cho phần này:

```bash wait=30 timeout=600
$ prepare-environment networking/network-policies
```

:::

Mặc định, Kubernetes cho phép tất cả các pod giao tiếp tự do với nhau mà không có hạn chế nào. Network Policies của Kubernetes cho phép bạn xác định và áp dụng các quy tắc về luồng dữ liệu giữa các pod, namespace và các block IP (phạm vi CIDR). Chúng hoạt động như một tường lửa ảo, cho phép bạn phân đoạn và bảo mật cụm của mình bằng cách chỉ định các quy tắc lưu lượng mạng ingress (đầu vào) và egress (đầu ra) dựa trên các tiêu chí khác nhau như nhãn pod, namespace, địa chỉ IP và cổng.

Dưới đây là một ví dụ về một network policy,

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

Phần mô tả network policy chứa các phần quan trọng sau:
- **metadata**: tương tự như các đối tượng Kubernetes khác, cho phép bạn chỉ định tên và namespace cho network policy cụ thể.
- **spec.podSelector**: cho phép lựa chọn các pod cụ thể dựa trên các nhãn của chúng trong namespace mà network policy cụ thể sẽ được áp dụng. Nếu một pod selector trống hoặc matchLabels được chỉ định trong phần mô tả, thì policy sẽ được áp dụng cho tất cả các pod trong namespace.
- **spec.policyTypes**: chỉ định xem policy sẽ được áp dụng cho lưu lượng ingress, lưu lượng egress hoặc cả hai cho các pod được chọn. Nếu bạn không chỉ định trường này, thì hành vi mặc định là áp dụng network policy cho lưu lượng ingress mà không chỉ định phần egress, trừ khi network policy có một phần egress, trong trường hợp đó network policy sẽ được áp dụng cho cả lưu lượng ingress và egress.
- **ingress**: cho phép cấu hình các quy tắc ingress mà chỉ định từ các pod (podSelector), namespace (namespaceSelector) hoặc phạm vi CIDR (ipBlock) mà lưu lượng được phép đến các pod được chọn và cổng hoặc phạm vi cổng nào có thể được sử dụng. Nếu không chỉ định cổng hoặc phạm vi cổng, bất kỳ cổng nào cũng có thể được sử dụng cho việc giao tiếp.

Để biết thêm thông tin về những khả năng được phép hoặc bị hạn chế của network policies trong Kubernetes, xem tài liệu [Kubernetes](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

Ngoài network policies, Amazon VPC CNI ở chế độ IPv4 cung cấp một tính năng mạnh mẽ được gọi là "security group s for Pods". Tính năng này cho phép bạn sử dụng các security group Amazon EC2 để xác định các quy tắc toàn diện điều chỉnh lưu lượng mạng đến và đi từ các pod triển khai trên các nút của bạn. Trong khi có sự trùng lặp về công năng giữa các security group cho các pod và các network policy, nhưng có một số khác biệt quan trọng.
- Các security group cho phép điều khiển lưu lượng ingress và egress đến các phạm vi CIDR, trong khi network policies cho phép điều khiển lưu lượng ingress và egress đến các pod, namespace cũng như các phạm vi CIDR.
- Các security group cho phép điều khiển lưu lượng ingress và egress từ các security group khác, điều này không có sẵn cho các network policies.

Amazon EKS đặc biệt khuyến khích sử dụng network policies kết hợp với các security group để hạn chế giao tiếp mạng giữa các pod, từ đó giảm bề mặt tấn công và giảm thiểu các lỗ hổng tiềm ẩn.