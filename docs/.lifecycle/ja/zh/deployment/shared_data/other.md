---
displayed_sidebar: Chinese
---

# 他のクラウドストレージをベースにしたデプロイメント

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## システムアーキテクチャ

![共有データアーキテクチャ](../../assets/share_data_arch.png)

## StarRocks ストレージ分離クラスタのデプロイ

StarRocks ストレージ分離クラスタのデプロイ方法は、ストレージ統合クラスタのデプロイ方法と似ていますが、ストレージ分離クラスタでは CN ノードをデプロイする必要があり、BE ノードは不要です。このセクションでは、StarRocks ストレージ分離クラスタをデプロイする際に FE と CN の設定ファイル **fe.conf** と **cn.conf** に追加する必要がある追加の設定項目のみをリストします。StarRocks クラスタのデプロイの詳細については、[StarRocks のデプロイ](../deploy_manually.md)を参照してください。

> **注意**
>
> クラスタの設定が完了する前にクラスタを起動しないでください。

## ストレージ分離デプロイ FE 設定

### FE 設定例

StarRocks は [AWS Signature Version 4 protocol](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html) を使用して S3 プロトコルと互換性のあるオブジェクトストレージにアクセスするため、`aws_s3` をプレフィックスとする設定項目を設定する必要があります。すべての FE ノードの設定ファイル **fe.conf** に以下の設定項目を追加します：

- 阿里云 OSS を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>

  # 例：cn-zhangjiakou
  aws_s3_region = <region>

  # 例：https://oss-cn-zhangjiakou-internal.aliyuncs.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

- 华为云 OBS を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>

  # 例：cn-north-4
  aws_s3_region = <region>

  # 例：https://obs.cn-north-4.myhuaweicloud.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

- 腾讯云 COS を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>

  # 例：ap-beijing
  aws_s3_region = <region>

  # 例：https://cos.ap-beijing.myqcloud.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

- 火山引擎 TOS を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>

  # 例：cn-beijing
  aws_s3_region = <region>

  # 例：https://tos-s3-cn-beijing.ivolces.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

- 金山云を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>

  # 例：BEIJING
  aws_s3_region = <region>
  
  # 三階層ドメインを使用してください、金山云は二階層ドメインをサポートしていません
  # 例：jeff-test.ks3-cn-beijing.ksyuncs.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

- Ceph S3 を使用する場合：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例 testbucket/subpath
  aws_s3_path = <s3_path>
  
  # 例：http://172.26.xx.xxx:7480
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <access_key>
  aws_s3_secret_key = <secret_key>
  ```

クラスタ作成後に手動でデフォルトのストレージボリュームを作成したい場合は、以下の設定項目のみを追加します：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

### FE 設定説明

#### run_mode

StarRocks クラスタの実行モード。有効な値：

- `shared_data`：ストレージ分離モードで StarRocks を実行します。
- `shared_nothing`（デフォルト）：ストレージ統合モードで StarRocks を実行します。

> **説明**
>
> - StarRocks クラスタは、ストレージ分離モードとストレージ統合モードの混在デプロイをサポートしていません。
> - クラスタのデプロイが完了した後に `run_mode` を変更しないでください。そうすると、クラスタは再起動できなくなります。ストレージ統合クラスタからストレージ分離クラスタへ、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタデータサービスのリスニングポート。

- デフォルト値：`6090`

#### enable_load_volume_from_conf

StarRocks が FE 設定ファイルに指定されたストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成することを許可するかどうか。v3.1.0 からサポートされています。有効な値：

- `true`（デフォルト）：新しいストレージ分離クラスタを作成する際にこの項目を `true` に設定すると、StarRocks は FE 設定ファイルのストレージ関連のプロパティを使用して組み込みのストレージボリューム `builtin_storage_volume` を作成し、それをデフォルトのストレージボリュームとして設定します。ただし、ストレージ関連のプロパティを指定していない場合、StarRocks は起動できません。
- `false`：新しいストレージ分離クラスタを作成する際にこの項目を `false` に設定すると、StarRocks は直接起動し、組み込みのストレージボリュームは作成されません。StarRocks でオブジェクトを作成する前に、手動でストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#使用-starrocks-存算分离集群)を参照してください。

> **注意**
>
> 既存の v3.0 ストレージ分離クラスタをアップグレードする際は、この項目をデフォルトの設定 `true` のままにすることをお勧めします。この項目を `false` に変更すると、アップグレード前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

#### cloud_native_storage_type

使用するストレージタイプ。ストレージ分離モードでは、StarRocks はデータを HDFS、Azure Blob（v3.1.1 からサポート）、および S3 プロトコルと互換性のあるオブジェクトストレージ（例：AWS S3、Google GCP、阿里云 OSS、MinIO）に保存することをサポートしています。有効な値：

- `S3`（デフォルト値）
- `AZBLOB`
- `HDFS`

> **説明**
>
> - この項目を `S3` に設定する場合は、`aws_s3` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `AZBLOB` に設定する場合は、`azure_blob` をプレフィックスとする設定項目を追加する必要があります。
> - この項目を `HDFS` に設定する場合は、`cloud_native_hdfs_url` のみを指定する必要があります。

#### aws_s3_path

データを保存するためのストレージスペースのパス。ストレージバケットの名前とその下のサブパス（存在する場合）で構成されています。例：`testbucket/subpath`。

#### aws_s3_endpoint

ストレージスペースへのアクセスエンドポイント。

#### aws_s3_region

アクセスが必要なストレージスペースのリージョン。例：`us-west-2`。

#### aws_s3_access_key

ストレージスペースへのアクセスキー。

#### aws_s3_secret_key

ストレージスペースへのシークレットキー。

> **注意**
>
> ストレージ分離クラスタを正常に作成した後、セキュリティ資格情報に関連する設定項目のみを変更することができます。元のストレージパスに関連する設定項目を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることができなくなります。

## ストレージ分離デプロイ CN 設定

<SharedDataCNconf />

## StarRocks ストレージ分離クラスタの使用

<SharedDataUseIntro />

以下の例では、Access Key および Secret Key 認証を使用して OSS ストレージスペース `defaultbucket` にストレージボリューム `def_volume` を作成し、それをアクティブにしてデフォルトのストレージボリュームとして設定します：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "cn-zhangjiakou",
    "aws.s3.endpoint" = "http://oss-cn-zhangjiakou-internal.aliyuncs.com",
    "aws.s3.access_key" = "<oss_access_key>",
    "aws.s3.secret_key" = "<oss_secret_key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
