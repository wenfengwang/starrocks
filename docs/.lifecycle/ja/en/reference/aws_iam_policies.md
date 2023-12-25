---
displayed_sidebar: English
---

# AWS IAM ポリシー

AWS IAM のポリシーは、特定の AWS リソースに対する一連の権限を宣言します。ポリシーを作成した後、IAM ロールまたはユーザーにアタッチする必要があります。これにより、IAM ロールまたはユーザーは、ポリシーで宣言された権限を持って指定された AWS リソースにアクセスできるようになります。

StarRocks の異なる操作には、異なる AWS リソースへのアクセス権が必要ですので、異なるポリシーを設定する必要があります。

このトピックでは、[インスタンスプロファイル、仮定されたロール、または IAM ユーザーベースの認証方法を選択した場合](../integrations/authenticate_to_aws_resources.md#preparations)に、StarRocks がさまざまな AWS リソースと統合するために必要な IAM ポリシーを提供します。

## AWS S3 からのバッチデータロード

S3 バケットからデータをロードしたい場合は、以下の IAM ポリシーを設定してください。

> **注意**
>
> 次の JSON ポリシーテンプレート内の `<bucket_name>` を、データファイルを保存している S3 バケットの名前に置き換えてください。

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

## AWS S3 の読み書き

S3 バケットからデータをクエリしたい場合は、以下の IAM ポリシーを設定してください。

> **注意**
>
> 次の JSON ポリシーテンプレート内の `<bucket_name>` を、データファイルを保存している S3 バケットの名前に置き換えてください。

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

AWS Glue Data Catalog と連携したい場合は、以下の IAM ポリシーを設定してください。

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
