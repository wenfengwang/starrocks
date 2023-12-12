---
displayed_sidebar: "Japanese"
---

# 共有データのMinIOの使用

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## アーキテクチャ

![Shared-data Architecture](../../assets/share_data_arch.png)

## 共有データStarRocksクラスターのデプロイ

共有データStarRocksクラスターのデプロイは、共有しないStarRocksクラスターのものと類似しています。唯一の違いは、共有データクラスターではBEではなくCNをデプロイする必要がある点です。このセクションでは、共有データStarRocksクラスターをデプロイする際にFEとCNの構成ファイル**fe.conf**および**cn.conf**に追加する必要がある追加の構成項目のみを列挙しています。StarRocksクラスターのデプロイの詳細な手順については、[StarRocksのデプロイ](../../deployment/deploy_manually.md)を参照してください。

### 共有データStarRocksのFEノードの構成

FEを開始する前に、FEの構成ファイル**fe.conf**に以下の構成項目を追加してください。

#### run_mode

StarRocksクラスターの実行モード。有効な値：

- `shared_data`
- `shared_nothing`（デフォルト）。

> **注意**
>
> StarRocksクラスターでは`shared_data`モードと`shared_nothing`モードを同時に採用することはできません。混在デプロイはサポートされていません。
>
> `run_mode`はクラスターがデプロイされた後に変更しないでください。そうしないと、クラスターが再起動に失敗します。共有しないクラスターから共有データクラスター、またはその逆への変換はサポートされていません。

#### cloud_native_meta_port

クラウドネイティブメタサービスのRPCポート。

- デフォルト：`6090`

#### enable_load_volume_from_conf

StarRocksがFE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用してデフォルトのストレージボリュームを作成し、それをデフォルトのストレージボリュームとして設定するかどうか。有効な値：

- `true`（デフォルト）新しい共有データクラスターを作成する際にこの項目を`true`として指定した場合、StarRocksはFE構成ファイルで指定されたオブジェクトストレージ関連のプロパティを使用して組み込みストレージボリューム`builtin_storage_volume`を作成し、デフォルトのストレージボリュームとして設定します。ただし、オブジェクトストレージ関連プロパティが指定されていない場合、StarRocksは起動に失敗します。
- `false`新しい共有データクラスターを作成する際にこの項目を`false`として指定した場合、StarRocksは組み込みストレージボリュームを作成せずに直接起動します。この場合、StarRocksでオブジェクトを作成する前にストレージボリュームを手動で作成し、それをデフォルトのストレージボリュームとして設定する必要があります。詳細は[デフォルトのストレージボリュームの作成](#create-default-storage-volume)を参照してください。

v3.1.0以降でサポートされています。

> **注意**
>
> 既存のv3.0の共有データクラスターをアップグレードしている間は、この項目を`true`に設定したままにすることを強くお勧めします。この項目を`false`に設定すると、アップグレード前に作成したデータベースとテーブルは読み取り専用になり、データをロードすることができません。

#### cloud_native_storage_type

使用するオブジェクトストレージのタイプ。共有データモードでは、StarRocksはAzure Blob（v3.1.1以降でサポート）およびS3プロトコルに対応するオブジェクトストレージ（例：AWS S3、Google GCP、MinIO）にデータを格納することができます。有効な値：

- `S3`（デフォルト）
- `AZBLOB`

> 注意
>
> このパラメータを`S3`と指定した場合、`aws_s3`で接頭辞が付いたパラメータを追加する必要があります。
>
> このパラメータを`AZBLOB`と指定した場合、`azure_blob`で接頭辞が付いたパラメータを追加する必要があります。

#### aws_s3_path

データを格納するために使用するS3パス。S3バケットの名前とそれに含まれるサブパス（ある場合）から構成されます。例：`testbucket/subpath`。

#### aws_s3_endpoint

S3バケットにアクセスするために使用するエンドポイント。例：`https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

S3バケットが存在するリージョン。例：`us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

[AWS SDKのデフォルト認証プロバイダーチェーン](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)を使用するかどうか。有効な値：

- `true`
- `false`（デフォルト）

#### aws_s3_use_instance_profile 

S3へのアクセスのためのインスタンスプロファイルおよび仮定されるロールを認証方法として使用するかどうか。有効な値：

- `true`
- `false`（デフォルト）

S3へのアクセスにIAMユーザーベースの認証（アクセスキーおよびシークレットキー）を使用する場合、この項目を`false`として指定し、`aws_s3_access_key`および`aws_s3_secret_key`を指定する必要があります。

S3へのアクセスにインスタンスプロファイルを使用する場合、この項目を`true`として指定する必要があります。

S3へのアクセスに仮定される役割を使用する場合、この項目を`true`として指定し、`aws_s3_iam_role_arn`を指定する必要があります。また、外部AWSアカウントを使用する場合は、`aws_s3_external_id`も指定する必要があります。

#### aws_s3_access_key

S3バケットにアクセスするために使用するアクセスキーID。

#### aws_s3_secret_key

S3バケットにアクセスするために使用するシークレットアクセスキー。

#### aws_s3_iam_role_arn

データファイルが格納されているS3バケットに権限を持つIAMロールのARN。

#### aws_s3_external_id

S3バケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。

#### azure_blob_path

データを格納するために使用するAzure Blob Storageのパス。このパスは、ストレージアカウント内のコンテナの名前とそのコンテナ内のサブパス（ある場合）から構成されます。例：`testcontainer/subpath`。

#### azure_blob_endpoint

Azure Blob Storageアカウントのエンドポイント。例：`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

Azure Blob Storageのリクエストを認可するために使用される共有キー。

#### azure_blob_sas_token

Azure Blob Storageのリクエストを認可するために使用される共有アクセス署名（SAS）。

> **注意**
>
> 共有データStarRocksクラスターが作成された後は認証関連の構成項目のみを変更できます。元のストレージパス関連の構成項目を変更した場合、変更前に作成したデータベースおよびテーブルは読み取り専用になり、データをロードすることができません。

クラスター作成後にデフォルトのストレージボリュームを手動で作成する場合は、以下の構成項目を追加するだけです：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 共有データStarRocksのCNノードの構成

<SharedDataCNconf />

## 共有データStarRocksクラスターの使用

<SharedDataUseIntro />

次の例では、MinIOバケット`defaultbucket`用のストレージボリューム`def_volume`をアクセスキーとシークレットキーで作成し、ストレージボリュームを有効にして、デフォルトのストレージボリュームとして設定します：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://hostname.domainname.com:portnumber",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />