---
title: "Các Bước Chuẩn Bị"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 2. </b>"
---

#### Các Bước Chuẩn Bị

#### IAM Roles cho Service Accounts

#### TRƯỚC KHI BẮT ĐẦU
Chuẩn bị môi trường cho phần này:

```bash
prepare-environment security/irsa
```

Điều này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:

- Tạo một bảng Amazon DynamoDB
- Tạo một vai trò IAM cho các tải trọng công việc AmazonEKS để truy cập vào bảng DynamoDB
- Cài đặt AWS Load Balancer Controller trong cụm Amazon EKS


Ứng dụng trong các container của Pod có thể sử dụng AWS SDK hoặc AWS CLI để thực hiện các yêu cầu API đến các dịch vụ AWS bằng quyền IAM. Ví dụ, ứng dụng có thể cần tải lên các tệp vào một bucket S3 hoặc truy vấn một bảng DynamoDB. Để làm điều này, các ứng dụng phải ký các yêu cầu API AWS của họ bằng thông tin đăng nhập AWS. **IAM Roles for Service Accounts (IRSA)** cung cấp khả năng quản lý thông tin đăng nhập cho các ứng dụng của bạn, tương tự như cách các Hồ sơ Thể hiện IAM cung cấp thông tin đăng nhập cho các thể hiện Amazon EC2. Thay vì tạo và phân phối thông tin đăng nhập AWS của bạn cho các container hoặc phụ thuộc vào Hồ sơ Thể hiện Amazon EC2 để cấp quyền, bạn phải kết hợp một Vai trò IAM với một Tài khoản Dịch vụ Kubernetes và cấu hình Pods của bạn để sử dụng Tài khoản Dịch vụ đó.

Trong chương này, chúng ta sẽ cấu hình lại một trong các thành phần ứng dụng mẫu để tận dụng một API AWS và cung cấp cho nó xác thực phù hợp.



