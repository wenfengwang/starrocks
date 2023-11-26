---
displayed_sidebar: "Japanese"
---

# GCSを使用してStarRocksをデプロイする

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスタのデプロイ

共有データStarRocksクラスタのデプロイは、共有しないStarRocksクラスタのデプロイと似ています。唯一の違いは、共有データクラスタではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスタをデプロイする際に、FEおよびCNの設定ファイル（**fe.conf**および**cn.conf**）に追加する必要がある追加のFEおよびCNの設定項目のみをリストアップしています。StarRocksクラスタをデプロイする詳しい手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注意**
>
> 次のセクションのドキュメントで共有ストレージに設定するまで、クラスタを開始しないでください。

## 共有データStarRocksのFEノードの設定

クラスタを開始する前に、FEおよびCNを設定してください。以下に例として提供される設定を参考にして、各パラメータの詳細を確認してください。

### GCS用のFE設定の例

`fe.conf`の共有データの追加例は、各FEノードの`fe.conf`ファイルに追加できます。GCSストレージは、[Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview)を使用してアクセスされるため、パラメータは`aws_s3`プレフィックスを使用します。

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例: testbucket/subpath
  aws_s3_path = <s3_path>

  # 例: us-east1
  aws_s3_region = <region>

  # 例: https://storage.googleapis.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <HMAC access_key>
  aws_s3_secret_key = <HMAC secret_key>
  ```

### GCSと共有ストレージに関連するすべてのFEパラメータ

#### run_mode

StarRocksクラスタの実行モード。有効な値:

- `shared_data`
- `shared_nothing` (デフォルト)

> **注意**
>
> StarRocksクラスタでは、`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。ミックスデプロイはサポートされていません。
>
> クラスタをデプロイした後に`run_mode`を変更しないでください。そうしないと、クラスタの再起動に失敗します。共有しないクラスタから共有データクラスタまたはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFE設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成するかどうか。有効な値:

- `true` (デフォルト) 新しい共有データクラスタを作成する際にこの項目を`true`と指定すると、StarRocksはFE設定ファイルのオブジェクトストレージ関連のプロパティを使用して組み込みのストレージボリューム`builtin_storage_volume`を作成し、デフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動できません。
- `false` 新しい共有データクラスタを作成する際にこの項目を`false`と指定すると、StarRocksは組み込みのストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、ストレージボリュームを手動で作成し、デフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0以降でサポートされています。

> **注意**
>
> 既存の共有データクラスタをv3.0からアップグレードする場合、この項目を`true`のままにしておくことを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルに互換性のあるオブジェクトストレージ（AWS S3、Google GCS、MinIOなど）にデータを保存することができます。有効な値:

- `S3` (デフォルト)
- `AZBLOB`

#### aws_s3_path

データを保存するために使用するS3パス。S3バケットの名前とそれに含まれるサブパス（ある場合）で構成されます。例: `testbucket/subpath`

#### aws_s3_endpoint

S3バケットにアクセスするために使用するエンドポイント。例: `https://storage.googleapis.com/`

#### aws_s3_region

S3バケットが存在するリージョン。例: `us-west-2`

#### aws_s3_use_instance_profile

GCSへのアクセスにおいて、インスタンスプロファイルとアサムドロールを認証方法として使用するかどうか。有効な値:

- `true`
- `false` (デフォルト)

IAMユーザーベースの認証（アクセスキーとシークレットキー）を使用してGCSにアクセスする場合は、この項目を`false`と指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

インスタンスプロファイルを使用してGCSにアクセスする場合は、この項目を`true`と指定する必要があります。

アサムドロールを使用してGCSにアクセスする場合は、この項目を`true`と指定し、`aws_s3_iam_role_arn`を指定する必要があります。

外部のAWSアカウントを使用する場合は、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

GCSバケットにアクセスするために使用するHMACアクセスキーID。

#### aws_s3_secret_key

GCSバケットにアクセスするために使用するHMACシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが格納されているGCSバケットに特権を持つIAMロールのARN。

#### aws_s3_external_id

GCSバケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

> **注意**
>
> 共有データStarRocksクラスタを作成した後は、認証関連の設定項目のみを変更できます。元のストレージパス関連の設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

クラスタが作成された後に手動でデフォルトのストレージボリュームを作成する場合は、次の設定項目のみを追加する必要があります。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードの設定
<SharedDataCNconf />

## 共有データStarRocksクラスタの使用方法

<SharedDataUseIntro />

次の例では、HMACアクセスキーとシークレットキーを使用してGCSバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、ストレージボリュームを有効にし、デフォルトのストレージボリュームとして設定しています。

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-east1",
    "aws.s3.endpoint" = "https://storage.googleapis.com",
    "aws.s3.access_key" = "<HMAC access key>",
    "aws.s3.secret_key" = "<HMAC secret key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
