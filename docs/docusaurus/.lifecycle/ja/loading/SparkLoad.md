---
displayed_sidebar: "Japanese"
---

# Spark Loadを使用して大量データをロードする

このロードでは、外部のApache Spark™リソースを使用してインポートされたデータの事前処理を行い、インポートのパフォーマンスを向上させ、コンピュートリソースを節約します。これは主にStarRocks（データ量がTBレベルまでの）への**初回の移行**および**大規模データのインポート**に使用されます。

Spark Loadは、**非同期**のインポートメソッドであり、ユーザーはMySQLプロトコルを介してSparkタイプのインポートジョブを作成し、`SHOW LOAD`を使用してインポート結果を表示する必要があります。

> **注意**
>
> - StarRocksテーブルにINSERT権限を持つユーザーのみが、このテーブルにデータをロードできます。必要な権限を付与するには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で提供されている手順に従うことができます。
> - Spark Loadは、プライマリキーのテーブルにデータをロードするために使用することはできません。

## 用語の説明

- **Spark ETL**: インポートプロセスの中でのデータのETLに主に責任を持ち、グローバルディクショナリの構築（BITMAPタイプ）、パーティション分割、ソート、集約などを含みます。
- **Broker**: ブローカーは独立した状態を持たないプロセスです。ファイルシステムインターフェースをカプセル化し、StarRocksにリモートストレージシステムのファイルを読み込む機能を提供します。
- **グローバルディクショナリ**: オリジナルの値からエンコードされた値へのデータをマッピングするデータ構造を保存します。オリジナルの値は任意のデータ型である一方、エンコードされた値は整数です。グローバルディクショナリは、厳密なカウントの予め計算されたシナリオで主に使用されます。

## 背景情報

StarRocks v2.4以前では、Spark LoadはBrokerプロセスに依存してStarRocksクラスターとストレージシステムとの接続を設定していました。Spark Loadジョブを作成する際に、`WITH BROKER "<broker_name>"`を入力して使用するブローカーを指定する必要がありました。ブローカーは独立した状態を持たないプロセスで、ファイルシステムインターフェースと統合されています。ブローカープロセスを使用することで、StarRocksはストレージシステムに保存されているデータファイルをアクセスして読み取り、自身のコンピュートリソースを使用してこれらのデータファイルの事前処理およびデータのロードを行うことができます。

StarRocks v2.5以降、Spark LoadはもはやBrokerプロセスに依存する必要がありません。Spark Loadジョブを作成する際に、もはやブローカーを指定する必要はありませんが、`WITH BROKER`キーワードは引き続き保持する必要があります。

> **注意**
>
> ブローカープロセスを使用しないでロードすると、複数のHDFSクラスターまたは複数のKerberosユーザーを持っているといった特定の状況で動作しない場合があります。このような状況では、ブローカープロセスを使用してデータをロードすることができます。

## 基本

ユーザーはMySQLクライアントを介してSparkタイプのインポートジョブを提出します。FEはメタデータを記録し、提出の結果を返します。

spark loadタスクの実行は、次の主要なフェーズに分割されます。

1. ユーザーはSpark LoadジョブをFEに提出します。
2. FEはETLタスクの提出をApache Spark™クラスターにスケジュールして実行します。
3. Apache Spark™クラスターはグローバルディクショナリ構築（BITMAPタイプ）、パーティション分割、ソート、集約などを含むETLタスクを実行します。
4. ETLタスクが完了した後、FEは各事前処理スライスのデータパスを取得し、関連するBEにプッシュタスクの実行をスケジュールします。
5. BEは、ブローカープロセス経由でHDFSからデータを読み込み、StarRocksストレージ形式に変換します。
   > ブローカープロセスを使用しない場合は、BEはHDFSから直接データを読み込みます。
6. FEは有効なバージョンをスケジュールし、インポートジョブを完了します。

以下のダイアグラムは、spark loadの主要なフローを示しています。

![Spark load](../assets/4.3.2-1.png)

---

## グローバルディクショナリ

### 適用シナリオ

現在、StarRocksのBITMAP列はRoaringbitmapを使用して実装されており、入力データ型は整数のみです。そのため、インポートプロセスのBITMAP列の事前計算を実装する場合は、入力データ型を整数に変換する必要があります。

StarRocksの既存のインポートプロセスでは、グローバルディクショナリのデータ構造がHiveテーブルをベースに実装され、オリジナルの値からエンコードされた値へのマッピングが保存されます。

### ビルドプロセス

1. 上流データソースからデータを読み込み、`hive-table`という一時的なHiveテーブルを生成します。
2. `hive-table`の強調されたフィールドの値を抽出し、`distinct-value-table`という新しいHiveテーブルを生成します。
3. オリジナルの値用の1つの列とエンコードされた値用の列を持つ`dict-table`という新しいグローバルディクショナリテーブルを作成します。
4. `distinct-value-table`と`dict-table`の左結合を行い、次にウィンドウ関数を使用してこのセットをエンコードします。最終的に、重複のない列のオリジナル値とエンコードされた値の両方を`dict-table`に書き込みます。
5. `dict-table`と`hive-table`の結合を行い、`hive-table`のオリジナル値を整数値に置換する作業を終了します。
6. `hive-table`は次回のデータ事前処理で読み込まれ、計算後にStarRocksにインポートされます。

## データ事前処理

データ事前処理の基本的なプロセスは次のとおりです。

1. 上流データソース（HDFSファイルまたはHiveテーブル）からデータを読み込みます。
2. 読み込んだデータのフィールドマッピングと計算を完了し、パーティション情報に基づいて`bucket-id`を生成します。
3. StarRocksテーブルのロールアップメタデータに基づいて、RollupTreeを生成します。
4. RollupTreeを反復処理し、階層的な集計操作を実行します。次の階層のRollupは、前の階層のRollupから計算できます。
5. 集計計算が完了するたびに、データは`bucket-id`に基づいてバケット化され、その後HDFSに書き込まれます。
6. 次のブローカープロセスは、HDFSからファイルを取得してこれらをStarRocks BEノードにインポートします。

## 基本操作

### 前提条件

ブローカープロセスを介して引き続きデータをロードする場合は、StarRocksクラスターにブローカープロセスがデプロイされていることを確認する必要があります。

[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスターでデプロイされているブローカーを確認できます。ブローカーがデプロイされていない場合は、[Deploy Broker](../deployment/deploy_broker.md)で提供されている手順に従い、ブローカープロセスをデプロイする必要があります。

### ETLクラスターの構成

Apache Spark™をStarRocksでETL作業に使用する外部計算リソースとして設定します。他にもSpark/GPU for query、外部ストレージのHDFS/S3、ETLのMapReduceなど、StarRocksで使用されるこれらの外部リソースを管理するために、「リソース管理」を導入しています。

Apache Spark™のインポートジョブを提出する前に、ETLタスクを実行するためのApache Spark™クラスターを構成します。操作の構文は次のとおりです。

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
-- yarnクラスターモード
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

-- yarn HAクラスターモード
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

`resource-name`はStarRocksで構成されたApache Spark™リソースの名前です。

`PROPERTIES`にはApache Spark™リソースに関連するパラメータが含まれます。
> **注意**
>
> Apache Spark™のRESOURCE PROPERTIESの詳細な説明については、[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md)を参照してください。

- Spark関連のパラメータ:
  - `type` : リソースの種類、必須、現在は`spark`のみサポートされています。
  - `spark.master` : 必須、現在は`yarn`のみサポートされています。
    - `spark.submit.deployMode` : Apache Spark™プログラムのデプロイモード、必須、現在は`cluster`および`client`の両方がサポートされています。
    - `spark.hadoop.fs.defaultFS` : masterがyarnの場合は必須。
    - Yarnリソースマネージャに関連するパラメータ、必須。
      - 単一ノード上のResourceManager
        `spark.hadoop.yarn.resourcemanager.address` : 単一ポイントのリソースマネージャのアドレス。
      - ResourceManager HA
        > リソースマネージャのホスト名またはアドレスを指定できます。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled` : リソースマネージャHAを有効にし、`true`に設定します。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids` : リソースマネージャの論理IDのリスト。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id` : 各rm-idについて、リソースマネージャに対応するホスト名を指定します。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id` : 各rm-idについて、クライアントがジョブを提出するための`host:port`を指定します。

- `*working_dir` : ETLで使用されるディレクトリ。Apache Spark™をETLリソースとして使用する場合に必要です。例: `hdfs://host:port/tmp/starrocks`。

- Broker関連のパラメータ:
  - `broker` : Broker名。Apache Spark™をETLリソースとして使用する場合に必要です。事前に`ALTER SYSTEM ADD BROKER`コマンドを使用して構成を完了する必要があります。
  - `broker.property_key` : BrokerプロセスがETLによって生成された中間ファイルを読み込む際に指定する情報（例: 認証情報）。

**注意**:

上記はBrokerプロセスを介してデータをロードするパラメータの説明です。Brokerプロセスなしでデータをロードする場合は、以下に注意してください。

- `broker`を指定する必要はありません。
- ユーザ認証およびNamenodeノードのHAを構成する場合、HDFSクラスタのhdfs-site.xmlファイルでパラメータを構成する必要があります。パラメータの説明については、[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。そして、**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**および各BEの**$BE_HOME/conf**に移動する必要があります。

> **注意**
>
> HDFSファイルが特定のユーザーのみアクセスできる場合、`broker.name`でHDFSユーザー名を指定し、`broker.password`でユーザーパスワードを指定する必要があります。

- リソースの表示

通常のアカウントは`USAGE-PRIV`アクセス権を持つリソースのみを表示できます。rootおよび管理者アカウントはすべてのリソースを表示できます。

- リソース権限

リソース権限は`GRANT REVOKE`を介して管理され、現在は`USAGE-PRIV`権限のみがサポートされています。ユーザーまたはロールに`USAGE-PRIV`権限を付与できます。

~~~sql
-- spark0リソースへのuser0へのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- spark0リソースへのrole0へのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- すべてのリソースへのuser0へのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- すべてのリソースへのrole0へのアクセス権を付与
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- user0からspark0リソースの使用権を取り消す
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### Sparkクライアントの構成

FEのSparkクライアントを構成し、`spark-submit`コマンドを実行してSparkタスクを提出できるようにします。公式バージョンのSpark2 2.4.5以上を使用することをお勧めします [spark download address](https://archive.apache.org/dist/spark/)。ダウンロード後、以下の手順を実行して構成を完了してください。

- `SPARK-HOME`の構成

SparkクライアントをFEと同じマシン上のディレクトリに配置し、FE構成ファイルの`spark_home_default_dir`をこのディレクトリに設定します。これはデフォルトでFEルートディレクトリの`lib/spark2x`パスですが、空にできません。

- **SPARK依存パッケージの構成**

依存パッケージを構成するには、Sparkクライアントのjarsフォルダーに存在するすべてのjarファイルをzip化しアーカイブ化し、FE構成の`spark_resource_path`項目をこのzipファイルに設定します。この構成が空の場合、FEはデフォルトでFEルートディレクトリの`lib/spark2x/jars/spark-2x.zip`ファイルを見つけようとします。FEがそれを見つけられない場合、エラーが発生します。

spark loadジョブが提出されると、アーカイブされた依存ファイルはリモートリポジトリにアップロードされます。デフォルトのリポジトリパスは`working_dir/{cluster_id}`ディレクトリの下に`--spark-repository--{resource-name}`という名前であり、クラスタ内のリソースがリモートリポジトリに対応していることを意味します。ディレクトリ構造は以下を参照してください：

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

spark依存関係（デフォルトで`spark-2x.zip`と呼ばれる）に加えて、FEはDPP依存関係もリモートリポジトリにアップロードします。Spark loadによって提出されたすべての依存関係が既にリモートリポジトリに存在する場合、再度依存関係をアップロードする必要がなく、時間を節約できます。大量のファイルを繰り返しアップロードする手間が省けます。

### YARNクライアントの構成

FEのYARNクライアントを構成し、FEが実行中のアプリケーションのステータスを取得したり、アプリケーションを停止したりできるようにします。公式バージョンのHadoop2 2.5.2以上を使用することをお勧めします ([hadoop download address](https://archive.apache.org/dist/hadoop/common/))。ダウンロード後、以下の手順を実行して構成を完了してください：

- **YARN実行可能ファイルの構成**

FEと同じマシン上のディレクトリにダウンロードされたyarnクライアントを配置し、FE構成ファイルの`yarn_client_path`項目をyarnのバイナリ実行ファイルに設定します。これはデフォルトでFEルートディレクトリの`lib/yarn-client/hadoop/bin/yarn`パスです。

- **YARNを生成するために必要な構成ファイルへのパスの構成（オプション）**

FEは、デフォルトでStarRocksは`lib/yarn-config`パスの中に実行するために必要な構成ファイルを生成します。このパスはFEルートディレクトリの`yarn_config_dir`エントリで設定できます。現在では、`core-site.xml`および`yarn-site.xml`が含まれます。

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

**Example 1**: アップストリームのデータソースがHDFSの場合

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
```sql
外部リソース
(
    "timeout" = "3600"
);

**例2**：上流データソースがHiveである場合。

- ステップ1：新しいHiveリソースを作成する。

~~~sql
CREATE EXTERNAL RESOURCE hive0
外部リソース
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://0.0.0.0:8080"
);
 ~~~

- ステップ2：新しいHive外部テーブルを作成する。

~~~sql
CREATE EXTERNAL TABLE hive_t1
(
    k1 INT,
    K2 SMALLINT,
    k3 varchar(50),
    uuid varchar(100)
)
ENGINE=hive
外部リソース
( 
    "resource" = "hive0",
    "database" = "tmp",
    "table" = "t1"
);
 ~~~

- ステップ3：カラム名をスターロックステーブル内でImport時に使用し、Hive外部テーブルに存在することを要求してロードコマンドを送信する。

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
外部リソース
(
    "timeout" = "3600"
);
 ~~~

Spは今後より多くの統計情報を含む場合もあります。content-end

スターロックスへのSparkロード（import）のパラメータの紹介：

- **Label**
  
importジョブのラベル。各importジョブには、データベース内でユニークなLabelがあり、Broker Loadと同じルールに従います。

- **データの記述クラスパラメータ**
  
現在、サポートされているデータソースはCSVとHiveテーブルです。他のルールはBroker Loadと同じです。

- **Import Job Parameters**
  
importジョブパラメータはimportステートメントの`opt_properties`セクションに属するパラメータを指します。これらのパラメータはimportジョブ全体に適用されます。ルールはBroker Loadと同じです。

- **Spark Resource Parameters**
  
Sparkリソースは事前にStarRocksに設定する必要があり、ユーザーにはリソースをSparkロードに適用する前にUSAGE-PRIV権限が与えられる必要があります。
Sparkリソースパラメータは、一時的な必要がある場合に設定できます。例えば、ジョブにリソースを追加したり、Sparkの構成を変更したりする場合です。この設定はこのジョブにのみ有効であり、StarRocksクラスター内の既存の構成に影響を与えません。

~~~sql
外部リソース 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

- **データソースがHiveの場合のImport**
  
現在、importプロセスでHiveテーブルを使用するためには、Hiveタイプの外部テーブルを作成し、その名前をimportコマンドの送信時に指定する必要があります。

- **グローバルディクショナリを構築するImportプロセス**
  
loadコマンドでは、次のフォーマットでグローバルディクショナリを構築するために必要なフィールドを指定できます：`StarRocksフィールド名=bitmap_dict(Hiveテーブルフィールド名)` 現在のところ、**グローバルディクショナリは上流データソースがHiveテーブルの場合にのみサポートされています**。

- **ビットマップバイナリ型のカラムのImport**

StarRocksテーブルの集計カラムに適用可能なデータ型はビットマップ型であり、対応する列のデータソースHiveテーブルまたはHDFSファイルのタイプはバイナリ（spark-dpp内のFEクラスシリアル化のcom.starrocks.load.loadv2.dpp.BitmapValue経由）タイプです。

グローバルディクショナリを構築する必要はありません。単にloadコマンドで対応フィールドを指定します。フォーマットは次のとおりです：```StarRocksフィールド名=bitmap_from_binary(Hiveテーブルフィールド名)```

## Import Jobの表示

Sparkロードimportは非同期であり、Broker Loadと同様です。ユーザーはimportジョブのラベルを記録し、`SHOW LOAD`コマンドでimportの結果を表示する必要があります。importを表示するためのコマンドは、すべてのimport方法で共通です。以下に例を示します。

返されるパラメータの詳細な説明はBroker Loadを参照してください。以下は違いです。

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
  
importジョブの現在のステージ。
PENDING：ジョブはコミットされています。
ETL：Spark ETLがコミットされています。
LOADING：FEはBEをスケジュールしてプッシュ操作を実行します。
FINISHED：プッシュが完了し、バージョンが有効になりました。

importジョブには、`CANCELLED`と`FINISHED`の2つの最終ステージがあります。どちらもimportジョブの完了を示します。`CANCELLED`はimportが失敗したことを、`FINISHED`はimportが成功したことを示します。

- **Progress**
  
importジョブの進行状況の説明。ETLとLOADの2つの進行状況があり、importプロセスの2つのフェーズであるETLとLOADに対応しています。

LOADの進行の範囲は0〜100%です。
`LOADの進捗 = 現在完了したすべてのレプリケーションのインポートのタブレットの数 / このimportジョブの総タブレット数 * 100%`です。

すべてのテーブルがインポートされた場合、LOADの進捗は99%になり、importが最後のバリデーションフェーズに入ると100%に変わります。

importの進捗は直線的ではありません。進捗に変化がない場合、importが実行されていないという意味ではありません。

- **Type**

 importジョブの種類。spark loadに対してはSPARKです。

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

これらの値は、importが作成された時刻、ETLフェーズの開始時刻、ETLフェーズの完了時刻、LOADフェーズの開始時刻、およびimportジョブ全体の完了時刻を表します。

- **JobDetails**

importジョブの詳細な実行ステータスを表示し、インポートされたファイルの数、合計サイズ（バイト単位）、サブタスクの数、処理されている行の数などを含みます。例：

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **URL**

対応するアプリケーションのWebインタフェースにアクセスするために、ブラウザに入力をコピーできます。

### Apache Spark™ランチャーのコミットログの表示

Apache Spark™のジョブコミット時に生成された詳細なログを表示する必要がある場合、デフォルトでは、ログはFEルートディレクトリのパス`log/spark_launcher_log`に保存され、`spark-launcher-{load-job-id}-{label}.log`という名前で保存されます。ログは一定の期間このディレクトリに保存されますが、FEメタデータ内のimport情報がクリーンアップされると削除されます。デフォルトの保持期間は3日です。

### Importのキャンセル

Sparkロード（import）ジョブの状態が`CANCELLED`または`FINISHED`でない場合、ユーザーはimportジョブのラベルを指定して手動でキャンセルできます。

---

## 関連システム設定

**FE設定：** Sparkロードのシステムレベルの設定は、すべてのSparkロードimportジョブに適用されます。設定値は主に`fe.conf`を修正することで変更できます。

- enable-spark-load：デフォルト値がfalseのSparkロードとリソース作成を有効にします。
- spark-load-default-timeout-second：ジョブのデフォルトのタイムアウトは259200秒（3日）です。
- spark-home-default-dir：Sparkクライアントパス（`fe/lib/spark2x`）。
- spark-resource-path：パッケージ化されたSpark依存ファイルのパス（デフォルトは空）。
- spark-launcher-log-dir：Sparkクライアントのコミットログが保存されるディレクトリ（`fe/log/spark-launcher-log`）。
- yarn-client-path：yarnバイナリ実行ファイルのパス（`fe/lib/yarn-client/hadoop/bin/yarn`）。
- yarn-config-dir：Yarnの設定ファイルパス（`fe/lib/yarn-config`）。

---

## ベストプラクティス

Sparkロードを使用する最適なシナリオは、生データがファイルシステム（HDFS）にあり、データ量が数十GBからTBレベルの場合です。データ量が小さい場合はStream LoadまたはBroker Loadを使用してください。
```
完全なSparkロードのインポート例については、GitHubのデモを参照してください：[https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## よくある質問

- `エラー: 'yarn'マスターで実行する場合、環境にHADOOP-CONF-DIRまたはYARN-CONF-DIRを設定する必要があります。`

 Sparkクライアントの`spark-env.sh`で`HADOOP-CONF-DIR`環境変数を構成せずにSparkロードを使用しています。

- `エラー: プログラム "xxx/bin/spark-submit"を実行できません：error=2、そのようなファイルやディレクトリはありません。`

 Sparkロードを使用する際に、`spark_home_default_dir`構成項目がSparkクライアントのルートディレクトリを指定していません。

- `エラー: ファイルxxx/jars/spark-2x.zipが存在しません。`

 Sparkロードを使用する際に、`spark-resource-path`構成項目がパッケージ化されたzipファイルを指していません。

- `エラー: yarnクライアントがパスに存在しません：xxx/yarn-client/hadoop/bin/yarn`

 Sparkロードを使用する際に、yarn-executable-path構成項目がyarn実行可能ファイルを指定していません。

- `エラー: hadoop-yarn/bin/... /libexec/yarn-config.shを実行できません。`

 CDHでHadoopを使用する場合、`HADOOP_LIBEXEC_DIR`環境変数を構成する必要があります。
 `hadoop-yarn`とhadoopディレクトリが異なるため、デフォルトの`libexec`ディレクトリは`hadoop-yarn/bin/... /libexec`を探しますが、`libexec`はhadoopディレクトリにあります。
 ```yarn application status```コマンドは、Sparkタスクのステータスを取得してインポートジョブを失敗させるエラーを報告しました。