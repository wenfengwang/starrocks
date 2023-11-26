---
displayed_sidebar: "Japanese"
---

# MySQLからのリアルタイム同期

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、MySQLからのリアルタイムデータ同期をサポートしており、秒単位での超低遅延のリアルタイム分析を実現し、ユーザーが発生したデータをリアルタイムにクエリできるようにします。

このチュートリアルでは、リアルタイム分析をビジネスとユーザーにもたらす方法を学びます。以下のツールを使用して、MySQLからStarRocksへのデータをリアルタイムに同期する方法を示します：StarRocks Migration Tools（SMT）、Flink、Flink CDC Connector、およびflink-starrocks-connector。

<InsertPrivNote />

## 動作原理

次の図は、全体の同期プロセスを示しています。

![img](../assets/4.9.2.png)

MySQLからのリアルタイム同期は、データベースとテーブルのスキーマの同期とデータの同期の2つのステージで実装されます。まず、SMTはMySQLデータベースとテーブルのスキーマをStarRocksのテーブル作成ステートメントに変換します。次に、FlinkクラスターはFlinkジョブを実行して、MySQLのフルおよび増分データをStarRocksに同期します。

> **注意**
>
> 同期プロセスは、正確に一度だけのセマンティクスを保証します。

**同期プロセス**：

1. データベースとテーブルのスキーマを同期します。

   SMTは、同期するMySQLデータベースとテーブルのスキーマを読み取り、StarRocksで宛先データベースとテーブルを作成するためのSQLファイルを生成します。この操作は、SMTの設定ファイル内のMySQLおよびStarRocksの情報に基づいて行われます。

2. データを同期します。

   a. Flink SQLクライアントは、データローディングステートメント`INSERT INTO SELECT`を実行して、Flinkクラスターに1つ以上のFlinkジョブを送信します。

   b. Flinkクラスターは、Flinkジョブを実行してデータを取得します。[Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md)は、まずソースデータベースからフル履歴データを読み取り、その後、増分読み取りにシームレスに切り替え、データをflink-starrocks-connectorに送信します。

   c. flink-starrocks-connectorは、データをミニバッチで蓄積し、各バッチのデータをStarRocksに同期します。

> **注意**
>
> MySQLのデータ操作言語（DML）の操作のみがStarRocksに同期されます。データ定義言語（DDL）の操作は同期できません。

## シナリオ

MySQLからのリアルタイム同期は、データが常に変更される場合に使用される幅広いユースケースがあります。リアルワールドのユースケース「商品販売のリアルタイムランキング」を例に説明します。

Flinkは、MySQLの元の注文テーブルを基に商品販売のリアルタイムランキングを計算し、リアルタイムでStarRocksのプライマリキーテーブルにランキングを同期します。ユーザーは、可視化ツールをStarRocksに接続してランキングをリアルタイムで表示し、オンデマンドの運用インサイトを得ることができます。

## 準備

### 同期ツールのダウンロードとインストール

MySQLからデータを同期するには、次のツールをインストールする必要があります：SMT、Flink、Flink CDCコネクタ、およびflink-starrocks-connector。

1. Flinkをダウンロードしてインストールし、Flinkクラスターを起動します。または、[Flink公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)の手順に従ってこの手順を実行できます。

   a. Flinkを実行する前に、オペレーティングシステムにJava 8またはJava 11をインストールします。インストールされたJavaのバージョンを確認するには、次のコマンドを実行します。

    ```Bash
        # Javaのバージョンを表示します。
        java -version
        
        # 次の出力が返された場合、Java 8がインストールされています。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. [Flinkインストールパッケージ](https://flink.apache.org/downloads.html)をダウンロードして解凍します。Flink 1.14以降を使用することをお勧めします。最小許可バージョンはFlink 1.11です。このトピックでは、Flink 1.14.5を使用します。

   ```Bash
      # Flinkをダウンロードします。
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Flinkを解凍します。
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Flinkディレクトリに移動します。
      cd flink-1.14.5
    ```

   c. Flinkクラスターを起動します。

   ```Bash
      # Flinkクラスターを起動します。
      ./bin/start-cluster.sh
      
      # 次の出力が返された場合、Flinkクラスターが起動されています。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. [Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/releases)をダウンロードします。このトピックでは、データソースとしてMySQLを使用するため、`flink-sql-connector-mysql-cdc-x.x.x.jar`をダウンロードします。コネクタのバージョンは[Flink](https://github.com/ververica/flink-cdc-connectors/releases)のバージョンと一致する必要があります。詳細なバージョンマッピングについては、[サポートされるFlinkのバージョン](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)を参照してください。このトピックでは、Flink 1.14.5を使用し、`flink-sql-connector-mysql-cdc-2.2.0.jar`をダウンロードできます。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)をダウンロードします。バージョンはFlinkのバージョンと一致する必要があります。

    > flink-connector-starrocksパッケージ`x.x.x_flink-y.yy _ z.zz.jar`には、3つのバージョン番号が含まれています。
    >
    > - `x.x.x`はflink-connector-starrocksのバージョン番号です。
    > - `y.yy`はサポートされるFlinkのバージョンです。
    > - `z.zz`はFlinkでサポートされるScalaのバージョンです。Flinkのバージョンが1.14.x以前の場合は、Scalaのバージョンを持つパッケージをダウンロードする必要があります。
    >
    > このトピックでは、Flink 1.14.5およびScala 2.11を使用します。したがって、次のパッケージをダウンロードできます：`1.2.3_flink-14_2.11.jar`。

4. Flink CDCコネクタ（`flink-sql-connector-mysql-cdc-2.2.0.jar`）およびflink-connector-starrocks（`1.2.3_flink-1.14_2.11.jar`）のJARパッケージをFlinkの`lib`ディレクトリに移動します。

    > **注意**
    >
    > システムで既にFlinkクラスターが実行されている場合は、Flinkクラスターを停止して再起動して、JARパッケージを読み込んで検証する必要があります。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. [SMTパッケージ](https://www.starrocks.io/download/community)をダウンロードして解凍し、`flink-1.14.5`ディレクトリに配置します。StarRocksはLinux x86およびmacos ARM64向けのSMTパッケージを提供しています。オペレーティングシステムとCPUに基づいて1つを選択できます。

    ```Bash
    # Linux x86向け
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # macOS ARM64向け
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### MySQLバイナリログの有効化

MySQLからデータをリアルタイムに同期するためには、システムがMySQLバイナリログ（binlog）からデータを読み取り、データを解析し、データをStarRocksに同期する必要があります。MySQLバイナリログが有効になっていることを確認してください。

1. MySQLの設定ファイル`my.cnf`（デフォルトパス：`/etc/my.cnf`）を編集して、MySQLバイナリログを有効にします。

    ```Bash
    # MySQLバイナリログを有効にします。
    log_bin = ON
    # Binlogの保存パスを設定します。
    log_bin =/var/lib/mysql/mysql-bin
    # server_idを設定します。
    # MySQL 5.7.3以降ではserver_idが設定されていないと、MySQLサービスを使用できません。
    server_id = 1
    # Binlogの形式をROWに設定します。
    binlog_format = ROW
    # Binlogファイルのベース名です。各Binlogファイルを識別するために識別子が追加されます。
    log_bin_basename =/var/lib/mysql/mysql-bin
    # Binlogファイルのインデックスファイルです。すべてのBinlogファイルのディレクトリを管理します。
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 次のいずれかのコマンドを実行して、変更した設定ファイルを適用するためにMySQLを再起動します。

    ```Bash
    # サービスを使用してMySQLを再起動します。
    service mysqld restart
    # mysqldスクリプトを使用してMySQLを再起動します。
    /etc/init.d/mysqld restart
    ```

3. MySQLに接続し、MySQLバイナリログが有効になっているかどうかを確認します。

    ```Plain
    -- MySQLに接続します。
    mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

    -- MySQLバイナリログが有効になっているかどうかを確認します。
    mysql> SHOW VARIABLES LIKE 'log_bin'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       | ON    |
    +---------------+-------+
    1 row in set (0.00 sec)
    ```

## データベースとテーブルのスキーマを同期する

1. SMTの設定ファイルを編集します。
   SMTの`conf`ディレクトリに移動し、MySQLの接続情報、同期するデータベースとテーブルのマッチングルール、およびflink-starrocks-connectorの設定情報などを含む`config_prod.conf`設定ファイルを編集します。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocksのBE数
    be_num = 3
    # StarRocks-1.18.1以降でサポートされる`decimal_v3`です。
    use_decimal_v3 = true
    # 生成されるDDL SQLを保存するファイル
    output_dir = ./result

    [table-rule.1]
    # プロパティを設定するためのデータベースのパターン
    database = ^demo.*$
    # プロパティを設定するためのテーブルのパターン
    table = ^.*$

    ############################################
    ### Flink sink configurations
    ### `connector`、`table-name`、`database-name`を設定しないでください。これらは自動生成されます。
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
       - `host`：MySQLサーバーのIPアドレス。
       - `port`：MySQLデータベースのポート番号。デフォルトは`3306`です。
       - `user`：MySQLデータベースにアクセスするためのユーザー名
       - `password`：ユーザーのパスワード

    - `[table-rule]`：データベースとテーブルのマッチングルールと対応するflink-connector-starrocksの設定。

       - `Database`、`table`：MySQLのデータベースとテーブルの名前。正規表現がサポートされています。
       - `flink.starrocks.*`：flink-connector-starrocksの設定情報。詳細な設定と情報については、[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を参照してください。

       > 異なるテーブルに対して異なるflink-connector-starrocksの設定を使用する必要がある場合は、異なるテーブルに対して異なるflink-connector-starrocksの設定を使用する必要があります。たとえば、一部のテーブルが頻繁に更新され、データのロードを高速化する必要がある場合は、[異なるテーブルに対して異なるflink-connector-starrocksの設定を使用する](#use-different-flink-connector-starrocks-configurations-for-different-tables)を参照してください。MySQLのシャーディング後に複数のテーブルをStarRocksの同じテーブルに同期する必要がある場合は、[MySQLのシャーディング後に複数のテーブルをStarRocksの1つのテーブルに同期する](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)を参照してください。

    - `[other]`：その他の情報
       - `be_num`：StarRocksクラスターのBE数（このパラメータは、後続のStarRocksテーブル作成で適切な数のタブレットを設定するために使用されます）。
       - `use_decimal_v3`：[Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)を有効にするかどうか。Decimal V3が有効になると、データがStarRocksに同期される際にMySQLの10進数データがDecimal V3データに変換されます。
       - `output_dir`：生成されるSQLファイルを保存するパス。SQLファイルはStarRocksでデータベースとテーブルを作成し、FlinkジョブをFlinkクラスターに送信するために使用されます。デフォルトのパスは`./result`ですが、デフォルトの設定を保持することをお勧めします。

2. SMTを実行して、MySQLのデータベースとテーブルのスキーマを読み取り、`./result`ディレクトリに基づいてSQLファイルを生成します。`starrocks-create.all.sql`ファイルは、StarRocksでデータベースとテーブルを作成するために使用され、`flink-create.all.sql`ファイルは、FlinkジョブをFlinkクラスターに送信するために使用されます。

    ```Bash
    # SMTを実行します。
    ./starrocks-migrate-tool

    # resultディレクトリに移動し、このディレクトリ内のファイルを確認します。
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 次のコマンドを実行してStarRocksに接続し、`starrocks-create.all.sql`ファイルを実行してStarRocksでデータベースとテーブルを作成します。このSQLファイルのデフォルトのテーブル作成ステートメントを使用して、[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)のテーブルを作成することをお勧めします。

    > **注意**
    >
    > ビジネスのニーズに基づいてテーブル作成ステートメントを変更し、プライマリキーテーブルを使用しないテーブルを作成することもできます。ただし、ソースMySQLデータベースのDELETE操作は、非プライマリキーテーブルに同期できません。そのようなテーブルを作成する場合は注意してください。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    データをFlinkで処理してから宛先のStarRocksテーブルに書き込む必要がある場合、ソーステーブルと宛先テーブルのスキーマは異なります。この場合、テーブル作成ステートメントを変更する必要があります。この例では、宛先テーブルには`product_id`と`product_name`の列と商品販売のリアルタイムランキングのみが必要です。次のテーブル作成ステートメントを使用できます。

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
    > StarRocksはv2.5.7以降、テーブルまたはパーティションを作成する際にバケット数（BUCKETS）を自動的に設定できるようになりました。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## データを同期する

Flinkクラスターを実行し、Flinkジョブを送信してMySQLからStarRocksにフルおよび増分データを連続して同期します。

1. Flinkディレクトリに移動し、次のコマンドを実行して`flink-create.all.sql`ファイルをFlink SQLクライアントで実行します。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    このSQLファイルでは、動的テーブル`source table`と`sink table`、クエリステートメント`INSERT INTO SELECT`、およびコネクタ、ソースデータベース、および宛先データベースが定義されています。このファイルが実行されると、FlinkジョブがFlinkクラスターに送信され、データ同期が開始されます。

    > **注意**
    >
    > - Flinkクラスターが起動していることを確認してください。Flinkクラスターを起動するには、`flink/bin/start-cluster.sh`を実行します。
    > - Flinkのバージョンが1.13より前の場合、SQLファイル`flink-create.all.sql`を直接実行できない場合があります。この場合、SQLクライアントのコマンドラインインターフェース（CLI）でSQLステートメントを1つずつ実行する必要があります。また、`\`文字をエスケープする必要があります。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **同期中のデータの処理**：

    データを同期する際にデータを処理する必要がある場合、データをグループ化したり、データを結合したりするなどの処理を行う場合は、`flink-create.all.sql`ファイルを変更できます。次の例では、COUNT (*)とGROUP BYを実行して商品販売のリアルタイムランキングを計算しています。

    ```Bash
        $ ./bin/sql-client.sh -f flink-create.all.sql
        No default environment is specified.
        Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
        [INFO] Executing SQL from file.

        Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
        [INFO] Execute statement succeed.

        -- MySQLの注文テーブルを基に商品販売のリアルタイムランキングを計算する動的テーブル`source table`を作成します。
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
        [INFO] Execute statement succeed.

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
        [INFO] Execute statement succeed.

        -- 商品販売のリアルタイムランキングを実装します。`sink table`が動的に更新され、`source table`のデータ変更を反映します。
        Flink SQL> 
        INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
        [INFO] Submitting SQL update statement to the cluster...
        [INFO] SQL update statement has been successfully submitted to the cluster:
        Job ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

    データの一部のみを同期する必要がある場合、たとえば支払い日が2021年12月21日以降のデータのみを同期する場合は、`INSERT INTO SELECT`の`WHERE`句を使用してフィルタ条件を設定できます。たとえば、`WHERE pay_dt > '2021-12-21'`という条件を設定すると、この条件を満たさないデータはStarRocksに同期されません。

    Flinkジョブがフルおよび増分の同期のために送信された場合、次の結果が返されます。

    ```SQL
    [INFO] Submitting SQL update statement to the cluster...
    [INFO] SQL update statement has been successfully submitted to the cluster:
    Job ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

2. [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を使用するか、Flink SQLクライアントで`bin/flink list -running`コマンドを実行して、Flinkクラスターで実行中のFlinkジョブとジョブIDを表示できます。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        Waiting for response...
        ------------------ Running/Restarting Jobs -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
        --------------------------------------------------------------
    ```

    > **注意**
    >
    > ジョブが異常な場合は、Flink WebUIを使用するか、Flink 1.14.5の`/log`ディレクトリのログファイルを表示してトラブルシューティングを実行できます。

## FAQ

### 異なるテーブルに対して異なるflink-connector-starrocksの設定を使用する

データソースの一部のテーブルが頻繁に更新され、flink-connector-starrocksのロード速度を高速化する必要がある場合は、SMTの設定ファイル`config_prod.conf`で各テーブルに対して個別のflink-connector-starrocksの設定を行う必要があります。

```Bash
[table-rule.1]
# プロパティを設定するためのデータベースのパターン
database = ^order.*$
# プロパティを設定するためのテーブルのパターン
table = ^.*$

############################################
### Flink sink configurations
### `connector`、`table-name`、`database-name`を設定しないでください。これらは自動生成されます
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000[table-rule.2]
# プロパティを設定するためのデータベースのパターン
database = ^order2.*$
# プロパティを設定するためのテーブルのパターン
table = ^.*$

############################################
### Flink sink configurations
### `connector`、`table-name`、`database-name`を設定しないでください。これらは自動生成されます
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

### MySQLのシャーディング後に複数のテーブルをStarRocksの1つのテーブルに同期する

シャーディングが行われた後、1つのMySQLテーブルのデータは複数のテーブルに分割されるか、さらには複数のデータベースに分散される場合があります。すべてのテーブルは同じスキーマを持ちます。この場合、`[table-rule]`を設定してこれらのテーブルを1つのStarRocksテーブルに同期できます。たとえば、MySQLに`edu_db_1`と`edu_db_2`という2つのデータベースがあり、それぞれに`course_1`と`course_2`という2つのテーブルがあり、すべてのテーブルのスキーマが同じである場合、次の`[table-rule]`設定を使用して、これらのテーブルをすべて1つのStarRocksテーブルに同期できます。

> **注意**
>
> StarRocksテーブルの名前はデフォルトで`course__auto_shard`になります。異なる名前を使用する場合は、SQLファイル`starrocks-create.all.sql`および`flink-create.all.sql`で変更できます。

```Bash
[table-rule.1]
# プロパティを設定するためのデータベースのパターン
database = ^edu_db_[0-9]*$
# プロパティを設定するためのテーブルのパターン
table = ^course_[0-9]*$

############################################
### Flink sink configurations
### `connector`、`table-name`、`database-name`を設定しないでください。これらは自動生成されます
############################################
flink.starrocks.jdbc-url = jdbc: mysql://xxx.xxx.x.x:xxxx
flink.starrocks.load-url = xxx.xxx.x.x:xxxx
flink.starrocks.username = user2
flink.starrocks.password = xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
flink.starrocks.sink.buffer-flush.interval-ms = 5000
```

### JSON形式でデータをインポートする

前述の例では、データはCSV形式でインポートされています。適切な区切り文字を選択できない場合は、`[table-rule]`の`flink.starrocks.*`パラメータを置き換える必要があります。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> この方法は、ロード速度がわずかに低下します。

### 複数のINSERT INTOステートメントを1つのFlinkジョブとして実行する

`flink-create.all.sql`ファイルで[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)構文を使用して、複数のINSERT INTOステートメントを1つのFlinkジョブとして実行できます。これにより、複数のステートメントが多くのFlinkジョブリソースを占有することを防ぎ、複数のクエリを実行する効率を向上させることができます。

> **注意**
>
> Flinkは、1.13以降からSTATEMENT SET構文をサポートしています。

1. `result/flink-create.all.sql`ファイルを開きます。

2. ファイル内のSQLステートメントを変更します。すべてのINSERT INTOステートメントをファイルの末尾に移動します。最初のINSERT INTOステートメントの前に`EXECUTE STATEMENT SET BEGIN`を配置し、最後のINSERT INTOステートメントの後に`END;`を配置します。

> **注意**
>
> CREATE DATABASEおよびCREATE TABLEの位置は変更しません。

```SQL
CREATE DATABASE IF NOT EXISTS db;
CREATE TABLE IF NOT EXISTS db.a1;
CREATE TABLE IF NOT EXISTS db.b1;
CREATE TABLE IF NOT EXISTS db.a2;
CREATE TABLE IF NOT EXISTS db.b2;
EXECUTE STATEMENT SET 
BEGIN-- one or more INSERT INTO statements
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;
```
