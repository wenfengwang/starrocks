---
displayed_sidebar: English
---

# AWSリソースへの認証

StarRocksは、インスタンスプロファイルベースの認証、アサムドロールベースの認証、IAMユーザーベースの認証の3つの方法をサポートしており、AWSリソースとの統合が可能です。このトピックでは、これらの認証方法を使用してAWSクレデンシャルを設定する方法について説明します。

## 認証方法

### インスタンスプロファイルベースの認証

インスタンスプロファイルベースの認証では、StarRocksクラスタが実行されているEC2インスタンスのインスタンスプロファイルに指定された権限を継承します。理論的には、クラスタにログインできる任意のユーザーが、設定されたAWS IAMポリシーに従ってAWSリソースに対して許可されたアクションを実行できます。このユースケースの典型的なシナリオは、クラスタ内の複数のユーザー間でAWSリソースへのアクセス制御を必要としない場合です。この認証方法は、同じクラスタ内での分離が不要であることを意味します。

しかし、この認証方法は、クラスタにログインできるユーザーがクラスタ管理者によって制御されるため、クラスタレベルで安全なアクセス制御ソリューションと見なすことができます。

### アサムドロールベースの認証

インスタンスプロファイルベースの認証と異なり、アサムドロールベースの認証では、AWS IAMロールを引き受けてAWSリソースにアクセスすることがサポートされています。詳細は[ロールの引き受け](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)を参照してください。

<!--具体的には、複数のカタログを特定のアサムドロールにバインドすることで、それぞれのカタログが特定のAWSリソース（例えば、特定のS3バケット）に接続できるようになります。これにより、現在のSQLセッションのカタログを変更することで異なるデータソースにアクセスできます。さらに、クラスタ管理者が異なるカタログに対して異なるユーザーに権限を付与する場合、同じクラスタ内の異なるユーザーが異なるデータソースにアクセスできるようなアクセス制御ソリューションが実現されます。-->

### IAMユーザーベースの認証

IAMユーザーベースの認証では、IAMユーザーのクレデンシャルを使用してAWSリソースにアクセスします。詳細は[IAMユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)を参照してください。

<!--複数のカタログを特定のIAMユーザーのクレデンシャル（アクセスキーとシークレットキー）にバインドすることで、同じクラスタ内の異なるユーザーが異なるデータソースにアクセスできるようになります。-->

## 準備

まず、StarRocksクラスタが実行されているEC2インスタンスに関連付けられているIAMロール（以下、EC2インスタンスロールと呼びます）を見つけ、そのロールのARNを取得します。インスタンスプロファイルベースの認証にはEC2インスタンスロールが必要であり、アサムドロールベースの認証にはEC2インスタンスロールとそのARNが必要です。

次に、アクセスしたいAWSリソースのタイプとStarRocks内での特定の操作シナリオに基づいてIAMポリシーを作成します。AWS IAMのポリシーは、特定のAWSリソースに対する一連の権限を宣言します。ポリシーを作成したら、それをIAMロールまたはユーザーにアタッチする必要があります。これにより、IAMロールまたはユーザーに、指定されたAWSリソースにアクセスするためのポリシーで宣言された権限が割り当てられます。

> **注意**
>
> これらの準備を行うには、[AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインし、IAMユーザーとロールを編集する権限が必要です。

特定のAWSリソースにアクセスするために必要なIAMポリシーについては、以下のセクションを参照してください。

- [AWS S3からのバッチデータロード](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [AWS S3の読み書き](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [AWS Glueとの統合](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### インスタンスプロファイルベースの認証の準備

必要なAWSリソースにアクセスするための[IAMポリシー](../reference/aws_iam_policies.md)をEC2インスタンスロールにアタッチします。

### アサムドロールベースの認証の準備

#### IAMロールを作成し、ポリシーをアタッチする

アクセスしたいAWSリソースに応じて、1つ以上のIAMロールを作成します。[IAMロールの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)を参照してください。次に、必要なAWSリソースにアクセスするための[IAMポリシー](../reference/aws_iam_policies.md)を作成したIAMロールにアタッチします。

例えば、StarRocksクラスタがAWS S3とAWS Glueにアクセスする場合、1つのIAMロール（例：`s3_assumed_role`）を作成し、そのロールにAWS S3とAWS Glueにアクセスするためのポリシーを両方アタッチすることを選択できます。または、2つの異なるIAMロール（例：`s3_assumed_role`と`glue_assumed_role`）を作成し、それぞれにこれらのポリシーをアタッチすることもできます（つまり、AWS S3にアクセスするためのポリシーを`s3_assumed_role`に、AWS Glueにアクセスするためのポリシーを`glue_assumed_role`にアタッチします）。

作成したIAMロールは、StarRocksクラスタのEC2インスタンスロールによって引き受けられ、指定されたAWSリソースにアクセスします。

このセクションでは、1つのアサムドロール`S3_assumed_role`を作成し、そのロールにAWS S3とAWS Glueにアクセスするためのポリシーを両方追加したことを前提としています。

#### 信頼関係を設定する

次の手順に従ってアサムドロールを設定します：

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**アクセス管理** > **ロール**を選択します。
3. アサムドロール（`s3_assumed_role`）を見つけて、その名前をクリックします。
4. ロールの詳細ページで、**信頼関係**タブをクリックし、**信頼関係**タブで**信頼ポリシーの編集**をクリックします。
5. **信頼ポリシーの編集**ページで、既存のJSONポリシードキュメントを削除し、以下のIAMポリシーを貼り付けます。このとき、`<cluster_EC2_iam_role_ARN>`をあなたが取得したEC2インスタンスロールのARNに置き換える必要があります。その後、**ポリシーを更新**をクリックします。

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

異なるAWSリソースにアクセスするために異なる仮想ロールを作成した場合、上記の手順を繰り返して他の仮想ロールを設定する必要があります。たとえば、AWS S3とAWS Glueにそれぞれアクセスするために`s3_assumed_role`と`glue_assumed_role`を作成した場合、この状況では`glue_assumed_role`の設定も同様の手順を繰り返す必要があります。

EC2インスタンスロールを次のように設定します：

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**アクセス管理** > **ロール**を選択します。
3. EC2インスタンスロールを探して、その名前をクリックします。
4. ロールの詳細ページの**アクセス許可ポリシー**セクションで、**アクセス許可の追加**をクリックし、**インラインポリシーの作成**を選択します。
5. **アクセス許可の指定**ステップで、**JSON**タブをクリックし、既存のJSONポリシードキュメントを削除して、以下のIAMポリシーを貼り付けます。ここで`<s3_assumed_role_ARN>`を`s3_assumed_role`のARNに置き換えてください。その後、**ポリシーの確認**をクリックします。

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

   異なるAWSリソースにアクセスするために異なる仮想ロールを作成した場合、上記のIAMポリシーの**Resource**要素にこれらすべての仮想ロールのARNを入力し、それらをコンマ(,)で区切る必要があります。たとえば、AWS S3とAWS Glueにそれぞれアクセスするために`s3_assumed_role`と`glue_assumed_role`を作成した場合、この状況では**Resource**要素に`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`という形式で`s3_assumed_role`と`glue_assumed_role`のARNを入力する必要があります。

6. **ポリシーの確認**ステップで、ポリシー名を入力し、**ポリシーの作成**をクリックします。

### IAMユーザーベースの認証の準備

IAMユーザーを作成します。[AWSアカウントでのIAMユーザーの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)を参照してください。

次に、作成したIAMユーザーに必要なAWSリソースへのアクセスに必要な[IAMポリシー](../reference/aws_iam_policies.md)をアタッチします。

## 認証方法の比較

以下の図は、StarRocksにおけるインスタンスプロファイルベースの認証、仮想ロールベースの認証、およびIAMユーザーベースの認証のメカニズムの違いを高レベルで説明しています。

![認証方法の比較](../assets/authenticate_s3_credential_methods.png)

## AWSリソースとの接続を構築

### AWS S3へのアクセスにおける認証パラメータ

StarRocksがAWS S3と統合する必要があるさまざまなシナリオで、たとえば外部カタログを作成したり、ファイル外部テーブルを作成したり、AWS S3からデータを取り込んだり、バックアップしたり、復元したりする場合、AWS S3へのアクセスにおける認証パラメータを次のように設定します：

- インスタンスプロファイルベースの認証の場合、`aws.s3.use_instance_profile`を`true`に設定します。
- 仮想ロールベースの認証の場合、`aws.s3.use_instance_profile`を`true`に設定し、AWS S3へのアクセスに使用する仮想ロールのARNを`aws.s3.iam_role_arn`に設定します（例：上記で作成した`s3_assumed_role`のARN）。
- IAMユーザーベースの認証の場合、`aws.s3.use_instance_profile`を`false`に設定し、AWS IAMユーザーのアクセスキーとシークレットキーをそれぞれ`aws.s3.access_key`と`aws.s3.secret_key`に設定します。

以下の表は、パラメータを説明しています。

| パラメータ                    | 必須 | 説明                                                  |
| --------------------------- | ---- | ---------------------------------------------------- |
| aws.s3.use_instance_profile | はい | インスタンスプロファイルベースの認証方法と仮想ロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`、`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | いいえ  | AWS S3バケットに対する権限を持つIAMロールのARN。仮想ロールベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.access_key           | いいえ  | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ  | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |

### AWS Glueへのアクセスにおける認証パラメータ

StarRocksがAWS Glueと統合する必要があるさまざまなシナリオで、たとえば外部カタログを作成する場合、AWS Glueへのアクセスにおける認証パラメータを次のように設定します：

- インスタンスプロファイルベースの認証の場合、`aws.glue.use_instance_profile`を`true`に設定します。
- 仮想ロールベースの認証の場合、`aws.glue.use_instance_profile`を`true`に設定し、AWS Glueへのアクセスに使用する仮想ロールのARNを`aws.glue.iam_role_arn`に設定します（例：上記で作成した`glue_assumed_role`のARN）。
- IAMユーザーベースの認証の場合、`aws.glue.use_instance_profile`を`false`に設定し、AWS IAMユーザーのアクセスキーとシークレットキーをそれぞれ`aws.glue.access_key`と`aws.glue.secret_key`に設定します。

以下の表は、パラメータを説明しています。

| パラメータ                      | 必須 | 説明                                                  |
| ----------------------------- | ---- | ---------------------------------------------------- |
| aws.glue.use_instance_profile | はい | インスタンスプロファイルベースの認証方法と仮想ロールベースの認証を有効にするかどうかを指定します。有効な値：`true`、`false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn         | いいえ  | AWS Glue Data Catalogに対する権限を持つIAMロールのARN。仮想ロールベースの認証方法を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.access_key           | いいえ  | AWS IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |

| aws.glue.secret_key           | いいえ       | AWS IAM ユーザーのシークレットキーです。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

## 統合例

### 外部カタログ

StarRocks クラスターに外部カタログを作成することは、次の2つの主要コンポーネントで構成されるターゲットデータレイクシステムとの統合を構築することを意味します：

- テーブルファイルを保存するためのファイルストレージ（例：AWS S3）
- テーブルファイルのメタデータと場所を保存するメタストア（例：Hive メタストアや AWS Glue）

StarRocks は以下のタイプのカタログをサポートしています：

- [Hive カタログ](../data_source/catalog/hive_catalog.md)
- [Iceberg カタログ](../data_source/catalog/iceberg_catalog.md)
- [Hudi カタログ](../data_source/catalog/hudi_catalog.md)
- [Delta Lake カタログ](../data_source/catalog/deltalake_catalog.md)

以下の例では、使用するメタストアの種類に応じて `hive_catalog_hms` または `hive_catalog_glue` という名前の Hive カタログを作成し、Hive クラスターからデータをクエリします。構文とパラメータの詳細については、[Hive カタログ](../data_source/catalog/hive_catalog.md)を参照してください。

#### インスタンスプロファイルベースの認証

- Hive クラスターで Hive メタストアを使用する場合、以下のコマンドを実行します：

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

- Amazon EMR Hive クラスターで AWS Glue を使用する場合、以下のコマンドを実行します：

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

#### 想定ロールベースの認証

- Hive クラスターで Hive メタストアを使用する場合、以下のコマンドを実行します：

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

- Amazon EMR Hive クラスターで AWS Glue を使用する場合、以下のコマンドを実行します：

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

#### IAM ユーザーベースの認証

- Hive クラスターで Hive メタストアを使用する場合、以下のコマンドを実行します：

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

- Amazon EMR Hive クラスターで AWS Glue を使用する場合、以下のコマンドを実行します：

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

ファイル外部テーブルは `default_catalog` という名前の内部カタログに作成する必要があります。

以下の例では、`test_s3_db` という名前の既存のデータベースに `file_table` という名前のファイル外部テーブルを作成します。構文とパラメータの詳細については、[ファイル外部テーブル](../data_source/file_external_table.md)を参照してください。

#### インスタンスプロファイルベースの認証

以下のコマンドを実行します：

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

#### 想定ロールベースの認証

以下のコマンドを実行します：

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

#### IAM ユーザーベースの認証

以下のコマンドを実行します：

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

### データ取り込み

AWS S3 からデータを取り込むには、LOAD LABEL を使用できます。

以下の例では、`s3a://test-bucket/test_brokerload_ingestion` パスに格納されているすべての Parquet データファイルから `test_ingestion_2` テーブルにデータを取り込みます。このテーブルは `test_s3_db` という名前の既存のデータベースにあります。構文およびパラメータの詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### インスタンスプロファイルベースの認証

以下のコマンドを実行します：

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

#### 想定ロールベースの認証

以下のコマンドを実行します：

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

