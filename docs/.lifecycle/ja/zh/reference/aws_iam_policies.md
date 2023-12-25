---
displayed_sidebar: Chinese
---

# AWS IAM ポリシー

IAM ポリシーは、特定の AWS リソースへのアクセス権を宣言するために使用されます。IAM ポリシーを作成した後、そのポリシーを IAM ユーザーまたはロールに追加することで、その IAM ユーザーまたはロールにポリシーで宣言された特定の AWS リソースへのアクセス権を付与します。

StarRocks では、異なる操作が異なる AWS リソースを必要とします。したがって、アクセスする AWS リソースに基づいて IAM ポリシーを作成する必要があります。

この記事では、異なるシナリオで [Instance Profile、Assumed Role、および IAM User 認証方式](../integrations/authenticate_to_aws_resources.md#準備作業) を選択した場合に、StarRocks が関連する AWS リソースに正しくアクセスするために必要な IAM ポリシーをどのように設定するかを説明します。

## AWS S3 からのバルクデータインポート

S3 バケットからバルクデータをインポートする必要がある場合は、以下のように IAM ポリシーを設定してください：

> **注意**
>
> 下記のポリシー中の `<bucket_name>` をデータが存在する S3 バケットの名前に置き換えてください。

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

## AWS S3 からのデータの読み書き

S3 バケットからデータをクエリする必要がある場合は、以下のように IAM ポリシーを設定してください：

> **注意**
>
> 下記のポリシー中の `<bucket_name>` をデータが存在する S3 バケットの名前に置き換えてください。

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

## AWS Glue との連携

AWS Glue と連携する必要がある場合は、以下のように IAM ポリシーを設定してください：

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
