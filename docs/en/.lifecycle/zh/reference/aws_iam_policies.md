---
displayed_sidebar: English
---

# AWS IAM 策略

AWS IAM 中的策略声明了对特定 AWS 资源的一组权限。创建策略后，您需要将其附加到 IAM 角色或用户。因此，IAM 角色或用户被分配了策略中声明的权限，以访问指定的 AWS 资源。

StarRocks 中的不同操作需要您对不同的 AWS 资源拥有访问权限，因此您需要配置不同的策略。

本主题提供了在选择 [实例配置文件、代入角色或 IAM 用户认证方式](../integrations/authenticate_to_aws_resources.md#preparations) 时，需要为 StarRocks 配置的 IAM 策略，以便在各种业务场景下与不同的 AWS 资源集成。

## 从 AWS S3 批量加载数据

如果您想要从您的 S3 存储桶加载数据，请配置以下 IAM 策略：

> **注意**
>
> 请将以下 JSON 策略模板中的 `<bucket_name>` 替换为存储数据文件的 S3 存储桶名称。

```SQL
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "s3",
          "Effect": "Allow",
          "Action": [
              "s3:GetObject"
          ],
          "Resource": [
              "arn:aws:s3:::<bucket_name>/*"
          ]
      },
      {
          "Sid": "s3list",
          "Effect": "Allow",
          "Action": [
              "s3:ListBucket"
          ],
          "Resource": [
              "arn:aws:s3:::<bucket_name>"
          ]
      }
  ]
}
```

## 读/写 AWS S3

如果您想要从您的 S3 存储桶查询数据，请配置以下 IAM 策略：

> **注意**
>
> 请将以下 JSON 策略模板中的 `<bucket_name>` 替换为存储数据文件的 S3 存储桶名称。

```SQL
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "s3",
          "Effect": "Allow",
          "Action": [
              "s3:GetObject", 
              "s3:PutObject"
          ],
          "Resource": [
              "arn:aws:s3:::<bucket_name>/*"
          ]
      },
      {
          "Sid": "s3list",
          "Effect": "Allow",
          "Action": [
              "s3:ListBucket"
          ],
          "Resource": [
              "arn:aws:s3:::<bucket_name>"
          ]
      }
  ]
}
```

## 与 AWS Glue 集成

如果您想要与 AWS Glue 数据目录集成，请配置以下 IAM 策略：

```SQL
{
    "Version": "2012-10-17",
    "Statement": [
      {
          "Effect": "Allow",
          "Action": [
              "glue:GetDatabase",
              "glue:GetDatabases",
              "glue:GetPartition",
              "glue:GetPartitions",
              "glue:GetTable",
              "glue:GetTableVersions",
              "glue:GetTables",
              "glue:GetConnection",
              "glue:GetConnections",
              "glue:GetDevEndpoint",
              "glue:GetDevEndpoints",
              "glue:BatchGetPartition"
          ],
          "Resource": [
              "*"
            ]
        }
    ]
}