---
displayed_sidebar: "Japanese"
---

# Spark Loadを使用して一括データをロードする

このロードは、インポートされたデータの前処理に外部のApache Spark™リソースを使用し、インポートのパフォーマンスを向上させ、計算リソースを節約します。これは主に**初期移行**および**大規模データインポート**を行うために使用され、StarRocks（データ量がTBレベルまで）にデータをインポートします。

Sparkロードは、ユーザーがMySQLプロトコルを介してSparkタイプのインポートジョブを作成し、`SHOW LOAD`を使用してインポート結果を表示する**非同期**のインポート方法です。

> **注意**
>
> - StarRocksテーブルにINSERT権限を持つユーザーのみがこのテーブルにデータをロードできます。必要な権限を付与するための手順は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供されている手順に従ってください。
> - Spark Loadは、プライマリキーテーブルにデータをロードするために使用できません。

## 用語の説明

- **Spark ETL**: インポートプロセスのデータのETLを主に担当し、グローバルディクショナリの構築（BITMAPタイプ）、パーティショニング、ソート、集計などを行います。
- **Broker**: Brokerは独立したステートレスプロセスです。ファイルシステムインターフェースをカプセル化し、StarRocksにリモートストレージシステムからファイルを読み取る機能を提供します。
- **グローバルディクショナリ**: オリジナルの値からエンコードされた値へのデータのマッピングを保存するデータ構造です。オリジナルの値は任意のデータ型になりますが、エンコードされた値は整数です。グローバルディクショナリは、正確な重複カウントが事前に計算されるシナリオで主に使用されます。

## 背景情報

StarRocks v2.4以前では、Spark LoadはBrokerプロセスに依存してStarRocksクラスタとストレージシステムの間の接続を設定していました。Spark Loadジョブを作成する際には、`WITH BROKER "<broker_name>"`を入力して使用するBrokerを指定する必要があります。Brokerは独立したステートレスプロセスであり、ファイルシステムインターフェースをカプセル化し、StarRocksがストレージシステムに保存されているデータファイルをアクセスして読み取り、これらのデータファイルのデータを前処理およびロードするために独自の計算リソースを使用できるようにします。

StarRocks v2.5以降では、Spark LoadはもはやBrokerプロセスに依存せず、StarRocksクラスタとストレージシステムの間の接続を設定する必要がありません。Spark Loadジョブを作成する際には、Brokerを指定する必要はありませんが、`WITH BROKER`キーワードを引き続き指定する必要があります。

> **注意**
>
> 特定の状況では、Brokerプロセスを使用せずにロードすることができない場合があります。たとえば、複数のHDFSクラスタや複数のKerberosユーザーがある場合などです。この場合、Brokerプロセスを使用してデータをロードすることもできます。

## 基本原則

ユーザーはMySQLクライアントを介してSparkタイプのインポートジョブを提出します。FEはメタデータを記録し、提出結果を返します。

Sparkロードタスクの実行は、次の主要なフェーズに分割されます。

1. ユーザーはSparkロードジョブをFEに提出します。
2. FEはApache Spark™クラスタにETLタスクの提出をスケジュールします。
3. Apache Spark™クラスタは、グローバルディクショナリの構築（BITMAPタイプ）、パーティショニング、ソート、集計などを含むETLタスクを実行します。
4. ETLタスクが完了した後、FEは各前処理スライスのデータパスを取得し、関連するBEをスケジュールしてプッシュタスクを実行します。
5. BEはBrokerプロセスを介してHDFSからデータを読み取り、StarRocksのストレージ形式に変換します。
    > Brokerプロセスを使用しない場合、BEはHDFSから直接データを読み取ります。
6. FEは有効なバージョンをスケジュールし、インポートジョブを完了します。

次の図は、Sparkロードの主なフローを示しています。

![Sparkロード](../assets/4.3.2-1.png)

---

## グローバルディクショナリ

### 適用シナリオ

現在、StarRocksのBITMAP列はRoaringbitmapを使用して実装されており、入力データ型は整数のみです。したがって、インポートプロセスのBITMAP列の事前計算を実装する場合は、入力データ型を整数に変換する必要があります。

StarRocksの既存のインポートプロセスでは、グローバルディクショナリのデータ構造はHiveテーブルに基づいて実装されており、オリジナルの値からエンコードされた値へのマッピングを保存しています。

### 構築プロセス

1. 上流データソースからデータを読み取り、一時的なHiveテーブルである`hive-table`を生成します。
2. `hive-table`の非強調フィールドの値を抽出して、新しいHiveテーブルである`distinct-value-table`を生成します。
3. オリジナルの値用の1つの列とエンコードされた値用の1つの列を持つ新しいグローバルディクショナリテーブルである`dict-table`を作成します。
4. `distinct-value-table`と`dict-table`の間で左結合を行い、ウィンドウ関数を使用してこのセットをエンコードします。最終的に、重複のない列のオリジナル値とエンコードされた値の両方が`dict-table`に書き戻されます。
5. `dict-table`と`hive-table`の間で結合を行い、`hive-table`のオリジナル値を整数のエンコードされた値で置き換えるジョブを完了します。
6. `hive-table`は次回のデータ前処理で読み取られ、計算後にStarRocksにインポートされます。

## データ前処理

データ前処理の基本的なプロセスは次のとおりです。

1. 上流データソース（HDFSファイルまたはHiveテーブル）からデータを読み取ります。
2. 読み取ったデータのフィールドマッピングと計算を完了し、パーティション情報に基づいて`bucket-id`を生成します。
3. StarRocksテーブルのロールアップメタデータに基づいてRollupTreeを生成します。
4. RollupTreeを反復処理し、階層的な集計操作を実行します。次の階層のロールアップは、前の階層のロールアップから計算できます。
5. 集計計算が完了するたびに、データは`bucket-id`に基づいてバケット分割され、次にHDFSに書き込まれます。
6. 次のBrokerプロセスは、HDFSからファイルを取得し、StarRocks BEノードにインポートします。

## 基本操作

### 前提条件

Brokerプロセスを介して引き続きデータをロードする場合は、StarRocksクラスタにBrokerプロセスがデプロイされていることを確認する必要があります。

[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタにデプロイされているBrokerを確認できます。Brokerがデプロイされていない場合は、[Deploy Broker](../deployment/deploy_broker.md)で提供されている手順に従ってBrokerをデプロイする必要があります。

### ETLクラスタの設定

ETL作業のために、StarRocksで外部の計算リソースとしてApache Spark™を使用します。StarRocksには、Spark/GPUを使用したクエリ、外部ストレージとしてのHDFS/S3、ETLとしてのMapReduceなど、他の外部リソースが追加される場合があります。そのため、これらの外部リソースをStarRocksが使用するために`Resource Management`を導入して管理します。

Apache Spark™を使用したインポートジョブを提出する前に、ETLタスクを実行するためにApache Spark™クラスタを構成します。操作の構文は次のとおりです。

~~~sql
-- Apache Spark™リソースを作成
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = spark,
 spark_conf_key = spark_conf_value,
 working_dir = path,
 broker = broker_name,
 broker.property_key = property_value
);

-- Apache Spark™リソースを削除
DROP RESOURCE resource_name;

-- リソースを表示
SHOW RESOURCES
SHOW PROC "/resources";

-- 権限
GRANT USAGE_PRIV ON RESOURCE resource_name TO user_identityGRANT USAGE_PRIV ON RESOURCE resource_name TO ROLE role_name;
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM user_identityREVOKE USAGE_PRIV ON RESOURCE resource_name FROM ROLE role_name;
~~~

- リソースの作成

**例**:

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

`PROPERTIES`には、Apache Spark™リソースに関連するパラメータが含まれます。たとえば:

- リソースの作成

**例**:

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

- Spark関連のパラメータ:
  - `type`: リソースタイプ、必須、現在は`spark`のみサポートされています。
  - `spark.master`: 必須、現在は`yarn`のみサポートされています。
    - `spark.submit.deployMode`: Apache Spark™プログラムのデプロイモード、必須、現在は`cluster`および`client`の両方をサポートしています。
    - `spark.hadoop.fs.defaultFS`: masterがyarnの場合は必須です。
    - yarnリソースマネージャに関連するパラメータ、必須です。
      - 単一ノードのResourceManager
        `spark.hadoop.yarn.resourcemanager.address`: 単一ポイントリソースマネージャのアドレス。
      - ResourceManager HA
        > ResourceManagerのホスト名またはアドレスを指定することができます。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`: リソースマネージャHAを有効にするには、`true`に設定します。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`: リソースマネージャの論理IDのリスト。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`: 各rm-idに対して、リソースマネージャに対してジョブを提出するための`host:port`を指定します。

- `*working_dir`: ETLに使用されるディレクトリ。Apache Spark™がETLリソースとして使用される場合は必須です。たとえば: `hdfs://host:port/tmp/starrocks`。

- Broker関連のパラメータ:
  - `broker`: Broker名。Apache Spark™がETLリソースとして使用される場合は必須です。事前に`ALTER SYSTEM ADD BROKER`コマンドを使用して構成を完了する必要があります。
  - `broker.property_key`: ETLで生成された中間ファイルをBrokerプロセスが読み取るときに指定する情報（認証情報など）。

**注意**:

上記はBrokerプロセスを介してロードするパラメータの説明です。Brokerプロセスを使用せずにデータをロードする場合は、次の点に注意してください。

- `broker`を指定する必要はありません。
- ユーザー認証を構成する必要がある場合、およびNameNodeノードのHAを構成する必要がある場合は、HDFSクラスタのhdfs-site.xmlファイルでパラメータを構成する必要があります。パラメータの説明については、[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。また、各FEの**$FE_HOME/conf**および各BEの**$BE_HOME/conf**に**hdfs-site.xml**ファイルを移動する必要があります。

> 注意
>
> HDFSファイルに特定のユーザーのみがアクセスできる場合は、`broker.name`でHDFSユーザー名を指定し、`broker.password`でユーザーパスワードを指定する必要があります。

- リソースの表示

通常のアカウントは、アクセス権限があるリソースのみを表示できます。ルートアカウントと管理者アカウントはすべてのリソースを表示できます。

- リソースの権限

リソースの権限は、`GRANT REVOKE`を介して管理されます。現在は`USAGE_PRIV`権限のみをサポートしています。ユーザーまたはロールに`USAGE_PRIV`権限を付与できます。

~~~sql
-- ユーザーuser0に対してspark0リソースのアクセス権を付与する
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- ロールrole0に対してspark0リソースのアクセス権を付与する
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- ユーザーuser0に対してすべてのリソースへのアクセス権を付与する
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- ロールrole0に対してすべてのリソースへのアクセス権を付与する
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- ユーザーuser0からspark0リソースの使用権限を取り消す
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### Sparkクライアントの設定

FEでSparkクライアントを設定し、FEが`spark-submit`コマンドを実行してSparkタスクを提出できるようにします。公式バージョンのSpark2 2.4.5以上を使用することをお勧めします。[sparkのダウンロードアドレス](https://archive.apache.org/dist/spark/)からダウンロードした後、次の手順で設定を完了します。

- `SPARK-HOME`の設定
  
SparkクライアントをFEと同じマシンのディレクトリに配置し、FEの設定ファイルである`spark_home_default_dir`をこのディレクトリに設定します。デフォルトでは、FEのルートディレクトリの`lib/spark2x`パスになりますが、空にすることはできません。

- **SPARKの依存パッケージの設定**
  
依存パッケージを設定するには、Sparkクライアントの`jars`フォルダーのすべてのjarファイルをzipアーカイブしてアーカイブし、FEの設定ファイルである`spark_resource_path`項目をこのzipファイルに設定します。この設定が空の場合、FEはFEのルートディレクトリの`lib/spark2x/jars/spark-2x.zip`ファイルを見つけようとします。FEが見つからない場合は、エラーが報告されます。

Sparkロードジョブが提出されると、依存関係のあるファイルがリモートリポジトリにアップロードされます。デフォルトのリポジトリパスは、`working_dir/{cluster_id}`ディレクトリの`--spark-repository--{resource-name}`という名前で、クラスタのリソースにリモートリポジトリが対応します。ディレクトリ構造は次のようになります。

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

Sparkの依存関係（デフォルトでは`spark-2x.zip`という名前）以外にも、FEはDPPの依存関係をリモートリポジトリにアップロードします。Sparkロードによって提出された依存関係がすでにリモートリポジトリに存在する場合、毎回大量のファイルを繰り返しアップロードする必要はありません。

### YARNクライアントの設定

FEでYARNクライアントを設定し、FEが実行中のアプリケーションのステータスを取得したり、アプリケーションをキルしたりするためにyarnコマンドを実行できるようにします。公式バージョンのHadoop2 2.5.2以上を使用することをお勧めします([hadoopのダウンロードアドレス](https://archive.apache.org/dist/hadoop/common/))。ダウンロードした後、次の手順で設定を完了します。

- **YARN実行可能ファイルのパスの設定**
  
FEと同じマシンのディレクトリにダウンロードしたyarnクライアントを配置し、FEの設定ファイルである`yarn_client_path`項目をyarnのバイナリ実行ファイルに設定します。デフォルトでは、FEのルートディレクトリの`lib/yarn-client/hadoop/bin/yarn`パスです。

- **YARN（オプション）の生成に必要な設定ファイルのパスの設定**
  
FEは、デフォルトでは、FEのルートディレクトリの`lib/yarn-config`パスにある`core-site.xml`および`yarn-site.xml`という名前の設定ファイルを実行するために必要な設定ファイルを生成します。このパスは、FEの設定ファイルである`yarn_config_dir`エントリを設定することで変更できます。

### インポートジョブの作成

**構文**:

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

**例1**: 上流データソースがHDFSの場合

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

**例2**: 上流データソースがHiveの場合

- ステップ1: 新しいHiveリソースを作成する

~~~sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://0.0.0.0:8080"
);
 ~~~

- ステップ2: 新しいHive外部テーブルを作成する

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

- ステップ3: ロードコマンドを提出し、インポートするStarRocksテーブルの列がHive外部テーブルに存在することを要求します。

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

Sparkロードのパラメータの説明:

- **Label**
  
インポートジョブのラベル。各インポートジョブには、データベース内で一意のラベルがあり、brokerロードと同じルールに従います。

- **データ記述クラスパラメータ**
  
現在、サポートされているデータソースはCSVとHiveテーブルです。その他のルールはbrokerロードと同じです。

- **インポートジョブパラメータ**
  
インポートジョブパラメータは、インポートステートメントの`opt_properties`セクションに属するパラメータを指します。これらのパラメータは、インポートジョブ全体に適用されます。ルールはbrokerロードと同じです。

- **Sparkリソースパラメータ**
  
Sparkリソースは事前にStarRocksに構成する必要があり、ユーザーにはUSAGE-PRIV権限が与えられている必要があります。Sparkリソースパラメータは、ユーザーが一時的に必要な場合など、一時的なニーズがある場合に設定できます。この設定はこのジョブにのみ影響を与え、StarRocksクラスタの既存の設定には影響しません。

~~~sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

- **Hiveデータソースの場合のインポート**
  
現在、インポートプロセスでHiveテーブルを使用するには、`Hive`タイプの外部テーブルを作成し、インポートコマンドを提出する際にその名前を指定する必要があります。

- **グローバルディクショナリの構築プロセスのインポート**
  
ロードコマンドでは、グローバルディクショナリの構築に必要なフィールドを次の形式で指定できます: `StarRocksフィールド名=bitmap_dict(Hiveテーブルフィールド名)`ただし、現在**グローバルディクショナリは、上流データソースがHiveテーブルの場合のみサポートされています**。

- **ビットマップバイナリタイプの列のインポート**

StarRocksテーブルの集計列に適用されるデータ型はビットマップタイプであり、データソースのHiveテーブルまたはHDFSファイルタイプの対応する列のデータ型はバイナリ（FEクラスのシリアル化であるspark-dppのcom.starrocks.load.loadv2.dpp.BitmapValueを介して）です。

グローバルディクショナリの構築は必要ありません。ロードコマンドで対応するフィールドを指定するだけです。形式は次のとおりです: ```StarRocksフィールド名=bitmap_from_binary(Hiveテーブルフィールド名)```

## インポートジョブの表示

Sparkロードインポートは非同期であり、brokerロードと同様です。ユーザーはインポートジョブのラベルを記録し、`SHOW LOAD`コマンドでインポート結果を表示する必要があります。インポートの表示コマンドは、すべてのインポート方法で共通です。例を以下に示します。

詳細なパラメータの説明については、Broker Loadを参照してください。違いは次のとおりです。

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
  
インポートジョブの現在のステージ。
PENDING: ジョブが提出されました。
ETL: Spark ETLが提出されました。
LOADING: FEがBEをスケジュールしてプッシュ操作を実行します。
FINISHED: プッシュが完了し、バージョンが有効になります。

インポートジョブには2つの最終ステージがあります - `CANCELLED`および`FINISHED`、どちらもロードジョブが完了したことを示します。`CANCELLED`はインポートの失敗を示し、`FINISHED`はインポートの成功を示します。

- **Progress**
  
インポートジョブの進捗状況の説明。インポートプロセスの2つのフェーズ、ETLとLOADに対応します。

- LOADの進捗状況は0〜100%の範囲です。
  
`LOADの進捗状況 = インポートジョブのすべてのレプリケーションの現在完了したタブレットの数 / このインポートジョブの総タブレット数 * 100%`です。

- すべてのテーブルがインポートされた場合、LOADの進捗は99%になり、インポートが最終的な検証フェーズに入ると100%に変わります。

- インポートの進捗は直線的ではありません。一定期間進捗が変化しない場合、インポートが実行されていないことを意味しません。

- **Type**

 インポートジョブのタイプ。Sparkロードの場合はSPARKです。

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

これらの値は、インポートが作成された時点、ETLフェーズが開始された時点、ETLフェーズが完了した時点、LOADINGフェーズが開始された時点、インポートジョブ全体が完了した時点を表します。

- **JobDetails**

ジョブの詳細な実行状況を表示します。インポートされたファイルの数、合計サイズ（バイト単位）、サブタスクの数、処理中の生の行数などが含まれます。例:

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **URL**

対応するアプリケーションのWebインターフェースにアクセスするために、入力をブラウザにコピーできます。

### Apache Spark™ Launcherのコミットログの表示

Apache Spark™ジョブのコミット時に生成された詳細なログを表示する必要がある場合があります。デフォルトでは、ログはFEのルートディレクトリの`log/spark_launcher_log`パスに保存され、`spark-launcher-{load-job-id}-{label}.log`という名前で保存されます。ログは一定期間このディレクトリに保存され、FEメタデータのインポート情報がクリーンアップされると削除されます。デフォルトの保持期間は3日です。

### インポートのキャンセル

Sparkロードジョブのステータスが`CANCELLED`または`FINISHED`でない場合、ユーザーはラベルを指定して手動でキャンセルできます。

---
## 関連するシステム設定

**FEの設定:** 以下の設定は、すべてのSparkロードインポートジョブに適用されるSparkロードのシステムレベルの設定です。設定値は主に`fe.conf`を変更することで調整できます。

- enable-spark-load: Sparkロードとリソースの作成を有効にする。デフォルト値はfalseです。
- spark-load-default-timeout-second: ジョブのデフォルトタイムアウトは259200秒（3日）です。
- spark-home-default-dir: Sparkクライアントのパス（`fe/lib/spark2x`）です。
- spark-resource-path: パッケージ化されたSparkの依存ファイルのパス（デフォルトは空です）。
- spark-launcher-log-dir: Sparkクライアントのコミットログが保存されるディレクトリ（`fe/log/spark-launcher-log`）です。
- yarn-client-path: yarnバイナリの実行可能ファイルのパス（`fe/lib/yarn-client/hadoop/bin/yarn`）です。
- yarn-config-dir: Yarnの設定ファイルのパス（`fe/lib/yarn-config`）です。

---

## ベストプラクティス

Sparkロードを使用する最適なシナリオは、生データがファイルシステム（HDFS）にあり、データのボリュームが数十GBからTBレベルの場合です。データボリュームが小さい場合は、Stream LoadまたはBroker Loadを使用してください。

完全なSparkロードインポートの例については、githubのデモを参照してください：[https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## よくある質問

- `Error: When running with master 'yarn' either HADOOP-CONF-DIR or YARN-CONF-DIR must be set in the environment.`

 Spark Loadを使用する際に、Sparkクライアントの`spark-env.sh`で`HADOOP-CONF-DIR`環境変数を設定していない場合のエラーです。

- `Error: Cannot run program "xxx/bin/spark-submit": error=2, No such file or directory`

 Spark Loadを使用する際に、`spark-home-default-dir`の設定項目がSparkクライアントのルートディレクトリを指定していない場合のエラーです。

- `Error: File xxx/jars/spark-2x.zip does not exist.`

 Spark Loadを使用する際に、`spark-resource-path`の設定項目がパッケージ化されたzipファイルを指すようにされていない場合のエラーです。

- `Error: yarn client does not exist in path: xxx/yarn-client/hadoop/bin/yarn`

 Spark Loadを使用する際に、`yarn-client-path`の設定項目がyarnの実行可能ファイルを指定していない場合のエラーです。

- `ERROR: Cannot execute hadoop-yarn/bin/... /libexec/yarn-config.sh`

 CDHでHadoopを使用する場合、`HADOOP_LIBEXEC_DIR`環境変数を設定する必要があります。
 `hadoop-yarn`とhadoopディレクトリが異なるため、デフォルトの`libexec`ディレクトリは`hadoop-yarn/bin/... /libexec`を探しますが、`libexec`はhadoopディレクトリにあります。
 ```yarn application status``コマンドでSparkタスクのステータスを取得する際にエラーが発生し、インポートジョブが失敗します。
