---
displayed_sidebar: Chinese
---

# HDFS をベースにしたデプロイメント

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## システムアーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## StarRocks ストレージとコンピューティングの分離クラスターのデプロイ

StarRocks のストレージとコンピューティングの分離クラスターのデプロイ方法は、統合型クラスターのデプロイ方法と似ていますが、分離クラスターでは CN ノードをデプロイする必要があり、BE ノードは不要です。このセクションでは、StarRocks のストレージとコンピューティングの分離クラスターをデプロイする際に、FE と CN の設定ファイル **fe.conf** と **cn.conf** に追加する必要がある追加の設定項目のみをリストします。StarRocks クラスターのデプロイに関する詳細な説明については、[StarRocks のデプロイ](../deploy_manually.md)を参照してください。

> **注意**
>
> クラスターの設定が完了する前にクラスターを起動しないでください。

## ストレージとコンピューティングの分離デプロイメントのための FE 設定

### FE 設定の例

すべての FE ノードの設定ファイル **fe.conf** に以下の設定項目を追加します：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
cloud_native_storage_type = HDFS

# 例えば hdfs://127.0.0.1:9000/user/starrocks/
cloud_native_hdfs_url = <hdfs_url>
```

クラスターを作成した後にデフォルトのストレージボリュームを手動で作成したい場合は、以下の設定項目のみを追加します：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

### FE 設定の説明

#### run_mode

StarRocks クラスターの実行モード。有効な値：

- `shared_data`：ストレージとコンピューティングの分離モードで StarRocks を実行します。
- `shared_nothing`（デフォルト）：統合型モードで StarRocks を実行します。

> **説明**
>
> - StarRocks クラスターは、ストレージとコンピューティングの分離モードと統合型モードの混在デプロイをサポートしていません。
> - クラスターのデプロイが完了した後に `run_mode` を変更しないでください。そうすると、クラスターは再起動できなくなります。統合型クラスターから分離クラスターへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブのメタデータサービスのリスニングポート。

- デフォルト値：`6090`

#### enable_load_volume_from_conf

StarRocks が FE の設定ファイルに指定されたストレージ関連の属性を使用してデフォルトのストレージボリュームを作成することを許可するかどうか。バージョン 3.1.0 からサポートされています。有効な値：

- `true`（デフォルト）：新しいストレージとコンピューティングの分離クラスターを作成する際にこの項目を `true` に設定すると、StarRocks は FE の設定ファイルに指定されたストレージ関連の属性を使用して組み込みのストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、ストレージ関連の属性を指定していない場合、StarRocks は起動できません。
- `false`：新しいストレージとコンピューティングの分離クラスターを作成する際にこの項目を `false` に設定すると、StarRocks は直接起動し、組み込みのストレージボリュームは作成しません。StarRocks で任意のオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#使用-starrocks-存算分离集群)を参照してください。

> **注意**
>
> 既存のバージョン 3.0 のストレージとコンピューティングの分離クラスターをアップグレードする際には、この項目をデフォルトの設定 `true` のままにしておくことをお勧めします。この項目を `false` に変更すると、アップグレード前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

#### cloud_native_storage_type

使用しているストレージタイプ。ストレージとコンピューティングの分離モードでは、StarRocks は HDFS、Azure Blob（バージョン 3.1.1 からサポート）、および S3 プロトコルと互換性のあるオブジェクトストレージ（例：AWS S3、Google GCP、アリババクラウド OSS、MinIO）にデータを保存することをサポートしています。有効な値：

- `S3`（デフォルト値）
- `AZBLOB`
- `HDFS`

> **説明**
>
> - この項目を `S3` に設定する場合は、`aws_s3` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `AZBLOB` に設定する場合は、`azure_blob` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `HDFS` に設定する場合は、`cloud_native_hdfs_url` を指定するだけで十分です。

#### cloud_native_hdfs_url

HDFS ストレージの URL、例えば `hdfs://127.0.0.1:9000/user/xxx/starrocks/`。

> **注意**
>
> ストレージとコンピューティングの分離クラスターを正常に作成した後、セキュリティ資格情報に関連する設定項目のみを変更することができます。元のストレージパスに関連する設定項目を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

## ストレージとコンピューティングの分離デプロイメントのための CN 設定

<SharedDataCNconf />

## StarRocks ストレージとコンピューティングの分離クラスターの使用

<SharedDataUseIntro />

以下の例では、HDFS ストレージに `def_volume` というストレージボリュームを作成し、それをアクティブにしてデフォルトのストレージボリュームとして設定します：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = HDFS
LOCATIONS = ("hdfs://127.0.0.1:9000/user/starrocks/");

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
