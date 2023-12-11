---
displayed_sidebar: "Japanese"
---

# 共有データにMinIOを使用する

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![Shared-data Architecture](../../assets/share_data_arch.png)

## 共有データStarRocksクラスターのデプロイ

共有データStarRocksクラスターのデプロイは、共有データStarRocksクラスターのデプロイと似ています。唯一の違いは、共有データクラスターでは、BEの代わりにCNをデプロイする必要があるということです。このセクションでは、共有データStarRocksクラスターのデプロイ時にFEおよびCNの構成ファイル**fe.conf**および**cn.conf**に追加する必要がある追加のFEおよびCN構成項目のみをリストアップしています。StarRocksクラスターのデプロイの詳細については、[StarRocksをデプロイ](../../deployment/deploy_manually.md)を参照してください。

### 共有データStarRocks用のFEノードの構成

FEを開始する前に、FE構成ファイル**fe.conf**に以下の構成項目を追加してください。

#### run_mode

StarRocksクラスターの実行モード。有効な値：

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注意**
>
> StarRocksクラスターで`shared_data`および`shared_nothing`のモードを同時に採用することはできません。混在したデプロイはサポートされていません。
>
> クラスターをデプロイした後に`run_mode`を変更しないでください。さもないと、クラスターの再起動に失敗します。共有データクラスターやその逆の変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト：`6090`

#### enable_load_volume_from_conf

StarRocksがFE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。有効な値：

- `true`（デフォルト）：新しい共有データクラスターを作成する際にこの項目を`true`と指定した場合、StarRocksはFE構成ファイルのオブジェクトストレージ関連のプロパティを使用して組込ストレージボリューム`builtin_storage_volume`を作成し、これをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動に失敗します。
- `false`：新しい共有データクラスターを作成する際にこの項目を`false`と指定した場合、StarRocksは組込のストレージボリュームを作成せずに直接起動します。StarRocks内でオブジェクトを作成する前にストレージボリュームを手動で作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームを作成](#create-default-storage-volume)を参照してください。

v3.1.0からサポートされています。

> **注意**
>
> 既存のv3.0以降の共有データクラスターをアップグレードする間は、この項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースやテーブルは読み取り専用になり、そこにデータをロードすることはできません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）、およびS3プロトコルと互換性のあるオブジェクトストレージにデータを保存することができます。有効な値：

- `S3`（デフォルト）
- `AZBLOB`

> 注
>
> このパラメータを`S3`として指定する場合は、`aws_s3`で接頭辞が付いたパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`として指定する場合は、`azure_blob`で接頭辞が付いたパラメータを追加する必要があります。

#### aws_s3_path

データを保存するために使用するS3パス。S3バケットの名前と、それのサブパス（あれば）で構成されています。例：`testbucket/subpath`

#### aws_s3_endpoint

S3バケットにアクセスするためのエンドポイント。例：`https://s3.us-west-2.amazonaws.com`

#### aws_s3_region

S3バケットが存在するリージョン。例：`us-west-2`

#### aws_s3_use_aws_sdk_default_behavior

[Amazon S3のデフォルト資格情報プロバイダーチェーン](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)を使用するかどうか。有効な値：

- `true`
- `false`（デフォルト）

#### aws_s3_use_instance_profile

S3にアクセスするための資格情報メソッドとしてInstance ProfileおよびAssumed Roleを使用するかどうか。有効な値：

- `true`
- `false`（デフォルト）

S3にIAMユーザーベースの資格情報（アクセスキーおよびシークレットキー）を使用する場合は、この項目を`false`と指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

S3にInstance Profileを使用する場合は、この項目を`true`と指定する必要があります。

S3にAssumed Roleを使用する場合は、この項目を`true`と指定し、`aws_s3_iam_role_arn`を指定する必要があります。また、外部のAWSアカウントを使用する場合は、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

S3バケットにアクセスするために使用するアクセスキーID。

#### aws_s3_secret_key

S3バケットにアクセスするために使用するシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが保存されているS3バケットに対する権限を持つIAMロールのARN。

#### aws_s3_external_id

S3バケットへの他のAWSアカウントからのアクセスを許可する外部ID。

#### azure_blob_path

データを保存するために使用するAzure Blob Storageパス。保存アカウンタ内のコンテナの名前と、それのサブパス（あれば）で構成されています。例：`testcontainer/subpath`

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例：`https://test.blob.core.windows.net`

#### azure_blob_shared_key

Azure Blob Storageのリクエストを承認するために使用される共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを承認するために使用される共有アクセス署名（SAS）。

> **注意**
>
> 共有データStarRocksクラスターを作成した後で、資格情報関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースやテーブルは読み取り専用になり、そこにデータをロードすることはできません。

クラスターが作成された後で手動でデフォルトのストレージボリュームを作成したい場合は、次の構成項目を追加するだけでよいです。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocks用のCNノードの構成

<SharedDataCNconf />

## 共有データStarRocksクラスターを使用する

<SharedDataUseIntro />

次の例は、MinIOバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、アクセスキーおよびシークレットキーを有効にし、このストレージボリュームをデフォルトのストレージボリュームとして設定します。

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