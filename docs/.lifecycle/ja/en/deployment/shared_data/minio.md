---
displayed_sidebar: English
---

# MinIOを用いた共有データの利用

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスタのデプロイ

共有データStarRocksクラスタのデプロイは、共有ノーシングStarRocksクラスタのデプロイと似ています。唯一の違いは、共有データクラスタではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスタをデプロイする際に、FEおよびCNの設定ファイル**fe.conf**と**cn.conf**に追加する必要がある追加のFEおよびCNの設定項目のみをリストします。StarRocksクラスタのデプロイの詳細については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

### 共有データStarRocksのFEノードを設定する

FEを起動する前に、FEの設定ファイル**fe.conf**に以下の設定項目を追加します。

#### run_mode

StarRocksクラスタの実行モード。有効な値:

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注**
>
> StarRocksクラスタでは`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> クラスタがデプロイされた後に`run_mode`を変更しないでください。そうしないと、クラスタは再起動に失敗します。共有ノーシングクラスタから共有データクラスタへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスRPCポート。

- デフォルト：`6090`

#### enable_load_volume_from_conf

StarRocksがFEの設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。有効な値:

- `true`（デフォルト）新しい共有データクラスタを作成する際にこの項目を`true`として指定すると、StarRocksはFEの設定ファイルのオブジェクトストレージ関連のプロパティを使用してビルトインストレージボリューム`builtin_storage_volume`を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動に失敗します。
- `false`新しい共有データクラスタを作成する際にこの項目を`false`として指定すると、StarRocksはビルトインストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームを作成する](#use-your-shared-data-starrocks-cluster)を参照してください。

v3.1.0からサポート。

> **警告**
>
> 既存の共有データクラスタをv3.0からアップグレードする場合、この項目を`true`のままにしておくことを強く推奨します。この項目を`false`に設定すると、アップグレード前に作成したデータベースとテーブルが読み取り専用になり、データをロードすることができなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルと互換性のあるオブジェクトストレージ（AWS S3、Google GCP、MinIOなど）にデータを保存することをサポートしています。有効な値:

- `S3`（デフォルト）
- `AZBLOB`.

> 注
>
> このパラメータを`S3`として指定する場合は、`aws_s3`で始まるパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`として指定する場合は、`azure_blob`で始まるパラメータを追加する必要があります。

#### aws_s3_path

データを保存するためのS3パス。これは、S3バケットの名前とその下のサブパス（ある場合）で構成されます。例: `testbucket/subpath`。

#### aws_s3_endpoint

S3バケットにアクセスするためのエンドポイント。例: `https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

S3バケットが存在するリージョン。例: `us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

[AWS SDKのデフォルト認証情報プロバイダーチェーン](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)を使用するかどうか。有効な値:

- `true`
- `false`（デフォルト）。

#### aws_s3_use_instance_profile

S3へのアクセスにインスタンスプロファイルと仮定されたロールを認証方法として使用するかどうか。有効な値:

- `true`
- `false`（デフォルト）。

IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してS3にアクセスする場合、この項目を`false`に設定し、`aws_s3_access_key`と`aws_s3_secret_key`を指定する必要があります。

インスタンスプロファイルを使用してS3にアクセスする場合、この項目を`true`に設定する必要があります。

仮定されたロールを使用してS3にアクセスする場合、この項目を`true`に設定し、`aws_s3_iam_role_arn`を指定する必要があります。

外部AWSアカウントを使用する場合、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

S3バケットにアクセスするためのアクセスキーID。

#### aws_s3_secret_key

S3バケットにアクセスするためのシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが保存されているS3バケットに権限を持つIAMロールのARN。

#### aws_s3_external_id

S3バケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

#### azure_blob_path

データを保存するためのAzure Blob Storageパス。これは、ストレージアカウント内のコンテナの名前と、コンテナの下のサブパス（ある場合）で構成されます。例: `testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例: `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを認証するために使用される共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを認証するために使用される共有アクセス署名（SAS）。

> 注
>
> 共有データStarRocksクラスタが作成された後に変更できるのは認証情報関連の設定項目のみです。元のストレージパス関連の設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができなくなります。

クラスタが作成された後にデフォルトのストレージボリュームを手動で作成したい場合は、以下の設定項目のみを追加する必要があります。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードを設定する

<SharedDataCNconf />

## 共有データStarRocksクラスタを使用する

<SharedDataUseIntro />

以下の例では、アクセスキーとシークレットキーの認証情報を使用してMinIOバケット`defaultbucket`のためのストレージボリューム`def_volume`を作成し、ストレージボリュームを有効にして、それをデフォルトのストレージボリュームとして設定します。

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://hostname.domainname.com:portnumber",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
