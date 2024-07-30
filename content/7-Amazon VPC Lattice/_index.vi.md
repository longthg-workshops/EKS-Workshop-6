---
title: "Amazon VPC Lattice"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b> 7. </b>"
---

#### Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment networking/vpc-lattice
```

Việc này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:
- Tạo một vai trò IAM cho bộ điều khiển API Gateway để truy cập vào các API AWS
- Cài đặt AWS Load Balancer Controller trong cụm Amazon EKS

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/tree/VAR::MANIFESTS_REF/manifests/modules/networking/vpc-lattice/.workshop/terraform).

[Amazon VPC Lattice](https://aws.amazon.com/vpc/lattice/) là một dịch vụ mạng tầng ứng dụng cung cấp cách liên tục để kết nối, bảo mật và giám sát giao tiếp giữa dịch vụ mà không cần có kiến thức về mạng trước. Với VPC Lattice, bạn có thể cấu hình quyền truy cập mạng, quản lý lưu lượng và giám sát mạng để cho phép giao tiếp giữa dịch vụ một cách đồng nhất trên các VPC và tài khoản, không phụ thuộc vào loại tính toán cơ bản bao gồm các cụm Kubernetes.

Amazon VPC Lattice giải quyết các thao tác mạng phổ biến như phát hiện các thành phần, định tuyến lưu lượng giữa các công việc cá nhân và ủy quyền truy cập, loại bỏ nhu cầu cho các nhà phát triển phải làm điều này thông qua phần mềm hoặc mã bổ sung. Chỉ trong vài cú nhấp chuột hoặc cuộc gọi API, các nhà phát triển có thể cấu hình các chính sách để xác định cách ứng dụng của họ nên giao tiếp mà không cần kiến thức về mạng trước.

Các lợi ích chính của việc sử dụng Lattice là:

* **Tăng năng suất phát triển**: Lattice tăng năng suất phát triển bằng cách cho phép người ta tập trung vào việc xây dựng các tính năng quan trọng cho doanh nghiệp, trong khi nó xử lý các thách thức về mạng, bảo mật và quan sát một cách đồng nhất trên tất cả các nền tảng tính toán
* **Cải thiện tư duy bảo mật**: Lattice cho phép các nhà phát triển dễ dàng xác thực và bảo mật giao tiếp giữa các ứng dụng, mà không phải chịu chi phí vận hành của các cơ chế hiện có (ví dụ: quản lý chứng chỉ). Với các chính sách truy cập Lattice, các nhà phát triển và quản trị viên đám mây có thể áp dụng kiểm soát truy cập cụ thể. Lattice cũng có thể áp dụng mã hóa cho lưu lượng đang truyền, tăng thêm tư duy bảo mật
* **Cải thiện tính mở rộng và tính ổn định của ứng dụng**: Lattice giúp dễ dàng tạo ra một mạng lưới các ứng dụng triển khai với định tuyến, xác thực, ủy quyền, giám sát và nhiều tính năng khác. Lattice cung cấp tất cả các lợi ích này mà không tăng thêm tài nguyên cho các công việc và có thể hỗ trợ triển khai quy mô lớn và nhiều yêu cầu mỗi giây mà không tăng thêm độ trễ đáng kể.
* **Tính linh hoạt trong triển khai với cơ sở hạ tầng đa dạng**: Lattice cung cấp các tính năng đồng nhất trên tất cả các dịch vụ tính toán - EC2, ECS, EKS, Lambda, và có thể bao gồm các dịch vụ tại chỗ, cho phép tổ chức linh hoạt chọn lựa cơ sở hạ tầng tính toán tối ưu cho tình huống sử dụng của họ.

[Các thành phần](https://docs.aws.amazon.com/vpc-lattice/latest/ug/what-is-vpc-service-network.html#vpc-service-network-components-overview) của Amazon VPC Lattices bao gồm:

* **Mạng dịch vụ**:
Một nhóm logic có thể chia sẻ và quản lý chứa Dịch vụ và Chính sách.

* **Dịch vụ**:
Đại diện cho một Đơn vị Ứng dụng với một tên DNS và có thể mở rộng trên tất cả các tính toán - máy ảo, container, không máy chủ. Nó bao gồm Người nghe, Nhóm Mục tiêu, Mục tiêu.

* **Thư mục dịch vụ**:
Một đăng ký trong một tài khoản AWS chứa một cái nhìn toàn cầu về Dịch vụ theo phiên bản và tên DNS của chúng.

* **Chính sách bảo mật**:
Một chính sách mô tả quyền truy cập của các Dịch vụ được phép giao tiếp như thế nào. Chúng có thể được xác định tại cấp độ Dịch vụ hoặc cấp độ Mạng dịch vụ.

![Các thành phần của Amazon VPC Lattice](assets/vpc_lattice_building_blocks.png)