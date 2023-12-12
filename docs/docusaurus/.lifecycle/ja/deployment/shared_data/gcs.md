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

## 共有データStarRocksクラスターのデプロイ

共有データStarRocksクラスターのデプロイは、共有データではないStarRocksクラスターのデプロイと類似しています。唯一の違いは、共有データクラスターではBEの代わりにCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスターをデプロイする際に、FEおよびCNの**fe.conf**および**cn.conf**の設定ファイルに追加する必要がある追加のFEおよびCN構成項目のみをリストアップしています。StarRocksクラスターのデプロイの詳細については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注意**
>
> 次のセクションの文書で共有ストレージに構成されるまでクラスターを開始しないでください。

## 共有データStarRocksのFEノードを設定する

クラスターを開始する前に、FEおよびCNを構成します。例として以下に構成内容を示します。その後、各パラメータの詳細を提供します。

### GCSのためのFE構成例

あなたの各FEノードの`fe.conf`ファイルに`fe.conf`ファイルに共有データの追加例を追加できます。
GCSストレージは[Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview)を使用してアクセスされるため、パラメータは`aws_s3`接頭辞を使用しています。

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

### GCSと共有するすべてのFEパラメータ

#### run_mode

StarRocksクラスターの実行モード。有効な値:

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注意**
>
> StarRocksクラスターで`shared_data`と`shared_nothing`モードを同時に採用することはできません。混合デプロイはサポートされていません。
> クラスターの展開後に`run_mode`を変更しないでください。そうしないと、クラスターが再起動に失敗します。共有データクラスターから共有データではないクラスターやその逆に変換することはサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトストレージボリュームを作成することを許可するかどうか。有効な値:

- `true` (デフォルト) この項目を`true`と指定した場合、共有データクラスターを新たに作成する際、StarRocksはFE構成ファイルのオブジェクトストレージ関連のプロパティを使用して組み込みストレージボリューム`builtin_storage_volume`を作成し、デフォルトストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動に失敗します。
- `false` この項目を`false`と指定した場合、共有データクラスターを新たに作成する際、StarRocksは組み込みストレージボリュームを作成せずに直接起動します。StarRocksを作成する前にストレージボリュームを手動で作成し、それをデフォルトストレージボリュームとして設定する必要があります。詳細については、[デフォルトストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0からサポートされています。

> **注意**
>
> 既存の共有データクラスターをv3.0からアップグレードする間は、この項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースやテーブルは読み取り専用になり、データをロードすることができなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルに準拠したオブジェクトストレージにデータの保存をサポートします。有効な値:

- `S3` (デフォルト)
- `AZBLOB`

#### aws_s3_path

データを保存するために使用されるS3パス。あなたのS3バケットの名前と、それの下のサブパス（ある場合）で構成されています。

#### aws_s3_endpoint

S3バケットにアクセスするために使用されるエンドポイント。

#### aws_s3_region

S3バケットが存在するリージョン。

#### aws_s3_use_instance_profile

GCSへのアクセスにインスタンスプロファイルおよび仮定されたロールを資格情報方法として使用するかどうか。有効な値:

- `true`
- `false` (デフォルト)。

GCSにIAMユーザーベースの資格情報（アクセスキーおよびシークレットキー）を使用する場合、この項目を`false`と指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

GCSへのインスタンスプロファイルを使用する場合、この項目を`true`と指定する必要があります。

GCSへの仮定されたロールを使用する場合、この項目を`true`と指定し、`aws_s3_iam_role_arn`を指定する必要があります。

外部AWSアカウントを使用する場合、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

GCSバケットにアクセスするために使用されるHMACアクセスキーID。

#### aws_s3_secret_key

GCSバケットにアクセスするために使用されるHMACシークレットアクセスキー。

#### aws_s3_iam_role_arn

あなたのデータファイルが保存されているGCSバケットで特権を持つIAMロールのARN。

#### aws_s3_external_id

あなたのGCSバケットへのクロスアカウントアクセスに使用するAWSアカウントの外部ID。

> **注意**
>
> 共有データStarRocksクラスターを作成した後には、資格情報関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースやテーブルは読み取り専用になり、データをロードすることができなくなります。

クラスターが作成された後にデフォルトストレージボリュームを手動で作成する場合、次の構成項目のみを追加する必要があります:

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードを構成する
<SharedDataCNconf />

## 共有データStarRocksクラスターを使用する

<SharedDataUseIntro />

次の例は、HMACアクセスキーとシークレットキーを使用して、GCSバケット`defaultbucket`用のストレージボリューム`def_volume`を作成し、そのストレージボリュームを有効にし、デフォルトストレージボリュームとして設定するものです:

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