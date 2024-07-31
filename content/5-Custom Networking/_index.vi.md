---
title: "Xây Dựng Mạng Tùy Chỉnh"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 5. </b>"
---

#### Xây Dựng Mạng Tùy Chỉnh

##### **Chuẩn bị môi trường cho phần này:**

```bash wait=30 timeout=300
$ prepare-environment networking/custom-networking
```

Điều này sẽ thực hiện các thay đổi sau đây trong môi trường thí nghiệm của bạn:
- Gắn một dải CIDR phụ vào VPC
- Tạo ba mạng con bổ sung từ dải CIDR phụ

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/tree/VAR::MANIFESTS_REF/manifests/modules/networking/custom-networking/.workshop/terraform).

Mặc định, Amazon VPC CNI sẽ gán cho Pods một địa chỉ IP được chọn từ mạng con chính. Mạng con chính là dải CIDR mà ENI chính được gắn vào, thường là mạng con của nút/ máy chủ.

Nếu dải CIDR của mạng con quá nhỏ, CNI có thể không thể có được đủ địa chỉ IP phụ để gán cho Pods của bạn. Điều này là một thách thức phổ biến đối với các cụm IPv4 EKS.

Mạng tùy chỉnh là một giải pháp cho vấn đề này.

Mạng tùy chỉnh giải quyết vấn đề cạn kiệt địa chỉ IP bằng cách gán các địa chỉ IP Pods từ không gian địa chỉ VPC phụ (CIDR). Mạng tùy chỉnh hỗ trợ tài nguyên tùy chỉnh ENIConfig. ENIConfig bao gồm một phạm vi CIDR mạng con thay thế (tách từ CIDR VPC phụ), cùng với các nhóm bảo mật mà các Pods sẽ thuộc về. Khi mạng tùy chỉnh được kích hoạt, VPC CNI tạo các ENI phụ trong mạng con được xác định trong ENIConfig. CNI gán cho Pods các địa chỉ IP từ một dải CIDR được xác định trong một tài nguyên CRD ENIConfig.

![EKS](/EKS-Workshop-6/images/0004/00016.png?featherlight=false&width=90pc)