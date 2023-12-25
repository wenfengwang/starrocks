---
displayed_sidebar: Chinese
---

# Azure Blob をベースにしたデプロイメント

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## システムアーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## StarRocks ストレージと計算の分離クラスターのデプロイメント

StarRocks のストレージと計算の分離クラスターのデプロイメント方法は、ストレージと計算が統合されたクラスターのデプロイメント方法と似ていますが、ストレージと計算の分離クラスターでは CN ノードをデプロイする必要があり、BE ノードは不要です。このセクションでは、StarRocks のストレージと計算の分離クラスターをデプロイする際に FE と CN の設定ファイル **fe.conf** と **cn.conf** に追加する必要がある追加の設定項目のみをリストします。StarRocks クラスターのデプロイメントに関する詳細な説明については、[StarRocks のデプロイメント](../deploy_manually.md)を参照してください。

> **注意**
>
> クラスターの設定が完了する前に起動しないでください。

## ストレージと計算の分離デプロイメントのための FE 設定

### FE 設定例

異なる認証方式には異なる FE パラメータが必要です。以下の各セクションの例を参考に、ビジネスニーズに応じた設定を行ってください。

クラスター作成後にデフォルトのストレージボリュームを手動で作成する場合は、以下の設定項目のみを追加する必要があります：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

#### Shared Key を使用して Azure Blob にアクセス

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

#### 共有アクセス署名（SAS）を使用して Azure Blob にアクセス

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
> Azure Blob Storage アカウントを作成する際は、階層型の名前空間を無効にする必要があります。

### FE 設定の説明

#### run_mode

StarRocks クラスターの実行モード。有効な値：

- `shared_data`：ストレージと計算の分離モードで StarRocks を実行します。
- `shared_nothing`（デフォルト）：ストレージと計算が統合されたモードで StarRocks を実行します。

> **説明**
>
> - StarRocks クラスターは、ストレージと計算の分離モードと統合モードを混在させてのデプロイメントはサポートしていません。
> - クラスターのデプロイメントが完了した後に `run_mode` を変更しないでください。そうするとクラスターは再起動できなくなります。ストレージと計算が統合されたクラスターから分離モードへの変更、またはその逆はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブのメタデータサービスがリッスンするポート。

- デフォルト値：`6090`

#### enable_load_volume_from_conf

StarRocks が FE の設定ファイルに指定されたストレージ関連の属性を使用してデフォルトのストレージボリュームを作成することを許可するかどうか。バージョン 3.1.0 からサポートされています。有効な値：

- `true`（デフォルト）：新しいストレージと計算の分離クラスターを作成する際にこの項目を `true` に設定した場合、StarRocks は FE の設定ファイルにあるストレージ関連の属性を使用して組み込みのストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、ストレージ関連の属性を指定していない場合、StarRocks は起動できません。
- `false`：新しいストレージと計算の分離クラスターを作成する際にこの項目を `false` に設定した場合、StarRocks は直接起動し、組み込みのストレージボリュームは作成されません。StarRocks でオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#使用-starrocks-存算分离集群)を参照してください。

> **注意**
>
> 既存のバージョン 3.0 のストレージと計算の分離クラスターをアップグレードする場合は、この項目をデフォルトの設定 `true` のままにしておくことをお勧めします。この項目を `false` に変更すると、アップグレード前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

#### cloud_native_storage_type

使用するストレージのタイプ。ストレージと計算の分離モードでは、StarRocks は HDFS、Azure Blob（バージョン 3.1.1 からサポート）、および S3 プロトコルと互換性のあるオブジェクトストレージ（例：AWS S3、Google GCP、アリババクラウド OSS、MinIO）にデータを保存することをサポートしています。有効な値：

- `S3`（デフォルト値）
- `AZBLOB`
- `HDFS`

> **説明**
>
> - この項目を `S3` に設定する場合は、`aws_s3` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `AZBLOB` に設定する場合は、`azure_blob` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `HDFS` に設定する場合は、`cloud_native_hdfs_url` のみを指定する必要があります。

#### azure_blob_path

Azure Blob Storage でデータを保存するためのパス。Storage Account 内のコンテナ名とその下のサブパス（存在する場合）で構成されます。例：`testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storage のエンドポイント。例：`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storage にアクセスするための Shared Key。

#### azure_blob_sas_token

Azure Blob Storage にアクセスするための共有アクセス署名（SAS）。

> **注意**
>
> ストレージと計算の分離クラスターを正常に作成した後、セキュリティ資格情報に関連する設定項目のみを変更することができます。元のストレージパスに関連する設定項目を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

## ストレージと計算の分離デプロイメントのための CN 設定

<SharedDataCNconf />

## StarRocks ストレージと計算の分離クラスターの使用

<SharedDataUseIntro />

以下の例では、Shared Key 認証を使用して Azure Blob ストレージスペース `defaultbucket` にストレージボリューム `def_volume` を作成し、それをアクティブにしてデフォルトのストレージボリュームとして設定します：

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
