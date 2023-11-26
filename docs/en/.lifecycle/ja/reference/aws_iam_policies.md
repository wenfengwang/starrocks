---
displayed_sidebar: "Japanese"
---

# AWS IAM ポリシー

AWS IAM におけるポリシーは、特定の AWS リソースに対する一連のアクセス許可を宣言します。ポリシーを作成した後は、IAM ロールまたはユーザーにポリシーをアタッチする必要があります。そのため、IAM ロールまたはユーザーには、ポリシーで宣言されたアクセス許可が割り当てられ、指定された AWS リソースにアクセスするための権限が与えられます。

StarRocks の異なる操作には、さまざまな AWS リソースへのアクセス許可が必要です。そのため、異なるポリシーを設定する必要があります。

このトピックでは、さまざまなビジネスシナリオで StarRocks を AWS リソースと統合するために設定する必要のある IAM ポリシーを提供します。

## AWS S3 からバッチでデータをロードする

S3 バケットからデータをロードする場合は、次の IAM ポリシーを設定してください：

> **注意**
>
> 次の JSON ポリシーテンプレート内の `<bucket_name>` を、データファイルを格納する S3 バケットの名前に置き換えてください。

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

## AWS S3 の読み取り/書き込み

S3 バケットからデータをクエリする場合は、次の IAM ポリシーを設定してください：

> **注意**
>
> 次の JSON ポリシーテンプレート内の `<bucket_name>` を、データファイルを格納する S3 バケットの名前に置き換えてください。

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

## AWS Glue との統合

AWS Glue Data Catalog と統合する場合は、次の IAM ポリシーを設定してください：

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
