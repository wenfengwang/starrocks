---
displayed_sidebar: "Japanese"
---

# AWSリソースへの認証

StarRocksは、AWSリソースと連携するために3つの認証方法をサポートしています: インスタンスプロファイルベースの認証、仮定ロールベースの認証、およびIAMユーザーベースの認証。このトピックでは、これらの認証方法を使用してAWS資格情報を構成する方法について説明します。

## 認証方法

### インスタンスプロファイルベースの認証

インスタンスプロファイルベースの認証方法を使用すると、StarRocksクラスタは、クラスタが実行されているEC2インスタンスのインスタンスプロファイルで指定された特権を継承できます。理論上、クラスタにログインできる任意のクラスタユーザは、構成したAWS IAMポリシーに従ってAWSリソース上で許可された操作を実行できます。このユースケースの典型的なシナリオは、クラスタ内の複数のユーザ間でAWSリソースへのアクセス制御が不要な場合です。この認証方法は、同じクラスタ内での分離が不要であることを意味します。

ただし、この認証方法は、誰でもクラスタにログインできるという点でクラスタレベルの安全なアクセス制御ソリューションと見なすことができます。

### 仮定ロールベースの認証

インスタンスプロファイルベースの認証とは異なり、仮定ロールベースの認証方法はAWS IAMロールの仮定をサポートし、AWSリソースへのアクセス権を獲得します。詳細については、「[ロールの仮定](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)」を参照してください。

### IAMユーザーベースの認証

IAMユーザーベースの認証方法は、IAMユーザーの資格情報を使用してAWSリソースへのアクセスをサポートします。詳細については、「[IAMユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)」を参照してください。

## 事前準備

まず、StarRocksクラスタが実行されているEC2インスタンスに関連付けられたIAMロール（以下、本トピックではEC2インスタンスロールと呼びます）を見つけ、そのロールのARNを取得します。インスタンスプロファイルベースの認証の準備にはEC2インスタンスロールが必要であり、仮定ロールベースの認証の準備にはEC2インスタンスロールとそのARNが必要です。

次に、アクセスしたいAWSリソースの種類とStarRocks内の特定の操作シナリオに基づいてIAMポリシーを作成します。AWS IAMにおけるポリシーは、特定のAWSリソースに対する一連の権限を宣言します。ポリシーを作成した後は、そのポリシーをIAMロールまたはユーザーにアタッチする必要があります。このようにして、IAMロールまたはユーザーには、ポリシーで宣言されたアクセス権を指定されたAWSリソースにアクセスするための権限が付与されます。

> **注意**
>
> これらの準備を行うためには、[AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインしてIAMユーザーおよびロールを編集する権限が必要です。

特定のAWSリソースにアクセスするために必要なIAMポリシーについては、以下のセクションを参照してください:

- [AWS S3からのバッチデータロード](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [AWS S3の読み取り/書き込み](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [AWS Glueとの統合](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### インスタンスプロファイルベースの認証の準備

必要なAWSリソースへのアクセスに対する[IAMポリシー](../reference/aws_iam_policies.md)をEC2インスタンスロールにアタッチします。

### 仮定ロールベースの認証の準備

#### IAMロールの作成とポリシーのアタッチ

アクセスしたいAWSリソースに応じて、1つ以上のIAMロールを作成します。[IAMロールの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)を参照してください。その後、作成したIAMロールに必要なAWSリソースへのアクセスのための[IAMポリシー](../reference/aws_iam_policies.md)をアタッチします。

例えば、StarRocksクラスタがAWS S3とAWS Glueにアクセスすることを希望する場合、1つのIAMロール（たとえば`s3_assumed_role`）を作成し、そのロールにAWS S3へのアクセス用のポリシーとAWS Glueへのアクセス用のポリシーの両方をアタッチすることができます。また、異なるIAMロール（たとえば`s3_assumed_role`および`glue_assumed_role`）を作成し、それぞれのロールにこれらのポリシーを個別にアタッチすることもできます。

作成したIAMロールは、StarRocksクラスタのEC2インスタンスロールによって仮定され、指定されたAWSリソースにアクセスします。

このセクションでは、1つの仮定されたロール`s3_assumed_role`を作成し、AWS S3へのアクセス用のポリシーとAWS Glueへのアクセス用のポリシーの両方をそのロールに追加したと仮定しています。

#### 信頼関係の構成

次のように仮定されるロールを構成します:

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**[アクセス管理] > [ロール]**を選択します。
3. 仮定されたロール（`s3_assumed_role`など）を見つけ、その名前をクリックします。
4. ロールの詳細ページで、**信頼関係**タブをクリックし、**信頼関係**タブで**信頼ポリシーの編集**をクリックします。
5. **信頼ポリシーの編集**ページで、既存のJSONポリシードキュメントを削除し、以下のIAMポリシーを貼り付けます。ただし、ここで`<cluster_EC2_iam_role_ARN>`を上記で取得したEC2インスタンスロールのARNに置き換える必要があります。その後、**ポリシーの更新**をクリックします。

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

異なるAWSリソースにアクセスするために異なる仮定役割を作成した場合、他の仮定されたロールを構成するためにこれらの手順を繰り返す必要があります。例えば、AWS S3とAWS Glueにアクセスするために`s3_assumed_role`および`glue_assumed_role`を作成した場合、これらの手順を繰り返して`glue_assumed_role`を構成する必要があります。

EC2インスタンスロールを構成するためには、次の手順を実行します:

1. [AWS IAMコンソール](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)にサインインします。
2. 左側のナビゲーションペインで、**[アクセス管理] > [ロール]**を選択します。
3. EC2インスタンスロールを見つけ、その名前をクリックします。
4. ロールの詳細ページの**アクセス許可ポリシー**セクションで、**許可の追加**をクリックし、**インラインポリシーの作成**を選択します。
5. **アクセス許可の指定**ステップで、**JSON**タブをクリックし、既存のJSONポリシードキュメントを削除し、以下のIAMポリシーを貼り付けます。ただし、ここで`<s3_assumed_role_ARN>`を`s3_assumed_role`のARNに置き換える必要があります。その後、**ポリシーの確認**をクリックします。

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

   異なるAWSリソースにアクセスするために異なる仮定役割を作成した場合、前述のIAMポリシーの**リソース**要素にこれらの仮定役割のARNをすべて入力し、コンマ（,）で区切って入力する必要があります。例えば、AWS S3およびAWS Glueにアクセスするために`s3_assumed_role`および`glue_assumed_role`を作成した場合、`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`の形式で**リソース**要素に`s3_assumed_role`および`glue_assumed_role`のARNを入力する必要があります。

6. **ポリシーの確認**ステップで、ポリシー名を入力し、**ポリシーの作成**をクリックします。

### IAMユーザーベースの認証の準備

IAMユーザーを作成します。[AWSアカウント内のIAMユーザーの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)を参照してください。

その後、作成したIAMユーザーに必要なAWSリソースへのアクセスのための[IAMポリシー](../reference/aws_iam_policies.md)をアタッチします。

## 認証方法の比較

以下の図は、StarRocksにおけるインスタンスプロファイルベースの認証、仮定ロールベースの認証、IAMユーザーベースの認証の仕組みの違いを高レベルで説明しています。

![認証方法の比較](../assets/authenticate_s3_credential_methods.png)

## AWSリソースとの接続の構築

### AWS S3へのアクセスのための認証パラメータ
StarRocksがAWS S3と統合する必要があるさまざまなシナリオでは、外部カタログや外部ファイルテーブルを作成する場合、またはAWS S3からデータを取り込んだり、バックアップしたり、復元する場合など、AWS S3へのアクセスのための認証パラメーターを次のように設定します：

- たとえばプロファイルベースの認証を使用する場合、`aws.s3.use_instance_profile`を`true`に設定します。
- 仮定される役割ベースの認証を使用する場合、`aws.s3.use_instance_profile`を`true`に設定し、アクセスするために使用する仮定される役割のARNである`aws.s3.iam_role_arn`を構成します（例えば、上記で作成した仮定される役割`s3_assumed_role`のARN）。
- IAMユーザーベースの認証を使用する場合、`aws.s3.use_instance_profile`を`false`に設定し、AWS IAMユーザーのアクセスキーとシークレットキーを`aws.s3.access_key`と`aws.s3.secret_key`として構成します。

以下の表はパラメーターを説明しています。

| パラメーター                   | 必須    | 説明                                                  |
| --------------------------- | ------ | ---------------------------------------------------- |
| aws.s3.use_instance_profile | はい    | インスタンスプロファイルベースの認証方法と仮定される役割ベースの認証を有効にするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ   | AWS S3バケットへの特権を持つIAMロールのARN。AWS S3へのアクセスに仮定される役割ベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。  |
| aws.s3.access_key           | いいえ   | AWS IAMユーザーのアクセスキー。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |
| aws.s3.secret_key           | いいえ   | AWS IAMユーザーのシークレットキー。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |

### AWS Glueへのアクセスのための認証パラメーター

StarRocksがAWS Glueと統合する必要があるさまざまなシナリオでは、外部カタログを作成する場合、AWS Glueへのアクセスのための認証パラメーターを次のように設定します：

- たとえばプロファイルベースの認証を使用する場合、`aws.glue.use_instance_profile`を`true`に設定します。
- 仮定される役割ベースの認証を使用する場合、`aws.glue.use_instance_profile`を`true`に設定し、AWS Glueにアクセスするために使用する仮定される役割のARNである`aws.glue.iam_role_arn`を構成します（例えば、上記で作成した仮定される役割`glue_assumed_role`のARN）。
- IAMユーザーベースの認証を使用する場合、`aws.glue.use_instance_profile`を`false`に設定し、AWS IAMユーザーのアクセスキーとシークレットキーを`aws.glue.access_key`と`aws.glue.secret_key`として構成します。

以下の表はパラメーターを説明しています。

| パラメーター                   | 必須    | 説明                                                  |
| --------------------------- | ------ | ---------------------------------------------------- |
| aws.glue.use_instance_profile | はい    | インスタンスプロファイルベースの認証方法と仮定される役割ベースの認証を有効にするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ   | AWS Glueデータカタログに特権を持つIAMロールのARN。AWS Glueへのアクセスに仮定される役割ベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |
| aws.glue.access_key           | いいえ   | AWS IAMユーザーのアクセスキー。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |
| aws.glue.secret_key           | いいえ   | AWS IAMユーザーのシークレットキー。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |

## 統合例

### 外部カタログ

StarRocksクラスターで外部カタログを作成することは、対象のデータレイクシステムとの統合を意味し、これには次の2つの主要なコンポーネントが含まれます：

- テーブルファイルを格納するAWS S3などのファイルストレージ
- テーブルファイルのメタデータと位置を保存するHiveメタストアまたはAWS Glueなどのメタストア

StarRocksは以下のタイプのカタログをサポートしています：

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)
- [Icebergカタログ](../data_source/catalog/iceberg_catalog.md)
- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)
- [Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)

以下の例では、Hiveクラスターからデータをクエリするために`hive_catalog_hms`または`hive_catalog_glue`というHiveカタログを作成します。詳細な構文とパラメーターについては、[Hiveカタログ](../data_source/catalog/hive_catalog.md)を参照してください。

#### インスタンスプロファイルベースの認証

- Hiveメタストアを使用する場合、次のようにコマンドを実行します：

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します：

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

#### 仮定される役割ベースの認証

- Hiveメタストアを使用する場合、次のようにコマンドを実行します：

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します：

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

- Hiveメタストアを使用する場合、次のようにコマンドを実行します：

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

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します：

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

### 外部ファイルテーブル

外部ファイルテーブルは、`default_catalog`という内部カタログに作成する必要があります。

以下の例では、既存のデータベース`test_s3_db`に`file_table`という名前の外部ファイルテーブルを作成します。詳細な構文とパラメーターについては、[File external table](../data_source/file_external_table.md)を参照してください。

#### インスタンスプロファイルベースの認証

次のようにコマンドを実行します：

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
```
```markdown
+ {R}
+ {R}
  + {R}
    + {R}
```

```markdown
+ {T}
+ {T}
  + {T}
    + {T}
```