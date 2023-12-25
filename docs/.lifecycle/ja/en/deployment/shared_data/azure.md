---
displayed_sidebar: English
---

# Azure Blob Storageを共有データとして使用する

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスタをデプロイする

共有データStarRocksクラスタのデプロイは、共有ノーシングStarRocksクラスタのデプロイと似ています。唯一の違いは、共有データクラスタではBEではなくCNをデプロイする必要があることです。このセクションでは、共有データStarRocksクラスタをデプロイする際に、FEおよびCNの設定ファイル**fe.conf**および**cn.conf**に追加する必要がある追加のFEおよびCN設定項目のみをリストします。StarRocksクラスタのデプロイの詳細については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注記**
>
> このドキュメントの次のセクションで共有ストレージ用に設定されるまで、クラスタを起動しないでください。

## 共有データStarRocksのFEノードを構成する

クラスタを開始する前に、FEとCNを設定します。以下に構成の例を示し、その後で各パラメータの詳細を提供します。

### Azure Blob Storage用のFE構成例

共有データの追加例は、各FEノードの`fe.conf`ファイルに追加できます。

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

- Shared Access Signature (SAS)を使用してAzure Blob Storageにアクセスする場合は、以下の設定項目を追加します：

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

> **警告**
>
> Azure Blob Storageアカウントを作成する際には、階層型名前空間を無効にする必要があります。

### Azure Blob Storageと共有ストレージに関連するすべてのFEパラメータ

#### run_mode

StarRocksクラスタの実行モード。有効な値：

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注記**
>
> StarRocksクラスタに対して`shared_data`と`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> クラスタがデプロイされた後に`run_mode`を変更しないでください。そうすると、クラスタは再起動に失敗します。共有ノーシングクラスタから共有データクラスタへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスRPCポート。

- デフォルト：`6090`

#### enable_load_volume_from_conf

StarRocksがFE設定ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。有効な値：

- `true`（デフォルト）新しい共有データクラスタを作成する際にこの項目を`true`として指定すると、StarRocksはFE設定ファイルのオブジェクトストレージ関連のプロパティを使用してビルトインストレージボリューム`builtin_storage_volume`を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティが指定されていない場合、StarRocksは起動に失敗します。
- `false`新しい共有データクラスタを作成する際にこの項目を`false`として指定すると、StarRocksはビルトインストレージボリュームを作成せずに直接起動します。StarRocksでオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトのストレージボリュームを作成する](#use-your-shared-data-starrocks-cluster)を参照してください。

v3.1.0からサポート。

> **警告**
>
> 既存の共有データクラスタをv3.0からアップグレードする際には、この項目を`true`のままにしておくことを強く推奨します。この項目を`false`に設定すると、アップグレード前に作成したデータベースとテーブルが読み取り専用になり、データをロードできなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）、およびS3プロトコルと互換性のあるオブジェクトストレージ（AWS S3、Google GCP、MinIOなど）でのデータ保存をサポートしています。有効な値：

- `S3`（デフォルト）
- `AZBLOB`

> **注記**
>
> このパラメータを`S3`として指定する場合、`aws_s3`で始まるパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`として指定する場合、`azure_blob`で始まるパラメータを追加する必要があります。

#### azure_blob_path

データを保存するためのAzure Blob Storageのパス。これは、ストレージアカウント内のコンテナ名と、コンテナの下のサブパス（ある場合）で構成されています。例：`testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例：`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを認証するために使用される共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを認証するために使用されるShared Access Signature（SAS）。

> **注記**
>
> 共有データStarRocksクラスタが作成された後に変更できるのは、認証情報に関連する設定項目のみです。元のストレージパスに関連する設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用になり、データをロードできなくなります。

クラスタが作成された後にデフォルトのストレージボリュームを手動で作成する場合は、以下の設定項目のみを追加する必要があります：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードを構成する

<SharedDataCNconf />

## 共有データStarRocksクラスタを使用する

<SharedDataUseIntro />

次の例では、共有キーアクセスを使用するAzure Blob Storageバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、ストレージボリュームを有効にして、それをデフォルトのストレージボリュームとして設定します：

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
