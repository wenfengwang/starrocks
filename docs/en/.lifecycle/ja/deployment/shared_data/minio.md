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

![共有データのアーキテクチャ](../../assets/share_data_arch.png)

## 共有データのStarRocksクラスタをデプロイする

共有データのStarRocksクラスタのデプロイは、共有しないStarRocksクラスタのデプロイと似ています。唯一の違いは、共有データクラスタではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データのStarRocksクラスタをデプロイする際に、FEおよびCNの設定ファイル**fe.conf**および**cn.conf**に追加する必要のある追加のFEおよびCNの設定項目のみをリストします。StarRocksクラスタをデプロイする詳細な手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

### 共有データのStarRocksのFEノードを設定する

FEを起動する前に、FEの設定ファイル**fe.conf**に以下の設定項目を追加してください。

#### run_mode

StarRocksクラスタの実行モード。有効な値:

- `shared_data`
- `shared_nothing` (デフォルト).

> **注意**
>
> StarRocksクラスタでは、`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。ミックスデプロイはサポートされていません。
>
> クラスタをデプロイした後に`run_mode`を変更しないでください。そうしないと、クラスタの再起動に失敗します。共有しないクラスタから共有データクラスタまたはその逆の変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFEの設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成するかどうか。有効な値:

- `true` (デフォルト) 新しい共有データクラスタを作成する際にこの項目を`true`と指定すると、StarRocksはFEの設定ファイルのオブジェクトストレージ関連のプロパティを使用して組み込みのストレージボリューム`builtin_storage_volume`を作成し、デフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動できません。
- `false` 新しい共有データクラスタを作成する際にこの項目を`false`と指定すると、StarRocksは組み込みのストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、ストレージボリュームを手動で作成し、デフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームを作成する](#create-default-storage-volume)を参照してください。

v3.1.0以降でサポートされています。

> **注意**
>
> 既存の共有データクラスタをv3.0からアップグレードする場合、この項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルに互換性のあるオブジェクトストレージ（AWS S3、Google GCP、およびMinIOなど）にデータを保存することができます。有効な値:

- `S3` (デフォルト)
- `AZBLOB`.

> 注意
>
> このパラメータを`S3`と指定する場合、`aws_s3`で始まるパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`と指定する場合、`azure_blob`で始まるパラメータを追加する必要があります。

#### aws_s3_path

データを保存するために使用するS3パス。S3バケットの名前と、それに含まれるサブパス（ある場合）から構成されます。例: `testbucket/subpath`。

#### aws_s3_endpoint

S3バケットにアクセスするために使用するエンドポイント。例: `https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

S3バケットが存在するリージョン。例: `us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

[AWS SDKのデフォルトの資格情報プロバイダーチェーン](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)を使用するかどうか。有効な値:

- `true`
- `false` (デフォルト).

#### aws_s3_use_instance_profile

S3へのアクセスにInstance ProfileおよびAssumed Roleを資格情報の方法として使用するかどうか。有効な値:

- `true`
- `false` (デフォルト).

IAMユーザーベースの資格情報（アクセスキーとシークレットキー）を使用してS3にアクセスする場合、この項目を`false`と指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

Instance Profileを使用してS3にアクセスする場合、この項目を`true`と指定する必要があります。

Assumed Roleを使用してS3にアクセスする場合、この項目を`true`と指定し、`aws_s3_iam_role_arn`を指定する必要があります。

外部のAWSアカウントを使用する場合、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

S3バケットにアクセスするために使用するアクセスキーID。

#### aws_s3_secret_key

S3バケットにアクセスするために使用するシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが格納されているS3バケットに対して特権を持つIAMロールのARN。

#### aws_s3_external_id

S3バケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

#### azure_blob_path

データを保存するために使用するAzure Blob Storageのパス。ストレージアカウント内のコンテナの名前と、それに含まれるサブパス（ある場合）から構成されます。例: `testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例: `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを承認するために使用する共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを承認するために使用する共有アクセス署名（SAS）。

> **注意**
>
> 共有データのStarRocksクラスタを作成した後は、資格情報関連の設定項目のみを変更できます。元のストレージパス関連の設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

クラスタが作成された後にデフォルトのストレージボリュームを手動で作成する場合は、以下の設定項目のみを追加する必要があります。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データのStarRocksのCNノードを設定する

<SharedDataCNconf />

## 共有データのStarRocksクラスタを使用する

<SharedDataUseIntro />

以下の例では、MinIOバケット`defaultbucket`に対してAccess KeyとSecret Keyの資格情報を使用してストレージボリューム`def_volume`を作成し、ストレージボリュームを有効にし、デフォルトのストレージボリュームとして設定しています。

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
