```yaml
   に {T}
   された_sidebar: "Japanese"
   ---

   # AWS IAM ポリシー

   AWS IAM のポリシーは、特定のAWSリソースに対するアクセス許可を宣言します。ポリシーを作成した後は、それをIAMロールまたはユーザーにアタッチする必要があります。したがって、IAMロールまたはユーザーには、宣言されたポリシーで指定されたAWSリソースへのアクセス権が付与されます。

   StarRocksの異なる操作には、異なるAWSリソースへのアクセス許可が必要であり、そのために異なるポリシーを構成する必要があります。

   このトピックでは、StarRocksが異なるビジネスシナリオで様々なAWSリソースと統合するために構成する必要があるIAMポリシーを提供します。

   ## AWS S3 からのバッチデータロード

   S3バケットからデータをロードする場合、以下のIAMポリシーを構成します:

   > **注意**
   >
   > 以下のJSONポリシーテンプレート内の`<bucket_name>`を、データファイルを保存しているS3バケットの名前に置き換えてください。

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

   S3バケットからデータをクエリする場合、以下のIAMポリシーを構成します:

   > **注意**
   >
   > 以下のJSONポリシーテンプレート内の`<bucket_name>`を、データファイルを保存しているS3バケットの名前に置き換えてください。

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

   AWS Glueデータカタログと統合する場合、以下のIAMポリシーを構成します:

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
```