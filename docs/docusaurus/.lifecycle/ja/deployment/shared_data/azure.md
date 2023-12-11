---
displayed_sidebar: "Japanese"
---

# 共有データ用のAzure Blob Storageの使用

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスターのデプロイ

共有データStarRocksクラスターのデプロイは共有データのないStarRocksクラスターのそれと類似しています。唯一の違いは、共有データクラスターではBEの代わりにCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスターをデプロイする際にFEとCNの構成ファイル**fe.conf**および**cn.conf**に追加する必要がある追加のFEおよびCN構成アイテムのみをリストアップしています。StarRocksクラスターのデプロイの詳細な手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注意**
>
> このドキュメントの次のセクションで共有ストレージに構成されるまで、クラスターを開始しないでください。

## 共有データStarRocksのFEノードの構成

クラスターを開始する前に、FEおよびCNを構成します。以下に例の構成を示し、その後に各パラメータの詳細を提供します。

### Azure Blob StorageのFEの例の構成

お使いの各FEノードの`fe.conf`ファイルに共有データの例の追加設定を追加できます。

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例: testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例: https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_shared_key = <shared_key>
  ```

- Azure Blob Storageへのアクセスに共有アクセス署名（SAS）を使用する場合、次の構成項目を追加します。

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例: testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例: https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_sas_token = <sas_token>
  ```

> **注意**
>
> Azure Blob Storageアカウントを作成する際には、階層ネームスペースを無効にする必要があります。

### Azure Blob Storageと共有するすべてのFEパラメータ

#### run_mode

StarRocksクラスターの実行モード。有効な値:

- `shared_data`
- `shared_nothing` （既定値）。

> **注意**
>
> StarRocksクラスターでは、同時に`shared_data`および`shared_nothing`モードを採用することはできません。混合デプロイはサポートされていません。
>
> クラスターをデプロイした後は`run_mode`を変更しないでください。そうしないと、クラスターの再起動に失敗します。共有データクラスターから共有データのないクラスターへの変換、またはその逆の変換はサポートされていません。

#### cloud_native_meta_port

Cloud NativeメタサービスのRPCポート。

- 既定値: `6090`

#### enable_load_volume_from_conf

StarRocksがFE構成ファイルで指定されたオブジェクトストレージ関連プロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。有効な値:

- `true` （既定値）新しい共有データクラスターを作成する際に、`true`としてこの項目を指定すると、StarRocksはFE構成ファイルのオブジェクトストレージ関連プロパティを使用して組み込みのストレージボリューム`builtin_storage_volume`を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連プロパティを指定していない場合、StarRocksは起動に失敗します。
- `false` 新しい共有データクラスターを作成する際に、`false`としてこの項目を指定すると、StarRocksはデフォルトのストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、ストレージボリュームを手動で作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0からサポートされています。

> **注意**
>
> 既存の共有データクラスターをv3.0からアップグレードする際には、この項目を`true`のままにすることを強くお勧めします。この項目を`false`として指定すると、アップグレード前に作成したデータベースとテーブルが読み取り専用になり、データをロードできなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルと互換性のあるオブジェクトストレージ（AWS S3、Google GCP、MinIOなど）でデータを保存できます。有効な値:

- `S3` （既定値）
- `AZBLOB`。

> **Note**
>
> このパラメータを`S3`と指定する場合は、`aws_s3`で前置されたパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`と指定する場合は、`azure_blob`で前置されたパラメータを追加する必要があります。

#### azure_blob_path

データを保存するために使用するAzure Blob Storageのパス。これは、ストレージアカウント内のコンテナの名前と、コンテナ内のサブパス（あれば）から構成されます。例: `testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例: `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを承認するために使用する共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを承認するために使用する共有アクセス署名（SAS）。

> **注意**
>
> 共有データのStarRocksクラスターを作成した後は、認証関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースとテーブルが読み取り専用になり、データをロードできなくなります。

クラスター作成後にデフォルトのストレージボリュームを手動で作成する場合は、次の構成項目のみを追加する必要があります:

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードの構成

<SharedDataCNconf />

## 共有データStarRocksクラスターの使用

<SharedDataUseIntro />

以下は、共有キーを使用したAzure Blob Storageバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、そのストレージボリュームを有効にし、デフォルトのストレージボリュームに設定する例です。

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