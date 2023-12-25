---
displayed_sidebar: Chinese
---

# ストレージボリュームの作成

## 機能

リモートストレージシステムにストレージボリュームを作成します。この機能はv3.1からサポートされています。

ストレージボリュームは、リモートストレージシステムの属性と認証情報で構成されます。[StarRocks 分散ストレージクラスター](../../../deployment/shared_data/s3.md)でデータベースやクラウドネイティブテーブルを作成する際に、ストレージボリュームを参照できます。

> **注意**
>
> - SYSTEMレベルのCREATE STORAGE VOLUME権限を持つユーザーのみがこの操作を実行できます。
> - HDFSに基づいてストレージボリュームを作成する場合は、**HADOOP_CONF**や**core-site.xml/hdfs-site.xml**を無闇に変更しないことをお勧めします。これらのファイルのパラメータとストレージボリュームの作成時のパラメータが異なる場合、予期せぬ動作が発生する可能性があります。

## 構文

```SQL
CREATE STORAGE VOLUME [IF NOT EXISTS] <storage_volume_name>
TYPE = { S3 | HDFS | AZBLOB }
LOCATIONS = ('<remote_storage_path>')
[ COMMENT '<comment_string>' ]
PROPERTIES
("key" = "value",...)
```

## パラメータ説明

| **パラメータ**            | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | ストレージボリュームの名前。`builtin_storage_volume`という名前のストレージボリュームは作成できません。この名前は内蔵ストレージボリュームの作成に使用されています。 |
| TYPE                | リモートストレージシステムのタイプ。有効な値：`S3`、`AZBLOB`、`HDFS`。`S3`はAWS S3またはS3プロトコルと互換性のあるストレージシステムを表します。`AZBLOB`はAzure Blob Storageを表します（v3.1.1からサポート）。`HDFS`はHDFSクラスターを表します。 |
| LOCATIONS           | リモートストレージシステムの位置。形式は以下の通りです：<ul><li>AWS S3またはS3プロトコルと互換性のあるストレージシステム：`s3://<s3_path>`。`<s3_path>`は絶対パスでなければなりません。例：`s3://testbucket/subpath`。</li><li>Azure Blob Storage: `azblob://<azblob_path>`。`<azblob_path>`は絶対パスでなければなりません。例：`azblob://testcontainer/subpath`。</li><li>HDFS：`hdfs://<host>:<port>/<hdfs_path>`。`<hdfs_path>`は絶対パスでなければなりません。例：`hdfs://127.0.0.1:9000/user/xxx/starrocks`。</li><li>WebHDFS：`webhdfs://<host>:<http_port>/<hdfs_path>`、`<http_port>`はNameNodeのHTTPポートです。`<hdfs_path>`は絶対パスでなければなりません。例：`webhdfs://127.0.0.1:50070/user/xxx/starrocks`。</li><li>ViewFS：`viewfs://<ViewFS_cluster>/<viewfs_path>`、`<ViewFS_cluster>`はViewFSクラスター名です。`<viewfs_path>`は絶対パスでなければなりません。例：`viewfs://myviewfscluster/user/xxx/starrocks`。</li></ul> |
| COMMENT             | ストレージボリュームのコメント。                                               |
| PROPERTIES          | `"key" = "value"`の形式のパラメータペアで、リモートストレージシステムへのアクセスに必要な属性と認証情報を指定します。詳細は[PROPERTIES](#properties)を参照してください。 |

## PROPERTIES

- AWS S3を使用する場合：

  - AWS SDKのデフォルト認証情報を使用する場合、以下のプロパティを設定してください：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "true"
    ```

  - IAMユーザーベースの認証を使用する場合、以下のプロパティを設定してください：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<access_key>",
    "aws.s3.secret_key" = "<secret_key>"
    ```

  - Instance Profile認証を使用する場合、以下のプロパティを設定してください：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true"
    ```

  - Assumed Role認証を使用する場合、以下のプロパティを設定してください：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>"
    ```

  - 外部AWSアカウントを通じてAssumed Role認証を使用する場合、以下のプロパティを設定してください：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>",
    "aws.s3.external_id" = "<external_id>"
    ```

- GCP Cloud Storageを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例：https://storage.googleapis.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- 阿里云OSSを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：cn-zhangjiakou
  "aws.s3.region" = "<region>",
  
  -- 例：https://oss-cn-zhangjiakou-internal.aliyuncs.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- 华为云OBSを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：cn-north-4
  "aws.s3.region" = "<region>",
  
  -- 例：https://obs.cn-north-4.myhuaweicloud.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- 腾讯云COSを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：ap-beijing
  "aws.s3.region" = "<region>",
  
  -- 例：https://cos.ap-beijing.myqcloud.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- 火山引擎TOSを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：cn-beijing
  "aws.s3.region" = "<region>",
  
  -- 例：https://tos-s3-cn-beijing.ivolces.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- 金山云を使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：BEIJING
  "aws.s3.region" = "<region>",
  
  -- 三階層のドメイン名を使用してください。金山云は二階層のドメイン名をサポートしていません。
  -- 例：jeff-test.ks3-cn-beijing.ksyuncs.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- MinIOを使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例：http://172.26.xx.xxx:39000
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- Ceph S3を使用する場合、以下のプロパティを設定してください：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：http://172.26.xx.xxx:7480
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

  | **プロパティ**                            | **説明**                                                     |
  | ----------------------------------- | ------------------------------------------------------------ |
  | enabled                             | 現在のストレージボリュームを有効にするかどうか。デフォルト値は`false`です。無効にされたストレージボリュームは参照できません。 |
  | aws.s3.region                       | アクセスするS3ストレージのリージョン。例：`us-west-2`。                 |
  | aws.s3.endpoint                     | S3ストレージへのエンドポイントURL。例：`https://s3.us-west-2.amazonaws.com`。 |
  | aws.s3.use_aws_sdk_default_behavior | AWS SDKのデフォルト認証情報を使用するかどうか。有効な値は`true`または`false`（デフォルト）。 |
  | aws.s3.use_instance_profile         | Instance ProfileまたはAssumed Roleをセキュリティクレデンシャルとして使用してS3にアクセスするかどうか。有効な値は`true`または`false`（デフォルト）。<ul><li>IAMユーザークレデンシャル（Access KeyとSecret Key）を使用してS3にアクセスする場合は、これを`false`に設定し、`aws.s3.access_key`と`aws.s3.secret_key`を指定する必要があります。</li><li>Instance Profileを使用してS3にアクセスする場合は、これを`true`に設定します。</li><li>Assumed Roleを使用してS3にアクセスする場合は、これを`true`に設定し、`aws.s3.iam_role_arn`を指定する必要があります。</li><li>外部AWSアカウントを介してAssumed Role認証を使用してS3にアクセスする場合は、これを`true`に設定し、`aws.s3.iam_role_arn`と`aws.s3.external_id`を追加で指定する必要があります。</li></ul> |
  | aws.s3.access_key                   | S3ストレージへのアクセスキー。                              |
  | aws.s3.secret_key                   | S3ストレージへのシークレットキー。                              |
  | aws.s3.iam_role_arn                 | S3ストレージへのアクセス権を持つIAMロールのARN。                     |
  | aws.s3.external_id                  | 外部AWSアカウントを介してS3ストレージにアクセスするための外部ID。                   |

- Azure Blob Storageを使用する場合（v3.1.1からサポート）：

  - 共有キー(Shared Key)認証を使用する場合は、以下の PROPERTIES を設定してください:

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
    ```

  - 共有アクセス署名(SAS)認証を使用する場合は、以下の PROPERTIES を設定してください:

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.sas_token" = "<sas_token>"
    ```

  > **注意**
  >
  > Azure Blob Storage アカウントを作成する際には、階層型ネームスペースを無効にする必要があります。

  | **属性**              | **説明**                                                     |
  | --------------------- | ------------------------------------------------------------ |
  | enabled               | 現在のストレージボリュームを有効にするかどうか。デフォルト値：`false`。無効にされたストレージボリュームは参照できません。 |
  | azure.blob.endpoint   | Azure Blob Storage のエンドポイントURL。例：`https://test.blob.core.windows.net`。 |
  | azure.blob.shared_key | Azure Blob Storage へのアクセスに使用する共有キー(Shared Key)。           |
  | azure.blob.sas_token  | Azure Blob Storage へのアクセスに使用する共有アクセス署名(SAS)。              |

- HDFS ストレージを使用する場合:

  - 認証なしで HDFS にアクセスする場合は、以下の属性を設定してください:

    ```SQL
    "enabled" = "{ true | false }"
    ```

  - 簡易認証(Username)で HDFS にアクセスする場合(v3.2 以降でサポート)は、以下の属性を設定してください:

    ```SQL
    "enabled" = "{ true | false }",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>"
    ```

  - Kerberos Ticket Cache 認証で HDFS にアクセスする場合(v3.2 以降でサポート)は、以下の属性を設定してください:

    ```SQL
    "enabled" = "{ true | false }",
    "hadoop.security.authentication" = "kerberos",
    "hadoop.security.kerberos.ticket.cache.path" = "<ticket_cache_path>"
    ```

    > **注意**
    >
    > - この設定は、システムが KeyTab を使用して Kerberos 経由で HDFS にアクセスすることを強制するためのものです。BE または CN ノードが KeyTab ファイルにアクセスできることを確認してください。また、**/etc/krb5.conf** ファイルが正しく設定されていることも確認してください。
    > - Ticket cache は外部の kinit ツールによって生成されます。crontab や類似の定期的なタスクが Ticket を更新するようにしてください。

  - HDFS クラスターで NameNode HA 設定が有効になっている場合(v3.2 以降でサポート)は、以下の追加属性を設定してください:

    ```SQL
    "dfs.nameservices" = "<ha_cluster_name>",
    "dfs.ha.namenodes.<ha_cluster_name>" = "<NameNode1>,<NameNode2> [, ...]",
    "dfs.namenode.rpc-address.<ha_cluster_name>.<NameNode1>" = "<hdfs_host>:<hdfs_port>",
    "dfs.namenode.rpc-address.<ha_cluster_name>.<NameNode2>" = "<hdfs_host>:<hdfs_port>",
    [...]
    "dfs.client.failover.proxy.provider.<ha_cluster_name>" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
    ```

    詳細については、[HDFS HA ドキュメント](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)を参照してください。

    - WebHDFS を使用する場合(v3.2 以降でサポート)は、以下の属性を設定してください:

    ```SQL
    "enabled" = "{ true | false }"
    ```

    詳細については、[WebHDFS ドキュメント](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)を参照してください。


  - Hadoop ViewFS を使用する場合(v3.2 以降でサポート)は、以下の属性を設定してください:

    ```SQL
    -- <ViewFS_cluster> を ViewFS クラスター名に置き換えてください。
    "fs.viewfs.mounttable.<ViewFS_cluster>.link./<viewfs_path_1>" = "hdfs://<hdfs_host_1>:<hdfs_port_1>/<hdfs_path_1>",
    "fs.viewfs.mounttable.<ViewFS_cluster>.link./<viewfs_path_2>" = "hdfs://<hdfs_host_2>:<hdfs_port_2>/<hdfs_path_2>",
    [, ...]
    ```

    詳細については、[ViewFS ドキュメント](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/ViewFs.html)を参照してください。

    | **属性**                                              | **説明**                                                     |
    | ----------------------------------------------------- | ------------------------------------------------------------ |
    | enabled                                               | 現在のストレージボリュームを有効にするかどうか。デフォルト値:`false`。無効にされたストレージボリュームは参照できません。       |
    | hadoop.security.authentication                        | 認証方式を指定します。有効な値:`simple`(デフォルト) および `kerberos`。`simple` は簡易認証、つまり Username を意味します。`kerberos` は Kerberos 認証を意味します。 |
    | username                                              | HDFS クラスター内の NameNode ノードにアクセスするためのユーザー名。                      |
    | hadoop.security.kerberos.ticket.cache.path            | kinit が生成する Ticket Cache ファイルのパスを指定します。                   |
    | dfs.nameservices                                      | HDFS クラスターのカスタム名を指定します。                                        |
    | dfs.ha.namenodes.`<ha_cluster_name\>`                   | NameNode のカスタム名を指定します。複数の名前はコンマ (,) で区切り、引用符の中にスペースを含めないでください。ここでの `<ha_cluster_name>` は `dfs.nameservices` で指定した HDFS サービスの名前です。 |
    | dfs.namenode.rpc-address.`<ha_cluster_name\>`.`<NameNode\>` | NameNode の RPC アドレス情報を指定します。ここでの `<NameNode>` は `dfs.ha.namenodes.<ha_cluster_name>` で指定した NameNode の名前です。 |
    | dfs.client.failover.proxy.provider                    | クライアントが接続する NameNode のプロバイダーを指定します。デフォルトは `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider` です。 |
    | fs.viewfs.mounttable.`<ViewFS_cluster\>`.link./`<viewfs_path\>` | マウントする ViewFS クラスターのパスを指定します。複数のパスはコンマ (,) で区切ります。ここでの `<ViewFS_cluster>` は `LOCATIONS` で指定した ViewFS クラスターの名前です。|

<!--

    | kerberos_keytab_content                               | Kerberos の keytab ファイルの内容を Base64 エンコードしたものを指定します。このパラメータは `kerberos_keytab` と選択して設定します。 |
    | password                                              | HDFS クラスター内の NameNode ノードにアクセスするためのパスワード。                   |
    | kerberos_principal                                    | Kerberos のユーザーまたはサービス (Principal) を指定します。各 Principal は HDFS クラスター内で一意であり、以下の三つの部分で構成されます：<ul><li>`username` または `servicename`：HDFS クラスター内のユーザーまたはサービスの名前。</li><li>`instance`：認証が必要な HDFS クラスターのノードが存在するサーバーの名前。ユーザーまたはサービスが全体で一意であることを保証するために使用されます。例えば、HDFS クラスターに複数の DataNode ノードがあり、それぞれが独立して認証を受ける必要があります。</li><li>`realm`：ドメインで、必ず大文字で記述します。</li></ul>例：`nn/zelda1@ZELDA.COM`。 |

-->

## 例

例一：AWS S3 ストレージスペース `defaultbucket` に `my_s3_volume` ストレージボリュームを作成し、IAM user-based 認証を使用して、そのストレージボリュームを有効にします。

```SQL
CREATE STORAGE VOLUME my_s3_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://s3.us-west-2.amazonaws.com",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);
```

例二：HDFS に `my_hdfs_volume` ストレージボリュームを作成し、そのストレージボリュームを有効にします。

```SQL
CREATE STORAGE VOLUME my_hdfs_volume
TYPE = HDFS
LOCATIONS = ("hdfs://127.0.0.1:9000/sr/test/")
PROPERTIES
(
    "enabled" = "true"
);
```

例三：簡易認証を使用して HDFS に `hdfsvolumehadoop` ストレージボリュームを作成します。

```sql
CREATE STORAGE VOLUME hdfsvolumehadoop
TYPE = HDFS
LOCATIONS = ("hdfs://127.0.0.1:9000/sr/test/")
PROPERTIES(
    "hadoop.security.authentication" = "simple",
    "username" = "starrocks"
);
```

例四：Kerberos Ticket Cache 認証を使用して HDFS にアクセスし、`hdfsvolkerberos` ストレージボリュームを作成します。

```sql
CREATE STORAGE VOLUME hdfsvolkerberos
TYPE = HDFS
LOCATIONS = ("hdfs://127.0.0.1:9000/sr/test/")
PROPERTIES(
    "hadoop.security.authentication" = "kerberos",
    "hadoop.security.kerberos.ticket.cache.path" = "/path/to/ticket/cache/path"
);
```

例五：NameNode HA 設定が有効な HDFS クラスターに `hdfsvolha` ストレージボリュームを作成します。

```sql
CREATE STORAGE VOLUME hdfsvolha
TYPE = HDFS
LOCATIONS = ("hdfs://myhacluster/data/sr")
PROPERTIES(
    "dfs.nameservices" = "myhacluster",
    "dfs.ha.namenodes.myhacluster" = "nn1,nn2,nn3",
    "dfs.namenode.rpc-address.myhacluster.nn1" = "machine1.example.com:8020",
    "dfs.namenode.rpc-address.myhacluster.nn2" = "machine2.example.com:8020",
    "dfs.namenode.rpc-address.myhacluster.nn3" = "machine3.example.com:8020",
    "dfs.namenode.http-address.myhacluster.nn1" = "machine1.example.com:9870",
    "dfs.namenode.http-address.myhacluster.nn2" = "machine2.example.com:9870",
    "dfs.namenode.http-address.myhacluster.nn3" = "machine3.example.com:9870",
    "dfs.client.failover.proxy.provider.myhacluster" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

例六：WebHDFS を使用して `webhdfsvol` ストレージボリュームを作成します。

```sql
CREATE STORAGE VOLUME webhdfsvol
TYPE = HDFS
LOCATIONS = ("webhdfs://namenode:9870/data/sr");
```

例七：Hadoop ViewFS を使用して `viewfsvol` ストレージボリュームを作成します。

```sql
CREATE STORAGE VOLUME viewfsvol
TYPE = HDFS
LOCATIONS = ("viewfs://clusterX/data/sr")
PROPERTIES(
    "fs.viewfs.mounttable.clusterX.link./data" = "hdfs://nn1-clusterx.example.com:8020/data",
    "fs.viewfs.mounttable.clusterX.link./project" = "hdfs://nn2-clusterx.example.com:8020/project"
);
```

## 関連 SQL

- [ALTER_STORAGE_VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP_STORAGE_VOLUME](./DROP_STORAGE_VOLUME.md)

- [デフォルトのストレージボリュームを設定](./SET_DEFAULT_STORAGE_VOLUME.md)
- [ストレージボリュームの説明](./DESC_STORAGE_VOLUME.md)
- [ストレージボリュームの表示](./SHOW_STORAGE_VOLUMES.md)
