---
displayed_sidebar: "Japanese"
---

# 共有データのためのAzure Blob Storageの使用

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データのアーキテクチャ](../../assets/share_data_arch.png)

## 共有データのStarRocksクラスタのデプロイ

共有データのStarRocksクラスタのデプロイは、共有しないStarRocksクラスタのデプロイと類似しています。唯一の違いは、共有データクラスタではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データのStarRocksクラスタをデプロイする際に、FEとCNの設定ファイル（**fe.conf**および**cn.conf**）に追加する必要がある追加のFEおよびCNの設定項目のみをリストアップしています。StarRocksクラスタをデプロイする詳しい手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注意**
>
> 次のセクションのドキュメントで共有ストレージに設定するまで、クラスタを開始しないでください。

## 共有データのStarRocksのFEノードの設定

クラスタを開始する前に、FEとCNの設定を行います。以下に例となる設定を提供し、その後、各パラメータの詳細を説明します。

### Azure Blob StorageのためのFE設定の例

`fe.conf`の共有データの追加例は、各FEノードの`fe.conf`ファイルに追加できます。

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例：testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例：https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_shared_key = <shared_key>
  ```

- Azure Blob Storageへのアクセスに共有アクセス署名（SAS）を使用する場合、次の設定項目を追加してください。

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例：testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例：https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_sas_token = <sas_token>
  ```

> **注意**
>
> Azure Blob Storageアカウントを作成する際に、階層型名前空間を無効にする必要があります。

### Azure Blob Storageと共有ストレージに関連するすべてのFEパラメータ

#### run_mode

StarRocksクラスタの実行モード。有効な値:

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注意**
>
> StarRocksクラスタでは、`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。ミックスデプロイはサポートされていません。
>
> クラスタをデプロイした後に`run_mode`を変更しないでください。そうしないと、クラスタの再起動に失敗します。共有データクラスタから共有しないクラスタまたはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksがFE設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成するかどうか。有効な値:

- `true`（デフォルト）: 新しい共有データクラスタを作成する際にこの項目を`true`と指定すると、StarRocksはFE設定ファイルのオブジェクトストレージ関連のプロパティを使用して組み込みのストレージボリューム`builtin_storage_volume`を作成し、デフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティを指定していない場合、StarRocksは起動できません。
- `false`: 新しい共有データクラスタを作成する際にこの項目を`false`と指定すると、StarRocksは組み込みのストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、ストレージボリュームを手動で作成し、デフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0以降でサポートされています。

> **注意**
>
> 既存の共有データクラスタをv3.0からアップグレードする場合、この項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）、およびS3プロトコルに互換性のあるオブジェクトストレージ（AWS S3、Google GCP、MinIOなど）にデータを保存することができます。有効な値:

- `S3`（デフォルト）
- `AZBLOB`.

> 注意
>
> このパラメータを`S3`と指定する場合、`aws_s3`で始まるパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`と指定する場合、`azure_blob`で始まるパラメータを追加する必要があります。

#### azure_blob_path

データを保存するために使用するAzure Blob Storageのパスです。ストレージアカウント内のコンテナの名前と、コンテナ内のサブパス（ある場合）から構成されます。例：`testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイントです。例：`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを承認するために使用される共有キーです。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを承認するために使用される共有アクセス署名（SAS）です。

> **注意**
>
> 共有データのStarRocksクラスタが作成された後は、認証関連の設定項目のみを変更できます。元のストレージパス関連の設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

クラスタが作成された後にデフォルトのストレージボリュームを手動で作成する場合は、次の設定項目のみを追加する必要があります。

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データのStarRocksのCNノードの設定

<SharedDataCNconf />

## 共有データのStarRocksクラスタの使用方法

<SharedDataUseIntro />

以下の例では、Azure Blob Storageバケット`defaultbucket`のストレージボリューム`def_volume`を共有キーアクセスで作成し、ストレージボリュームを有効にし、デフォルトのストレージボリュームとして設定しています。

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = AZBLOB
LOCATIONS = ("azblob://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
