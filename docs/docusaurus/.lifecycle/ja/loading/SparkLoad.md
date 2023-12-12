---
displayed_sidebar: "Japanese"
---

# Spark Loadを使用した一括データのロード

このロードは、インポートされたデータを前処理するために外部のApache Spark™リソースを使用し、インポートのパフォーマンスを向上させ、コンピューティングリソースを節約します。これは主に、**初期マイグレーション**と**スターロックス（TBレベルまでのデータ量）への**大規模なデータインポートに使用されます。

Spark Loadは**非同期**のインポート方法であり、ユーザーはMySQLプロトコルを介してSparkタイプのインポートジョブを作成し、インポート結果を`SHOW LOAD`で表示する必要があります。

> **注意**
>
> - StarRocksテーブルにINSERT権限を持つユーザーのみ、このテーブルにデータをロードできます。必要な権限を付与するには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供される手順に従うことができます。
> - Spark Loadは、プライマリキー テーブルにデータをロードするためには使用できません。

## 用語の説明

- **Spark ETL**: インポートプロセスにおけるデータのETL（抽出、変換、ロード）を主に担当し、グローバル辞書の構築（BITMAPタイプ）、パーティショニング、ソート、集計などを行います。
- **Broker**: Brokerは独立したステートレスプロセスです。ファイルシステムインターフェイスをカプセル化し、リモートストレージシステムからファイルを読み取り、StarRocksに提供します。
- **グローバル辞書**: 元の値から符号化された値へのデータのマッピング構造を保存します。元の値は任意のデータ型であるのに対し、符号化された値は整数です。グローバル辞書は、正確なカウントの異なり数が事前に計算されたシナリオで主に使用されます。

## 背景情報

StarRocks v2.4以前では、Spark LoadはBrokerプロセスに依存し、StarRocksクラスタとストレージシステムの間の接続を確立するために使用されていました。Spark Loadジョブを作成する際には、「WITH BROKER "<broker_name>"」を入力して使用するBrokerを指定する必要があります。Brokerは、独立したステートレスプロセスであり、ファイルシステムインターフェースが統合されています。Brokerプロセスを使用することで、StarRocksはストレージシステムに格納されたデータファイルにアクセスし、読み取り、および自身のコンピューティングリソースを使用して、これらのデータファイルのデータを前処理し、ロードできます。

StarRocks v2.5以降、Spark LoadはBrokerプロセスに依存する必要がなくなりました。Spark Loadジョブを作成する際には、Brokerを指定する必要はなくなりましたが、引き続き`WITH BROKER`キーワードを保持する必要があります。

> **注意**
>
> Brokerプロセスを使用せずにロードすると、複数のHDFSクラスタや複数のKerberosユーザーが存在する場合など、特定の状況で正常に動作しないことがあります。このような状況では、引き続きBrokerプロセスを使用してデータをロードすることができます。

## 基礎

ユーザーはMySQLクライアントを介してSparkタイプのインポートジョブを送信します。FEはメタデータを記録し、送信結果を返します。

spark loadタスクの実行は、次の主要なフェーズに分かれます。

1. ユーザーがspark loadジョブをFEに送信します。
2. FEは、ETLタスクの提出をApache Spark™クラスタにスケジュールします。
3. Apache Spark™クラスタは、グローバル辞書の構築（BITMAPタイプ）、パーティショニング、ソート、集計などを含むETLタスクを実行します。
4. ETLタスクが完了した後、FEは各前処理スライスのデータパスを取得し、関連するBEにプッシュタスクの実行をスケジュールします。
5. BEは、HDFSからBrokerプロセスを介してデータを読み取り、StarRocksストレージ形式に変換します。
    > Brokerプロセスを使用しない場合、BEはHDFSから直接データを読み取ります。
6. FEは有効なバージョンをスケジュールし、インポートジョブを完了します。

次の図は、Spark Loadの主要なフローを示しています。

![Spark load](../assets/4.3.2-1.png)

---

## グローバル辞書

### 適用シナリオ

現在、StarRocksのBITMAP列はRoaringbitmapを使用して実装されており、整数のみが入力データ型として使用されます。したがって、インポートプロセスのBITMAP列の事前計算を実装したい場合は、入力データ型を整数に変換する必要があります。

StarRocksの既存のインポートプロセスでは、グローバル辞書のデータ構造はHiveテーブルに基づいて実装されており、元の値から符号化された値へのマッピングを保存しています。

### 構築プロセス

1. 上流データソースからデータを読み取り、一時的なHiveテーブルである`hive-table`を生成します。
2. `hive-table`の軽減フィールドの値を抽出し、`distinct-value-table`という新しいHiveテーブルを生成します。
3. 元の値用の1つの列と符号化値用の1つの列を持つ新しいグローバル辞書テーブルである`dict-table`を作成します。
4. `distinct-value-table`と`dict-table`の左結合を行い、次にウィンドウ関数を使用してこのセットを符号化します。最終的に、重複排除された列の元の値と符号化された値の両方を`dict-table`に書き込みます。
5. `dict-table`と`hive-table`の間での結合を行い、`hive-table`内の元の値を整数符号化値で置き換えるジョブを完了します。
6. `hive-table`は、次回のデータ前処理で読み取られ、計算後にStarRocksにインポートされます。

## データの前処理

データの前処理の基本的な手順は次のとおりです。

1. 上流データソース（HDFSファイルまたはHiveテーブル）からデータを読み取ります。
2. 読み取ったデータのフィールドマッピングと計算を完了し、パーティション情報に基づいて`bucket-id`を生成します。
3. StarRocksテーブルのロールアップメタデータに基づいてRollupTreeを生成します。
4. RollupTreeを反復処理し、階層的な集計操作を実行します。次階層のロールアップは、前の階層のロールアップから計算できます。
5. 集約計算が完了するたびに、データは`bucket-id`に従ってバケット分けされ、次にHDFSに書き込まれます。
6. その後のBrokerプロセスは、HDFSからファイルを取得し、StarRocks BEノードにインポートします。

## 基本操作

### 前提条件

引き続きBrokerプロセスを介してデータをロードする場合、StarRocksクラスタにBrokerプロセスが展開されていることを確認する必要があります。

[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタに展開されているBrokerを確認できます。Brokerが展開されていない場合は、[Deploy Broker](../deployment/deploy_broker.md)で提供される手順に従ってBrokerを展開する必要があります。

### ETLクラスタの設定

Apache Spark™は、StarRocksの外部計算リソースとしてETLタスクを実行するために使用されます。StarRocksには、クエリ用のSpark/GPU、外部ストレージ用のHDFS/S3、ETL用のMapReduceなど、他の外部リソースが追加される場合があります。そのため、StarRocksが使用するこれらの外部リソースを管理するために、`Resource Management`を導入しています。

Apache Spark™のインポートジョブを提出する前に、Apache Spark™クラスタをETLタスクを実行するために設定します。操作の構文は次のとおりです。

~~~sql
-- リソースの作成
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = spark,
 spark_conf_key = spark_conf_value,
 working_dir = path,
 broker = broker_name,
 broker.property_key = property_value
);

-- リソースの削除
DROP RESOURCE resource_name;

-- リソースの表示
SHOW RESOURCES
SHOW PROC "/resources";

-- 権限
GRANT USAGE_PRIV ON RESOURCE resource_name TO user_identityGRANT USAGE_PRIV ON RESOURCE resource_name TO ROLE role_name;
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM user_identityREVOKE USAGE_PRIV ON RESOURCE resource_name FROM ROLE role_name;
~~~

- リソースの作成

**例**：

~~~sql
-- yarnクラスタモード
CREATE EXTERNAL RESOURCE "spark0"
PROPERTIES
(
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.jars" = "xxx.jar,yyy.jar",
    "spark.files" = "/tmp/aaa,/tmp/bbb",
    "spark.executor.memory" = "1g",
    "spark.yarn.queue" = "queue0",
    "spark.hadoop.yarn.resourcemanager.address" = "127.0.0.1:9999",
    "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
    "working_dir" = "hdfs://127.0.0.1:10000/tmp/starrocks",
    "broker" = "broker0",
    "broker.username" = "user0",
    "broker.password" = "password0"
);

-- yarn HAクラスタモード
CREATE EXTERNAL RESOURCE "spark1"
PROPERTIES
(
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.hadoop.yarn.resourcemanager.ha.enabled" = "true",
    "spark.hadoop.yarn.resourcemanager.ha.rm-ids" = "rm1,rm2",
    "spark.hadoop.yarn.resourcemanager.hostname.rm1" = "host1",
    "spark.hadoop.yarn.resourcemanager.hostname.rm2" = "host2",
    "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
    "working_dir" = "hdfs://127.0.0.1:10000/tmp/starrocks",
    "broker" = "broker1"
);
~~~

`resource-name`は、StarRocksで構成されたApache Spark™リソースの名前です。

`PROPERTIES`には、次のようなApache Spark™リソースに関連するパラメータが含まれます:
> **注意**
>
> Apache Spark™ リソースのPROPERTIESの詳細説明については、[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md) をご参照ください。

- Spark 関連のパラメータ:
  - `type`: リソースタイプ。必須。現在は `spark` のみをサポート。
  - `spark.master`: 必須。現在は `yarn` のみをサポート。
    - `spark.submit.deployMode`: Apache Spark™ プログラムの展開モード。必須。現在は `cluster` と `client` の両方をサポート。
    - `spark.hadoop.fs.defaultFS`: master が yarn の場合に必須。
    - yarn リソースマネージャに関連するパラメータ。必須。
      - 単一ノードでのResourceManager
        `spark.hadoop.yarn.resourcemanager.address`: 単一ポイントリソースマネージャのアドレス。
      - ResourceManager HA
        > ResourceManagerのホスト名またはアドレスを指定できます。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`: リソースマネージャのHAを有効にし、`true` に設定します。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`: リソースマネージャの論理IDのリスト。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`: 各 rm-id について、リソースマネージャに対応するホスト名を指定します。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id`: 各 rm-id について、クライアントがジョブを提出するための `host:port` を指定します。

- `*working_dir`: ETLで使用されるディレクトリ。Apache Spark™ をETLリソースとして使用する場合に必須です。例: `hdfs://host:port/tmp/starrocks`。

- Broker 関連のパラメータ:
  - `broker`: Broker 名。ETLリソースとしてApache Spark™を使用する場合に必須です。事前に`ALTER SYSTEM ADD BROKER` コマンドを使用して構成を完了する必要があります。
  - `broker.property_key`: ETLによって生成された中間ファイルを読み込むときに指定する情報（例: 認証情報）。

**注意**:

上記は、Brokerプロセスを介して読み込むためのパラメータの説明です。Brokerプロセスを使用せずにデータを読み込む場合は、次の点に注意してください。

- `broker` を指定する必要はありません。
- ユーザー認証や、NameNodeノードのHAの構成が必要な場合は、HDFSクラスタの`hdfs-site.xml`ファイルでパラメータを構成する必要があります。パラメータの詳細については[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照し、**hdfs-site.xml** ファイルを各FEの**$FE_HOME/conf**および各BEの**$BE_HOME/conf**に移動する必要があります。

> 注意
>
> HDFSファイルに特定のユーザーだけがアクセスできる場合、`broker.name` でHDFSのユーザー名を指定し、`broker.password` でユーザーのパスワードを指定する必要があります。

- リソースの表示

通常のアカウントでは、`USAGE-PRIV` アクセス権限を持つリソースのみを表示することができます。ルートおよび管理者アカウントでは、すべてのリソースを表示できます。

- リソース権限

リソース権限は `GRANT REVOKE` を介して管理され、現在は `USAGE-PRIV` 権限のみをサポートしています。ユーザーまたはロールに `USAGE-PRIV` 権限を与えることができます。

~~~sql
-- ユーザー0にspark0リソースへのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- ロール0にspark0リソースへのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- ユーザー0にすべてのリソースへのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- ロール0にすべてのリソースへのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- ユーザーuser0からspark0リソースの使用権を取り消し
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### Sparkクライアントの構成

FE用にSparkクライアントを構成して、`spark-submit` コマンドを実行してSparkタスクを提出できるようにします。公式バージョンのSpark2 2.4.5以上を使用することをお勧めします[sparkダウンロードアドレス](https://archive.apache.org/dist/spark/) からダウンロードした後、次の手順を実行して構成を完了してください。

- `SPARK-HOME` の構成
  
そのSparkクライアントをFEと同じマシン上のディレクトリに配置し、FE構成ファイルの`spark_home_default_dir`をこのディレクトリに設定します。これは、デフォルトでFEルートディレクトリ内の`lib/spark2x`パスですが、空にできません。

- **SPARK 依存パッケージの構成**
  
依存パッケージを構成するには、Sparkクライアントの`jars`フォルダー内のすべてのjarファイルをzip圧縮してアーカイブし、FE設定の`spark_resource_path`項目をこのzipファイルに設定します。この構成が空の場合、FEはデフォルトでFEルートディレクトリ内の`lib/spark2x/jars/spark-2x.zip`ファイルを検索しようとします。FEがそれを見つけられない場合は、エラーが発生します。

Sparkのロードジョブを提出するとき、アーカイブされた依存ファイルがリモートリポジトリにアップロードされます。デフォルトのリポジトリパスは、`working_dir/{cluster_id}`ディレクトリの下に`--spark-repository--{resource-name}` という名前が付けられたリソースに対応しています。ディレクトリ構造は次のように参照されます:

~~~bash
---spark-repository--spark0/

   |---archive-1.0.0/

   |        |\---lib-990325d2c0d1d5e45bf675e54e44fb16-spark-dpp-1.0.0\-jar-with-dependencies.jar

   |        |\---lib-7670c29daf535efe3c9b923f778f61fc-spark-2x.zip

   |---archive-1.1.0/

   |        |\---lib-64d5696f99c379af2bee28c1c84271d5-spark-dpp-1.1.0\-jar-with-dependencies.jar

   |        |\---lib-1bbb74bb6b264a270bc7fca3e964160f-spark-2x.zip

   |---archive-1.2.0/

   |        |-...

~~~

`spark-2x.zip`（デフォルト名）を含むSpark依存ファイルに加えて、FEはDPP依存ファイルもリモートリポジトリにアップロードします。Sparkロードで提出されたすべての依存ファイルがすでにリモートリポジトリに存在する場合は、毎回大量のファイルを繰り返しアップロードする手間を省きます。

### YARNクライアントの構成

FE用にYARNクライアントを構成して、FEが実行しているアプリケーションのステータスを取得したり、アプリケーションを停止したりできるようにします。公式バージョンのHadoop2 2.5.2以上を使用することをお勧めします（[hadoopダウンロードアドレス](https://archive.apache.org/dist/hadoop/common/)）。ダウンロード後、次の手順を実行して構成を完了してください:

- **Configure the YARN executable path**
  
ダウンロードしたYARNクライアントをFEと同じマシン上のディレクトリに配置し、FE構成ファイルの`yarn_client_path`項目をYARNのバイナリ実行ファイルに設定します。これは、デフォルトでFEルートディレクトリ内の`lib/yarn-client/hadoop/bin/yarn`パスです。

- **YARN（オプション）の構成ファイルのパスを構成する**
  
FEがYARNクライアントを介してアプリケーションのステータスを取得したり、アプリケーションを停止するために必要な構成ファイルを生成するために必要な構成ファイルをStarRocksがデフォルトでFEルートディレクトリの`lib/yarn-config`パスに生成します。このパスは、現在`core-site.xml`および`yarn-site.xml`を含んでいます。

### インポートジョブの作成

**構文:**

~~~sql
LOAD LABEL load_label
    (data_desc, ...)
WITH RESOURCE resource_name 
[resource_properties]
[PROPERTIES (key1=value1, ... )]

* load_label:
    db_name.label_name

* data_desc:
    DATA INFILE ('file_path', ...)
    [NEGATIVE]
    INTO TABLE tbl_name
    [PARTITION (p1, p2)]
    [COLUMNS TERMINATED BY separator ]
    [(col1, ...)]
    [COLUMNS FROM PATH AS (col2, ...)]
    [SET (k1=f1(xx), k2=f2(xx))]
    [WHERE predicate]

    DATA FROM TABLE hive_external_tbl
    [NEGATIVE]
    INTO TABLE tbl_name
    [PARTITION (p1, p2)]
    [SET (k1=f1(xx), k2=f2(xx))]
    [WHERE predicate]

* resource_properties:
(key2=value2, ...)
~~~

**例1**: 上流データソースがHDFSの場合のケース

~~~sql
LOAD LABEL db1.label1
(
    DATA INFILE("hdfs://abc.com:8888/user/starrocks/test/ml/file1")
    INTO TABLE tbl1
    COLUMNS TERMINATED BY ","
    (tmp_c1,tmp_c2)
    SET
    (
        id=tmp_c2,
        name=tmp_c1
    ),
    DATA INFILE("hdfs://abc.com:8888/user/starrocks/test/ml/file2")
    INTO TABLE tbl2
    COLUMNS TERMINATED BY ","
    (col1, col2)
    where col1 > 1
)
WITH RESOURCE 'spark0'
(
    "spark.executor.memory" = "2g",
    "spark.shuffle.compress" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
~~~

**Example 2**: The case where the upstream data source is Hive.

- Step 1: Create a new hive resource

~~~sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://0.0.0.0:8080"
);
 ~~~

- Step 2: Create a new hive external table

~~~sql
CREATE EXTERNAL TABLE hive_t1
(
    k1 INT,
    K2 SMALLINT,
    k3 varchar(50),
    uuid varchar(100)
)
ENGINE=hive
PROPERTIES
( 
    "resource" = "hive0",
    "database" = "tmp",
    "table" = "t1"
);
 ~~~

- Step 3: Submit the load command, requiring that the columns in the imported StarRocks table exist in the hive external table.

~~~sql
LOAD LABEL db1.label1
(
    DATA FROM TABLE hive_t1
    INTO TABLE tbl1
    SET
    (
        uuid=bitmap_dict(uuid)
    )
)
WITH RESOURCE 'spark0'
(
    "spark.executor.memory" = "2g",
    "spark.shuffle.compress" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
 ~~~

In the documentation for Spark load parameters:

- **Label**
  
Import job label. Each import job has a unique label within the database, following the same rules as broker load.

- **Data description class parameters**
  
Currently, CSV and Hive table are supported data sources. Other rules are the same as broker load.

- **Import Job Parameters**
  
Import job parameters refer to the parameters belonging to the `opt_properties` section of the import statement. These parameters are applicable to the entire import job. The rules are the same as broker load.

- **Spark Resource Parameters**
  
Spark resources need to be configured into StarRocks in advance and users need to be given USAGE-PRIV permissions before they can apply the resources to Spark load.
Spark resource parameters can be set when the user has a temporary need, such as adding resources for a job and modifying Spark configs. The setting only takes effect on this job and does not affect the existing configurations in the StarRocks cluster.

~~~sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

- **Import when the data source is Hive**
  
Currently, to use a Hive table in the import process, you need to create an external table of the `Hive` type and then specify its name when submitting the import command.

- **Import process to build a global dictionary**
  
In the load command, you can specify the required fields for building the global dictionary in the following format: `StarRocks field name=bitmap_dict(hive table field name)` Note that currently **the global dictionary is only supported when the upstream data source is a Hive table**.

- **Import of bitmap binary type columns**

The data type applicable to the StarRocks table aggregate column is bitmap type, and the data type of the corresponding column in the data source Hive table or HDFS file type is binary (through com.starrocks.load.loadv2.dpp.BitmapValue in spark-dpp in FE class serialization) type.
There is no need to build a global dictionary, just specify the corresponding fields in the load command. The format is: ```StarRocks field name=bitmap_from_binary(Hive table field name)```

## Viewing Import Jobs

The Spark load import is asynchronous, as is the broker load. The user must record the label of the import job and use it in the `SHOW LOAD` command to view the import results. The command to view the import is common to all import methods. The example is as follows.

Refer to Broker Load for a detailed explanation of returned parameters.The differences are as follows.

~~~sql
mysql> show load order by createtime desc limit 1\G
*************************** 1. row ***************************
  JobId: 76391
  Label: label1
  State: FINISHED
 Progress: ETL:100%; LOAD:100%
  Type: SPARK
 EtlInfo: unselected.rows=4; dpp.abnorm.ALL=15; dpp.norm.ALL=28133376
 TaskInfo: cluster:cluster0; timeout(s):10800; max_filter_ratio:5.0E-5
 ErrorMsg: N/A
 CreateTime: 2019-07-27 11:46:42
 EtlStartTime: 2019-07-27 11:46:44
 EtlFinishTime: 2019-07-27 11:49:44
 LoadStartTime: 2019-07-27 11:49:44
LoadFinishTime: 2019-07-27 11:50:16
  URL: http://1.1.1.1:8089/proxy/application_1586619723848_0035/
 JobDetails: {"ScannedRows":28133395,"TaskNumber":1,"FileNumber":1,"FileSize":200000}
~~~

- **State**
  
The current stage of the imported job.
PENDING: The job is committed.
ETL: Spark ETL is committed.
LOADING: The FE schedule an BE to execute push operation.
FINISHED: The push is completed and the version is effective.

There are two final stages of the import job –      `CANCELLED` and `FINISHED`, both indicating the load job is complete. `CANCELLED` indicates import failure and `FINISHED` indicates import success.

- **Progress**
  
Description of the import job progress. There are two types of progress –ETL and LOAD, which correspond to the two phases of the import process, ETL and LOADING.

- The range of progress for LOAD is 0~100%.
  
`LOAD progress = the number of currently completed tablets of all replications imports / the total number of tablets of this import job * 100%`.

- If all tables have been imported, the LOAD progress is 99%, and changes to 100% when the import enters the final validation phase.

- The import progress is not linear. If there is no change in progress for a period of time, it does not mean that the import is not executing.

- **Type**

 The type of the import job. SPARK for spark load.

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

These values represent the time when the import was created, when the ETL phase started, when the ETL phase completed,      when the LOADING phase started, and when the entire import job was completed.

- **JobDetails**

Displays the detailed running status of the job, including the number of imported files, total size (in bytes), number of subtasks, number of raw rows being processed, etc. For example:

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **URL**

You can copy the input to your browser to access  the web interface of the corresponding application.

### View Apache Spark™ Launcher commit logs

Sometimes users need to view the detailed logs generated during a Apache Spark™ job commit. By  default, the logs are saved in the path `log/spark_launcher_log` in the FE root directory      named as `spark-launcher-{load-job-id}-{label}.log`. The logs are      saved in this directory for a period of time and will be erased when the import information in FE metadata is cleaned up. The default retention time is 3 days.

### Cancel Import

When the Spark load job status is not `CANCELLED` or `FINISHED`, it can be cancelled manually by the user by specifying the Label of the import job.

---

## Related System Configurations

**FE Configuration:** The following configuration is the system-level configuration of Spark load, which applies to all Spark load import jobs. The configuration values can be adjusted mainly by modifying `fe.conf`.

- enable-spark-load: Enable Spark load and resource creation with a default value of false.
- spark-load-default-timeout-second: The default timeout for the job is 259200 seconds (3 days).
- spark-home-default-dir: The Spark client path (`fe/lib/spark2x`).
- spark-resource-path: The path to the packaged S     park dependency file (empty by default).
- spark-launcher-log-dir: The directory where the commit log of the Spark client is stored (`fe/log/spark-launcher-log`).
- yarn-client-path: The path to the yarn binary executable (`fe/lib/yarn-client/hadoop/bin/yarn`).
- yarn-config-dir: Yarn's configuration file path (`fe/lib/yarn-config`).

---

## Best Practices

The most suitable scenario for using Spark load is when the raw data is in the file system (HDFS) and the data volume is in the tens of GB to TB level. Use Stream Load or Broker Load for smaller data volumes.
完全なSparkロードのインポート例については、GitHubのデモを参照してください: [https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## よくある質問

- `Error: When running with master 'yarn' either HADOOP-CONF-DIR or YARN-CONF-DIR must be set in the environment.`

 「HADOOP-CONF-DIR」環境変数をSparkクライアントの`spark-env.sh`で設定せずにSpark Loadを使用するとこのエラーが発生します。

- `Error: Cannot run program "xxx/bin/spark-submit": error=2, No such file or directory`

 Spark Loadを使用する際に、`spark_home_default_dir`構成項目がSparkクライアントのルートディレクトリを指定していないことが原因です。

- `Error: File xxx/jars/spark-2x.zip does not exist.`

 Spark Loadを使用する際に、`spark-resource-path`構成項目がパッケージ化されたzipファイルを指していないことが原因です。

- `Error: yarn client does not exist in path: xxx/yarn-client/hadoop/bin/yarn`

 Spark Loadを使用する際に、`yarn-client-path`構成項目がyarnの実行可能ファイルを指定していないことが原因です。

- `ERROR: Cannot execute hadoop-yarn/bin/... /libexec/yarn-config.sh`

 CDHでHadoopを使用する際には、`HADOOP_LIBEXEC_DIR`環境変数を設定する必要があります。 
 `hadoop-yarn`ディレクトリとhadoopディレクトリが異なるため、デフォルトの`libexec`ディレクトリは`hadoop-yarn/bin/... /libexec`を探しに行きますが、実際はhadoopディレクトリに`libexec`があります。 
 ```yarn application status```コマンドはSparkタスクのステータスを取得する際にエラーを報告しており、これがインポートジョブの失敗の原因です。