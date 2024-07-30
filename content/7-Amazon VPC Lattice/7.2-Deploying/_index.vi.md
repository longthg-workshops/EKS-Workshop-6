---
title: "Triển khai AWS Gateway API Controller"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 7.2 </b>"
---

#### Triển khai AWS Gateway API Controller

Hãy tuân theo các hướng dẫn sau để tạo một cụm và triển khai AWS Gateway API Controller.

Đầu tiên, cấu hình nhóm bảo mật để nhận lưu lượng từ mạng Lattice trong VPC. Bạn phải thiết lập các nhóm bảo mật để chúng cho phép tất cả các Pod giao tiếp với VPC Lattice để cho phép lưu lượng từ các danh sách tiền tố do VPC Lattice quản lý. Xem [Kiểm soát lưu lượng đến các tài nguyên bằng cách sử dụng các nhóm bảo mật](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) để biết chi tiết. Lattice có sẵn cả danh sách tiền tố IPv4 và IPv6.

```bash
$ CLUSTER_SG=$(aws eks describe-cluster --name $EKS_CLUSTER_NAME --output json| jq -r '.cluster.resourcesVpcConfig.clusterSecurityGroupId')
$ PREFIX_LIST_ID=$(aws ec2 describe-managed-prefix-lists --query "PrefixLists[?PrefixListName=="\'com.amazonaws.$AWS_REGION.vpc-lattice\'"].PrefixListId" | jq -r '.[]')
$ aws ec2 authorize-security-group-ingress --group-id $CLUSTER_SG --ip-permissions "PrefixListIds=[{PrefixListId=${PREFIX_LIST_ID}}],IpProtocol=-1"
$ PREFIX_LIST_ID_IPV6=$(aws ec2 describe-managed-prefix-lists --query "PrefixLists[?PrefixListName=="\'com.amazonaws.$AWS_REGION.ipv6.vpc-lattice\'"].PrefixListId" | jq -r '.[]')
$ aws ec2 authorize-security-group-ingress --group-id $CLUSTER_SG --ip-permissions "PrefixListIds=[{PrefixListId=${PREFIX_LIST_ID_IPV6}}],IpProtocol=-1"

{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0cd3ce1b3cd1c8987",
            "GroupId": "sg-02182c4bf5e0c9756",
            "GroupOwnerId": "475846101383",
            "IsEgress": false,
            "IpProtocol": "-1",
            "FromPort": -1,
            "ToPort": -1,
            "PrefixListId": "pl-0cbf975b710a608ea"
        }
    ]
}
```

Bước này sẽ cài đặt bộ điều khiển và các CRD (Custom Resource Definitions) cần thiết để tương tác với Kubernetes Gateway API.

```bash wait=30
$ aws ecr-public get-login-password --region us-east-1 \
  | helm registry login --username AWS --password-stdin public.ecr.aws
$ helm install gateway-api-controller \
    oci://public.ecr.aws/aws-application-networking-k8s/aws-gateway-controller-chart \
    --version=v1.0.1 \
    --create-namespace \
    --set=aws.region=${AWS_REGION} \
    --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"="$LATTICE_IAM_ROLE" \
    --set=defaultServiceNetwork=${EKS_CLUSTER_NAME} \
    --namespace gateway-api-controller \
    --wait
```

Bộ điều khiển sẽ bắt đầu chạy như một triển khai:

```bash
$ kubectl get deployment -n gateway-api-controller
NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
gateway-api-controller-aws-gateway-controller-chart   2/2     2            2           24s
```