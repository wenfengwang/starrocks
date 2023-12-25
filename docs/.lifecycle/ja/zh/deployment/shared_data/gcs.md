---
displayed_sidebar: Chinese
---

# GCS をベースにしたデプロイメント

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## システムアーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## StarRocks ストレージとコンピューティングの分離クラスタのデプロイ

StarRocks ストレージとコンピューティングの分離クラスタのデプロイ方法は、ストレージとコンピューティングの統合クラスタのデプロイ方法に似ていますが、ストレージとコンピューティングの分離クラスタではCNノードをデプロイする必要があり、BEノードは不要です。このセクションでは、StarRocks ストレージとコンピューティングの分離クラスタをデプロイする際に、FE と CN の設定ファイル **fe.conf** と **cn.conf** に追加する必要がある追加の設定項目のみをリストしています。StarRocks クラスタのデプロイに関する詳細な説明については、[StarRocks のデプロイ](../deploy_manually.md)を参照してください。

> **注意**
>
> クラスタの設定が完了する前にクラスタを起動しないでください。

## ストレージとコンピューティングの分離デプロイメントのための FE 設定

### FE 設定例

StarRocks は [Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview) を介して GCS にアクセスするため、`aws_s3` をプレフィックスとする設定項目を設定する必要があります。すべての FE ノードの設定ファイル **fe.conf** に以下の設定項目を追加します：

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

  aws_s3_access_key = <HMAC_access_key>
  aws_s3_secret_key = <HMAC_secret_key>
  ```

  クラスタ作成後にデフォルトのストレージボリュームを手動で作成したい場合は、以下の設定項目のみを追加します：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  enable_load_volume_from_conf = false
  ```

### FE 設定の説明

#### run_mode

StarRocks クラスタの実行モード。有効な値：

- `shared_data`：ストレージとコンピューティングの分離モードで StarRocks を実行します。
- `shared_nothing`（デフォルト）：ストレージとコンピューティングの統合モードで StarRocks を実行します。

> **説明**
>
> - StarRocks クラスタは、ストレージとコンピューティングの分離モードと統合モードの混在デプロイをサポートしていません。
> - クラスタのデプロイが完了した後で `run_mode` を変更しないでください。そうすると、クラスタを再起動できなくなります。ストレージとコンピューティングの統合クラスタから分離クラスタへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタデータサービスのリスニングポート。

- デフォルト値：`6090`

#### enable_load_volume_from_conf

StarRocks が FE 設定ファイルに指定されたストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。v3.1.0 からサポートされています。有効な値：

- `true`（デフォルト）：新しいストレージとコンピューティングの分離クラスタを作成する際にこの項目を `true` に設定すると、StarRocks は FE 設定ファイルに指定されたストレージ関連のプロパティを使用して組み込みのストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、ストレージ関連のプロパティを指定していない場合、StarRocks は起動できません。
- `false`：新しいストレージとコンピューティングの分離クラスタを作成する際にこの項目を `false` に設定すると、StarRocks は直接起動し、組み込みのストレージボリュームは作成されません。StarRocks でオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#StarRocks-ストレージとコンピューティングの分離クラスタの使用)を参照してください。

> **注意**
>
> 既存の v3.0 ストレージとコンピューティングの分離クラスタをアップグレードする際は、この項目をデフォルト設定の `true` のままにすることをお勧めします。この項目を `false` に変更すると、アップグレード前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

#### cloud_native_storage_type

使用するストレージタイプ。ストレージとコンピューティングの分離モードでは、StarRocks は HDFS、Azure Blob（v3.1.1 からサポート）、および S3 プロトコルと互換性のあるオブジェクトストレージ（例：AWS S3、Google GCP、アリババクラウド OSS、MinIO）にデータを保存することをサポートしています。有効な値：

- `S3`（デフォルト値）
- `AZBLOB`
- `HDFS`

> **説明**
>
> - この項目を `S3` に設定する場合は、`aws_s3` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `AZBLOB` に設定する場合は、`azure_blob` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `HDFS` に設定する場合は、`cloud_native_hdfs_url` のみを指定する必要があります。

#### aws_s3_path

データを保存する GCS ストレージスペースのパス。GCS バケットの名前とその下のサブパス（存在する場合）で構成されます。例：`testbucket/subpath`。

#### aws_s3_endpoint

GCS ストレージスペースにアクセスするためのエンドポイント。例：`https://storage.googleapis.com`。

#### aws_s3_region

アクセスする GCS ストレージスペースのリージョン。例：`us-west-2`。

#### aws_s3_access_key

GCS ストレージスペースにアクセスするための HMAC Access Key。

#### aws_s3_secret_key

GCS ストレージスペースにアクセスするための HMAC Secret Key。

> **注意**
>
> ストレージとコンピューティングの分離クラスタを正常に作成した後、セキュリティ資格情報に関連する設定項目のみを変更することができます。元のストレージパスに関連する設定項目を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

## ストレージとコンピューティングの分離デプロイメントのための CN 設定

<SharedDataCNconf />

## StarRocks ストレージとコンピューティングの分離クラスタの使用

<SharedDataUseIntro />

以下の例では、HMAC Access Key および Secret Key を使用して、GCS ストレージスペース `defaultbucket` に `def_volume` ストレージボリュームを作成し、それをアクティブにしてデフォルトのストレージボリュームとして設定します：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-east1",
    "aws.s3.endpoint" = "https://storage.googleapis.com",
    "aws.s3.access_key" = "<HMAC_access_key>",
    "aws.s3.secret_key" = "<HMAC_secret_key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
