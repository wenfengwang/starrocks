---
displayed_sidebar: Chinese
---

# Spark Loadを使用したバッチデータのインポート

Spark Loadは外部のSparkリソースを利用してデータのプリプロセスを行い、StarRocksの大量データのインポート性能を向上させ、StarRocksクラスターの計算リソースを節約します。主に**初回の移行**や**大量データのインポート**のシナリオ（データ量はTBレベルまで）に使用されます。

この文書では、インポートタスクの操作プロセス（関連するクライアント設定、タスクの作成と確認など）、システム設定、ベストプラクティス、およびよくある質問について説明します。

> **注意**
>
> * Spark Load操作には対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。
> * Spark Loadはプライマリキーモデルのテーブルへのインポートをサポートしていません。

## 背景情報

StarRocks v2.4以前のバージョンでは、Spark LoadはBrokerプロセスを介して外部ストレージシステムにアクセスする必要がありました。ETLタスクを実行するSparkクラスターを設定する際には、Brokerグループを指定する必要がありました。Brokerは独立したステートレスプロセスで、ファイルシステムインターフェースをカプセル化しています。Brokerプロセスを介して、StarRocksは外部ストレージシステム上のデータファイルにアクセスして読み取ることができます。
StarRocks v2.5以降では、Spark LoadはBrokerプロセスを介さずに外部ストレージシステムにアクセスできるようになりました。
> **説明**
>
> Brokerプロセスを使用しないインポート方法は、一部のシナリオでは制限を受けます。複数のHDFSクラスターや複数のKerberosユーザーを設定している場合、Brokerプロセスを介さないインポート方法はまだサポートされていません。このような場合は、Brokerプロセスを介してインポートを続ける必要があります。

## 使用説明

Brokerプロセスを介してインポートを続ける場合は、StarRocksクラスターにBrokerがデプロイされていることを確認する必要があります。
[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用して、クラスターにデプロイされているBrokerを確認できます。クラスターにBrokerがデプロイされていない場合は、[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してBrokerのデプロイを完了してください。

## サポートされるデータ形式

* CSV
* ORC（バージョン2.0以降でサポート）
* PARQUET（バージョン2.0以降でサポート）

## 基本原理

ユーザーはMySQLクライアントを通じてSparkタイプのインポートタスクをFEに提出し、FEはメタデータを記録してユーザーに成功を返します。

Spark Loadタスクの実行は主に以下の段階に分かれます：

1. ユーザーがFEにSpark Loadタスクを提出する。
2. FEがETLタスクをSparkクラスターにスケジュールして実行する。
3. SparkクラスターがETLを実行し、インポートデータのプリプロセスを完了する。これには、グローバル辞書の構築（BITMAPタイプ）、パーティショニング、ソート、集約などが含まれます。プリプロセスされたデータはHDFSに保存されます。
4. ETLタスクが完了すると、FEはプリプロセスされた各シャードのデータパスを取得し、関連するBEにPushタスクをスケジュールする。
5. BEはBrokerプロセスを介してHDFSデータを読み取り、StarRocksのストレージ形式に変換する。
    > **説明**
    >
    > Brokerプロセスを使用しない場合は、BEが直接HDFSデータを読み取ります。
6. FEがバージョンを有効にしてインポートタスクを完了する。

以下の図は、Spark Loadの主要なプロセスを示しています：

![spark load](../assets/4.3.2-1.png)

---

## 基本操作

Spark Loadを使用してデータをインポートするには、`リソースの作成 -> Sparkクライアントの設定 -> YARNクライアントの設定 -> Spark Loadインポートタスクの作成`のプロセスに従って実行する必要があります。具体的な各部分については、以下の説明を参照してください。

### ETLクラスターの設定

Sparkは外部の計算リソースとしてStarRocksでETL作業を行うために使用されるため、StarRocksが使用する外部リソースを管理するためにリソースマネジメントを導入しました。

Sparkインポートタスクを提出する前に、ETLタスクを実行するSparkクラスターを設定する必要があります。操作構文：

#### リソースの作成

**例**：

~~~sql
-- yarn clusterモード
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
    "spark.hadoop.yarn.resourcemanager.address" = "resourcemanager_host:8032",
    "spark.hadoop.fs.defaultFS" = "hdfs://namenode_host:9000",
    "working_dir" = "hdfs://namenode_host:9000/tmp/starrocks",
    "broker" = "broker0",
    "broker.username" = "user0",
    "broker.password" = "password0"
);

-- yarn HA clusterモード
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
    "spark.hadoop.fs.defaultFS" = "hdfs://namenode_host:9000",
    "working_dir" = "hdfs://namenode_host:9000/tmp/starrocks",
    "broker" = "broker1"
);

-- HDFS HA clusterモード
CREATE EXTERNAL RESOURCE "spark2"
PROPERTIES
(
    "type" = "spark", 
    "spark.master" = "yarn",
    "spark.hadoop.yarn.resourcemanager.address" = "resourcemanager_host:8032",
    "spark.hadoop.fs.defaultFS" = "hdfs://myha",
    "spark.hadoop.dfs.nameservices" = "myha",
    "spark.hadoop.dfs.ha.namenodes.myha" = "mynamenode1,mynamenode2",
    "spark.hadoop.dfs.namenode.rpc-address.myha.mynamenode1" = "nn1_host:rpc_port",
    "spark.hadoop.dfs.namenode.rpc-address.myha.mynamenode2" = "nn2_host:rpc_port",
    "spark.hadoop.dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider",
    "working_dir" = "hdfs://myha/tmp/starrocks",
    "broker" = "broker2",
    "broker.dfs.nameservices" = "myha",
    "broker.dfs.ha.namenodes.myha" = "mynamenode1,mynamenode2",
    "broker.dfs.namenode.rpc-address.myha.mynamenode1" = "nn1_host:rpc_port",
    "broker.dfs.namenode.rpc-address.myha.mynamenode2" = "nn2_host:rpc_port",
    "broker.dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
~~~

`spark0`、`spark1`、`spark2`はStarRocksに設定されたSparkリソースの名前です。

PROPERTIESはSparkリソースに関連するパラメータで、重要なパラメータについて以下で説明します：
> Sparkリソースのすべてのパラメータと説明については、[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md#spark-リソース)を参照してください。

* Sparkクラスター関連のパラメータ
  * `type`：必須、リソースのタイプで、`spark`を指定します。
  * `spark.master`: 必須、Sparkのクラスターマネージャー。現在はYARNのみをサポートしているため、`yarn`を指定します。
  * `spark.submit.deployMode`: 必須、Sparkドライバーのデプロイモード。`cluster`または`client`を指定します。詳細は[Launching Spark on YARN](https://spark.apache.org/docs/3.3.0/running-on-yarn.html#launching-spark-on-yarn)を参照してください。
  * `spark.hadoop.fs.defaultFS`: 必須、HDFS内のNameNodeのアドレス。形式は`hdfs://namenode_host:port`です。
  * YARN ResourceManager関連のパラメータ
    * SparkがシングルノードのResourceManagerの場合は`spark.hadoop.yarn.resourcemanager.address`を設定し、ResourceManagerのアドレスを指定します。
    * SparkがResourceManager HAの場合は以下を設定します（hostnameとaddressはどちらか一方を設定）：
      * `spark.hadoop.yarn.resourcemanager.ha.enabled`: ResourceManagerのHAを有効にし、`true`を設定します。
      * `spark.hadoop.yarn.resourcemanager.ha.rm-ids`: ResourceManagerの論理IDリスト。

      * `spark.hadoop.yarn.resourcemanager.hostname.rm-id`: 各rm-idについて、ResourceManagerのホスト名を指定します。
      * `spark.hadoop.yarn.resourcemanager.address.rm-id`: 各rm-idについて、クライアントがジョブをサブミットするための`host:port`を指定します。
* Broker 関連パラメータ
  * `broker`: Broker グループの名前。`ALTER SYSTEM ADD BROKER` コマンドを使用して事前に設定が必要です。
  * `broker.property_key`: Broker がETLで生成された中間ファイルを読み取る際に必要な認証情報などを指定します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。
* その他のパラメータ
  * `working_dir`: 必須項目で、ETLジョブで生成されたファイルを保存するためのHDFSファイルパスです。例：`hdfs://host:port/tmp/starrocks`。

**注意**

上記はBrokerプロセスを介してインポートを実行する際のパラメータ説明です。Brokerプロセスを使用しないインポート方法を使用する場合は、以下の点に注意してください：

* `broker`を渡す必要はありません。
* ユーザー認証やNameNodeのHA設定が必要な場合は、HDFSクラスターの **hdfs-site.xml** にパラメータを設定する必要があります。具体的なパラメータと説明については、[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs) を参照してください。そして、**hdfs-site.xml** を各FEの **$FE_HOME/conf** および各BEの **$BE_HOME/conf** に配置してください。

    > **説明**
    >
    > HDFSファイルが特定のユーザーのみによってアクセス可能な場合は、HDFSユーザー名 `broker.name` とHDFSユーザーパスワード `broker.password` を引き続き渡す必要があります。

#### リソースの表示

~~~sql
show resources;
~~~

> **注意**
>
> 一般ユーザーは、USAGE権限を持つリソースのみを表示できます。`db_admin` ロールはグローバルリソースを表示できます。

リソース権限は GRANT と REVOKE で管理されます。関連する権限を特定のユーザーやロールに付与するには、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してください。

### Spark クライアントの設定

FEは内部的に `spark-submit` コマンドを実行してSparkタスクをサブミットするため、FEにSparkクライアントを設定する必要があります。Spark2の公式バージョン2.4.5以上の使用を推奨します。[Spark ダウンロードリンク](https://archive.apache.org/dist/spark/)からダウンロードした後、以下の手順で設定を完了してください：

* **SPARK-HOME 環境変数の設定**
  
  spark_home_default_dir: FEの設定で、Sparkクライアントのディレクトリを指します。この設定項目はデフォルトでFEのルートディレクトリ下の `lib/spark2x` パスになっており、この項目は空にできません。

* **SPARK 依存関係パッケージの設定**
  
    spark_resource_path: Sparkクライアントのjarパッケージ圧縮ファイル。デフォルトは `fe/lib/spark2x/jars/spark-2x.zip` ファイルで、Sparkクライアントのjarsフォルダ内の全てのjarパッケージをアーカイブして `spark-2x.zip` ファイル（デフォルトは `spark-2x.zip` という名前）にパッケージする必要があります。見つからない場合はファイルが存在しないというエラーが発生します。

Spark Loadタスクをサブミットするとき、アーカイブされた依存ファイルはリモートリポジトリにアップロードされます。デフォルトのリポジトリパスは `working_dir/{cluster_id}` ディレクトリ下にマウントされ、`--spark-repository--{resource-name}` という名前で、クラスタ内のリソースごとに1つのリモートリポジトリが対応しています。リモートリポジトリのディレクトリ構造は以下の通りです：

~~~text
---spark-repository--spark0/
   |---archive-1.0.0/
   |   |---lib-990325d2c0d1d5e45bf675e54e44fb16-spark-dpp-1.0.0-jar-with-dependencies.jar
   |   |---lib-7670c29daf535efe3c9b923f778f61fc-spark-2x.zip
   |---archive-1.1.0/
   |   |---lib-64d5696f99c379af2bee28c1c84271d5-spark-dpp-1.1.0-jar-with-dependencies.jar
   |   |---lib-1bbb74bb6b264a270bc7fca3e964160f-spark-2x.zip
   |---archive-1.2.0/
   |   |-...
~~~

Spark依存関係の他に、FEはDPPの依存パッケージもリモートリポジトリにアップロードします。今回のSpark Loadでサブミットされる全ての依存ファイルがリモートリポジトリに既に存在する場合は、依存ファイルを再アップロードする必要はありません。これにより、毎回大量のファイルを繰り返しアップロードする時間が節約されます。

### YARN クライアントの設定

FEは内部的にyarnコマンドを実行して実行中のアプリケーションの状態を取得し、アプリケーションを終了するため、FEにYARNクライアントを設定する必要があります。Hadoop2の公式バージョン2.5.2以上の使用を推奨します。[Hadoop ダウンロードリンク](https://archive.apache.org/dist/hadoop/common/)からダウンロードした後、以下の手順で設定を完了してください：

* **YARN 実行ファイルパスの設定**
  
yarn_client_path: FEの設定で、デフォルトはFEのルートディレクトリ下の `lib/yarn-client/hadoop/bin/yarn` パスです。

* **YARNの設定ファイル生成パスの設定（オプション）**
  
yarn_config_dir: FEの設定で、デフォルトはFEのルートディレクトリ下の `lib/yarn-config` パスで、yarnコマンドの実行に必要な設定ファイルを生成します。現在生成される設定ファイルには `core-site.xml` と `yarn-site.xml` が含まれます。

### インポートタスクの作成

**例 1**: 上流データソースがHDFSファイルの場合

~~~sql
LOAD LABEL db1.label1 #このラベルは記録しておくことをお勧めします。タスクのクエリに使用できます。
(
    DATA INFILE("hdfs://abc.com:8888/user/starRocks/test/ml/file1")
    INTO TABLE tbl1
    COLUMNS TERMINATED BY ","
    (tmp_c1,tmp_c2)
    SET
    (
        id=tmp_c2,
        name=tmp_c1
    ),
    DATA INFILE("hdfs://abc.com:8888/user/starRocks/test/ml/file2")
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

**例 2**: 上流データソースがHDFSのORCファイルの場合

~~~sql
LOAD LABEL db1.label2
(
    DATA INFILE("hdfs://abc.com:8888/user/starRocks/test/ml/file3")
    INTO TABLE tbl3
    COLUMNS TERMINATED BY ","
    FORMAT AS "orc"
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

**例 3**: 上流データソースがHiveテーブルの場合

* ステップ 1: Hiveリソースを新規作成します。

    ~~~sql
    CREATE EXTERNAL RESOURCE hive0
   PROPERTIES
    ( 
        "type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:8080"
    );
    ~~~

* ステップ 2: Hive外部テーブルを新規作成します。

    ~~~sql
    CREATE EXTERNAL TABLE hive_t1
    (
        k1 INT,
        k2 SMALLINT,
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

* ステップ 3: loadコマンドをサブミットし、StarRocksテーブルにインポートする列がHive外部テーブルに存在する必要があります。

    ~~~sql
    LOAD LABEL db1.label3
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

インポートの詳細な文法については [SPARK LOAD](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) を参照してください。ここでは、Spark Loadのインポート作成文法のパラメータの意味と注意点を主に説明します。

* **ラベル**
  
インポートタスクの識別子。各インポートタスクには、単一のデータベース内で一意のラベルがあります。具体的なルールはBroker Loadと同じです。

* **インポートジョブパラメータ**
  
インポートジョブパラメータは、主にSpark Loadがインポートステートメントを作成する際の`opt_properties`セクションに属するパラメータを指します。インポートジョブパラメータは、インポートジョブ全体に適用されます。ルールはBroker Loadと同じです。

* **Sparkリソースパラメータ**
  
Sparkリソースは、StarRocksシステムに事前に設定され、USAGE-PRIV権限がユーザーに付与された後にSpark Loadを使用できます。
ユーザーが一時的なニーズを持っている場合、例えばタスクのリソースを増やすためにSparkの設定を変更する場合は、ここで設定できます。設定はこのタスクにのみ有効であり、StarRocksクラスター内の既存の設定には影響しません。

~~~sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

* **データソースがHiveテーブルの場合のインポート**
  
現在、インポートプロセスでHiveテーブルをデータソースとして使用する場合は、まずHiveタイプの外部テーブルを新規作成し、その後インポートコマンドを送信する際に外部テーブルのテーブル名を指定する必要があります。

* **インポートプロセスでグローバル辞書を構築する**
  
StarRocksの集約列のデータタイプがbitmapタイプの場合に適用されます。ロードコマンドでグローバル辞書を構築するフィールドを指定するだけで、形式は次のとおりです：`StarRocksのフィールド名=bitmap_dict(Hiveテーブルのフィールド名)` 現在、**データソースがHiveテーブルの場合にのみ**グローバル辞書の構築をサポートしていることに注意してください。

* **binaryタイプのデータをインポートする**

バージョン2.5.17から、Spark Loadはインポート時にbitmap_from_binary関数を使用して、binaryタイプをbitmapタイプに変換することをサポートしています。HiveテーブルまたはHDFSファイルの列のデータタイプがbinaryタイプであり、StarRocksテーブルの対応する列がbitmapタイプの集約列である場合、グローバル辞書を構築する必要はありません。インポートコマンドで対応するフィールドを指定するだけです。形式は次のとおりです：`StarRocksのフィールド名=bitmap_from_binary(Hiveテーブルのフィールド名)`。

### インポートタスクを確認する

Spark LoadによるインポートはBroker Loadと同様に非同期であり、ユーザーはインポートを作成したラベルを記録しておき、`SHOW LOAD`コマンドでラベルを使用してインポート結果を確認する必要があります。インポートを確認するコマンドは、すべてのインポート方法で共通です。具体的な構文は[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を参照してください。以下に例を示します：

~~~sql
mysql > show load where label="label1"\G
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
    JobDetails: {"ScannedRows":28133395, "TaskNumber":1, "FileNumber":1,"FileSize":200000}
~~~

返された結果セットのパラメータの意味は、[インポートステータスの確認](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md#返された結果の説明)を参照してください。

### インポートをキャンセルする

Spark LoadジョブのステータスがCANCELLEDまたはFINISHEDでない場合、ユーザーによって手動でキャンセルすることができます。キャンセルする際には、キャンセルするインポートタスクのラベルを指定する必要があります。キャンセルコマンドの構文は[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)を参照してください。以下に例を示します：

~~~sql
CANCEL LOAD FROM db1 WHERE LABEL = "label1";
~~~

---

### Spark Launcherの送信ログを確認する

時には、ユーザーがSparkタスクの送信プロセスで生成された詳細なログを確認する必要があります。ログはデフォルトでFEのルートディレクトリの`log/spark_launcher_log`に保存され、`spark-launcher-{load-job-id}-{label}.log`という名前で命名されます。ログはこのディレクトリに一定期間保存され、FEのメタデータにインポート情報がクリアされると、関連するログもクリアされます。デフォルトの保存期間は3日間です。

## 関連するシステム設定

**FE設定:** 以下の設定はSpark Loadのシステムレベルの設定であり、すべてのSpark Loadインポートタスクに適用される設定です。主にfe.confを編集して設定値を調整します。

* `spark_load_default_timeout_second`：タスクのデフォルトタイムアウトは86400秒（1日）です。
* `spark_home_default_dir`：Sparkクライアントのパス(fe/lib/spark2x)。
* `spark_resource_path`：パッケージ化されたSpark依存ファイルのパス（デフォルトは空）。
* `spark_launcher_log_dir`：Sparkクライアントの送信ログが保存されるディレクトリ(fe/log/spark_launcher_log)。
* `yarn_client_path`：yarnバイナリ実行ファイルのパス(fe/lib/yarn-client/hadoop/bin/yarn)。
* `yarn_config_dir`：yarn設定ファイルの生成パス(fe/lib/yarn-config)。

---

## ベストプラクティス

### グローバル辞書

#### 適用シナリオ

現在、StarRocksのBITMAP列はRoaringbitmapライブラリを使用して実装されており、Roaringbitmapの入力データタイプは整数型のみです。したがって、インポートプロセスでBITMAP列の事前計算を実現するには、入力データのタイプを整数型に変換する必要があります。

StarRocksの既存のインポートプロセスでは、グローバル辞書のデータ構造はHiveテーブルに基づいて実装されており、元の値からエンコードされた値へのマッピングが保存されています。

#### 構築プロセス

**1.** 上流のデータソースからデータを読み取り、Hiveの一時テーブルを生成します。これをhive-tableと呼びます。

**2.** hive-tableから重複排除が必要なフィールドの重複排除値を抽出し、新しいHiveテーブルを生成します。これをdistinct-value-tableと呼びます。

**3.** グローバル辞書テーブルを新規作成し、これをdict-tableと呼びます。1列は元の値、もう1列はエンコードされた値です。

**4.** distinct-value-tableとdict-tableをleft joinし、新たに追加された重複排除値のセットを計算し、その後このセットにウィンドウ関数を使用してエンコードを行います。この時点で、重複排除列の元の値にはエンコードされた値の列が追加され、最後にこれら2列のデータをdict-tableに書き戻します。

**5.** dict-tableとhive-tableをjoinし、hive-table内の元の値を整数型のエンコード値に置き換える作業を完了します。

**6.** hive-tableは次のデータ前処理ステップで読み取られ、計算後にStarRocksにインポートされます。

### Sparkプログラムによるインポート

完全なSpark Loadインポートの例は、GitHubのデモを参照してください：[sparkLoad2StarRocks](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)。

---

## よくある質問

* Q：エラーが発生しました。マスター 'yarn' で実行する場合は、環境に HADOOP_CONF_DIR または YARN_CONF_DIR を設定する必要があります。

  A：Spark Loadを使用する際に、Sparkクライアントのspark-env.shにHADOOP_CONF_DIR環境変数が設定されていない。

* Q：spark-submitコマンドを使用してSparkジョブを送信する際にエラーが発生しました：プログラム "xxx/bin/spark-submit" を実行できません：エラー = 2、そのようなファイルやディレクトリはありません

  A：Spark Loadを使用する際に、`spark_home_default_dir`設定項目が指定されていないか、間違ったSparkクライアントのルートディレクトリが指定されています。

* Q：エラーが発生しました。ファイルxxx/jars/spark-2x.zipが存在しません。

  A：Spark Loadを使用する際に、spark_resource_path設定項目がパッケージ化されたzipファイルを指していないか、ファイルパスとファイル名が一致していないことを確認してください。

* Q：エラーが発生しました yarn client がパス xxx/yarn-client/hadoop/bin/yarn に存在しません

  A：Spark Load を使用する際に、yarn-client-path 設定項目で yarn の実行ファイルが指定されていません。

* Q：エラーが発生しました hadoop-yarn/bin/../libexec/yarn-config.sh を実行できません

  A：CDH の Hadoop を使用する際には、HADOOP_LIBEXEC_DIR 環境変数を設定する必要があります。hadoop-yarn と hadoop のディレクトリが異なるため、デフォルトの libexec ディレクトリは hadoop-yarn/bin/../libexec を探しますが、libexec は hadoop ディレクトリに存在します。そのため、yarn application status コマンドで Spark タスクの状態を取得する際にエラーが発生し、インポートジョブが失敗します。
