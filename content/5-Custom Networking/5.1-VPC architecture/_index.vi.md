---
title: "Kiến trúc VPC"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 5.1 </b>"
---

#### Kiến trúc VPC

##### **Khám phá VPC đã được thiết lập**

Chúng ta có thể bắt đầu bằng cách kiểm tra VPC đã được thiết lập. Ví dụ, mô tả VPC:

```bash wait=30
$ aws ec2 describe-vpcs --vpc-ids $VPC_ID
{
    "Vpcs": [
        {
            "CidrBlock": "10.42.0.0/16",
            "DhcpOptionsId": "dopt-0b9864a5c5bbe59bf",
            "State": "available",
            "VpcId": "vpc-0512db3d3af8fa5b0",
            "OwnerId": "188130284088",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-04cf2a625fa24724b",
                    "CidrBlock": "10.42.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                },
                {
                    "AssociationId": "vpc-cidr-assoc-0453603b1ab691914",
                    "CidrBlock": "100.64.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false,
            "Tags": [
                {
                    "Key": "created-by",
                    "Value": "eks-workshop-v2"
                },
                {
                    "Key": "env",
                    "Value": "cluster"
                },
                {
                    "Key": "Name",
                    "Value": "eks-workshop-vpc"
                }
            ]
        }
    ]
}
```

Ở đây, chúng ta thấy có hai dải CIDR được liên kết với VPC:

1. Phạm vi `10.42.0.0/16` là CIDR "chính"
2. Phạm vi `100.64.0.0/16` là CIDR "phụ"

Bạn cũng có thể xem điều này trong bảng điều khiển AWS:

[https://console.aws.amazon.com/vpc/home#vpcs:tag:created-by=eks-workshop-v2](https://console.aws.amazon.com/vpc/home#vpcs:tag:created-by=eks-workshop-v2)

Việc mô tả các subnet liên kết với VPC sẽ hiển thị 9 subnet:

```bash wait=30
$ aws ec2 describe-subnets --filters "Name=tag:created-by,Values=eks-workshop-v2" \
  --query "Subnets[*].CidrBlock"
[
    "10.42.64.0/19",
    "100.64.32.0/19",
    "100.64.0.0/19",
    "100.64.64.0/19",
    "10.42.160.0/19",
    "10.42.0.0/19",
    "10.42.96.0/19",
    "10.42.128.0/19",
    "10.42.32.0/19"
]
```

Các subnet này được chia thành:

- Subnet công cộng: Một cho mỗi khu vực có sẵn sử dụng một khối CIDR từ dải CIDR chính
- Subnet riêng: Một cho mỗi khu vực có sẵn sử dụng một khối CIDR từ dải CIDR chính
- Subnet riêng phụ: Một cho mỗi khu vực có sẵn sử dụng một khối CIDR từ dải CIDR **phụ**

![EKS](/EKS-Workshop-6/images/0004/00017.png?featherlight=false&width=60pc)

Bạn có thể xem các subnet này trong bảng điều khiển AWS:

[https://console.aws.amazon.com/vpc/home#subnets:tag:created-by=eks-workshop-v2;sort=desc:CidrBlock](https://console.aws.amazon.com/vpc/home#subnets:tag:created-by=eks-workshop-v2;sort=desc:CidrBlock)

Hiện tại, các pods của chúng ta đang tận dụng các subnet riêng `10.42.96.0/19`, `10.42.128.0/19` và `10.42.160.0/19`. Trong bài thực hành này, chúng ta sẽ di chuyển chúng để sử dụng địa chỉ IP từ các subnet `100.64`.

