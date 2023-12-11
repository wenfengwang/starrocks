---
displayed_sidebar: "Japanese"
---

# GCSを使用したStarRocksのデプロイ

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データのアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスタのデプロイ

共有データStarRocksクラスタのデプロイは、共有しないStarRocksクラスタのデプロイと類似しています。唯一の違いは、共有データクラスタではBEの代わりにCNをデプロイする必要がある点です。このセクションでは、共有データStarRocksクラスタのデプロイ時にFEとCNの**fe.conf**および**cn.conf**の構成ファイルに追加する必要のある追加の設定項目のみをリストしています。StarRocksクラスタのデプロイの詳細な手順については、「[StarRocksのデプロイ](../../deployment/deploy_manually.md)」を参照してください。

> **注意**
>
> 次のドキュメントのセクションで共有ストレージ用に設定されるまで、クラスタを起動しないでください。

## 共有データStarRocksのFEノードの構成

クラスタの起動前に、FEおよびCNの構成を行います。以下に構成の例を示し、それぞれのパラメータの詳細を提供します。

### GCSのためのFE構成の例

GCSストレージは、[Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview)を使用してアクセスされるため、パラメータには`aws_s3`プレフィックスが使用されます。

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

  aws_s3_access_key = <HMACアクセスキー>
  aws_s3_secret_key = <HMACシークレットキー>
  ```

### GCSと共有ストレージに関連するすべてのFEパラメータ

#### run_mode

StarRocksクラスタの実行モード。有効な値: 

- `shared_data`
- `shared_nothing` (デフォルト)。

> **注意**
>
> StarRocksクラスタで`shared_data`と`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> クラスタのデプロイ後に`run_mode`を変更しないでください。さもなければ、クラスタが再起動に失敗します。共有しないクラスタから共有データクラスタ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトストレージボリュームとして設定することを許可するかどうか。有効な値:

- `true` (デフォルト) この項目を`true`と指定して新しい共有データクラスタを作成する場合、StarRocksはFE構成ファイル内のオブジェクトストレージ関連のプロパティを使用して組込みストレージボリューム`builtin_storage_volume`を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティが指定されていない場合、StarRocksは起動に失敗します。
- `false` この項目を`false`と指定して新しい共有データクラスタを作成する場合、StarRocksはデフォルトのストレージボリュームを作成せずに直接起動します。StarRocks内のオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、「[デフォルトのストレージボリュームを作成する](#create-default-storage-volume)」を参照してください。

v3.1.0からサポートされています。

> **注意**
>
> 既存のv3.0にアップグレード中はこの項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコル互換のオブジェクトストレージ（AWS S3、Google GCS、MinIOなど）にデータを保存することができます。有効な値:

- `S3` (デフォルト)
- `AZBLOB`

#### aws_s3_path

データを保存するために使用されるS3パス。それは、S3バケットの名前と、それに属するサブパス（ある場合）から構成されます。例: `testbucket/subpath`

#### aws_s3_endpoint

S3バケットにアクセスするために使用されるエンドポイント。例: `https://storage.googleapis.com/`

#### aws_s3_region

S3バケットが存在するリージョン。例: `us-east1`

#### aws_s3_use_instance_profile

GCSへのアクセス時にインスタンスプロファイルと仮定されるロールを資格情報の方法として使用するかどうか。有効な値:

- `true`
- `false` (デフォルト)

GCSへのアクセスにIAMユーザーベースの資格情報（アクセスキーおよびシークレットキー）を使用する場合は、この項目を`false`と指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

GCSへのアクセスにインスタンスプロファイルを使用する場合は、この項目を`true`と指定する必要があります。

GCSへのアクセスに仮定されるロールを使用する場合は、この項目を`true`と指定し、`aws_s3_iam_role_arn`を指定する必要があります。また、外部AWSアカウントを使用する場合は、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

GCSバケットにアクセスするために使用されるHMACアクセスキーID。

#### aws_s3_secret_key

GCSバケットにアクセスするために使用されるHMACシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが保存されているGCSバケットに権限のあるIAMロールのARN。

#### aws_s3_external_id

GCSバケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

> **注意**
>
> 共有データのStarRocksクラスタを作成した後は、資格情報関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

クラスタの作成後、デフォルトのストレージボリュームを手動で作成したい場合は、次の構成項目を追加するだけです:

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードの構成
<SharedDataCNconf />

## 共有データStarRocksクラスタの利用方法

<SharedDataUseIntro />

次の例は、HMACアクセスキーとシークレットキーを持つGCSバケット `defaultbucket` にストレージボリューム`def_volume`を作成し、それを有効にしてデフォルトのストレージボリュームとして設定するものです:

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-east1",
    "aws.s3.endpoint" = "https://storage.googleapis.com",
    "aws.s3.access_key" = "<HMACアクセスキー>",
    "aws.s3.secret_key" = "<HMACシークレットキー>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
