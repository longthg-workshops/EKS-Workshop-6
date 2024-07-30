---
title: "AWS Gateway API Controller"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 7.1 </b>"
---

# Giới Thiệu về Gateway API

Gateway API là một dự án mã nguồn mở được quản lý bởi cộng đồng mạng lưới của Kubernetes. Đây là một bộ sưu tập các tài nguyên mô hình mạng ứng dụng trong Kubernetes. Gateway API hỗ trợ các tài nguyên như GatewayClass, Gateway và Route đã được triển khai bởi nhiều nhà cung cấp và được hỗ trợ rộng rãi trong ngành công nghiệp.

Được tạo ra ban đầu như một sự kế thừa Ingress API nổi tiếng, những lợi ích của Gateway API bao gồm (nhưng không giới hạn ở) sự hỗ trợ tường minh cho nhiều giao thức mạng thông dụng, cũng như sự hỗ trợ tích hợp chặt chẽ cho Transport Layer Security (TLS).

Tại AWS, chúng tôi triển khai Gateway API để tích hợp với Amazon VPC Lattice thông qua AWS Gateway API Controller. Khi được cài đặt trong cụm của bạn, bộ điều khiển sẽ theo dõi việc tạo ra các tài nguyên Gateway API như gateway và route và cung cấp các đối tượng Amazon VPC Lattice tương ứng theo bản đồ trong hình ảnh dưới đây. AWS Gateway API Controller là một dự án mã nguồn mở và được hỗ trợ hoàn toàn bởi Amazon.

![Các Đối Tượng Gateway API của Kubernetes và Các Thành Phần VPC Lattice](assets/fundamentals-mapping.png)

Như được hiển thị trong hình ảnh, các vai trò khác nhau liên quan đến các cấp độ kiểm soát khác nhau trong Gateway API của Kubernetes:

- Nhà cung cấp cơ sở hạ tầng: Tạo ra `GatewayClass` của Kubernetes để xác định VPC Lattice là GatewayClass.
- Nhà điều hành cụm: Tạo ra `Gateway` của Kubernetes, nhận thông tin từ VPC Lattice liên quan đến mạng dịch vụ.
- Nhà phát triển ứng dụng: Tạo ra các đối tượng `HTTPRoute` để chỉ định cách lưu lượng được chuyển hướng từ Gateway đến các dịch vụ Kubernetes phía sau.

AWS Gateway API Controller tích hợp với Amazon VPC Lattice và cho phép bạn:

- Xử lý kết nối mạng một cách liền mạch giữa các dịch vụ qua các VPC và tài khoản.
- Phát hiện các dịch vụ này bao gồm nhiều cụm Kubernetes.
- Triển khai một chiến lược bảo vệ đa lớp để bảo vệ việc giao tiếp giữa các dịch vụ đó.
- Quan sát lưu lượng yêu cầu/phản hồi qua các dịch vụ.

Trong chương này, chúng ta sẽ tạo một phiên bản mới của vi dịch vụ `checkout` và sử dụng Amazon VPC Lattice để thực hiện thử nghiệm A/B một cách liền mạch.

