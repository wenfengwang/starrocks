---
displayed_sidebar: Chinese
---

# MySQLからのリアルタイム同期

この記事では、MySQLのデータをリアルタイム（秒単位）でStarRocksに同期し、企業がリアルタイムで大量データの分析と処理を行うニーズをサポートする方法について説明します。

> **注意**
>
> インポート操作には、対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## 基本原理

![MySQL 同期](../assets/4.9.2.png)

MySQLからStarRocksへのリアルタイム同期は、同期するデータベースとテーブルの構造、データの2つのフェーズに分けて行います。まず、StarRocks Migration Tool（データ移行ツール、以下SMTと略）がMySQLのデータベースとテーブルの構造を読み取り、StarRocksのデータベースとテーブルを作成するSQL文に変換します。次に、FlinkクラスターがFlinkジョブを実行し、MySQLの全量および増分データをStarRocksに同期します。具体的な同期プロセスは以下の通りです：

> 説明：
> MySQLからStarRocksへのリアルタイム同期は、エンドツーエンドでexactly-onceのセマンティック一貫性を保証できます。

1. **データベースとテーブルの構造の同期**

   SMTは、設定ファイルに記載されたソースMySQLとターゲットStarRocksの情報に基づいて、同期対象のMySQLデータベースとテーブルの構造を読み取り、StarRocks内で対応するデータベースとテーブルを作成するためのSQLファイルを生成します。

2. **データの同期**

   Flink SQLクライアントがデータインポートのSQL文（`INSERT INTO SELECT`文）を実行し、Flinkクラスターに1つまたは複数の長時間実行されるFlinkジョブを提出します。FlinkクラスターはFlinkジョブを実行し、[Flink CDCコネクタ](https://ververica.github.io/flink-cdc-connectors/master/content/快速上手/build-real-time-data-lake-tutorial-zh.html)がまずデータベースの履歴全量データを読み取り、次に増分データの読み取りにシームレスに切り替え、flink-starrocks-connectorに送信し、最終的にflink-starrocks-connectorがマイクロバッチデータをStarRocksに同期します。

   > **注意**
   >
   > DMLの同期のみをサポートし、DDLの同期はサポートしていません。

## ビジネスシナリオ

商品の累計販売量のリアルタイムランキングを例にとると、MySQLに保存された元の注文テーブルをFlinkで処理し、製品の販売量のリアルタイムランキングを計算し、StarRocksのプライマリキーモデルテーブルにリアルタイムで同期します。最終的に、ユーザーは可視化ツールを介してStarRocksに接続し、リアルタイムで更新されるランキングを表示できます。

## 準備作業

### 同期ツールのダウンロードとインストール

同期にはSMT、Flink、Flink CDCコネクタ、flink-starrocks-connectorが必要で、ダウンロードとインストールの手順は以下の通りです：

1. **Flinkクラスターのダウンロード、インストール、および起動**。
   > 説明：ダウンロードとインストール方法は、[Flink公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)を参照することもできます。

   1. Flinkを正常に実行するために、あらかじめオペレーティングシステムにJava 8またはJava 11をインストールする必要があります。以下のコマンドでインストールされているJavaのバージョンを確認できます。

      ```Bash
      # Javaのバージョンを確認
      java -version
      
      # 以下の表示があればJava 8がインストールされています
      openjdk version "1.8.0_322"
      OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
      OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
      ```

   2. [Flink](https://flink.apache.org/downloads.html)をダウンロードして解凍します。この例ではFlink 1.14.5を使用しています。
      > 説明：1.14以上のバージョンを推奨し、最低でも1.11バージョンをサポートしています。

      ```Bash
      # Flinkをダウンロード
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Flinkを解凍
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Flinkディレクトリに移動
      cd flink-1.14.5
      ```

   3. Flinkクラスターを起動します。

      ```Bash
      # Flinkクラスターを起動
      ./bin/start-cluster.sh
      
      # 以下の情報が返された場合、Flinkクラスターの起動に成功しています
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
      ```

2. **[Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/releases)をダウンロード**します。この例のデータソースはMySQLなので、flink-sql-connector-mysql-cdc-x.x.x.jarをダウンロードします。また、対応するFlinkバージョンをサポートする必要があります。バージョンのサポートについては、[Supported Flink Versions](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)を参照してください。この記事ではFlink 1.14.5を使用しているため、flink-sql-connector-mysql-cdc-2.2.0.jarを使用できます。

      ```Bash
      wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.2.0/flink-sql-connector-mysql-cdc-2.2.0.jar
      ```

3. **[flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)をダウンロード**します。そのバージョンはFlinkのバージョンに対応する必要があります。

      > flink-connector-starrocksのJARファイル（**x.x.x_flink-y.yy_z.zz.jar**）には3つのバージョン番号が含まれています：
      >
      > - 最初のバージョン番号x.x.xはflink-connector-starrocksのバージョン番号です。
      >
      > - 2番目のバージョン番号y.yyはサポートされているFlinkのバージョン番号です。
      >
      > - 3番目のバージョン番号z.zzはFlinkがサポートするScalaのバージョン番号です。Flinkが1.14.x以前のバージョンの場合は、Scalaのバージョン番号が付いたflink-connector-starrocksをダウンロードする必要があります。

   この記事ではFlinkバージョン1.14.5、Scalaバージョン2.11を使用しているため、flink-connector-starrocks JARファイル**1.2.3_flink-1.14_2.11.jar**をダウンロードできます。

4. Flink CDCコネクタとflink-connector-starrocksのJARファイル**flink-sql-connector-mysql-cdc-2.2.0.jar**と**1.2.3_flink-1.14_2.11.jar**をFlinkの**lib**ディレクトリに移動します。

   > **注意**
   >
   > Flinkが既に実行中の場合は、Flinkを一度停止してから再起動し、JARファイルを読み込んで有効にする必要があります。
   >
   > ```Bash
   > ./bin/stop-cluster.sh
   > ./bin/start-cluster.sh
   > ```

5. [SMT](https://www.mirrorship.cn/zh-CN/download/community)をダウンロードして解凍し、**flink-1.14.5**ディレクトリに配置します。オペレーティングシステムとCPUアーキテクチャに応じて、対応するSMTインストールパッケージを選択できます。

   ```Bash
   # Linux x86用
   wget https://releases.starrocks.io/resources/smt.tar.gz
   # macOS ARM64用
   wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
  
   ```

### MySQL Binlogログの有効化

リアルタイム同期にはMySQL Binlogログのデータを読み取り、解析してStarRocksに同期する必要があるため、MySQL Binlogログが有効になっていることを確認する必要があります。

1. MySQLの設定ファイル**my.cnf**（デフォルトのパスは**/etc/my.cnf**）を編集して、MySQL Binlogを有効にします。

   ```Bash
   # Binlogログを有効にする
   log_bin = ON
   # Binlogの保存場所を設定する
   log_bin =/var/lib/mysql/mysql-bin
   # server_idを設定する
   # MySQL 5.7.3以降のバージョンでは、server_idがないとbinlogを設定してもMySQLサービスを起動できません
   server_id = 1
   # BinlogモードをROWに設定する
   binlog_format = ROW
   # binlogログの基本ファイル名で、各Binlogファイルを識別するために後ろに識別子が追加されます
   log_bin_basename =/var/lib/mysql/mysql-bin
   # binlogファイルのインデックスファイルで、すべてのBinlogファイルのディレクトリを管理します
   log_bin_index =/var/lib/mysql/mysql-bin.index
   ```

2. 以下のコマンドを実行してMySQLを再起動し、変更後の設定ファイルを有効にします：

   ```Bash
    # serviceを使用して起動する
    service mysqld restart
    # mysqldスクリプトを使用して起動する
    /etc/init.d/mysqld restart
   ```

3. MySQLに接続し、以下のステートメントを実行してBinlogが有効になっているかどうかを確認します：

   ```Plain
   -- MySQLに接続する
   mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

   -- MySQL Binlogが有効になっているかどうかを確認する、`ON`と表示されれば有効です

   mysql> SHOW VARIABLES LIKE 'log_bin'; 
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | log_bin       | ON    |
   +---------------+-------+
   1 row in set (0.00 sec)
   ```

## 同期ライブラリとテーブル構造

1. SMTの設定ファイルを設定します。
   SMTの**conf**ディレクトリに入り、設定ファイル**config_prod.conf**を編集します。例えば、ソースMySQLの接続情報、同期するライブラリとテーブルのマッチングルール、flink-starrocks-connectorの設定情報などです。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocksのバックエンド数
    be_num = 3
    # `decimal_v3`はStarRocks-1.18.1からサポートされています
    use_decimal_v3 = true
    # 変換されたDDL SQLを保存するファイル
    output_dir = ./result

    [table-rule.1]
    # プロパティを設定するためのデータベースのマッチングパターン
    database = ^demo.*$
    # プロパティを設定するためのテーブルのマッチングパターン
    table = ^.*$

    ############################################
    ### flink sinkの設定
    ### `connector`、`table-name`、`database-name`は設定しないでください。これらは自動生成されます。
    ############################################
    flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
    flink.starrocks.load-url= <fe_host>:<fe_http_port>
    flink.starrocks.username=user2
    flink.starrocks.password=xxxxxx
    flink.starrocks.sink.properties.format=csv
    flink.starrocks.sink.properties.column_separator=\x01
    flink.starrocks.sink.properties.row_delimiter=\x02
    flink.starrocks.sink.buffer-flush.interval-ms=15000
    ```

    - `[db]`：MySQLの接続情報。
        - `host`：MySQLがあるサーバーのIPアドレス。
        - `port`：MySQLのポート番号、デフォルトは`3306`です。
        - `user`：ユーザー名。
        - `password`：ユーザーログインパスワード。

    - `[table-rule]`：ライブラリとテーブルのマッチングルール、および対応するflink-connector-starrocksの設定。

        > - 異なるテーブルに異なるflink-connector-starrocksの設定をマッチさせる必要がある場合、例えば一部のテーブルが頻繁に更新され、インポート速度を向上させる必要がある場合は、[補足説明](./Flink_cdc_load.md#常见问题)を参照してください。
        > - MySQLのシャーディング後の複数のテーブルをStarRocksの1つのテーブルにインポートする必要がある場合は、[補足説明](./Flink_cdc_load.md#常见问题)を参照してください。

        - `database`、`table`：MySQL内の同期対象のライブラリとテーブル名、正規表現がサポートされています。

        - `flink.starrocks.*`：flink-connector-starrocksの設定情報、詳細な設定と説明については、[Flink-connector-starrocks](./Flink-connector-starrocks.md#参数说明)を参照してください。

    - `[other]`：その他の情報
        - `be_num`：StarRocksクラスターのBEノード数（後で生成されるStarRocksのテーブル作成SQLファイルはこのパラメータを参照し、適切なバケット数を設定します）。
        - `use_decimal_v3`：[decimalV3](../sql-reference/sql-statements/data-types/DECIMAL.md)を有効にするかどうか。有効にすると、MySQLの小数点データがStarRocksに同期される際にdecimalV3に変換されます。
        - `output_dir`：生成されるSQLファイルのパス。SQLファイルはStarRocksクラスターでライブラリとテーブルを作成し、FlinkクラスターにFlinkジョブを提出するために使用されます。デフォルトは`./result`で、変更はお勧めしません。

2. 以下のコマンドを実行すると、SMTはMySQL内の同期対象のライブラリとテーブル構造を読み取り、設定ファイルの情報と組み合わせて、**result**ディレクトリにSQLファイルを生成します。これはStarRocksクラスターでライブラリとテーブルを作成するためのものです（**starrocks-create.all.sql**）、Flinkクラスターに同期データのFlinkジョブを提出するためのものです（**flink-create.all.sql**）。
   また、ソーステーブルが異なる場合、**starrocks-create.all.sql**内のテーブル作成ステートメントがデフォルトで作成するデータモデルも異なります。

   - ソーステーブルにPrimary Key、Unique Keyがない場合は、デフォルトで詳細モデルを作成します。
   - ソーステーブルにPrimary Key、Unique Keyがある場合は、以下の状況に応じて区別されます：
      - ソーステーブルがHiveテーブル、ClickHouse MergeTreeテーブルの場合は、デフォルトで詳細モデルを作成します。
      - ソーステーブルがClickHouse SummingMergeTreeテーブルの場合は、デフォルトで集約モデルを作成します。
      - ソーステーブルが他のタイプの場合は、デフォルトでプライマリキーモデルを作成します。

    ```Bash
    # SMTを実行
    ./starrocks-migrate-tool

    # resultディレクトリに入り、ファイルを確認
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 以下のコマンドを実行してStarRocksに接続し、SQLファイル**starrocks-create.all.sql**を実行して、目的のライブラリとテーブルを作成します。SQLファイルにデフォルトで含まれているテーブル作成ステートメントの使用を推奨します。この例では、デフォルトで作成されるデータモデルは[プライマリキーモデル](../table_design/table_types/primary_key_table.md)です。

    > **注意**
    >
    > - ご自身のビジネスニーズに応じて、SQLファイル内のテーブル作成ステートメントを変更し、他のモデルに基づいて目的のテーブルを作成することもできます。
    > - 非プライマリキーモデルに基づいて目的のテーブルを作成する場合、StarRocksはソーステーブルのDELETE操作を非プライマリキーモデルのテーブルに同期することはサポートしていませんので、ご注意ください。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    データをFlinkで処理した後に目的のテーブルに書き込む必要がある場合、目的のテーブルとソーステーブルの構造が異なるため、SQLファイル**starrocks-create.all.sql**内のテーブル作成ステートメントを変更する必要があります。この例では、目的のテーブルは商品ID（product_id）、商品名（product_name）のみを保持し、商品の売上をリアルタイムでランキングするため、以下のテーブル作成ステートメントを使用できます。

    ```Bash
    CREATE DATABASE IF NOT EXISTS `demo`;
    
    CREATE TABLE IF NOT EXISTS `demo`.`orders` (
    `product_id` INT(11) NOT NULL COMMENT "",
    `product_name` STRING NOT NULL COMMENT "",
    `sales_cnt` BIGINT NOT NULL COMMENT ""
    ) ENGINE=olap
    PRIMARY KEY(`product_id`)
    DISTRIBUTED BY HASH(`product_id`)
    PROPERTIES (
    "replication_num" = "3"
    );
    ```

   > **注意**
   >
   > バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

## データ同期

Flinkクラスターを実行し、Flinkジョブを提出して、ストリーミングジョブを開始し、MySQLデータベース内の全量および増分データをStarRocksに継続的に同期します。

1. Flinkディレクトリに入り、以下のコマンドを実行して、Flink SQLクライアントでSQLファイル**flink-create.all.sql**を実行します。

    このSQLファイルは、動的テーブルsource table、sink table、クエリステートメントINSERT INTO SELECTを定義し、connector、ソースデータベース、およびターゲットデータベースを指定します。Flink SQLクライアントがこのSQLファイルを実行すると、FlinkジョブがFlinkクラスターに提出され、同期タスクが開始されます。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    > **注意**
    >
    > - Flinkクラスターが既に起動していることを確認してください。`flink/bin/start-cluster.sh`コマンドで起動できます。
    >
    > - Flink 1.13以前のバージョンを使用している場合、SQLファイル**flink-create.all.sql**を直接実行することはできないかもしれません。SQLクライアントのコマンドラインインターフェースで、SQLファイル**flink-create.all.sql**内のSQLステートメントを逐次実行し、`\`文字に対してエスケープ処理を行う必要があります。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

   **同期データの処理**

   同期プロセス中にデータを処理する必要がある場合、たとえばGROUP BY、JOINなど、SQLファイル**flink-create.all.sql**を変更することができます。この例では、count(*)とGROUP BYを実行して、製品の売上実績をリアルタイムでランキングすることができます。

   ```Bash
   $ ./bin/sql-client.sh -f flink-create.all.sql
   No default environment specified.
   Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
   [INFO] Executing SQL from file.

   Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
   [INFO] Execute statement succeed.

   -- MySQLの注文テーブルに基づいて動的テーブルsource tableを作成
   Flink SQL> 
   CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_src` (
     `order_id` BIGINT NOT NULL,
     `product_id` INT NULL,
     `order_date` TIMESTAMP NOT NULL,
     `customer_name` STRING NOT NULL,
     `product_name` STRING NOT NULL,
     `price` DECIMAL(10, 5) NULL,
     PRIMARY KEY(`order_id`)
    NOT ENFORCED
   ) with (
     'connector' = 'mysql-cdc',
     'hostname' = 'xxx.xx.xxx.xxx',
     'port' = '3306',
     'username' = 'root',
     'password' = '',
     'database-name' = 'demo',
     'table-name' = 'orders'
   );
   [INFO] Execute statement succeed.

   -- sink table の作成
   Flink SQL> 
   CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_sink` (
    `product_id` INT NOT NULL,
    `product_name` STRING NOT NULL,
    `sales_cnt` BIGINT NOT NULL,
    PRIMARY KEY(`product_id`)
   NOT ENFORCED
   ) with (
     'sink.max-retries' = '10',
     'jdbc-url' = 'jdbc:mysql://<fe_host>:<fe_query_port>',
     'password' = '',
     'sink.properties.strip_outer_array' = 'true',
     'sink.properties.format' = 'json',
     'load-url' = '<fe_host>:<fe_http_port>',
     'username' = 'root',
     'sink.buffer-flush.interval-ms' = '15000',
     'connector' = 'starrocks',
     'database-name' = 'demo',
     'table-name' = 'orders'
   );
   [INFO] Execute statement succeed.

   -- クエリの実行により、リアルタイムで製品ランキングを実現し、source table の変更を sink table に反映させる
   Flink SQL> 
   INSERT INTO `default_catalog`.`demo`.`orders_sink` SELECT product_id, product_name, COUNT(*) AS cnt FROM `default_catalog`.`demo`.`orders_src` GROUP BY product_id, product_name;
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

   特定のデータのみを同期する必要がある場合、例えば2021年01月01日以降の支払いデータのみを同期する場合は、INSERT INTO SELECT ステートメントで WHERE 句を使用して order_date > '2021-01-01' というフィルタ条件を設定できます。この条件を満たさないデータ、つまり2021年01月01日以前の支払いデータは StarRocks に同期されません。

   ```sql
   INSERT INTO `default_catalog`.`demo`.`orders_sink` SELECT product_id, product_name, COUNT(*) AS cnt FROM `default_catalog`.`demo`.`orders_src` WHERE order_date > '2021-01-01 00:00:01' GROUP BY product_id, product_name;
   ```

   以下の結果が返された場合、Flink job が送信され、全量および増分データの同期が開始されたことを意味します。

   ```SQL
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

2. [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) または Flink コマンドラインで `bin/flink list -running` コマンドを実行して、Flink クラスターで実行中の Flink job および Flink job ID を確認できます。
      1. Flink WebUI のインターフェース
         ![タスクトポロジ](../assets/4.9.3.png)

      2. Flink コマンドラインで `bin/flink list -running` コマンドを実行

         ```Bash
         $ bin/flink list -running
         Waiting for response...
         ------------------ Running/Restarting Jobs -------------------
         13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
         --------------------------------------------------------------
         ```

           > 説明
           >
           > タスクに異常が発生した場合は、Flink WebUI または **flink-1.14.5/log** ディレクトリのログファイルで調査できます。

## よくある質問

### **異なるテーブルに対して異なる flink-connector-starrocks の設定をする方法**

例えば、データソースの一部のテーブルが頻繁に更新され、flink connector sr のインポート速度を向上させる必要がある場合は、SMT 設定ファイル **config_prod.conf** でこれらのテーブルに対して個別の flink-connector-starrocks 設定を行う必要があります。

```Bash
[table-rule.1]
# データベースに適用するプロパティをマッチさせるパターン
database = ^order.*$
# テーブルに適用するプロパティをマッチさせるパターン
table = ^.*$

############################################
### flink sink 設定
### `connector`, `table-name`, `database-name` は自動生成されるため設定しないでください
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000

[table-rule.2]
# データベースに適用するプロパティをマッチさせるパターン
database = ^order2.*$
# テーブルに適用するプロパティをマッチさせるパターン
table = ^.*$

############################################
### flink sink 設定
### `connector`, `table-name`, `database-name` は自動生成されるため設定しないでください
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=10000
```

### **MySQL の分割されたデータベースとテーブルを StarRocks の単一テーブルに同期する**

データソースの MySQL がデータベースとテーブルを分割し、複数のテーブルにデータを分散させ、すべてのテーブルの構造が同じである場合、`[table-rule]` を設定してこれらのテーブルを StarRocks の単一テーブルに同期できます。例えば、MySQL に edu_db_1、edu_db_2 の2つのデータベースがあり、それぞれに course_1、course_2 の2つのテーブルがあり、すべてのテーブルの構造が同じである場合、以下のように `[table-rule]` を設定することで StarRocks の単一テーブルに同期できます。

> **説明**
>
> 複数のデータソーステーブルを StarRocks の単一テーブルに同期する場合、テーブル名はデフォルトで course__auto_shard になります。変更する必要がある場合は、**result** ディレクトリの SQL ファイル **starrocks-create.all.sql、flink-create.all.sql** で変更できます。

```Bash
[table-rule.1]
# データベースに適用するプロパティをマッチさせるパターン
database = ^edu_db_[0-9]*$
# テーブルに適用するプロパティをマッチさせるパターン
table = ^course_[0-9]*$

############################################
### flink sink 設定
### `connector`, `table-name`, `database-name` は自動生成されるため設定しないでください
############################################
flink.starrocks.jdbc-url = jdbc:mysql://xxx.xxx.x.x:xxxx
flink.starrocks.load-url = xxx.xxx.x.x:xxxx
flink.starrocks.username = user2
flink.starrocks.password = xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
flink.starrocks.sink.buffer-flush.interval-ms = 5000
```

### **JSON 形式でのデータインポート**

上記の例では CSV 形式でデータをインポートしていますが、適切な区切り文字を選ぶことができない場合は、`[table-rule]` 内の `flink.starrocks.*` の以下のパラメータを置き換える必要があります。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

以下のパラメータを入力して、JSON 形式でデータをインポートします。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> この方法はインポート速度に影響を与える可能性があります。

### 複数の INSERT INTO ステートメントを単一の Flink ジョブに統合する

**flink-create.all.sql** ファイルで [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) を使用し、複数の INSERT INTO ステートメントを単一の Flink ジョブに統合して、多くの Flink ジョブリソースを占有することを避けます。

   > 説明
   >
   > Flink はバージョン 1.13 から [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) の構文をサポートしています。

1. **result/flink-create.all.sql** ファイルを開きます。

2. ファイル内の SQL ステートメントを変更し、すべての INSERT INTO ステートメントをファイルの末尾に配置します。そして、最初の INSERT INTO ステートメントの前に `EXECUTE STATEMENT SET BEGIN` を追加し、最後の INSERT INTO ステートメントの後に `END;` を追加します。

   > **注意**
   >
   > CREATE DATABASE、CREATE TABLE の位置は変更しないでください。

   ```SQL
   CREATE DATABASE IF NOT EXISTS db;
   CREATE TABLE IF NOT EXISTS db.a1;
   CREATE TABLE IF NOT EXISTS db.b1;
   CREATE TABLE IF NOT EXISTS db.a2;
   CREATE TABLE IF NOT EXISTS db.b2;
   EXECUTE STATEMENT SET 
   BEGIN
     -- 1つまたは複数の INSERT INTO ステートメント
   INSERT INTO db.a1 SELECT * FROM db.b1;
   INSERT INTO db.a2 SELECT * FROM db.b2;
   END;
   ```

より多くの一般的な質問については、[MySQL リアルタイム同期から StarRocks へのよくある質問](../faq/loading/synchronize_mysql_into_sr.md)を参照してください。

