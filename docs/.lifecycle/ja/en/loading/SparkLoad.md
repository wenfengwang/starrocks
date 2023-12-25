---
displayed_sidebar: English
---

# Spark Loadを使用してデータを一括で読み込む

このロードでは、外部のApache Spark™リソースを使用してインポートされたデータを前処理し、インポートのパフォーマンスを向上させ、計算リソースを節約します。これは主に**初期移行**と**大量データのインポート**に使用されます（データ量はTBレベルまで）。

Spark Loadは**非同期**インポート方法であり、ユーザーはMySQLプロトコルを介してSparkタイプのインポートジョブを作成し、`SHOW LOAD`を使用してインポート結果を確認する必要があります。

> **注意**
>
> - StarRocksテーブルにINSERT権限を持つユーザーのみが、このテーブルにデータをロードできます。必要な権限を付与するには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)の指示に従ってください。
> - Spark Loadは、プライマリキーテーブルにデータをロードするために使用することはできません。

## 用語の説明

- **Spark ETL**: インポートプロセス中のデータのETL（抽出、変換、ロード）を主に担当し、グローバル辞書の構築（BITMAP型）、パーティショニング、ソート、集約などを含みます。
- **Broker**: Brokerは独立したステートレスプロセスです。ファイルシステムインターフェースをカプセル化し、StarRocksがリモートストレージシステムからファイルを読み取る機能を提供します。
- **グローバル辞書**: 元の値からエンコードされた値へのデータマッピングを保存するデータ構造です。元の値は任意のデータ型であり、エンコードされた値は整数です。グローバル辞書は、正確なカウントdistinctが事前に計算されるシナリオで主に使用されます。

## 背景情報

StarRocks v2.4以前では、Spark LoadはBrokerプロセスに依存してStarRocksクラスターとストレージシステム間の接続を確立します。Spark Loadジョブを作成する際には、`WITH BROKER "<broker_name>"`を入力して使用するBrokerを指定する必要があります。Brokerは、ファイルシステムインターフェースと統合された独立したステートレスプロセスです。Brokerプロセスを使用することで、StarRocksはストレージシステムに保存されているデータファイルにアクセスし、読み取り、そのデータファイルのデータを前処理してロードするための計算リソースを使用できます。

StarRocks v2.5以降、Spark LoadはBrokerプロセスに依存せずにStarRocksクラスターとストレージシステム間の接続を確立できるようになりました。Spark Loadジョブを作成する際にBrokerを指定する必要はなくなりましたが、`WITH BROKER`キーワードは引き続き必要です。

> **注記**
>
> Brokerプロセスを使用しないロードは、複数のHDFSクラスターや複数のKerberosユーザーが存在するなど、特定の状況では機能しないことがあります。このような状況では、Brokerプロセスを使用してデータをロードすることができます。

## 基本原則

ユーザーはMySQLクライアントを介してSparkタイプのインポートジョブを提出し、FEはメタデータを記録し、提出結果を返します。

Spark Loadタスクの実行は以下の主要なフェーズに分かれます。

1. ユーザーがFEにSpark Loadジョブを提出します。
2. FEはApache Spark™クラスターにETLタスクの提出をスケジュールします。
3. Apache Spark™クラスターは、グローバル辞書の構築（BITMAP型）、パーティショニング、ソート、集約などを含むETLタスクを実行します。
4. ETLタスクが完了した後、FEは前処理された各スライスのデータパスを取得し、関連するBEにプッシュタスクの実行をスケジュールします。
5. BEはBrokerプロセスを介してHDFSからデータを読み取り、それをStarRocksのストレージ形式に変換します。
    > Brokerプロセスを使用しない場合、BEは直接HDFSからデータを読み取ります。
6. FEは有効なバージョンをスケジュールし、インポートジョブを完了します。

以下の図は、Spark Loadの主要なフローを示しています。

![Spark Load](../assets/4.3.2-1.png)

---

## グローバル辞書

### 適用シナリオ

現在、StarRocksのBITMAP列はRoaringbitmapを使用して実装されており、入力データタイプは整数のみです。したがって、インポートプロセス中にBITMAP列の事前計算を実装する場合は、入力データタイプを整数に変換する必要があります。

StarRocksの既存のインポートプロセスでは、グローバル辞書のデータ構造はHiveテーブルに基づいて実装され、元の値からエンコードされた値へのマッピングが保存されています。

### 構築プロセス

1. 上流データソースからデータを読み取り、`hive-table`という名前の一時的なHiveテーブルを生成します。
2. `hive-table`の非強調フィールドの値を抽出して、`distinct-value-table`という名前の新しいHiveテーブルを生成します。
3. 元の値用の列とエンコードされた値用の列を持つ新しいグローバル辞書テーブル`dict-table`を作成します。
4. `distinct-value-table`と`dict-table`の間で左結合を行い、ウィンドウ関数を使用してこのセットをエンコードします。最終的に、重複排除されたカラムの元の値とエンコードされた値が`dict-table`に書き戻されます。
5. `dict-table`と`hive-table`の間で結合を行い、`hive-table`の元の値を整数でエンコードされた値に置き換える作業を完了します。
6. `hive-table`は次回のデータ前処理時に読み取られ、計算後にStarRocksにインポートされます。

## データ前処理

データ前処理の基本的なプロセスは以下の通りです。

1. 上流データソース（HDFSファイルまたはHiveテーブル）からデータを読み取ります。
2. 読み取ったデータに対してフィールドマッピングと計算を行い、パーティション情報に基づいて`bucket-id`を生成します。
3. StarRocksテーブルのRollupメタデータに基づいてRollupTreeを生成します。
4. RollupTreeを反復処理し、階層的な集約操作を実行します。次の階層のRollupは、前の階層のRollupから計算できます。
5. 集約計算が完了するたびに、`bucket-id`に基づいてデータをバケット化し、HDFSに書き込みます。
6. その後のBrokerプロセスがHDFSからファイルをプルし、StarRocksのBEノードにインポートします。

## 基本操作

### 前提条件

Brokerプロセスを介してデータをロードし続ける場合は、StarRocksクラスターにBrokerプロセスがデプロイされていることを確認する必要があります。

[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocks クラスターにデプロイされている Broker を確認できます。Broker がデプロイされていない場合は、[Broker のデプロイ](../deployment/deploy_broker.md)に記載されている手順に従って Broker をデプロイする必要があります。

### ETL クラスターの設定

Apache Spark™ は、ETL 作業用の StarRocks の外部計算リソースとして使用されます。StarRocks には、クエリ用の Spark/GPU、外部ストレージ用の HDFS/S3、ETL 用の MapReduce など、他の外部リソースが追加される可能性があります。そのため、StarRocks が使用するこれらの外部リソースを管理する `Resource Management` を導入しています。

Apache Spark™ インポートジョブを送信する前に、ETL タスクを実行する Apache Spark™ クラスターを設定します。操作の構文は次のとおりです。

~~~sql
-- Apache Spark™ リソースを作成
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = 'spark',
 spark_conf_key = 'spark_conf_value',
 working_dir = 'path',
 broker = 'broker_name',
 broker.property_key = 'property_value'
);

-- Apache Spark™ リソースを削除
DROP RESOURCE resource_name;

-- リソースを表示
SHOW RESOURCES;
SHOW PROC "/resources";

-- 権限
GRANT USAGE_PRIV ON RESOURCE resource_name TO 'user_identity';
GRANT USAGE_PRIV ON RESOURCE resource_name TO ROLE 'role_name';
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM 'user_identity';
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM ROLE 'role_name';
~~~

- リソースの作成

**例**:

~~~sql
-- yarn クラスターモード
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

-- yarn HA クラスターモード
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

`resource_name` は StarRocks で設定された Apache Spark™ リソースの名前です。

`PROPERTIES` には、Apache Spark™ リソースに関連するパラメーターが含まれており、以下の通りです。
> **注記**
>
> Apache Spark™ リソースの PROPERTIES の詳細な説明については、[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md) を参照してください。

- Spark 関連のパラメーター:
  - `type`: リソースの種類、必須、現在は `spark` のみをサポート。
  - `spark.master`: 必須、現在は `yarn` のみをサポート。
    - `spark.submit.deployMode`: Apache Spark™ プログラムのデプロイモード、必須、現在は `cluster` と `client` の両方をサポート。
    - `spark.hadoop.fs.defaultFS`: master が yarn の場合は必須。
    - yarn リソースマネージャーに関連するパラメーター、必須。
      - シングルノード上の ResourceManager
        `spark.hadoop.yarn.resourcemanager.address`: シングルポイントリソースマネージャーのアドレス。
      - ResourceManager HA
        > ResourceManager のホスト名またはアドレスを指定することができます。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`: リソースマネージャーの HA を有効にし、`true` に設定。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`: リソースマネージャーの論理 ID のリスト。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`: 各 rm-id に対して、リソースマネージャーのホスト名を指定。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id`: 各 rm-id に対して、クライアントがジョブを送信するための `host:port` を指定。

- `working_dir`: ETL に使用されるディレクトリ。Apache Spark™ を ETL リソースとして使用する場合は必須。例: `hdfs://host:port/tmp/starrocks`。

- Broker 関連のパラメーター:
  - `broker`: Broker 名。Apache Spark™ を ETL リソースとして使用する場合は必須。`ALTER SYSTEM ADD BROKER` コマンドを使用して事前に設定を完了する必要があります。
  - `broker.property_key`: ETL が生成した中間ファイルを Broker プロセスが読み取る際に指定する情報（例: 認証情報）。

**注意**:

上記は Broker プロセスを介してデータをロードするためのパラメーターの説明です。Broker プロセスなしでデータをロードする場合は、以下の点に注意してください。

- `broker` を指定する必要はありません。
- NameNode ノードのユーザー認証と HA を設定する必要がある場合、HDFS クラスターの hdfs-site.xml ファイルでパラメーターを設定する必要があります（パラメーターの説明については、[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs) を参照）。また、各 FE の `$FE_HOME/conf` と各 BE の `$BE_HOME/conf` の下に **hdfs-site.xml** ファイルを移動する必要があります。

> 注記
>
> HDFS ファイルに特定のユーザーのみがアクセスできる場合、`broker.username` で HDFS ユーザー名を指定し、`broker.password` でユーザーパスワードを指定する必要があります。

- リソースの表示

通常アカウントは、`USAGE_PRIV` アクセス権のあるリソースのみを表示できます。root アカウントと admin アカウントは、すべてのリソースを表示できます。

- リソースの権限

リソースの権限は `GRANT` と `REVOKE` を使用して管理され、現在は `USAGE_PRIV` 権限のみをサポートしています。ユーザーやロールに `USAGE_PRIV` 権限を付与できます。

~~~sql
-- user0 に対して spark0 リソースへのアクセスを許可
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- role0 に対して spark0 リソースへのアクセスを許可
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- user0 に対してすべてのリソースへのアクセスを許可
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- role0 に対してすべてのリソースへのアクセスを許可
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- user0 から spark0 リソースの使用権限を取り消す
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### Spark クライアントの設定

FE で Spark クライアントを設定し、`spark-submit` コマンドを実行して Spark タスクを送信できるようにします。Spark2 2.4.5 以降の公式バージョンを使用することをお勧めします。[Spark ダウンロードアドレス](https://archive.apache.org/dist/spark/)。ダウンロード後、以下の手順で設定を完了してください。

- `SPARK_HOME` の設定
  
Spark クライアントを FE と同じマシン上のディレクトリに配置し、FE の設定ファイルで `spark_home_default_dir` をこのディレクトリに設定します。デフォルトは FE のルートディレクトリ内の `lib/spark2x` パスで、空にすることはできません。

- **Spark 依存パッケージの設定**
  
依存関係パッケージを設定するには、Spark クライアントの `jars` フォルダ内のすべての jar ファイルを zip 形式でアーカイブし、FE の設定で `spark_resource_path` 項目をこの zip ファイルに設定します。この設定が空の場合、FE は FE のルートディレクトリ内の `lib/spark2x/jars/spark-2x.zip` ファイルを探します。FE がファイルを見つけられない場合は、エラーを報告します。

spark load ジョブが送信されると、アーカイブされた依存関係ファイルがリモートリポジトリにアップロードされます。デフォルトのリポジトリパスは `working_dir/{cluster_id}` ディレクトリの下に `--spark-repository--{resource-name}` という名前で作成されます。これは、クラスタ内のリソースがリモートリポジトリに対応していることを意味します。ディレクトリ構造は以下の通りです：

~~~bash
---spark-repository--spark0/

   |---archive-1.0.0/

   |        |\---lib-990325d2c0d1d5e45bf675e54e44fb16-spark-dpp-1.0.0-jar-with-dependencies.jar

   |        |\---lib-7670c29daf535efe3c9b923f778f61fc-spark-2x.zip

   |---archive-1.1.0/

   |        |\---lib-64d5696f99c379af2bee28c1c84271d5-spark-dpp-1.1.0-jar-with-dependencies.jar

   |        |\---lib-1bbb74bb6b264a270bc7fca3e964160f-spark-2x.zip

   |---archive-1.2.0/

   |        |-...

~~~

デフォルトで `spark-2x.zip` と名付けられた Spark の依存関係に加えて、FE は DPP の依存関係もリモートリポジトリにアップロードします。Spark load によって送信された依存関係がリモートリポジトリに既に存在する場合、依存関係を再度アップロードする必要はなく、毎回大量のファイルを繰り返しアップロードする時間が節約されます。

### YARN クライアントの設定

FE が YARN コマンドを実行して実行中のアプリケーションのステータスを取得したり、アプリケーションを終了させたりできるように、FE に YARN クライアントを設定します。Hadoop 2.5.2 以降の公式バージョンの使用を推奨します（[Hadoop ダウンロードアドレス](https://archive.apache.org/dist/hadoop/common/)）。ダウンロード後、以下の手順で設定を完了してください：

- **YARN 実行可能パスの設定**
  
ダウンロードした YARN クライアントを FE と同じマシン上のディレクトリに配置し、FE の設定ファイルで `yarn_client_path` 項目を YARN のバイナリ実行ファイルに設定します。デフォルトでは、FE のルートディレクトリ内の `lib/yarn-client/hadoop/bin/yarn` パスです。

- **YARN コマンド実行に必要な設定ファイルのパスの設定（オプション）**
  
FE が YARN クライアントを介してアプリケーションのステータスを取得したり、アプリケーションを終了させたりする際に、デフォルトでは StarRocks は FE のルートディレクトリの `lib/yarn-config` パスに必要な設定ファイルを生成します。このパスは、FE の設定ファイルで `yarn_config_dir` 項目を設定することで変更できます。現在は `core-site.xml` と `yarn-site.xml` を含んでいます。

### インポートジョブの作成

**構文：**

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

**例 1**: アップストリームデータソースが HDFS の場合

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
    WHERE col1 > 1
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

**例 2**: アップストリームデータソースが Hive の場合。

- ステップ 1: 新しい Hive リソースを作成する

~~~sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:8080"
);
 ~~~

- ステップ 2: 新しい Hive 外部テーブルを作成する

~~~sql
CREATE EXTERNAL TABLE hive_t1
(
    k1 INT,
    k2 SMALLINT,
    k3 VARCHAR(50),
    uuid VARCHAR(100)
)
ENGINE=hive
PROPERTIES
( 
    "resource" = "hive0",
    "database" = "tmp",
    "table" = "t1"
);
 ~~~

- ステップ 3: Load コマンドを送信し、インポートされる StarRocks テーブルの列が Hive 外部テーブルに存在することを要求する。

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

Spark ロードのパラメーター紹介：

- **ラベル**
  
インポートジョブのラベル。各インポートジョブにはデータベース内で一意のラベルがあり、ブローカーロードと同じルールに従います。

- **データ記述クラスのパラメーター**
  
現在サポートされているデータソースは CSV と Hive テーブルです。他のルールはブローカーロードと同じです。

- **インポートジョブのパラメーター**
  
インポートジョブのパラメーターは、インポートステートメントの `opt_properties` セクションに属するパラメーターです。これらのパラメーターはインポートジョブ全体に適用されます。ルールはブローカーロードと同じです。

- **Spark リソースのパラメーター**
  
Spark リソースは事前に StarRocks に設定されている必要があり、リソースを Spark ロードに適用する前にユーザーに USAGE-PRIV 権限を付与する必要があります。
Spark リソースのパラメーターは、ジョブにリソースを追加したり、Spark の設定を変更したりするなど、ユーザーが一時的なニーズを持っている場合に設定できます。この設定はこのジョブにのみ影響し、StarRocks クラスターの既存の設定には影響しません。

~~~sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

- **データソースが Hive の場合のインポート**
  

現在、インポートプロセスでHiveテーブルを使用するには、`Hive`型の外部テーブルを作成し、インポートコマンドを送信する際にその名前を指定する必要があります。

- **グローバル辞書を構築するためのインポートプロセス**
  
ロードコマンドでは、グローバル辞書を構築するために必要なフィールドを次の形式で指定できます: `StarRocksフィールド名=bitmap_dict(Hiveテーブルフィールド名)`。現在、**グローバル辞書はアップストリームデータソースがHiveテーブルの場合にのみサポートされています**。

- **バイナリタイプデータのロード**

v2.5.17以降、Spark Loadは`bitmap_from_binary`関数をサポートしており、バイナリデータをビットマップデータに変換できます。HiveテーブルまたはHDFSファイルのカラムタイプがバイナリで、StarRocksテーブルの対応するカラムがビットマップ型集約カラムの場合、ロードコマンドでフィールドを次の形式で指定できます: `StarRocksフィールド名=bitmap_from_binary(Hiveテーブルフィールド名)`。これによりグローバル辞書の構築が不要になります。

## インポートジョブの確認

Spark Loadによるインポートは非同期であり、ブローカーロードも同様です。ユーザーはインポートジョブのラベルを記録し、`SHOW LOAD`コマンドでインポート結果を確認する必要があります。インポートを確認するコマンドはすべてのインポート方法に共通です。以下が例です。

返されるパラメーターの詳細な説明については、Broker Loadを参照してください。違いは以下の通りです。

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

- **状態**
  
インポートジョブの現在のステージです。
PENDING: ジョブがコミットされました。
ETL: Spark ETLがコミットされました。
LOADING: FEがBEにプッシュ操作を実行するようスケジュールします。
FINISHED: プッシュが完了し、バージョンが有効になりました。

インポートジョブには`CANCELLED`と`FINISHED`の2つの最終ステージがあり、どちらもロードジョブが完了したことを示します。`CANCELLED`はインポート失敗を、`FINISHED`はインポート成功を示します。

- **進捗**
  
インポートジョブの進捗状況の説明です。ETLとLOADの2種類の進捗があり、これらはインポートプロセスの2フェーズ、ETLとLOADINGに対応しています。

- LOADの進捗範囲は0～100%です。
  
`LOAD進捗 = 現在完了した全レプリケーションインポートのタブレット数 / このインポートジョブのタブレット総数 * 100%`

- すべてのテーブルがインポートされた場合、LOAD進捗は99%で、インポートが最終検証フェーズに入ると100%に変わります。

- インポートの進捗は非線形です。一定期間進捗に変化がない場合でも、インポートが実行されていないわけではありません。

- **タイプ**

インポートジョブのタイプです。Spark Loadの場合はSPARKです。

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

これらの値は、インポートが作成された時刻、ETLフェーズが開始された時刻、ETLフェーズが完了した時刻、LOADINGフェーズが開始された時刻、およびインポートジョブ全体が完了した時刻を表します。

- **JobDetails**

インポートされたファイルの数、合計サイズ（バイト単位）、サブタスクの数、処理中の生の行の数など、ジョブの詳細な実行ステータスを表示します。例えば：

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **URL**

入力をブラウザにコピーして、対応するアプリケーションのWebインターフェースにアクセスできます。

### Apache Spark™ Launcherのコミットログを表示

ユーザーは、Apache Spark™ジョブのコミット中に生成された詳細なログを表示する必要がある場合があります。デフォルトでは、ログはFEルートディレクトリ内の`log/spark_launcher_log`パスに`spark-launcher-{load-job-id}-{label}.log`という名前で保存されます。ログはこのディレクトリに一定期間保存され、FEメタデータのインポート情報がクリーンアップされると削除されます。デフォルトの保持期間は3日間です。

### インポートのキャンセル

Spark Loadジョブのステータスが`CANCELLED`または`FINISHED`でない場合、ユーザーはインポートジョブのラベルを指定して手動でキャンセルできます。

---

## 関連するシステム設定

**FE設定:** 以下の設定は、すべてのSpark Loadインポートジョブに適用されるSpark Loadのシステムレベル設定です。設定値は主に`fe.conf`を変更することで調整できます。

- enable-spark-load: Spark Loadとリソース作成をデフォルト値falseで有効にします。
- spark-load-default-timeout-second: ジョブのデフォルトタイムアウトは259200秒（3日間）です。
- spark-home-default-dir: Sparkクライアントパス（`fe/lib/spark2x`）。
- spark-resource-path: パッケージ化されたSpark依存ファイルのパス（デフォルトでは空）。
- spark-launcher-log-dir: Sparkクライアントのコミットログが保存されるディレクトリ（`fe/log/spark-launcher-log`）。
- yarn-client-path: yarnバイナリ実行ファイルのパス（`fe/lib/yarn-client/hadoop/bin/yarn`）。
- yarn-config-dir: Yarnの設定ファイルパス（`fe/lib/yarn-config`）。

---

## ベストプラクティス

Spark Loadを使用する最適なシナリオは、生データがファイルシステム（HDFS）にあり、データボリュームが数十GBからTBレベルの場合です。データボリュームが小さい場合は、Stream LoadまたはBroker Loadを使用してください。

完全なSpark Loadインポートの例については、GitHubのデモを参照してください: [https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## よくある質問(FAQ)

- `Error: When running with master 'yarn' either HADOOP-CONF-DIR or YARN-CONF-DIR must be set in the environment.`

Sparkクライアントの`spark-env.sh`で環境変数`HADOOP-CONF-DIR`を設定せずにSpark Loadを使用する場合。

- `Error: Cannot run program "xxx/bin/spark-submit": error=2, No such file or directory`


`spark_home_default_dir` 設定項目は、Spark Loadを使用する際にSparkクライアントのルートディレクトリを指定しません。

- `Error: File xxx/jars/spark-2x.zip does not exist.`

`spark-resource-path` 設定項目は、Spark Loadを使用する際にパックされたzipファイルを指していません。

- `Error: yarn client does not exist in path: xxx/yarn-client/hadoop/bin/yarn`

`yarn-client-path` 設定項目は、Spark Loadを使用する際にyarn実行ファイルを指定していません。

- `ERROR: Cannot execute hadoop-yarn/bin/.../libexec/yarn-config.sh`

CDHを使用してHadoopを使用する場合、`HADOOP_LIBEXEC_DIR` 環境変数を設定する必要があります。
`hadoop-yarn` とhadoopディレクトリは異なるため、デフォルトの`libexec`ディレクトリは`hadoop-yarn/bin/.../libexec`を探しますが、`libexec`はhadoopディレクトリ内にあります。
コマンド`yarn application status`でSparkタスクの状態を取得する際にエラーが発生し、インポートジョブが失敗する原因となりました。
