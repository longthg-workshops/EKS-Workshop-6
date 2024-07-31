---
title: "Cài đặt các Egress Control"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Cài đặt các Egress Control


![EKS](/EKS-Workshop-6/images/0004/00015.png?featherlight=false&width=60pc)


Như đã hiển thị trong biểu đồ kiến trúc ở trên, thành phần 'ui' là ứng dụng phía trước. Vì vậy, chúng ta có thể bắt đầu triển khai các điều khiển mạng cho thành phần 'ui' bằng cách xác định một chính sách mạng sẽ chặn tất cả lưu lượng ra từ namespace 'ui'.

```file
manifests/modules/networking/network-policies/apply-network-policies/default-deny.yaml
```

>**Lưu ý**   : Không có namespace nào được chỉ định trong chính sách mạng, vì đó là một chính sách chung có thể được áp dụng cho bất kỳ namespace nào trong cụm của chúng ta.

```bash wait=30
$ kubectl apply -n ui -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/default-deny.yaml 
```

Bây giờ chúng ta hãy thử truy cập thành phần 'catalog' từ thành phần 'ui',

```bash expectError=true
$ kubectl exec deployment/ui -n ui -- curl -s http://catalog.catalog/health --connect-timeout 5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:03 --:--:--     0
curl: (28) Resolving timed out after 5000 milliseconds
lệnh đã kết thúc với mã thoát 28
```

Khi thực thi lệnh curl, đầu ra hiển thị phải có câu sau, cho thấy rằng thành phần 'ui' bây giờ không thể giao tiếp trực tiếp với thành phần 'catalog'.

```
curl: (28) Resolving timed out after 3000 milliseconds
```

Việc triển khai chính sách trên cũng sẽ khiến ứng dụng mẫu không hoạt động đúng cách nữa vì thành phần 'ui' cần truy cập vào dịch vụ 'catalog' và các thành phần dịch vụ khác. Để xác định một chính sách egress hiệu quả cho thành phần 'ui' đòi hỏi hiểu biết về các phụ thuộc mạng cho thành phần đó.

Trong trường hợp của thành phần 'ui', nó cần giao tiếp với tất cả các thành phần dịch vụ khác, chẳng hạn như 'catalog', 'orders', v.v. Ngoài ra, 'ui' cũng sẽ cần có khả năng giao tiếp với các thành phần trong các namespace hệ thống cụm. Ví dụ, để thành phần 'ui' hoạt động, nó cần có khả năng thực hiện tìm kiếm DNS, điều này đòi hỏi nó phải giao tiếp với dịch vụ CoreDNS trong namespace `kube-system``.

Chính sách mạng dưới đây đã được thiết kế dựa trên các yêu cầu trên. Nó có hai phần chính:

* Phần đầu tiên tập trung vào cho phép lưu lượng egress tới tất cả các thành phần dịch vụ như 'catalog', 'orders', v.v. mà không cung cấp quyền truy cập vào các thành phần cơ sở dữ liệu thông qua một sự kết hợp của namespaceSelector, cho phép lưu lượng egress tới bất kỳ namespace nào miễn là nhãn pod phù hợp với "app.kubernetes.io/component: service".
* Phần thứ hai tập trung vào cho phép lưu lượng egress tới tất cả các thành phần trong namespace kube-system, cho phép tìm kiếm DNS và các giao tiếp khác chính với các thành phần trong namespace hệ thống.

```file
manifests/modules/networking/network-policies/apply-network-policies/allow-ui-egress.yaml
```

Hãy áp dụng chính sách bổ sung này:

```bash wait=30
$ kubectl apply -f ~/environment/eks-workshop/modules/networking/network-policies/apply-network-policies/allow-ui-egress.yaml
```

Bây giờ, chúng ta có thể thử nghiệm xem liệu chúng ta có thể kết nối được với dịch vụ 'catalog' không:

```bash
$ kubectl exec deployment/ui -n ui -- curl http://catalog.catalog/health
OK
```

Như bạn có thể thấy từ đầu ra, chúng ta hiện có thể kết nối với dịch vụ 'catalog' nhưng không phải là cơ sở dữ liệu vì nó không có nhãn `app.kubernetes.io/component: service`:

```bash expectError=true
$ kubectl exec deployment/ui -n ui -- curl -v telnet://catalog-mysql.catalog:3306 --connect-timeout 5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
* Không thể kết nối đến catalog-mysql.catalog cổng 3306 sau 5000 ms: Đã đạt đến thời gian chờ
* Đóng kết nối 0
curl: (28) Không thể kết nối đến catalog-mysql.catalog cổng 3306 sau 5000 ms: Đã đạt đến thời gian chờ
```