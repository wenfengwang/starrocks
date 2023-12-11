---
displayed_sidebar: "Japanese"
---

# Amazon Web Services リソースへの認証

StarRocks は、Amazon Web Services (AWS) リソースと統合するための 3 つの認証方法をサポートしています：インスタンスプロファイルベースの認証、仮想ロールベースの認証、IAM ユーザーベースの認証。このトピックでは、これらの認証方法を使用して AWS 資格情報を構成する手順について説明します。

## 認証方法

### インスタンスプロファイルベースの認証

インスタンスプロファイルベースの認証方法を使用すると、StarRocks クラスタは、クラスタが実行されている EC2 インスタンスのインスタンスプロファイルに指定された特権を継承できます。理論上、クラスタにログインできる任意のユーザーは、構成した AWS IAM ポリシーに従って、AWS リソース上で許可されたアクションを実行できます。このユースケースの典型的なシナリオは、クラスタ内の複数のユーザー間で AWS リソースへのアクセス制御が必要ない場合です。この認証方法は、同じクラスタ内での分離が不要であることを意味します。

ただし、この認証方法は、クラスタ管理者によって制御されるため、依然としてクラスタレベルでの安全なアクセス制御ソリューションと見なすことができます。

### 仮想ロールベースの認証

インスタンスプロファイルベースの認証とは異なり、仮想ロールベースの認証方法は、AWS IAM ロールを仮想的に使用して AWS リソースにアクセスできるようになります。詳細については、[ロールの仮定](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)を参照してください。

<!--具体的には、複数のカタログをそれぞれ特定の仮想ロールにバインドすることで、それぞれのカタログが特定の AWS リソース（たとえば、特定の S3 バケット）に接続できるようになります。これにより、現在の SQL セッションのカタログを変更することで異なるデータソースにアクセスできます。さらに、クラスタ管理者が異なるユーザーに異なるカタログへの権限を付与すれば、クラスタ内の異なるユーザーが異なるデータソースにアクセスできるようになります。-->

### IAM ユーザーベースの認証

IAM ユーザーベースの認証方法では、IAM ユーザーの資格情報を使用して、AWS リソースにアクセスすることができます。詳細については、[IAM ユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)を参照してください。

<!--それぞれのカタログを特定の IAM ユーザーの資格情報（アクセスキーおよびシークレットキー）にバインドすることで、同じクラスタ内の異なるユーザーが異なるデータソースにアクセスできるようになります。-->

## 準備

まず、StarRocks クラスタが実行されている EC2 インスタンスに関連付けられている IAM ロール（以下、このトピックでは EC2 インスタンスロールと呼びます）を見つけ、そのロールの Amazon Resource Name（ARN）を取得してください。インスタンスプロファイルベースの認証には EC2 インスタンスロールが必要であり、仮想ロールベースの認証には EC2 インスタンスロールとその ARN が必要です。

次に、アクセスする AWS リソースの種類と StarRocks 内の特定の操作シナリオに基づいて IAM ポリシーを作成します。AWS IAM では、ポリシーは特定の AWS リソースに対するアクセス権限のセットを宣言します。ポリシーを作成したら、それを IAM ロールまたはユーザーにアタッチする必要があります。そのため、IAM ロールまたはユーザーには、ポリシーで宣言されたアクセス対象の AWS リソースへのアクセス権限が付与されます。

> **注意**
>
> これらの準備を行うためには、[AWS IAM コンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインして、IAM ユーザーやロールを編集する権限が必要です。

AWS リソースにアクセスするために必要な IAM ポリシーについては、以下のセクションを参照してください：

- [AWS S3 からのバッチデータロード](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [AWS S3 の読み取り/書き込み](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [AWS Glue との統合](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### インスタンスプロファイルベースの認証の準備

必要な AWS リソースにアクセスするための[IAM ポリシー](../reference/aws_iam_policies.md)を EC2 インスタンスロールにアタッチします。

### 仮想ロールベースの認証の準備

#### IAM ロールの作成とポリシーのアタッチ

アクセスしようとする AWS リソースに応じて、1 つまたは複数の IAM ロールを作成します。[IAM ロールの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)を参照してください。その後、作成した IAM ロールに、必要な AWS リソースへのアクセス用の[IAM ポリシー](../reference/aws_iam_policies.md)をアタッチします。

例えば、StarRocks クラスタが AWS S3 と AWS Glue にアクセスできるようにしたい場合、1 つの IAM ロール（たとえば `s3_assumed_role`）を作成し、そのロールに AWS S3 へのアクセス用ポリシーと AWS Glue へのアクセス用ポリシーの両方をアタッチすることができます。または、異なる IAM ロール（たとえば `s3_assumed_role` と `glue_assumed_role`）を作成し、それぞれのロールにそれぞれのポリシーをアタッチすることもできます。

作成した IAM ロールは、それらが EC2 インスタンスロールによって仮定され、指定された AWS リソースにアクセスします。

このセクションは、1 つの仮定されるロール(`s3_assumed_role`)のみが作成され、作成されたロールに AWS S3 へのアクセス用のポリシーと AWS Glue へのアクセス用のポリシーが両方追加されていることを想定しています。

#### 信頼関係の構成

次のように仮定されるロールを構成します：

1. [AWS IAM コンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションパネルで **Access management** > **Roles** を選択します。
3. 仮定されるロール（`s3_assumed_role`）を見つけ、その名前をクリックします。
4. ロールの詳細ページで、**信頼関係**タブをクリックし、**信頼関係**タブで **信頼ポリシーの編集** をクリックします。
5. **信頼ポリシーの編集**ページで、既存の JSON ポリシー文を削除し、次の IAM ポリシーを貼り付けます。この際、`<cluster_EC2_iam_role_ARN>`を上記で取得した EC2 インスタンスロールの ARNに置き換える必要があります。その後、**ポリシーの更新**をクリックします。

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

異なる AWS リソースにアクセスするために異なる仮定されるロールを作成した場合、前述の手順を繰り返して他の仮定されるロールを構成する必要があります。たとえば、AWS S3 と AWS Glue にアクセスするために `s3_assumed_role` と `glue_assumed_role` を作成した場合、`glue_assumed_role` を構成するために前述の手順を繰り返す必要があります。

EC2 インスタンスロールを構成するには次の手順を実行します：

1. [AWS IAM コンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションパネルで **Access management** > **Roles** を選択します。
3. EC2 インスタンスロールを見つけ、その名前をクリックします。
4. ロールの詳細ページの **Permissions policies** セクションで、**アクセス許可の追加**をクリックし、**インラインポリシーの作成**を選択します。
5. **権限の指定**ステップで、**JSON** タブをクリックし、既存の JSON ポリシー文を削除し、次の IAM ポリシーを貼り付けます。この際、`<s3_assumed_role_ARN>`を `s3_assumed_role` の ARNに置き換える必要があります。その後、**ポリシーのレビュー**をクリックします。

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

   異なる AWS リソースにアクセスするために異なる仮定されるロールを作成した場合、前述の IAM ポリシーの **Resource** 要素にそれらの仮定されるロールすべての ARNを入力し、コンマ（,）で分割する必要があります。たとえば、AWS S3 および AWS Glue にアクセスするために `s3_assumed_role` と `glue_assumed_role` を作成した場合、`s3_assumed_role` の ARN と `glue_assumed_role` の ARN を以下の形式で **Resource** 要素に入力する必要があります：`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`。

6. **ポリシーをレビュー**ステップで、ポリシー名を入力し、**ポリシーの作成**をクリックします。

### IAM ユーザーベースの認証の準備

IAM ユーザーを作成します。[AWS アカウント内に IAM ユーザーを作成する](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)を参照してください。

その後、作成した IAM ユーザーに、アクセスする必要のある AWS リソース用の[IAM ポリシー](../reference/aws_iam_policies.md)をアタッチします。

## 認証方法の比較

以下の図は、StarRocks でのインスタンスプロファイルベースの認証、仮想ロールベースの認証、IAM ユーザーベースの認証の仕組みの違いを説明しています。

![Authentication methods comparison between authentication methods](../assets/authenticate_s3_credential_methods.png)

## AWS リソースとの接続の構築

### AWS S3 へのアクセスのための認証パラメータ
さまざまなシナリオで、StarRocksがAWS S3と統合する必要がある場合、たとえば、外部カタログやファイル外部テーブルを作成したり、AWS S3からデータを取り込んだり、バックアップしたり、復元したりする場合、以下のようにAWS S3へのアクセス認証パラメータを構成します。

AWS S3へのアクセス認証には次のように認証パラメータを設定します。

- インスタンスプロファイルベースの認証の場合、`aws.s3.use_instance_profile` を `true` に設定します。
- 仮定された役割ベースの認証の場合、`aws.s3.use_instance_profile` を `true` に設定し、AWS S3へのアクセスに使用する仮定された役割のARNを `aws.s3.iam_role_arn` として構成します（たとえば、上記で作成した仮定された役割 `s3_assumed_role` のARN）。
- IAMユーザーベースの認証の場合、`aws.s3.use_instance_profile` を `false` に設定し、AWS IAMユーザーのアクセスキーとシークレットキーを `aws.s3.access_key` と `aws.s3.secret_key` として構成します。

以下の表にパラメータが説明されています。

| パラメータ                   | 必須     | 説明                                                  　    |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と仮定された役割ベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | No       | AWS S3バケツに権限を持つIAMロールのARNです。AWS S3へのアクセスに仮定された役割ベースの認証方法を使用する場合には、このパラメータを指定する必要があります。  |
| aws.s3.access_key           | No       | AWS IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合には、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No       | AWS IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合には、このパラメータを指定する必要があります。 |

### AWS Glueへのアクセス認証パラメータ

さまざまなシナリオで、StarRocksがAWS Glueと統合する必要がある場合、たとえば、外部カタログを作成する場合、以下のようにAWS Glueへのアクセス認証パラメータを構成します。

- インスタンスプロファイルベースの認証の場合、`aws.glue.use_instance_profile` を `true` に設定します。
- 仮定された役割ベースの認証の場合、`aws.glue.use_instance_profile` を `true` に設定し、AWS Glueへのアクセスに使用する仮定された役割のARNを `aws.glue.iam_role_arn` として構成します（たとえば、上記で作成した仮定された役割 `glue_assumed_role` のARN）。
- IAMユーザーベースの認証の場合、`aws.glue.use_instance_profile` を `false` に設定し、AWS IAMユーザーのアクセスキーとシークレットキーを `aws.glue.access_key` と `aws.glue.secret_key` として構成します。

以下の表にパラメータが説明されています。

| パラメータ                     | 必須     | 説明                                                  　    |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| aws.glue.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と仮定された役割ベースの認証を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | No       | AWS Glueデータカタログに権限を持つIAMロールのARNです。AWS Glueへのアクセスに仮定された役割ベースの認証方法を使用する場合には、このパラメータを指定する必要があります。 |
| aws.glue.access_key           | No       | AWS IAMユーザーのアクセスキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合には、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAMユーザーのシークレットキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合には、このパラメータを指定する必要があります。 |

## 統合の例

### 外部カタログ

StarRocksクラスターで外部カタログを作成する場合、ターゲットのデータレイクシステムとの統合が必要です。これは、次の2つの主要コンポーネントで構成されています。

- AWS S3などのファイルストレージは、テーブルファイルを保存するために使用します。
- HiveメタストアまたはAWS Glueのようなメタストアは、テーブルファイルのメタデータや位置を保存するために使用します。

StarRocksは以下の種類のカタログをサポートしています。

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)
- [Icebergカタログ](../data_source/catalog/iceberg_catalog.md)
- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)
- [Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)

以下の例では、Hiveクラスターからデータをクエリするために、使用するメタストアのタイプに応じて`hive_catalog_hms`または`hive_catalog_glue`という名前のHiveカタログを作成します。詳細な構文とパラメータについては、「Hiveカタログ」を参照してください。

#### インスタンスプロファイルベースの認証

- Hiveメタストアを使用する場合、以下のようなコマンドを実行します。

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のようなコマンドを実行します。

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

#### 仮定された役割ベースの認証

- Hiveメタストアを使用する場合、以下のようなコマンドを実行します。

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のようなコマンドを実行します。

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

- Hiveメタストアを使用する場合、以下のようなコマンドを実行します。

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のようなコマンドを実行します。

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

以下の例では、`test_s3_db`という既存のデータベースに`file_table`という名前のファイル外部テーブルを作成します。詳細な構文とパラメータについては、「File external table」を参照してください。

#### インスタンスプロファイルベースの認証

以下のようなコマンドを実行します。

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
```yaml
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "us-west-2"
);
```

#### 帰結される役割ベースの認証

以下のようなコマンドを実行します：

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

以下のようなコマンドを実行します：

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

### 取り込み

LOAD LABELを使用してAWS S3からデータをロードできます。

以下の例は、`test_s3_db`という既存のデータベース内の`test_ingestion_2`テーブルに格納されている`s3a://test-bucket/test_brokerload_ingestion`パスにあるすべてのParquetデータファイルからデータを読み込むものです。詳細な構文とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### インスタンスプロファイルベースの認証

以下のようなコマンドを実行します：

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

#### 帰結される役割ベースの認証

以下のようなコマンドを実行します：

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

以下のようなコマンドを実行します：

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