---
displayed_sidebar: "Chinese"
---

# AWS IAM政策

AWS IAM中的一个政策声明了特定AWS资源上一组权限。创建政策后，您需要将其附加到IAM角色或用户。因此，IAM角色或用户被分配在政策中声明的权限，以访问指定的AWS资源。

StarRocks中的不同操作需要您对不同的AWS资源具有访问权限，因此您需要配置不同的政策。

本主题提供了您需要为StarRocks配置的IAM政策，以便在各种业务场景中与不同的AWS资源集成。

## 从AWS S3批量加载数据

如果您想从S3存储桶中加载数据，请配置以下IAM政策：

> **注意**
>
> 请将以下JSON政策模板中的`<bucket_name>`替换为存储数据文件的S3存储桶的名称。

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

## 读取/写入AWS S3

如果您想要从S3存储桶查询数据，请配置以下IAM政策：

> **注意**
>
> 请将以下JSON政策模板中的`<bucket_name>`替换为存储数据文件的S3存储桶的名称。

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

## 与AWS Glue集成

如果您想要与您的AWS Glue数据目录集成，请配置以下IAM政策：

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