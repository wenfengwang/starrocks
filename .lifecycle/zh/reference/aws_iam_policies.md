---
displayed_sidebar: English
---

# AWS IAM 策略

在 AWS IAM 中，策略用于声明对特定 AWS 资源的权限集。创建策略之后，您需要将其关联到一个 IAM 角色或用户。这样，IAM 角色或用户就被授予了策略中声明的权限，以访问特定的 AWS 资源。

在 StarRocks 中执行不同的操作需要您对不同的 AWS 资源具有访问权限，因此您需要配置不同的策略。

本主题提供了您在选择[实例配置文件、扮演角色或基于 IAM 用户的认证方法](../integrations/authenticate_to_aws_resources.md#preparations)时，为了让 StarRocks 与不同业务场景中的各种 AWS 资源集成，需要配置的 IAM 策略。

## 从 AWS S3 批量加载数据

如果您想要从您的 S3 存储桶中加载数据，请配置以下 IAM 策略：

> **注意**
> 请将以下 JSON 策略模板中的 `\u003cbucket_name\u003e` 替换成您存储数据文件的 S3 存储桶名称。

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

如果您想要查询您的 S3 存储桶中的数据，请配置以下 IAM 策略：

> **注意**
> 请将以下 JSON 策略模板中的 `\u003cbucket_name\u003e` 替换成您存储数据文件的 S3 存储桶名称。

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

如果您想要将 StarRocks 与 AWS Glue 数据目录集成，请配置以下 IAM 策略：

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
```
