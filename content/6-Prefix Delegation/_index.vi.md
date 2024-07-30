---
title: "Ủy Quyền Tiền Tố"
date: "`r Sys.Date()`"
weight: 6
chapter: false
pre: "<b> 6. </b>"
---

#### Ủy Quyền Tiền Tố

Chuẩn bị môi trường của bạn cho phần này:

```bash timeout=300 wait=30
$ prepare-environment networking/prefix
```

Amazon VPC CNI gán các tiền tố mạng cho [giao diện mạng Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-prefix-eni.html) để tăng số lượng địa chỉ IP có sẵn cho các nút và tăng mật độ pod trên mỗi nút. Bạn có thể cấu hình phiên bản 1.9.0 hoặc mới hơn của tiện ích bổ sung Amazon VPC CNI để gán một tiền tố thay vì gán các địa chỉ IP phụ cụ thể cho các giao diện mạng.

Với chế độ gán tiền tố, số lượng Elastic Network Interface (ENI) tối đa trên mỗi loại instance vẫn giữ nguyên, nhưng bây giờ bạn có thể cấu hình Amazon VPC CNI để gán tiền tố địa chỉ IPv4 /28 (16 địa chỉ IP), thay vì gán các địa chỉ IPv4 cụ thể cho các khe trên các giao diện mạng trên loại instance EC2 nitro. Khi `ENABLE_PREFIX_DELEGATION` được đặt thành true, VPC CNI sẽ phân bổ một địa chỉ IP cho một Pod từ tiền tố được gán cho một ENI.

![EKS](/images/0004/00018.png?featherlight=false&width=90pc)

Trong quá trình khởi tạo nút worker, VPC CNI gán một hoặc nhiều tiền tố cho ENI chính. CNI tiền gán trước một tiền tố để khởi động pod nhanh hơn bằng cách duy trì một warm pool (vùng chứa các máy EC2 trong trạng thái sẵn sàng).

Khi có nhiều Pods được lên lịch, các tiền tố bổ sung sẽ được yêu cầu cho các ENI hiện có. Đầu tiên, VPC CNI cố gắng phân bổ một tiền tố mới cho một ENI hiện có. Nếu ENI đã đầy, VPC CNI sẽ cố gắng phân bổ một ENI mới cho nút. ENI mới sẽ được đính kèm cho đến khi đạt đến giới hạn ENI tối đa (được xác định bởi loại instance). Khi một ENI mới được đính kèm, ipamd sẽ phân bổ một hoặc nhiều tiền tố cần thiết để duy trì các cài đặt bể ấm.

![EKS](/images/0004/00019.jpeg?featherlight=false&width=90pc)

Vui lòng truy cập [hướng dẫn thực hành tốt nhất cho EKS](https://aws.github.io/aws-eks-best-practices/networking/prefix-mode/) để xem danh sách các khuyến nghị về việc sử dụng VPC CNI ở chế độ tiền tố.
