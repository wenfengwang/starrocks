---
displayed_sidebar: "Japanese"
---

# 共有データ用にAzure Blob Storageを使用する

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![共有データのアーキテクチャ](../../assets/share_data_arch.png)

## 共有データStarRocksクラスターをデプロイする

共有データStarRocksクラスターのデプロイは、共有無しStarRocksクラスターのものと類似しています。唯一の違いは、共有データクラスターでは、BEの代わりにCNをデプロイする必要がある点です。このセクションでは、共有データStarRocksクラスターをデプロイする際にFEとCNの構成ファイルで追加する必要がある項目の詳細を示します。StarRocksクラスターのデプロイの詳細な手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

> **注意**
>
> 次のドキュメントのセクションで共有ストレージ用に構成されるまで、クラスターを開始しないでください。

## 共有データStarRocksのFEノードを構成する

クラスターの開始前に、FEおよびCNを構成してください。以下に示す具体的な例の構成を提供し、その後に各パラメータの詳細を提供します。

### Azure Blob Storage用のFEの構成例

`fe.conf`に追加される共有データの例は、各FEノード上の`fe.conf`ファイルに追加できます。

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
> Azure Blob Storageアカウントを作成する際に、階層型名前空間を無効にする必要があります。

### Azure Blob Storageと共有するすべてのFEパラメータ

#### run_mode

StarRocksクラスターの実行モード。有効な値:

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注意**
>
> StarRocksクラスターに対して`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> クラスターがデプロイされた後は`run_mode`を変更しないでください。それ以外の場合、クラスターの再起動に失敗します。共有無しクラスターから共有データクラスターやその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト: `6090`

#### enable_load_volume_from_conf

StarRocksに、FE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用して、デフォルトのストレージボリュームを作成することを許可するかどうか。有効な値:

- `true`（デフォルト）: 新しい共有データクラスターを作成する際に、この項目を`true`と指定すると、StarRocksはFE構成ファイル内のオブジェクトストレージ関連のプロパティを使用して、組み込みストレージボリューム`builtin_storage_volume`を作成し、デフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連のプロパティが指定されていない場合、StarRocksは起動に失敗します。
- `false`：新しい共有データクラスターを作成する際に、この項目を`false`と指定すると、StarRocksは組み込みストレージボリュームを作成せずに直接起動します。StarRocks内の任意のオブジェクトを作成する前に、ストレージボリュームを手動で作成し、デフォルトのストレージボリュームとして設定する必要があります。詳細については、[デフォルトストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0以降でサポート。

> **注意**
>
> 既存のv3.0からの共有データクラスターをアップグレードする際に、この項目を`true`のままにすることを強くお勧めします。この項目を`false`と指定すると、変更前に作成したデータベースとテーブルが読み取り専用になり、それらにデータをロードすることができなくなります。

#### cloud_native_storage_type

使用するオブジェクトストレージの種類。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）、およびS3プロトコルに互換性のあるオブジェクトストレージ（AWS S3、Google GCP、MinIOなど）でデータを保存できます。有効な値:

- `S3`（デフォルト）
- `AZBLOB`。

> 注意
>
> このパラメータを`S3`と指定する場合は、`aws_s3`で接頭辞が付いたパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`と指定する場合は、`azure_blob`で接頭辞が付いたパラメータを追加する必要があります。

#### azure_blob_path

データを保存するために使用するAzure Blob Storageのパス。これは、ストレージアカウント内のコンテナの名前と、コンテナ内のサブパス（ある場合）からなります。たとえば、`testcontainer/subpath`となります。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。たとえば、`https://test.blob.core.windows.net`です。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを承認するために使用される共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを承認するために使用される共有アクセス署名（SAS）。

> **注意**
>
> 共有データStarRocksクラスターが作成された後は、資格情報関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースとテーブルが読み取り専用になり、それらにデータをロードすることができません。

クラスターが作成された後にデフォルトのストレージボリュームを手動で作成する場合は、以下の構成項目を追加するだけです:

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードを構成する

<SharedDataCNconf />

## 共有データStarRocksクラスターを使用する

<SharedDataUseIntro />

次の例では、共有キーアクセスを使用したAzure Blob Storageバケット`defaultbucket`のストレージボリューム`def_volume`を作成し、そのストレージボリュームを有効にし、デフォルトのストレージボリュームとして設定しています:

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