---
displayed_sidebar: Chinese
---

# MinIO をベースにしたデプロイメント

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## システムアーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## StarRocks のストレージと計算の分離クラスターのデプロイメント

StarRocks のストレージと計算の分離クラスターのデプロイメント方法は、ストレージと計算が統合されたクラスターのデプロイメント方法と似ていますが、ストレージと計算の分離クラスターでは CN ノードをデプロイする必要があり、BE ノードは不要です。このセクションでは、StarRocks のストレージと計算の分離クラスターをデプロイする際に、FE と CN の設定ファイル **fe.conf** および **cn.conf** に追加する必要がある追加の設定項目のみをリストしています。StarRocks クラスターのデプロイメントに関する詳細な説明については、[StarRocks のデプロイメント](../deploy_manually.md)を参照してください。

> **注意**
>
> クラスターの設定が完了する前に、クラスターを起動しないでください。

## ストレージと計算の分離デプロイメントのための FE 設定

### FE 設定例

StarRocks は [AWS Signature Version 4 protocol](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html) を使用して MinIO にアクセスするため、`aws_s3` をプレフィックスとする設定項目を設定する必要があります。すべての FE ノードの設定ファイル **fe.conf** に以下の設定項目を追加します：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例: testbucket/subpath
  aws_s3_path = <s3_path>

  # 例: us-east1
  aws_s3_region = <region>

  # 例: http://172.26.xx.xxx:39000
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <minio_access_key>
  aws_s3_secret_key = <minio_secret_key>
  ```

  クラスター作成後にデフォルトのストレージボリュームを手動で作成したい場合は、以下の設定項目のみを追加します：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  enable_load_volume_from_conf = false
  ```


### FE 設定の説明

#### run_mode

StarRocks クラスターの実行モード。有効な値：

- `shared_data`：ストレージと計算の分離モードで StarRocks を実行します。
- `shared_nothing`（デフォルト）：ストレージと計算が統合されたモードで StarRocks を実行します。

> **説明**
>
> - StarRocks クラスターは、ストレージと計算の分離モードと統合モードを混在させてデプロイすることはできません。
> - クラスターのデプロイメントが完了した後で `run_mode` を変更しないでください。そうすると、クラスターを再起動できなくなります。ストレージと計算が統合されたクラスターから分離モードへの変更、またはその逆はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブのメタデータサービスがリッスンするポート。

- デフォルト値：`6090`

#### enable_load_volume_from_conf

StarRocks が FE の設定ファイルに指定されたストレージ関連の属性を使用してデフォルトのストレージボリュームを作成することを許可するかどうか。バージョン 3.1.0 からサポートされています。有効な値：

- `true`（デフォルト）：新しいストレージと計算の分離クラスターを作成する際にこの項目を `true` に設定すると、StarRocks は FE の設定ファイルに指定されたストレージ関連の属性を使用して組み込みのストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、ストレージ関連の属性を指定していない場合、StarRocks は起動できません。
- `false`：新しいストレージと計算の分離クラスターを作成する際にこの項目を `false` に設定すると、StarRocks は直接起動し、組み込みのストレージボリュームは作成されません。StarRocks で任意のオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#使用-starrocks-存算分离集群)を参照してください。

> **注意**
>
> 既存のバージョン 3.0 のストレージと計算の分離クラスターをアップグレードする際は、この項目をデフォルトの設定 `true` のままにしておくことをお勧めします。この項目を `false` に変更すると、アップグレード前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

#### cloud_native_storage_type

使用するストレージタイプ。ストレージと計算の分離モードでは、StarRocks はデータを HDFS、Azure Blob（バージョン 3.1.1 からサポート）、および S3 プロトコルと互換性のあるオブジェクトストレージ（例：AWS S3、Google GCP、アリババクラウド OSS、MinIO）に保存することをサポートしています。有効な値：

- `S3`（デフォルト値）
- `AZBLOB`
- `HDFS`

> **説明**
>
> - この項目を `S3` に設定する場合は、`aws_s3` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `AZBLOB` に設定する場合は、`azure_blob` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `HDFS` に設定する場合は、`cloud_native_hdfs_url` のみを指定する必要があります。

#### aws_s3_path

MinIO ストレージスペースのパスで、MinIO バケットの名前とその下のサブパス（存在する場合）で構成されています。例：`testbucket/subpath`。

#### aws_s3_endpoint

MinIO ストレージスペースにアクセスするためのエンドポイントの URL。例：`http://172.26.xx.xxx:39000`。

#### aws_s3_region

アクセスする MinIO ストレージスペースのリージョン。例：`us-east1`。

#### aws_s3_access_key

MinIO ストレージスペースにアクセスするためのアクセスキー。

#### aws_s3_secret_key

MinIO ストレージスペースにアクセスするためのシークレットキー。

> **注意**
>
> ストレージと計算の分離クラスターを正常に作成した後は、セキュリティ資格情報に関連する設定項目のみを変更することができます。元のストレージパスに関連する設定項目を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

## ストレージと計算の分離デプロイメントのための CN 設定

<SharedDataCNconf />

## StarRocks のストレージと計算の分離クラスターの使用

<SharedDataUseIntro />

以下の例では、Access Key と Secret Key を使用して MinIO ストレージスペース `defaultbucket` にストレージボリューム `def_volume` を作成し、それをアクティブにしてデフォルトのストレージボリュームとして設定します：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-east1",
    "aws.s3.endpoint" = "http://172.26.xx.xxx:39000",
    "aws.s3.access_key" = "<minio_access_key>",
    "aws.s3.secret_key" = "<minio_secret_key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
