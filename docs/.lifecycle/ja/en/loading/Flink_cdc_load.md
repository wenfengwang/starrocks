---
displayed_sidebar: English
---

# MySQLからのリアルタイム同期

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、秒単位でMySQLからのリアルタイムデータ同期をサポートし、超低レイテンシのリアルタイム分析を大規模に提供し、ユーザーが発生するリアルタイムデータをクエリできるようにします。

このチュートリアルは、ビジネスとユーザーにリアルタイム分析をもたらす方法を学ぶのに役立ちます。StarRocks Migration Tools (SMT)、Flink、Flink CDC Connector、およびflink-starrocks-connectorを使用して、MySQLからStarRocksへリアルタイムでデータを同期する方法を実演します。

<InsertPrivNote />

## 仕組み

以下の図は、同期プロセス全体を示しています。

![img](../assets/4.9.2.png)

MySQLからのリアルタイム同期は、データベース＆テーブルスキーマの同期とデータの同期の2段階で実装されます。まず、SMTはMySQLのデータベース＆テーブルスキーマをStarRocksのテーブル作成ステートメントに変換します。その後、FlinkクラスタはFlinkジョブを実行して、MySQLのフルデータとインクリメンタルデータをStarRocksに同期します。

> **注**
>
> 同期プロセスは、exactly-onceセマンティクスを保証します。

**同期プロセス**:

1. データベース＆テーブルスキーマを同期します。

   SMTは同期するMySQLデータベース＆テーブルのスキーマを読み取り、StarRocksで宛先データベース＆テーブルを作成するSQLファイルを生成します。この操作は、SMTの設定ファイルにあるMySQLとStarRocksの情報に基づいています。

2. データを同期します。

   a. Flink SQLクライアントは、データロードステートメント `INSERT INTO SELECT` を実行して、1つ以上のFlinkジョブをFlinkクラスタに送信します。

   b. FlinkクラスタはFlinkジョブを実行してデータを取得します。[Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md)は最初にソースデータベースからのフルデータを読み取り、次にインクリメンタル読み取りにシームレスに切り替え、データをflink-starrocks-connectorに送信します。

   c. flink-starrocks-connectorはデータをミニバッチで蓄積し、各バッチのデータをStarRocksに同期します。

> **注**
>
> MySQLのデータ操作言語(DML)操作のみがStarRocksに同期されます。データ定義言語(DDL)の操作は同期されません。

## シナリオ

MySQLからのリアルタイム同期は、データが絶えず変更されるさまざまなユースケースに適用されます。例として、「商品販売のリアルタイムランキング」の実世界のユースケースを考えてみましょう。

Flinkは、MySQLのオリジナルのオーダーテーブルに基づいて商品販売のリアルタイムランキングを計算し、そのランキングをリアルタイムでStarRocksのプライマリキーテーブルに同期します。ユーザーは可視化ツールをStarRocksに接続してリアルタイムでランキングを表示し、オンデマンドで運用インサイトを得ることができます。

## 準備

### 同期ツールのダウンロードとインストール

MySQLからデータを同期するためには、以下のツールをインストールする必要があります：SMT、Flink、Flink CDCコネクタ、およびflink-starrocks-connector。

1. Flinkをダウンロードしてインストールし、Flinkクラスタを起動します。この手順は、[Flink公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)に従って実行することもできます。

   a. Flinkを実行する前に、オペレーティングシステムにJava 8またはJava 11をインストールしてください。以下のコマンドを実行して、インストールされているJavaのバージョンを確認できます。

    ```Bash
        # Javaのバージョンを表示します。
        java -version
        
        # 以下の出力が返された場合、Java 8がインストールされています。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. [Flinkインストールパッケージ](https://flink.apache.org/downloads.html)をダウンロードして解凍します。Flink 1.14以降の使用を推奨します。許可される最小バージョンはFlink 1.11です。このトピックではFlink 1.14.5を使用します。

   ```Bash
      # Flinkをダウンロードします。
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Flinkを解凍します。
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Flinkディレクトリに移動します。
      cd flink-1.14.5
    ```

   c. Flinkクラスタを起動します。

   ```Bash
      # Flinkクラスタを起動します。
      ./bin/start-cluster.sh
      
      # 以下の出力が返された場合、Flinkクラスタが起動しています。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. [Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/releases)をダウンロードします。このトピックではMySQLをデータソースとして使用するため、`flink-sql-connector-mysql-cdc-x.x.x.jar`をダウンロードします。コネクタのバージョンは[Flink](https://github.com/ververica/flink-cdc-connectors/releases)のバージョンと一致している必要があります。詳細なバージョンマッピングについては、[サポートされるFlinkバージョン](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)を参照してください。このトピックではFlink 1.14.5を使用しており、`flink-sql-connector-mysql-cdc-2.2.0.jar`をダウンロードできます。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.2.0/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)をダウンロードします。バージョンはFlinkのバージョンと一致する必要があります。

    > flink-connector-starrocksパッケージの`x.x.x_flink-y.yy_z.zz.jar`には3つのバージョン番号が含まれています：
    >
    > - `x.x.x`はflink-connector-starrocksのバージョン番号です。
    > - `y.yy`はサポートされているFlinkのバージョンです。
    > - `z.zz`はFlinkがサポートするScalaのバージョンです。Flinkのバージョンが1.14.x以前の場合、Scalaバージョンのパッケージをダウンロードする必要があります。
    >
    > このトピックではFlink 1.14.5とScala 2.11を使用しているため、以下のパッケージをダウンロードできます：`1.2.3_flink-1.14_2.11.jar`。

4. Flink CDCコネクタ(`flink-sql-connector-mysql-cdc-2.2.0.jar`)とflink-connector-starrocks(`1.2.3_flink-1.14_2.11.jar`)のJARパッケージをFlinkの`lib`ディレクトリに移動します。

    > **注**
    >
    > システムにFlinkクラスタが既に実行されている場合、Flinkクラスタを停止して再起動し、JARパッケージをロードして検証する必要があります。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. [SMTパッケージ](https://www.starrocks.io/download/community)をダウンロードして解凍し、`flink-1.14.5`ディレクトリに配置します。StarRocksはLinux x86とmacOS ARM64用のSMTパッケージを提供しています。オペレーティングシステムとCPUに基づいて選択できます。

    ```Bash
    # Linux x86用
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # macOS ARM64用
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### MySQLバイナリログを有効にする

リアルタイムでMySQLからデータを同期するためには、システムがMySQLバイナリログ(binlog)からデータを読み取り、解析し、その後StarRocksに同期する必要があります。MySQLバイナリログが有効であることを確認してください。

1. MySQL設定ファイル`my.cnf`（デフォルトパス：`/etc/my.cnf`）を編集して、MySQLバイナリログを有効にします。

    ```Bash
    # MySQLバイナリログを有効にします。
    log_bin = ON
    # バイナリログの保存パスを設定します。
    log_bin =/var/lib/mysql/mysql-bin
    # server_idを設定します。
    # MySQL 5.7.3以降でserver_idが設定されていない場合、MySQLサービスは使用できません。
    server_id = 1
    # Binlog形式をROWに設定します。
    binlog_format = ROW
    # Binlogファイルの基本名です。各Binlogファイルを識別するために識別子が追加されます。
    log_bin_basename =/var/lib/mysql/mysql-bin
    # すべてのBinlogファイルのディレクトリを管理するBinlogファイルのインデックスファイル。
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 変更された設定ファイルを有効にするために、以下のコマンドのいずれかを実行してMySQLを再起動します。

    ```Bash
    # serviceを使用してMySQLを再起動します。
    service mysqld restart
    # mysqldスクリプトを使用してMySQLを再起動します。
    /etc/init.d/mysqld restart
    ```

3. MySQLに接続し、MySQLバイナリログが有効かどうかを確認します。

    ```Plain
    -- MySQLに接続します。
    mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

    -- MySQLバイナリログが有効かどうかを確認します。
    mysql> SHOW VARIABLES LIKE 'log_bin'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       | ON    |
    +---------------+-------+
    1行セット (0.00秒)
    ```

## データベースとテーブルスキーマの同期

1. SMT設定ファイルを編集します。
   SMTの`conf`ディレクトリに移動し、`config_prod.conf`設定ファイルを編集して、MySQL接続情報、同期対象のデータベースとテーブルのマッチングルール、flink-starrocks-connectorの設定情報などを設定します。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocks内のBE数
    be_num = 3
    # `decimal_v3`はStarRocks-1.18.1以降でサポートされています。
    use_decimal_v3 = true
    # 変換されたDDL SQLを保存するファイル
    output_dir = ./result

    [table-rule.1]
    # プロパティ設定のためのデータベースマッチングパターン
    database = ^demo.*$
    # プロパティ設定のためのテーブルマッチングパターン
    table = ^.*$

    ############################################
    ### Flink sink設定
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

    - `[db]`: MySQL接続情報。
       - `host`: MySQLサーバーのIPアドレス。
       - `port`: MySQLデータベースのポート番号、デフォルトは`3306`
       - `user`: MySQLデータベースへのアクセスユーザー名
       - `password`: ユーザー名のパスワード

    - `[table-rule]`: データベースとテーブルのマッチングルールおよび対応するflink-connector-starrocks設定。

       - `database`, `table`: MySQLのデータベース名とテーブル名。正規表現がサポートされています。
       - `flink.starrocks.*`: flink-connector-starrocksの設定情報。より多くの設定と情報については、[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を参照してください。

       > 異なるテーブルに対して異なるflink-connector-starrocks設定を使用する必要がある場合、例えば、一部のテーブルが頻繁に更新され、データロードを加速する必要がある場合は、[異なるテーブルに対する異なるflink-connector-starrocks設定の使用](#use-different-flink-connector-starrocks-configurations-for-different-tables)を参照してください。MySQLシャーディングから取得した複数のテーブルをStarRocksの同じテーブルに同期する必要がある場合は、[MySQLシャーディング後の複数のテーブルをStarRocksの1つのテーブルに同期する](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)を参照してください。

    - `[other]`: その他の情報
       - `be_num`: StarRocksクラスター内のBEの数（このパラメータは、後続のStarRocksテーブル作成で適切なタブレット数を設定するために使用されます）。
       - `use_decimal_v3`: [Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)を使用するかどうか。Decimal V3を有効にすると、データがStarRocksに同期される際に、MySQLのdecimalデータがDecimal V3データに変換されます。
       - `output_dir`: 生成されるSQLファイルを保存するパス。SQLファイルは、StarRocksにデータベースとテーブルを作成し、FlinkジョブをFlinkクラスタに送信するために使用されます。デフォルトのパスは`./result`であり、デフォルトの設定を維持することを推奨します。

2. SMTを実行してMySQLのデータベースとテーブルスキーマを読み取り、設定ファイルに基づいて`./result`ディレクトリにSQLファイルを生成します。`starrocks-create.all.sql`ファイルはStarRocksにデータベースとテーブルを作成するために使用され、`flink-create.all.sql`ファイルはFlinkジョブをFlinkクラスタに送信するために使用されます。

    ```Bash
    # SMTを実行します。
    ./starrocks-migrate-tool

    # resultディレクトリに移動し、このディレクトリ内のファイルを確認します。
    cd result
    ls
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 次のコマンドを実行してStarRocksに接続し、`starrocks-create.all.sql`ファイルを実行してStarRocksにデータベースとテーブルを作成します。SQLファイルにあるデフォルトのテーブル作成ステートメントを使用して、[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)のテーブルを作成することを推奨します。

    > **注記**
    >
    > また、ビジネスニーズに基づいてテーブル作成ステートメントを変更し、プライマリキーテーブルを使用しないテーブルを作成することもできます。ただし、ソースMySQLデータベースのDELETE操作は、非プライマリキーテーブルに同期されません。このようなテーブルを作成する場合は注意してください。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    データがStarRocksの宛先テーブルに書き込まれる前にFlinkで処理する必要がある場合、ソーステーブルと宛先テーブルのテーブルスキーマは異なります。この場合、テーブル作成ステートメントを変更する必要があります。この例では、宛先テーブルには`product_id`と`product_name`の列のみが必要で、商品販売のリアルタイムランキングが必要です。次のテーブル作成ステートメントを使用できます。

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

    > **通知**
    >
    > v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数(BUCKETS)を自動的に設定できるようになりました。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## データの同期

Flinkクラスタを実行し、Flinkジョブを送信して、MySQLからStarRocksへのフルデータとインクリメンタルデータを継続的に同期します。

1. Flinkディレクトリに移動し、次のコマンドを実行してFlink SQLクライアントで`flink-create.all.sql`ファイルを実行します。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    このSQLファイルは、動的テーブル`source table`と`sink table`、クエリステートメント`INSERT INTO SELECT`を定義し、コネクタ、ソースデータベース、および宛先データベースを指定します。このファイルが実行されると、FlinkジョブがFlinkクラスタに送信され、データ同期が開始されます。

    > **注記**
    >
    > - Flinkクラスタが起動していることを確認してください。`flink/bin/start-cluster.sh`を実行してFlinkクラスタを起動できます。
    > - Flinkのバージョンが1.13より前の場合、`flink-create.all.sql`ファイルを直接実行することはできません。このファイル内のSQLステートメントを1つずつSQLクライアントのコマンドラインインターフェース（CLI）で実行する必要があります。また、`\`文字をエスケープする必要があります。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **同期中のデータ処理**:

    同期中にデータを処理する必要がある場合、例えばGROUP BYやJOINをデータに適用する場合は、`flink-create.all.sql`ファイルを修正できます。以下の例では、COUNT(*)とGROUP BYを実行して商品売上のリアルタイムランキングを計算します。

    ```Bash
        $ ./bin/sql-client.sh -f flink-create.all.sql
        デフォルトの環境が指定されていません。
        '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'を検索中...見つかりませんでした。
        [INFO] ファイルからSQLを実行中。

        Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
        [INFO] ステートメントの実行に成功しました。

        -- MySQLの注文テーブルに基づいて動的テーブル`source table`を作成します。
        Flink SQL> 
        CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_src` (`order_id` BIGINT NOT NULL,
        `product_id` INT NULL,
        `order_date` TIMESTAMP NOT NULL,
        `customer_name` STRING NOT NULL,
        `product_name` STRING NOT NULL,
        `price` DECIMAL(10, 5) NULL,
        PRIMARY KEY(`order_id`)
        NOT ENFORCED
        ) with ('connector' = 'mysql-cdc',
        'hostname' = 'xxx.xx.xxx.xxx',
        'port' = '3306',
        'username' = 'root',
        'password' = '',
        'database-name' = 'demo',
        'table-name' = 'orders'
        );
        [INFO] ステートメントの実行に成功しました。

        -- 動的テーブル`sink table`を作成します。
        Flink SQL> 
        CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_sink` (`product_id` INT NOT NULL,
        `product_name` STRING NOT NULL,
        `sales_cnt` BIGINT NOT NULL,
        PRIMARY KEY(`product_id`)
        NOT ENFORCED
        ) with ('sink.max-retries' = '10',
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
        [INFO] ステートメントの実行に成功しました。

        -- 商品売上のリアルタイムランキングを実装し、`sink table`が`source table`のデータ変更を動的に反映するようにします。
        Flink SQL> 
        INSERT INTO `default_catalog`.`demo`.`orders_sink` SELECT product_id, product_name, COUNT(*) AS cnt FROM `default_catalog`.`demo`.`orders_src` GROUP BY product_id, product_name;
        [INFO] クラスターにSQL更新ステートメントを送信中...
        [INFO] SQL更新ステートメントがクラスターに正常に送信されました:
        ジョブID: 5ae005c4b3425d8bb13fe660260a35da
    ```

    支払い日が2021年12月21日より後のデータのみを同期する必要がある場合など、データの一部のみを同期する必要がある場合は、`INSERT INTO SELECT`の`WHERE`句を使用してフィルター条件を設定できます。例えば`WHERE pay_dt > '2021-12-21'`。この条件を満たさないデータはStarRocksに同期されません。

    次の結果が返された場合、Flinkジョブは完全同期と増分同期のために送信されています。

    ```SQL
    [INFO] クラスターにSQL更新ステートメントを送信中...
    [INFO] SQL更新ステートメントがクラスターに正常に送信されました:
    ジョブID: 5ae005c4b3425d8bb13fe660260a35da
    ```

2. Flink WebUI[を使用する](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)か、Flink SQLクライアントで`bin/flink list -running`コマンドを実行して、Flinkクラスターで実行中のFlinkジョブとジョブIDを表示できます。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        応答を待っています...
        ------------------ 実行中/再起動中のジョブ -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (実行中)
        --------------------------------------------------------------
    ```

    > **注記**
    >
    > ジョブに異常がある場合は、Flink WebUIを使用するか、Flink 1.14.5の`/log`ディレクトリにあるログファイルを確認することでトラブルシューティングを行うことができます。

## FAQ

### テーブルごとに異なるflink-connector-starrocks設定を使用する

データソース内の一部のテーブルが頻繁に更新され、flink-connector-starrocksのロード速度を加速したい場合は、SMT設定ファイル`config_prod.conf`でテーブルごとに個別のflink-connector-starrocks設定を設定する必要があります。

```Bash
[table-rule.1]
# プロパティ設定のためのデータベースと一致するパターン
database = ^order.*$
# プロパティ設定のためのテーブルと一致するパターン
table = ^.*$

############################################
### Flink sink設定
### `connector`, `table-name`, `database-name`は設定しないでください。これらは自動生成されます
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url=<fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000
[table-rule.2]
# プロパティ設定のためのデータベースと一致するパターン
database = ^order2.*$
# プロパティ設定のためのテーブルと一致するパターン
table = ^.*$

############################################
### Flink sink設定
### `connector`, `table-name`, `database-name`は設定しないでください。これらは自動生成されます
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url=<fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=10000
```

### MySQLシャーディング後に複数のテーブルをStarRocksの1つのテーブルに同期する

シャーディングが行われた後、MySQLの1つのテーブルのデータが複数のテーブルに分割されたり、複数のデータベースに分散されたりすることがあります。すべてのテーブルは同じスキーマを持っています。この場合、`[table-rule]`を設定してこれらのテーブルを1つのStarRocksテーブルに同期することができます。例えば、MySQLには`edu_db_1`と`edu_db_2`という2つのデータベースがあり、それぞれに`course_1`と`course_2`という2つのテーブルがあり、すべてのテーブルのスキーマは同じです。以下の`[table-rule]`設定を使用して、すべてのテーブルを1つのStarRocksテーブルに同期できます。

> **注記**
>
> StarRocksテーブルの名前はデフォルトで`course__auto_shard`です。異なる名前を使用する必要がある場合は、`starrocks-create.all.sql`と`flink-create.all.sql`のSQLファイルで変更できます。

```Bash
[table-rule.1]
# プロパティ設定のためのデータベースと一致するパターン
database = ^edu_db_[0-9]*$
# プロパティ設定のためのテーブルと一致するパターン
table = ^course_[0-9]*$

############################################
### Flink sink設定
### `connector`, `table-name`, `database-name`は設定しないでください。これらは自動生成されます
############################################
flink.starrocks.jdbc-url=jdbc:mysql://xxx.xxx.x.x:xxxx
flink.starrocks.load-url=xxx.xxx.x.x:xxxx
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=5000
```

### JSON形式でデータをインポートする

前述の例ではデータはCSV形式でインポートされます。適切な区切り文字を選択できない場合は、`[table-rule]`内の`flink.starrocks.*`の以下のパラメータを置き換える必要があります。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
```

以下のパラメータを渡すことで、データはJSON形式でインポートされます。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注記**
>
> この方法ではロード速度が若干遅くなる可能性があります。

### 複数のINSERT INTOステートメントを1つのFlinkジョブとして実行する

`flink-create.all.sql`ファイル内で[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)構文を使用して、複数のINSERT INTOステートメントを1つのFlinkジョブとして実行できます。これにより、複数のステートメントがFlinkジョブリソースを過度に消費するのを防ぎ、複数のクエリを効率的に実行することができます。

> **注記**
>
> Flinkはバージョン1.13以降でSTATEMENT SET構文をサポートしています。

1. `result/flink-create.all.sql` ファイルを開きます。

2. ファイル内のSQLステートメントを変更します。すべてのINSERT INTOステートメントをファイルの末尾に移動させます。最初のINSERT INTOステートメントの前に `EXECUTE STATEMENT SET BEGIN` を配置し、最後のINSERT INTOステートメントの後に `END;` を配置します。

> **注記**
>
> CREATE DATABASEとCREATE TABLEの位置は変更しないでください。

```SQL
CREATE DATABASE IF NOT EXISTS db;
CREATE TABLE IF NOT EXISTS db.a1;
CREATE TABLE IF NOT EXISTS db.b1;
CREATE TABLE IF NOT EXISTS db.a2;
CREATE TABLE IF NOT EXISTS db.b2;
EXECUTE STATEMENT SET 
BEGIN -- 一つ以上のINSERT INTOステートメント
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;
```
