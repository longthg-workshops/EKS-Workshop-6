
---
title: "Sử dụng Amazon RDS"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 4.2 </b>"
---


Một cơ sở dữ liệu RDS đã được tạo trong tài khoản của chúng ta, hãy lấy địa chỉ kết nối và mật khẩu để sử dụng sau này:

```bash
$ export CATALOG_RDS_ENDPOINT_QUERY=$(aws rds describe-db-instances --db-instance-identifier $EKS_CLUSTER_NAME-catalog --query 'DBInstances[0].Endpoint')
$ export CATALOG_RDS_ENDPOINT=$(echo $CATALOG_RDS_ENDPOINT_QUERY | jq -r '.Address+":"+(.Port|tostring)')
$ echo $CATALOG_RDS_ENDPOINT
eks-workshop-catalog.cluster-cjkatqd1cnrz.us-west-2.rds.amazonaws.com:3306
$ export CATALOG_RDS_PASSWORD=$(aws ssm get-parameter --name $EKS_CLUSTER_NAME-catalog-db --region $AWS_REGION --query "Parameter.Value" --output text --with-decryption)
```

Bước đầu tiên trong quy trình này là cấu hình lại dịch vụ `catalog` để sử dụng một cơ sở dữ liệu Amazon RDS đã được tạo trước đó. Ứng dụng tải hầu hết cấu hình của nó từ một ConfigMap, hãy xem nó:

```bash
$ kubectl -n catalog get -o yaml cm catalog
```
```yaml
apiVersion: v1
data:
  DB_ENDPOINT: catalog-mysql:3306
  DB_READ_ENDPOINT: catalog-mysql:3306
kind: ConfigMap
metadata:
  name: catalog
  namespace: catalog
```

Kustomization sau đây ghi đè ConfigMap, thay điểm kết nối MySQL theo biến môi trường `CATALOG_RDS_ENDPOINT` để ứng dụng sẽ kết nối với cơ sở dữ liệu Amazon RDS đã được tạo sẵn cho chúng ta.

```kustomization
modules/networking/securitygroups-for-pods/rds/kustomization.yaml
ConfigMap/catalog
```

Hãy áp dụng thay đổi này để sử dụng cơ sở dữ liệu RDS:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/networking/securitygroups-for-pods/rds \
  | envsubst | kubectl apply -f-
```

Kiểm tra xem ConfigMap đã được cập nhật với các giá trị mới chưa:

```bash
$ kubectl get -n catalog cm catalog -o yaml
apiVersion: v1
data:
  DB_ENDPOINT: eks-workshop-catalog.cluster-cjkatqd1cnrz.us-west-2.rds.amazonaws.com:3306
  DB_READ_ENDPOINT: eks-workshop-catalog.cluster-cjkatqd1cnrz.us-west-2.rds.amazonaws.com:3306
kind: ConfigMap
metadata:
  labels:
    app: catalog
  name: catalog
  namespace: catalog
```

Bây giờ chúng ta cần tái tạo các Pods `catalog` để lấy nội dung ConfigMap mới của chúng ta:

```bash expectError=true
$ kubectl delete pod -n catalog -l app.kubernetes.io/component=service
pod "catalog-788bb5d488-9p6cj" deleted
$ kubectl rollout status -n catalog deployment/catalog --timeout 30s
Waiting for deployment "catalog" rollout to finish: 1 old replicas are pending termination...
error: timed out waiting for the condition
```

Chúng ta nhận được một lỗi, dường như các Pods danh mục của chúng ta đã không khởi động lại kịp thời. Đã xảy ra vấn đề gì? Hãy kiểm tra log của Pod để xem đã xảy ra gì:

```bash
$ kubectl -n catalog logs deployment/catalog
2022/12/19 17:43:05 Error: Failed to prep migration dial tcp 10.42.11.72:3306: i/o timeout
2022/12/19 17:43:05 Error: Failed to run migration dial tcp 10.42.11.72:3306: i/o timeout
2022/12/19 17:43:05 dial tcp 10.42.11.72:3306: i/o timeout
```

Pod của chúng ta không thể kết nối được đến cơ sở dữ liệu RDS. Chúng ta có thể kiểm tra Security group EC2 đã được áp dụng cho cơ sở dữ liệu RDS như sau:

```bash
$ aws ec2 describe-security-groups \
    --filters Name=vpc-id,Values=$VPC_ID Name=tag:Name,Values=$EKS_CLUSTER_NAME-catalog-rds | jq '.'
```
```yaml
{
  "SecurityGroups": [
    {
      "Description": "Catalog RDS security group",
      "GroupName": "eks-workshop-catalog-rds-20221220135004125100000005",
      "IpPermissions": [
        {
          "FromPort": 3306,
          "IpProtocol": "tcp",
          "IpRanges": [],
          "Ipv6Ranges": [],
          "PrefixListIds": [],
          "ToPort": 3306,
          "UserIdGroupPairs": [
            {
              "Description": "MySQL access from within VPC",
              "GroupId": "sg-037ec36e968f1f5e7",
              "UserId": "1234567890"
            }
          ]
        }
      ],
      "OwnerId": "1234567890",
      "GroupId": "sg-0b47cdc59485262ea",
      "IpPermissionsEgress": [],
      "Tags": [
        {
          "Key": "Name",
          "Value": "eks-workshop-catalog-rds"
        }
      ],
      "VpcId": "vpc-077ca8c89d111b3c1"
    }
  ]
}
```

Bạn cũng có thể xem Security group của RDS instance thông qua [bảng điều khiển AWS](https://console.aws.amazon.com/rds/home#database:id=eks-workshop-catalog;is-cluster=false).

Security group này chỉ cho phép lưu lượng truy cập vào cơ sở dữ liệu RDS trên cổng `3306` nếu nó đến từ một nguồn có một Security group cụ thể, trong ví dụ trên là `sg-037ec36e968f1f5e7`.