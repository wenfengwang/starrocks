---
displayed_sidebar: "Japanese"
---

# AWS IAMポリシー

AWS IAMのポリシーは、特定のAWSリソースに対する許可のセットを宣言します。ポリシーを作成した後は、そのポリシーをIAMロールまたはユーザーに添付する必要があります。したがって、IAMロールまたはユーザーには、指定されたAWSリソースにアクセスするためにポリシーで宣言された許可が割り当てられます。

StarRocksの異なる操作には、異なるAWSリソースへのアクセス許可が必要であり、そのために異なるポリシーを構成する必要があります。

このトピックでは、異なるビジネスシナリオでStarRocksが異なるAWSリソースと統合するために構成する必要のあるIAMポリシーを提供します。

## AWS S3からバッチデータをロード

S3バケットからデータをロードしたい場合は、以下のIAMポリシーを構成してください：

> **注意**
>
> 以下のJSONポリシーテンプレートの`<bucket_name>`を、データファイルを保存しているS3バケットの名前で置き換えてください。

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

## AWS S3の読み書き

S3バケットからデータをクエリしたい場合は、以下のIAMポリシーを構成してください：

> **注意**
>
> 以下のJSONポリシーテンプレートの`<bucket_name>`を、データファイルを保存しているS3バケットの名前で置き換えてください。

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

## AWS Glueと統合

AWS Glueデータカタログと統合したい場合は、以下のIAMポリシーを構成してください：

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