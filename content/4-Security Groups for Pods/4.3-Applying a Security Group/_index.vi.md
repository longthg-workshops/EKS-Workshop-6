---
title: "Áp dụng Security Group"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 4.3 </b>"
---

#### Áp dụng Security Group

## Kết nối Pod Catalog với RDS Instance trên AWS

Để Pod `catalog` của chúng ta kết nối thành công với RDS instance, chúng ta cần sử dụng security group đúng. Tuy nhiên, việc áp dụng security group này trực tiếp vào các node worker của EKS sẽ dẫn đến tất cả các công việc trong cluster của chúng ta đều có quyền truy cập mạng vào RDS instance. Thay vào đó, chúng ta sẽ áp dụng Security Group vào các Pod `catalog` để cho phép chúng truy cập vào RDS instance.

Một security group cho phép truy cập vào cơ sở dữ liệu RDS đã được thiết lập sẵn cho bạn và bạn có thể xem như sau:

```bash
$ export CATALOG_SG_ID=$(aws ec2 describe-security-groups \
    --filters Name=vpc-id,Values=$VPC_ID Name=group-name,Values=$EKS_CLUSTER_NAME-catalog \
    --query "SecurityGroups[0].GroupId" --output text)

$ aws ec2 describe-security-groups \
  --group-ids $CATALOG_SG_ID | jq '.'
```
```yaml
{
  "SecurityGroups": [
    {
      "Description": "Applied to catalog application pods",
      "GroupName": "eks-workshop-catalog",
      "IpPermissions": [
        {
          "FromPort": 8080,
          "IpProtocol": "tcp",
          "IpRanges": [
            {
              "CidrIp": "10.42.0.0/16",
              "Description": "Allow inbound HTTP API traffic"
            }
          ],
          "Ipv6Ranges": [],
          "PrefixListIds": [],
          "ToPort": 8080,
          "UserIdGroupPairs": []
        }
      ],
      "OwnerId": "1234567890",
      "GroupId": "sg-037ec36e968f1f5e7",
      "IpPermissionsEgress": [
        {
          "IpProtocol": "-1",
          "IpRanges": [
            {
              "CidrIp": "0.0.0.0/0",
              "Description": "Allow all egress"
            }
          ],
          "Ipv6Ranges": [],
          "PrefixListIds": [],
          "UserIdGroupPairs": []
        }
      ],
      "VpcId": "vpc-077ca8c89d111b3c1"
    }
  ]
}
```

Security group này:

- Cho phép lưu lượng vào cho HTTP API được phục vụ bởi Pod trên cổng 8080
- Cho phép tất cả lưu lượng ra
- Sẽ được phép truy cập vào cơ sở dữ liệu RDS như chúng ta đã thấy trước đó

Để Pod của chúng ta sử dụng security group này, chúng ta cần sử dụng CRD `SecurityGroupPolicy` để thông báo cho EKS security group nào sẽ được ánh xạ vào một tập hợp cụ thể của Pods. Đây là những gì chúng ta sẽ cấu hình:

```file
manifests/modules/networking/securitygroups-for-pods/sg/policy.yaml
```

Áp dụng điều này vào cluster sau đó tái khởi động các Pod `catalog` một lần nữa:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/networking/securitygroups-for-pods/sg \
  | envsubst | kubectl apply -f-
namespace/catalog unchanged
serviceaccount/catalog unchanged
configmap/catalog unchanged
configmap/catalog-env-97g7bft95f unchanged
configmap/catalog-sg-env-54k244c6t7 created
secret/catalog-db unchanged
service/catalog unchanged
service/catalog-mysql unchanged
service/ui-nlb unchanged
deployment.apps/catalog unchanged
statefulset.apps/catalog-mysql unchanged
securitygrouppolicy.vpcresources.k8s.aws/catalog-rds-access created
$ kubectl delete pod -n catalog -l app.kubernetes.io/component=service
pod "catalog-6ccc6b5575-glfxc" deleted
$ kubectl rollout status -n catalog deployment/catalog --timeout 30s
deployment "catalog" successfully rolled out
```

Lần này, Pod `catalog` sẽ khởi động và quá trình triển khai sẽ thành công. Bạn có thể kiểm tra log để xác nhận rằng nó đang kết nối với cơ sở dữ liệu RDS:

```bash
$ kubectl -n catalog logs deployment/catalog | grep Connect
2022/12/20 20:52:10 Connecting to catalog_user:xxxxxxxxxx@tcp(eks-workshop-catalog.cjkatqd1cnrz.us-west-2.rds.amazonaws.com:3306)/catalog?timeout=5s
2022/12/20 20:52:10 Connected
2022/12/20 20:52:10 Connecting to catalog_user:xxxxxxxxxx@tcp(eks-workshop-catalog.cjkatqd1cnrz.us-west-2.rds.amazonaws.com:3306)/catalog?timeout=5s
2022/12/20 20:52:10 Connected
```