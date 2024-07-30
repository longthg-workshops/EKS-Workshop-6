---
title: "Security Groups cho các Pod"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4. </b>"
---

#### Chuẩn bị môi trường cho phần này:

```bash timeout=900 wait=30
$ prepare-environment networking/securitygroups-for-pods
```

Thao tác này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:
- Tạo một trường hợp Amazon Relational Database Service
- Tạo một Security Group Amazon EC2 để cho phép truy cập vào trường hợp RDS

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/tree/VAR::MANIFESTS_REF/manifests/modules/networking/securitygroups-for-pods/.workshop/terraform).


Nhóm bảo mật (security group), hoạt động như các tường lửa mạng cấp độ trường hợp, là một trong những nền móng quan trọng và thường được sử dụng trong bất kỳ triển khai đám mây AWS nào. Các ứng dụng container thường cần truy cập vào các dịch vụ khác đang chạy trong cụm cũng như các dịch vụ AWS bên ngoài, chẳng hạn như Amazon Relational Database Service (Amazon RDS) hoặc Amazon ElastiCache. Trên AWS, việc kiểm soát truy cập mạng cấp độ giữa các dịch vụ thường được thực hiện thông qua các nhóm bảo mật EC2.

Mặc định, Amazon VPC CNI sẽ sử dụng các nhóm bảo mật được liên kết với ENI chính trên node. Cụ thể hơn, mỗi ENI được liên kết với trường hợp sẽ có các Nhóm Bảo Mật EC2 giống nhau. Do đó, mỗi Pod trên một node sẽ chia sẻ các nhóm bảo mật giống như node mà nó chạy trên. Nhóm bảo mật cho các pod giúp dễ dàng đạt được sự tuân thủ bảo mật mạng bằng cách chạy các ứng dụng có yêu cầu bảo mật mạng khác nhau trên tài nguyên tính toán chia sẻ. Các quy tắc bảo mật mạng liên kết từ pod đến pod và từ pod đến lưu lượng dịch vụ AWS bên ngoài có thể được xác định ở một nơi duy nhất với các nhóm bảo mật EC2, và áp dụng cho các ứng dụng với các API native của Kubernetes. Sau khi áp dụng các nhóm bảo mật ở mức pod, kiến ​​trúc ứng dụng và nhóm node của bạn có thể được đơn giản hóa như được hiển thị (dưới đây).

Bạn có thể kích hoạt nhóm bảo mật cho Pods bằng cách thiết lập `ENABLE_POD_ENI=true` cho VPC CNI. Khi bạn kích hoạt Pod ENI, [VPC Resource Controller](https://github.com/aws/amazon-vpc-resource-controller-k8s) chạy trên mặt trái kiểm soát (được quản lý bởi EKS) tạo và đính kèm một giao diện cụm gọi là "aws-k8s-trunk-eni" vào node. Giao diện cụm hoạt động như một giao diện mạng tiêu chuẩn được đính kèm vào trường hợp.

Bộ điều khiển cũng tạo ra các giao diện nhánh có tên "aws-k8s-branch-eni" và liên kết chúng với giao diện cụm. Pods được gán một nhóm bảo mật bằng cách sử dụng [SecurityGroupPolicy](https://github.com/aws/amazon-vpc-resource-controller-k8s/blob/master/config/crd/bases/vpcresources.k8s.aws_securitygrouppolicies.yaml) tài nguyên tùy chỉnh và được liên kết với một giao diện nhánh. Vì nhóm bảo mật được chỉ định với các giao diện mạng, chúng ta hiện có thể lên lịch cho các Pods yêu cầu các nhóm bảo mật cụ thể trên các giao diện mạng bổ sung này. Xem [hướng dẫn thực hành tốt nhất EKS](https://aws.github.io/aws-eks-best-practices/networking/sgpp/) để biết các khuyến nghị và [Hướng dẫn Sử dụng EKS](https://docs.aws.amazon.com/eks/latest/userguide/security-groups-for-pods.html), cho các yêu cầu triển khai trước.

![Nhận thức](/img/networking/securitygroupsperpod/overview.png)

Trong chương này, chúng ta sẽ cấu hình lại một trong các thành phần ứng dụng mẫu để tận dụng nhóm bảo mật cho Pods để truy cập vào một tài nguyên mạng bên ngoài.