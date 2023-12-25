---
displayed_sidebar: Chinese
---

# AWS認証情報の設定

StarRocksは、以下の3つの認証方式を通じてAWSリソースとの統合をサポートしています：Instance Profile、Assumed Role、IAM User。この記事では、これら3つの認証方式でAWSセキュリティクレデンシャルを設定する方法について説明します。

## 認証方式の紹介

### Instance Profileに基づく認証

Instance Profileを使用することで、StarRocksクラスターはAWS EC2インスタンスに関連付けられたInstance Profileの権限を直接継承することができます。このモードでは、クラスターにログインできるユーザーは、AWS IAMポリシーの設定に基づいて、関連するAWSリソース（例えば、特定のAWS S3バケットの読み書き操作）に対して許可された範囲内の操作を実行することができます。

### Assumed Roleに基づく認証

Instance Profileモードとは異なり、Assumed RoleはAWS IAM Roleを担当することで外部データソースへのアクセス認証を実現します。AWS公式ドキュメント[代入ロール](https://docs.aws.amazon.com/zh_cn/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)を参照してください。

<!--異なるCatalogに対して異なるAssumed Roleを設定することで、特定のAWSリソース（例えばS3バケット）へのアクセス権を持つことができます。管理者として、異なるユーザーに異なるCatalogの操作権限を付与することで、同じクラスター内の異なるユーザーが異なる外部データにアクセスすることを制御することができます。-->

### IAM Userに基づく認証

IAM UserはAWS IAM Userを通じて外部データソースへのアクセス認証をサポートします。AWS公式ドキュメント[IAMユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)を参照してください。

<!--異なるCatalogで異なるIAM UserのAccess KeyとSecret Keyを指定することで、同じクラスター内の異なるユーザーが異なる外部データにアクセスすることを制御することができます。-->

## 準備作業

まず、StarRocksクラスターが配置されているEC2インスタンスに関連付けられたIAMロール（以下、「EC2インスタンス関連ロール」と呼びます）を見つけ、そのロールのARNを取得します。Instance Profileに基づく認証方式を使用する場合は、このロールが必要です。Assumed Roleに基づく認証方式を使用する場合は、このロールとそのARNが必要です。

次に、アクセスするAWSリソースとStarRocksとの統合における具体的な操作シナリオに基づいてIAMポリシーを作成します。IAMポリシーは特定のAWSリソースへのアクセス権を宣言するために使用されます。IAMポリシーを作成した後、そのポリシーをIAMユーザーやロールに追加することで、そのIAMユーザーやロールが特定のAWSリソースへのアクセス権を持つことになります。

> **注意**
>
> [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にログインし、IAMユーザーやロールを編集する権限が必要です。

特定のAWSリソースにアクセスするために必要なIAMポリシーについては、以下を参照してください：

- [AWS S3からのバルクデータインポート](../reference/aws_iam_policies.md#从-aws-s3-批量导入数据)
- [AWS S3でのデータの読み書き](../reference/aws_iam_policies.md#从-aws-s3-读写数据)
- [AWS Glueとの連携](../reference/aws_iam_policies.md#对接-aws-glue)

### Instance Profileに基づく認証

特定のAWSリソースへのアクセスを宣言した[IAMポリシー](../reference/aws_iam_policies.md)をEC2インスタンス関連ロールに追加します。

### Assumed Roleに基づく認証

#### IAMロールの作成とポリシーの追加

アクセスするAWSリソースに応じて、一つまたは複数のIAMロールを作成することができます。具体的な操作はAWS公式ドキュメント[Creating IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)を参照してください。次に、特定のAWSリソースへのアクセスを宣言した[IAMポリシー](../reference/aws_iam_policies.md)を作成したIAMロールに追加します。

ここでは、StarRocksクラスターがAWS S3とAWS Glueにアクセスする必要がある操作シナリオを想定しています。この場合、一つのIAMロール（例えば`s3_assumed_role`）を作成し、AWS S3へのアクセス権を与えるポリシーとAWS Glueへのアクセス権を与えるポリシーの両方をそのロールに追加することができます。または、`s3_assumed_role`と`glue_assumed_role`という二つの異なるIAMロールを作成し、それぞれの異なるポリシーをこれらのロールに追加することもできます（つまり、AWS S3へのアクセス権を与えるポリシーを`s3_assumed_role`に、AWS Glueへのアクセス権を与えるポリシーを`glue_assumed_role`に追加します）。

StarRocksクラスターのEC2インスタンス関連ロールは、作成および設定したIAMロールを担当することで、特定のAWSリソースへのアクセス権を得ることができます。

この記事では、Assumed Role `s3_assumed_role`を一つだけ作成し、AWS S3へのアクセス権を与えるポリシーとAWS Glueへのアクセス権を与えるポリシーの両方をそのロールに追加したと仮定しています。

#### 信頼関係の設定

以下の手順に従ってAssumed Roleを設定してください：

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にログインします。
2. 左側のナビゲーションバーで **Access management** > **Roles** を選択します。
3. Assumed Role（`s3_assumed_role`）を見つけ、ロール名をクリックします。
4. ロールの詳細ページで、**Trust relationships** タブをクリックし、**Trust relationships** タブで **Edit trust policy** をクリックします。
5. **Edit trust policy** ページで、現在のJSON形式のポリシーを削除し、以下のポリシーをコピーして貼り付けます。以下のポリシーの `<cluster_EC2_iam_role_ARN>` をEC2インスタンス関連ロールのARNに置き換えてください。最後に、**Update policy** をクリックします。

   ```JSON
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "AWS": "<cluster_EC2_iam_role_ARN>"
               },
               "Action": "sts:AssumeRole"
           }
       ]
   }
   ```

異なるAWSリソースへのアクセス権を与えるために異なるAssumed Roleを作成した場合は、上記の手順に従って他のすべてのAssumed Roleの設定を完了する必要があります。例えば、`s3_assumed_role`と`glue_assumed_role`という二つのAssumed Roleを作成し、それぞれAWS S3とAWS Glueへのアクセス権を与えるために使用した場合、`glue_assumed_role`の設定を完了するために上記の手順を繰り返す必要があります。

EC2インスタンス関連ロールの設定は以下の手順に従って行ってください：

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にログインします。
2. 左側のナビゲーションバーで **Access management** > **Roles** を選択します。
3. EC2インスタンス関連ロールを見つけ、ロール名をクリックします。
4. ロールの詳細ページの **Permissions policies** セクションで、**Add permissions** をクリックし、**Create inline policy** を選択します。
5. **Specify permissions** ステップで、**JSON** タブをクリックし、現在のJSON形式のポリシーを削除して、以下のポリシーをコピーして貼り付けます。以下のポリシーの `<s3_assumed_role_ARN>` をAssumed Role `s3_assumed_role`のARNに置き換えてください。最後に、**Review policy** をクリックします。

   ```JSON
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": ["sts:AssumeRole"],
               "Resource": [
                   "<s3_assumed_role_ARN>"
               ]
           }
       ]
   }
   ```

異なるAWSリソースへのアクセス権を与えるために異なるAssumed Roleを作成した場合は、上記のポリシーの **Resource** フィールドにすべてのAssumed RoleのARNを入力する必要があります。例えば、`s3_assumed_role`と`glue_assumed_role`という二つのAssumed Roleを作成し、それぞれAWS S3とAWS Glueへのアクセス権を与えるために使用した場合、**Resource** フィールドには`s3_assumed_role`のARNと`glue_assumed_role`のARNを入力する必要があります。形式は次のとおりです：`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`。

6. **Review Policy** ステップで、ポリシー名を入力し、**Create policy** をクリックします。

### IAM Userに基づく認証

IAMユーザーを作成します。具体的な操作はAWS公式ドキュメント[Creating an IAM user in your AWS account](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)を参照してください。

次に、特定のAWSリソースへのアクセスを宣言した[IAMポリシー](../reference/aws_iam_policies.md)を作成したIAMユーザーに追加します。

## 原理図

StarRocksでのInstance Profile、Assumed Role、およびIAM Userの3つの認証方式の原理と違いは、以下の図に示されています。

![Credentials](../assets/authenticate_s3_credential_methods.png)

## パラメータ設定

### AWS S3へのアクセス認証パラメータ

StarRocksがAWS S3と統合する必要があるさまざまなシナリオで、例えばExternal Catalogやファイル外部テーブルの作成、AWS S3からのデータインポート、バックアップ、復元など、AWS S3の認証パラメータは以下のように設定する必要があります：

- Instance Profileに基づく認証と鑑定を行う場合は、`aws.s3.use_instance_profile`を`true`に設定します。
- Assumed Roleに基づく認証と鑑定を行う場合は、`aws.s3.use_instance_profile`を`true`に設定し、`aws.s3.iam_role_arn`にAWS S3へのアクセスに使用するAssumed RoleのARN（例えば、前述の`s3_assumed_role`のARN）を入力します。
- IAM Userに基づく認証と鑑定を行う場合は、`aws.s3.use_instance_profile`を`false`に設定し、`aws.s3.access_key`と`aws.s3.secret_key`にそれぞれAWS IAMユーザーのAccess KeyとSecret Keyを入力します。

パラメータの説明は以下の表を参照してください。

| パラメータ                        | 必須       | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |

| aws.s3.use_instance_profile | はい       | Instance Profile および Assumed Role の2つの認証方法を使用するかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 Bucket にアクセスする権限を持つ IAM Role の ARN。Assumed Role 認証方法で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.access_key           | いいえ       | IAM User の Access Key。IAM User 認証方法で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM User の Secret Key。IAM User 認証方法で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

### AWS Glue へのアクセス認証パラメータ

StarRocks が AWS Glue と統合するさまざまなシナリオで、例えば External Catalog を作成する際には、以下のように AWS Glue の認証パラメータを設定する必要があります：

- Instance Profile を使用して認証および認可を行う場合は、`aws.glue.use_instance_profile` を `true` に設定します。
- Assumed Role を使用して認証および認可を行う場合は、`aws.glue.use_instance_profile` を `true` に設定し、`aws.glue.iam_role_arn` に AWS Glue へのアクセスに使用する Assumed Role の ARN（例えば、先に作成した `glue_assumed_role` の ARN）を入力します。
- IAM User を使用して認証および認可を行う場合は、`aws.glue.use_instance_profile` を `false` に設定し、`aws.glue.access_key` および `aws.glue.secret_key` にそれぞれ AWS IAM User の Access Key と Secret Key を入力します。

パラメータの説明は以下の表を参照してください。

| パラメータ                        | 必須   | 説明                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| aws.glue.use_instance_profile | はい       | Instance Profile および Assumed Role の2つの認証方法を使用するかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalog へのアクセス権を持つ IAM Role の ARN。Assumed Role 認証方法で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.access_key           | いいえ       | IAM User の Access Key。IAM User 認証方法で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | IAM User の Secret Key。IAM User 認証方法で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

## 統合例

### External Catalog

StarRocks クラスターで External Catalog を作成する前に、データレイクの2つの重要なコンポーネントを統合する必要があります：

- 分散ファイルストレージ（例：AWS S3）は、テーブルファイルを保存するために使用されます。
- メタデータサービス（例：Hive Metastore（以下 HMS と略）または AWS Glue）は、テーブルファイルのメタデータと位置情報を保存するために使用されます。

StarRocks は以下のタイプの External Catalog をサポートしています：

- [Hive カタログ](../data_source/catalog/hive_catalog.md)
- [Iceberg カタログ](../data_source/catalog/iceberg_catalog.md)
- [Hudi カタログ](../data_source/catalog/hudi_catalog.md)
- [Delta Lake カタログ](../data_source/catalog/deltalake_catalog.md)

以下の例では、`hive_catalog_hms` または `hive_catalog_glue` という名前の Hive Catalog を作成し、Hive クラスター内のデータをクエリします。詳細な構文とパラメータの説明については、[Hive カタログ](../data_source/catalog/hive_catalog.md)を参照してください。

#### Instance Profile 認証認可に基づく

- Hive クラスターが HMS をメタデータサービスとして使用している場合、次のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- Amazon EMR Hive クラスターが AWS Glue をメタデータサービスとして使用している場合、次のように Hive カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2"
  );
  ```

#### Assumed Role 認証認可に基づく

- Hive クラスターが HMS をメタデータサービスとして使用している場合、次のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- Amazon EMR Hive クラスターが AWS Glue をメタデータサービスとして使用している場合、次のように Hive カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/glue_assumed_role",
      "aws.glue.region" = "us-west-2"
  );
  ```

#### IAM User 認証認可に基づく

- Hive クラスターが HMS をメタデータサービスとして使用している場合、次のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- Amazon EMR Hive クラスターが AWS Glue をメタデータサービスとして使用している場合、次のように Hive カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "false",
      "aws.glue.access_key" = "<iam_user_access_key>",
      "aws.glue.secret_key" = "<iam_user_secret_key>",
      "aws.glue.region" = "us-west-2"
  );
  ```

### ファイル外部テーブル

ファイル外部テーブルは Internal Catalog `default_catalog` で作成する必要があります。以下の例では、既存のデータベース `test_s3_db` に `file_table` という名前のファイル外部テーブルを作成しています。詳細な構文とパラメータの説明については、[ファイル外部テーブル](../data_source/file_external_table.md)を参照してください。

#### Instance Profile 認証認可に基づく

次のようにファイル外部テーブルを作成できます：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-2"
);
```

#### Assumed Role 認証認可に基づく

次のようにファイル外部テーブルを作成できます：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
    "aws.s3.region" = "us-west-2"
);
```

#### IAM User 認証認可に基づく

次のようにファイル外部テーブルを作成できます：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-2"
);
```

### データインポート

AWS S3 からデータをインポートできます。以下の例では、`s3a://test-bucket/test_brokerload_ingestion` パスにあるすべての Parquet 形式のデータファイルを既存のデータベース `test_s3_db` の `test_ingestion_2` テーブルにインポートしています。詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Instance Profile 認証認可に基づく

次のようにデータをインポートできます：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```

#### Assumed Role 認証認可に基づく

次のようにデータをインポートできます：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```

#### IAM User 認証認可に基づく

次のようにデータをインポートできます：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(

    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);