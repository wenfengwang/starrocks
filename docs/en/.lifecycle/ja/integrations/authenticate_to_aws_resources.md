---
displayed_sidebar: "Japanese"
---

# AWSリソースへの認証

StarRocksでは、3つの認証方法を使用してAWSリソースと統合することができます。インスタンスプロファイルベースの認証、アサムドロールベースの認証、IAMユーザーベースの認証の3つの認証方法を使用してAWSの認証情報を構成する方法について説明します。

## 認証方法

### インスタンスプロファイルベースの認証

インスタンスプロファイルベースの認証方法では、StarRocksクラスターは、クラスターが実行されているEC2インスタンスのインスタンスプロファイルに指定された特権を継承することができます。理論的には、クラスターにログインできるクラスターユーザーは、構成したAWS IAMポリシーに従ってAWSリソース上で許可されたアクションを実行することができます。このユースケースの典型的なシナリオは、クラスター内の複数のクラスターユーザー間でAWSリソースへのアクセス制御が必要ない場合です。この認証方法では、同じクラスター内での分離は必要ありません。

ただし、この認証方法は、クラスターにログインできるユーザーがクラスター管理者によって制御されるため、クラスターレベルの安全なアクセス制御ソリューションと見なすこともできます。

### アサムドロールベースの認証

インスタンスプロファイルベースの認証とは異なり、アサムドロールベースの認証方法では、AWS IAMロールをアサムドロールとして使用してAWSリソースにアクセスすることができます。詳細については、[ロールのアサムプション](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)を参照してください。

<!--具体的には、各カタログを特定のアサムドロールにバインドすることができます。これにより、これらのカタログはそれぞれ特定のAWSリソース（たとえば、特定のS3バケット）に接続することができます。これにより、現在のSQLセッションのカタログを変更することで、異なるデータソースにアクセスすることができます。さらに、クラスター管理者が異なるカタログに対して異なるユーザーに特権を付与する場合、同じクラスター内の異なるユーザーが異なるデータソースにアクセスできるようになります。-->

### IAMユーザーベースの認証

IAMユーザーベースの認証方法では、IAMユーザーの資格情報を使用してAWSリソースにアクセスすることができます。詳細については、[IAMユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)を参照してください。

<!--IAMユーザーベースの認証方法では、各カタログを特定のIAMユーザーの資格情報（アクセスキーとシークレットキー）にバインドすることができます。これにより、同じクラスター内の異なるユーザーが異なるデータソースにアクセスすることができます。-->

## 準備

まず、StarRocksクラスターが実行されているEC2インスタンスに関連付けられているIAMロール（以下、EC2インスタンスロールと呼びます）を見つけ、そのロールのARNを取得します。インスタンスプロファイルベースの認証にはEC2インスタンスロールが必要であり、アサムドロールベースの認証にはEC2インスタンスロールとそのARNが必要です。

次に、アクセスしたいAWSリソースの種類とStarRocks内の特定の操作シナリオに基づいてIAMポリシーを作成します。AWS IAMのポリシーは、特定のAWSリソースに対する一連のアクセス許可を宣言します。ポリシーを作成した後、それをIAMロールまたはユーザーにアタッチする必要があります。これにより、IAMロールまたはユーザーには、ポリシーで宣言されたアクセス許可が付与され、指定されたAWSリソースにアクセスすることができます。

> **注意**
>
> これらの準備を行うには、[AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインし、IAMユーザーやロールを編集する権限が必要です。

特定のAWSリソースにアクセスするために必要なIAMポリシーについては、次のセクションを参照してください。

- [AWS S3からのバッチデータのロード](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [AWS S3の読み取り/書き込み](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [AWS Glueとの統合](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### インスタンスプロファイルベースの認証の準備

必要なAWSリソースへのアクセスのための[IAMポリシー](../reference/aws_iam_policies.md)をEC2インスタンスロールにアタッチします。

### アサムドロールベースの認証の準備

#### IAMロールの作成とポリシーのアタッチ

アクセスしたいAWSリソースに応じて、1つ以上のIAMロールを作成します。[IAMロールの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)を参照してください。その後、作成したIAMロールに必要なAWSリソースへのアクセスのための[IAMポリシー](../reference/aws_iam_policies.md)をアタッチします。

たとえば、StarRocksクラスターがAWS S3とAWS Glueにアクセスする必要がある場合、1つのIAMロール（たとえば、`s3_assumed_role`）を作成し、AWS S3へのアクセスのためのポリシーとAWS Glueへのアクセスのためのポリシーの両方をそのロールにアタッチすることができます。または、2つの異なるIAMロール（たとえば、`s3_assumed_role`と`glue_assumed_role`）を作成し、それぞれのロールにこれらのポリシーをアタッチすることもできます（つまり、AWS S3へのアクセスのためのポリシーを`s3_assumed_role`にアタッチし、AWS Glueへのアクセスのためのポリシーを`glue_assumed_role`にアタッチします）。

作成したIAMロールは、StarRocksクラスターのEC2インスタンスロールによってアサムドされ、指定したAWSリソースにアクセスするために使用されます。

このセクションでは、1つのアサムドロール（`s3_assumed_role`）のみを作成し、AWS S3へのアクセスのためのポリシーとAWS Glueへのアクセスのためのポリシーの両方をそのロールに追加したものとします。

#### 信頼関係の構成

アサムドロールを以下のように構成します。

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**アクセス管理** > **ロール**を選択します。
3. アサムドロール（`s3_assumed_role`）を見つけ、その名前をクリックします。
4. ロールの詳細ページで、**信頼関係**タブをクリックし、**信頼関係**タブで**信頼ポリシーの編集**をクリックします。
5. **信頼ポリシーの編集**ページで、既存のJSONポリシードキュメントを削除し、以下のIAMポリシーを貼り付けます。ただし、`<cluster_EC2_iam_role_ARN>`を上記で取得したEC2インスタンスロールのARNで置き換える必要があります。その後、**ポリシーの更新**をクリックします。

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

異なるAWSリソースにアクセスするために異なるアサムドロールを作成した場合は、前述の手順を繰り返して他のアサムドロールを構成する必要があります。たとえば、AWS S3へのアクセスのために`s3_assumed_role`と`glue_assumed_role`の2つのアサムドロールを作成した場合、`glue_assumed_role`を構成するために前述の手順を繰り返す必要があります。

EC2インスタンスロールを以下のように構成します。

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**アクセス管理** > **ロール**を選択します。
3. EC2インスタンスロールを見つけ、その名前をクリックします。
4. ロールの詳細ページの**アクセス許可ポリシー**セクションで、**アクセス許可の追加**をクリックし、**インラインポリシーの作成**を選択します。
5. **アクセス許可の指定**ステップで、**JSON**タブをクリックし、既存のJSONポリシードキュメントを削除し、以下のIAMポリシーを貼り付けます。ただし、`<s3_assumed_role_ARN>`をアサムドロール`s3_assumed_role`のARNで置き換える必要があります。その後、**ポリシーの確認**をクリックします。

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

   異なるAWSリソースにアクセスするために異なるアサムドロールを作成した場合は、前述のIAMポリシーの**Resource**要素にこれらのアサムドロールのARNをすべて記入し、カンマ（,）で区切る必要があります。たとえば、AWS S3へのアクセスのために`s3_assumed_role`と`glue_assumed_role`の2つのアサムドロールを作成した場合、以下の形式を使用して**Resource**要素に`s3_assumed_role`のARNと`glue_assumed_role`のARNを記入する必要があります：`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`。

6. **ポリシーの確認**ステップで、ポリシー名を入力し、**ポリシーの作成**をクリックします。

### IAMユーザーベースの認証の準備

IAMユーザーを作成します。[AWSアカウントでIAMユーザーを作成する](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)を参照してください。

その後、作成したIAMユーザーに必要なAWSリソースへのアクセスのための[IAMポリシー](../reference/aws_iam_policies.md)をアタッチします。

## 認証方法の比較

次の図は、StarRocksにおけるインスタンスプロファイルベースの認証、アサムドロールベースの認証、IAMユーザーベースの認証のメカニズムの違いを高レベルで説明しています。

![認証方法の比較](../assets/authenticate_s3_credential_methods.png)

## AWSリソースとの接続の構築

### AWS S3へのアクセスのための認証パラメータ

StarRocksがAWS S3と統合するさまざまなシナリオでは、外部カタログの作成、ファイル外部テーブルの作成、データのインジェスト、バックアップ、またはリストアなどの場合、以下のようにAWS S3へのアクセスのための認証パラメータを構成します。

- インスタンスプロファイルベースの認証の場合、`aws.s3.use_instance_profile`を`true`に設定します。
- アサムドロールベースの認証の場合、`aws.s3.use_instance_profile`を`true`に設定し、`aws.s3.iam_role_arn`をAWS S3にアクセスするために使用するアサムドロールのARN（たとえば、上記で作成したアサムドロール`s3_assumed_role`のARN）を構成します。
- IAMユーザーベースの認証の場合、`aws.s3.use_instance_profile`を`false`に設定し、`aws.s3.access_key`と`aws.s3.secret_key`をAWS IAMユーザーのアクセスキーとシークレットキーに構成します。

以下の表に、パラメータの詳細を示します。

| パラメータ                   | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法とアサムドロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | No   | AWS S3バケットに特権を持つIAMロールのARNです。AWS S3へのアクセスにアサムドロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。  |
| aws.s3.access_key           | No   | IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No   | IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

### AWS Glueへのアクセスのための認証パラメータ

StarRocksがAWS Glueと統合するさまざまなシナリオでは、外部カタログの作成など、以下のようにAWS Glueへのアクセスのための認証パラメータを構成します。

- インスタンスプロファイルベースの認証の場合、`aws.glue.use_instance_profile`を`true`に設定します。
- アサムドロールベースの認証の場合、`aws.glue.use_instance_profile`を`true`に設定し、`aws.glue.iam_role_arn`をAWS Glueにアクセスするために使用するアサムドロールのARN（たとえば、上記で作成したアサムドロール`glue_assumed_role`のARN）を構成します。
- IAMユーザーベースの認証の場合、`aws.glue.use_instance_profile`を`false`に設定し、`aws.glue.access_key`と`aws.glue.secret_key`をAWS IAMユーザーのアクセスキーとシークレットキーに構成します。

以下の表に、パラメータの詳細を示します。

| パラメータ                     | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| aws.glue.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法とアサムドロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn         | No   | AWS Glueデータカタログに特権を持つIAMロールのARNです。AWS Glueへのアクセスにアサムドロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.access_key           | No   | AWS IAMユーザーのアクセスキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No   | AWS IAMユーザーのシークレットキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

## 統合の例

### 外部カタログ

StarRocksクラスターで外部カタログを作成することは、ターゲットのデータレイクシステムとの統合を構築することを意味します。データレイクシステムは、次の2つの主要なコンポーネントで構成されています。

- テーブルファイルを格納するためのAWS S3などのファイルストレージ
- テーブルファイルのメタデータと場所を格納するためのHiveメタストアまたはAWS Glueなどのメタストア

StarRocksでは、次のタイプのカタログをサポートしています。

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)
- [Icebergカタログ](../data_source/catalog/iceberg_catalog.md)
- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)
- [Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)

以下の例では、HiveメタストアまたはAWS Glueを使用して、Hiveクラスターからデータをクエリするための`hive_catalog_hms`または`hive_catalog_glue`という名前のHiveカタログを作成します。詳細な構文とパラメータについては、[Hiveカタログ](../data_source/catalog/hive_catalog.md)を参照してください。

#### インスタンスプロファイルベースの認証

- Hiveメタストアを使用する場合は、次のようなコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行します：

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

#### アサムドロールベースの認証

- Hiveメタストアを使用する場合は、次のようなコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行します：

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

#### IAMユーザーベースの認証

- Hiveメタストアを使用する場合は、次のようなコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行します：

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

ファイル外部テーブルは、`default_catalog`という名前の内部カタログに作成する必要があります。

以下の例では、既存の`test_s3_db`データベースに`file_table`という名前のファイル外部テーブルを作成します。詳細な構文とパラメータについては、[ファイル外部テーブル](../data_source/file_external_table.md)を参照してください。

#### インスタンスプロファイルベースの認証

次のようなコマンドを実行します：

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

#### アサムドロールベースの認証

次のようなコマンドを実行します：

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

#### IAMユーザーベースの認証

次のようなコマンドを実行します：

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

### インジェスト

AWS S3からデータをロードするためにLOAD LABELを使用することができます。

以下の例では、`s3a://test-bucket/test_brokerload_ingestion`パスに格納されているすべてのParquetデータファイルからデータを読み込んで、`test_s3_db`データベースの`test_ingestion_2`テーブルにデータをロードします。詳細な構文とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### インスタンスプロファイルベースの認証

次のようなコマンドを実行します：

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

#### アサムドロールベースの認証

次のようなコマンドを実行します：

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

#### IAMユーザーベースの認証

次のようなコマンドを実行します：

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
```
