---
displayed_sidebar: English
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

共有データStarRocksクラスターのデプロイは、共有ノーシングStarRocksクラスターのデプロイと似ています。唯一の違いは、共有データクラスターではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスターをデプロイする際に、FEおよびCNの設定ファイル**fe.conf**および**cn.conf**に追加する必要がある追加のFEおよびCNの設定項目のみをリストします。StarRocksクラスターのデプロイの詳細については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注記**
>
> このドキュメントの次のセクションで共有ストレージ用に設定されるまで、クラスターを起動しないでください。

## 共有データStarRocksのFEノードを設定する

クラスターを開始する前に、FEとCNを設定します。以下に構成の例を示し、その後で各パラメーターの詳細を提供します。

### GCS用のFE構成例

`fe.conf`ファイルに追加できる共有データの追加例は、各FEノードの`fe.conf`ファイルに追加できます。GCSストレージは[Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview)を使用してアクセスされるため、パラメーターは`aws_s3`プレフィックスを使用します。

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

### GCSとの共有ストレージに関連するすべてのFEパラメーター

#### run_mode

StarRocksクラスターの実行モード。有効な値:

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注記**
>
> StarRocksクラスターでは、`shared_data`と`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> クラスターがデプロイされた後に`run_mode`を変更しないでください。そうすると、クラスターは再起動に失敗します。共有ノーシングクラスターから共有データクラスターへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFE設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。有効な値:

- `true`（デフォルト）新しい共有データクラスターを作成するときにこの項目を`true`として指定すると、StarRocksはFE設定ファイルのオブジェクトストレージ関連のプロパティを使用してビルトインストレージボリューム`builtin_storage_volume`を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動に失敗します。
- `false`新しい共有データクラスターを作成するときにこの項目を`false`として指定すると、StarRocksはビルトインストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームを作成する](#use-your-shared-data-starrocks-cluster)を参照してください。

v3.1.0からサポートされています。

> **警告**
>
> 既存の共有データクラスターをv3.0からアップグレードする間は、この項目を`true`のままにしておくことを強く推奨します。この項目を`false`に設定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）、およびS3プロトコルと互換性のあるオブジェクトストレージ（AWS S3、Google GCS、MinIOなど）でのデータの保存をサポートしています。有効な値:

- `S3`（デフォルト）
- `AZBLOB`

#### aws_s3_path

データを保存するためのS3パス。これは、S3バケットの名前とその下のサブパス（ある場合）で構成されています。例: `testbucket/subpath`。

#### aws_s3_endpoint

S3バケットにアクセスするためのエンドポイント。例: `https://storage.googleapis.com`

#### aws_s3_region

S3バケットが存在するリージョン。例: `us-east1`

#### aws_s3_use_instance_profile

GCSへのアクセスにインスタンスプロファイルと仮定されたロールを認証方法として使用するかどうか。有効な値:

- `true`
- `false`（デフォルト）

IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してGCSにアクセスする場合、この項目を`false`に設定し、`aws_s3_access_key`と`aws_s3_secret_key`を指定する必要があります。

インスタンスプロファイルを使用してGCSにアクセスする場合、この項目を`true`に設定する必要があります。

仮定されたロールを使用してGCSにアクセスする場合、この項目を`true`に設定し、`aws_s3_iam_role_arn`を指定する必要があります。

外部AWSアカウントを使用する場合は、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

GCSバケットにアクセスするためのHMACアクセスキーID。

#### aws_s3_secret_key

GCSバケットにアクセスするためのHMACシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが保存されているGCSバケットに権限を持つIAMロールのARN。

#### aws_s3_external_id

GCSバケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

> **注記**
>
> 共有データStarRocksクラスターが作成された後に変更できるのは、認証情報に関連する設定項目のみです。元のストレージパスに関連する設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができなくなります。

クラスターが作成された後にデフォルトのストレージボリュームを手動で作成する場合は、以下の設定項目のみを追加する必要があります。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードを設定する

<SharedDataCNconf />

## 共有データStarRocksクラスターを使用する

<SharedDataUseIntro />

次の例では、HMACアクセスキーとシークレットキーを使用してGCSバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、ストレージボリュームを有効にして、それをデフォルトのストレージボリュームとして設定します。

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
