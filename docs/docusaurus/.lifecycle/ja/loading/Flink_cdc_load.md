---
displayed_sidebar: "Japanese"
---

# MySQL からのリアルタイム同期

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks は、数秒以内で MySQL からのリアルタイムデータ同期をサポートし、規模に応じた超低遅延のリアルタイム分析を提供し、リアルタイムデータをクエリできるようにします。

このチュートリアルでは、リアルタイム分析をビジネスとユーザーにもたらす方法を学ぶことができます。次のツールを使用して MySQL から StarRocks にデータをリアルタイムに同期する方法を示しています: StarRocks Migration Tools (SMT)、Flink、Flink CDC Connector、および flink-starrocks-connector。

<InsertPrivNote />

## 動作原理

次の図は、全体の同期プロセスを示しています。

![img](../assets/4.9.2.png)

MySQL からのリアルタイム同期は、次の2つの段階で実装されています: データベースとテーブルスキーマの同期、データの同期。まず、SMT は MySQL データベースとテーブルスキーマを StarRocks のテーブル作成ステートメントに変換します。その後、Flink クラスターは Flink ジョブを実行して、MySQL の全データおよび増分データを StarRocks に同期します。

> **ノート**
>
> 同期プロセスは、確実に一度だけのセマンティクスを保証します。

**同期プロセス**:

1. データベースとテーブルスキーマを同期する

   SMT は、同期される MySQL データベースとテーブルのスキーマを読み取り、SMT の設定ファイルに含まれる MySQL および StarRocks の情報に基づいて、StarRocks での宛先データベースおよびテーブルの作成用の SQL ファイルを生成します。

2. データを同期する

   a. Flink SQL クライアントは、データローディングステートメント `INSERT INTO SELECT` を実行して、複数の Flink ジョブを Flink クラスターに送信します。

   b. Flink クラスターは、データを取得するために Flink ジョブを実行します。[Flink CDC コネクタ](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md) はまずソースデータベースから全データを読み取り、シームレスに増分読み取りに切り替え、データを flink-starrocks-connector に送信します。

   c. flink-starrocks-connector はミニバッチでデータを蓄積し、各バッチのデータを StarRocks に同期します。

> **ノート**
>
> MySQL のデータ操作言語 (DML) のみが、StarRocks に同期できます。データ定義言語 (DDL) 操作は同期できません。

## シナリオ

MySQL からのリアルタイム同期は、データが常に変化する幅広いユースケースで使用されます。リアルタイムランキング取引の例を取り上げましょう。

Flink は MySQL の元の注文テーブルを使用して商品販売のリアルタイムランキングを計算し、ランキングを StarRocks のプライマリキー テーブルにリアルタイムに同期します。ユーザーは、視覚化ツールを StarRocks に接続して、リアルタイムでランキングを表示し、必要な操作インサイトを得ることができます。

## 準備

### 同期ツールのダウンロードおよびインストール

MySQL からデータを同期するには、次のツールをインストールする必要があります: SMT、Flink、Flink CDC Connector、および flink-starrocks-connector

1. Flink をダウンロードしてインストールし、Flink クラスターを開始します。または、[Flink 公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)の手順に従ってこの手順を実行できます。

   a. Flink を実行する前に、オペレーティングシステムに Java 8 または Java 11 をインストールします。インストールされた Java バージョンを確認するには、次のコマンドを実行します。

    ```Bash
        # Java バージョンを表示します。
        java -version
        
        # 次の出力が返された場合、Java 8 がインストールされています。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. [Flink インストールパッケージ](https://flink.apache.org/downloads.html)をダウンロードして展開します。Flink 1.14 またはそれ以降のバージョンを使用することをお勧めします。最小の許容バージョンは Flink 1.11 です。このトピックでは Flink 1.14.5 を使用しています。

   ```Bash
      # Flink をダウンロードします。
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Flink を展開します。
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Flink ディレクトリに移動します。
      cd flink-1.14.5
    ```

   c. Flink クラスターを開始します。

   ```Bash
      # Flink クラスターを開始します。
      ./bin/start-cluster.sh
      
      # 次の出力が返された場合、Flink クラスターが開始されています。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. [Flink CDC Connector](https://github.com/ververica/flink-cdc-connectors/releases)をダウンロードします。このトピックではデータソースとして MySQL を使用するため、`flink-sql-connector-mysql-cdc-x.x.x.jar` がダウンロードされます。コネクタのバージョンは[Flink](https://github.com/ververica/flink-cdc-connectors/releases)のバージョンに合わせる必要があります。詳細なバージョンのマッピングについては、[サポートされる Flink バージョン](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)を参照してください。このトピックでは Flink 1.14.5 を使用し、`flink-sql-connector-mysql-cdc-2.2.0.jar` をダウンロードしています。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)をダウンロードします。バージョンは Flink バージョンと一致する必要があります。

    > flink-connector-starrocks パッケージ `x.x.x_flink-y.yy _ z.zz.jar` には、3つのバージョン番号が含まれています:
    >
    > - `x.x.x` は flink-connector-starrocks のバージョン番号です。
    > - `y.yy` はサポートされる Flink バージョンです。
    > - `z.zz` は Flink でサポートされる Scala バージョンです。Flink バージョンが 1.14.x またはそれ以前の場合、Scala バージョンを含むパッケージをダウンロードする必要があります。
    >
    > このトピックでは Flink 1.14.5 および Scala 2.11 を使用しています。したがって、次のパッケージをダウンロードできます: `1.2.3_flink-14_2.11.jar`。

4. Flink CDC Connector (`flink-sql-connector-mysql-cdc-2.2.0.jar`) および flink-connector-starrocks (`1.2.3_flink-1.14_2.11.jar`) の JAR パッケージを Flink の `lib` ディレクトリに移動します。

    > **ノート**
    >
    > システムで既に Flink クラスターが実行中の場合、Flink クラスターを停止してから再度開始して JAR パッケージを読み込み、検証する必要があります。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. [SMT パッケージ](https://www.starrocks.io/download/community)をダウンロードして展開し、`flink-1.14.5` ディレクトリに配置します。StarRocks は Linux x86 および macOS ARM64 向けの SMT パッケージを提供しています。オペレーティングシステムと CPU に基づいて 1 つを選択できます。

    ```Bash
    # Linux x86 向け
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # macOS ARM64 向け
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### MySQL バイナリログの有効化

MySQL からのデータをリアルタイムで同期するには、システムが MySQL バイナリログ (binlog) からデータを読み取り、データをパースし、StarRocks にデータを同期する必要があります。MySQL バイナリログが有効になっていることを確認してください。

1. MySQL 構成ファイル `my.cnf` (デフォルトパス: `/etc/my.cnf`) を編集して、MySQL バイナリログを有効にします。

    ```Bash
    # MySQL バイナリログを有効にします。
    log_bin = ON
    # Binlog の保存パスを構成します。
    log_bin =/var/lib/mysql/mysql-bin
    # server_id を構成します。
    # MySQL 5.7.3 以降のバージョンでは server_id が構成されていない場合、MySQL サービスを使用できません。
    server_id = 1
    # Binlog 形式を ROW に設定します。
    binlog_format = ROW
    # Binlog ファイルのベース名。各 Binlog ファイルを識別するために識別子が追加されます。
    log_bin_basename =/var/lib/mysql/mysql-bin
    # すべての Binlog ファイルのディレクトリを管理する Binlog ファイルのインデックスファイルです。
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 修正された構成ファイルが有効になるように MySQL を再起動するには、次のコマンドのいずれかを実行します。

    ```Bash```
```markdown
    # MySQLの再起動にサービスを使用します。
    service mysqld restart
    # MySQLを再起動するにはmysqldスクリプトを使用します。
    /etc/init.d/mysqld restart
    ```

3. MySQLに接続し、MySQLのバイナリログが有効化されているかどうかを確認してください。

    ```Plain
    -- MySQLに接続します。
    mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

    -- MySQLのバイナリログが有効化されているかどうかを確認します。
    mysql> SHOW VARIABLES LIKE 'log_bin'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       | ON    |
    +---------------+-------+
    1 row in set (0.00 sec)
    ```

## データベースとテーブルのスキーマを同期する

1. SMTの構成ファイルを編集します。
   SMTの`conf`ディレクトリに移動し、MySQL接続情報、同期するデータベースとテーブルの一致ルール、flink-starrocks-connectorの構成情報などを編集した構成ファイル`config_prod.conf`を開きます。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocksのBE数
    be_num = 3
    # `decimal_v3`はStarRocks-1.18.1以降でサポートされます。
    use_decimal_v3 = true
    # 生成されたDDL SQLを保存するファイル
    output_dir = ./result

    [table-rule.1]
    # プロパティを設定するためのデータベースと一致するパターン
    database = ^demo.*$
    # プロパティを設定するためのテーブルと一致するパターン
    table = ^.*$

    ############################################
    ### Flinkシンクの構成
    ### `connector`、`table-name`、`database-name`は自動生成されます。
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
       - `host`: MySQLサーバーのIPアドレス
       - `port`: MySQLデータベースのポート番号（デフォルトは`3306`）
       - `user`: MySQLデータベースへのアクセス用のユーザー名
       - `password`: ユーザーのパスワード

    - `[table-rule]`: データベースとテーブルの一致ルールおよび対応するflink-connector-starrocksの構成

       - `Database`、`table`: MySQLのデータベースおよびテーブルの名前。正規表現がサポートされています。
       - `flink.starrocks.*`: flink-connector-starrocksの構成情報。詳細な設定情報については[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を参照してください。

       > 異なるテーブルに対して異なるflink-connector-starrocksの構成を使用する必要がある場合、[異なるテーブルに対して異なるflink-connector-starrocks構成を使用する](#use-different-flink-connector-starrocks-configurations-for-different-tables)を参照してください。MySQLのシャーディングから取得した複数のテーブルを同じStarRocksテーブルに同期する必要がある場合は、[MySQLのシャーディング後に複数のテーブルをStarRocksの1つのテーブルに同期する](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)を参照してください。

    - `[other]`: その他の情報
       - `be_num`: StarRocksクラスター内のBE数（このパラメータは、後続のStarRocksテーブル作成時に適切なテーブルトの数を設定するために使用されます。）
       - `use_decimal_v3`: [Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)を有効にするかどうか。Decimal V3を有効にした場合、MySQLのdecimalデータがStarRocksに同期される際にDecimal V3データに変換されます。
       - `output_dir`: 生成されるSQLファイルを保存するパス。SQLファイルはStarRocksでデータベースやテーブルを作成し、FlinkクラスターにFlinkジョブを提出するために使用されます。デフォルトのパスは`./result`であり、デフォルトの設定を保持することをお勧めします。

2. SMTを実行して、MySQLのデータベースとテーブルスキーマを読み込み、構成ファイルに基づいて`./result`ディレクトリにSQLファイルを生成します。`starrocks-create.all.sql`ファイルはStarRocksでデータベースとテーブルを作成するために使用され、`flink-create.all.sql`ファイルはFlinkクラスターにFlinkジョブを提出するために使用されます。

    ```Bash
    # SMTを実行します。
    ./starrocks-migrate-tool

    # resultディレクトリに移動して、このディレクトリ内のファイルを確認します。
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 以下のコマンドを実行して、StarRocksに接続し、`starrocks-create.all.sql`ファイルを実行してStarRocksでデータベースとテーブルを作成します。このSQLファイルのデフォルトのテーブル作成文を使用して、[プライマリキー表](../table_design/table_types/primary_key_table.md)のテーブルを作成することをお勧めします。

    > **注意**
    >
    > ビジネス上のニーズに基づいてテーブル作成文を変更し、プライマリキー表を使用しないテーブルを作成することもできます。ただし、ソースのMySQLデータベースでのDELETE操作は非プライマリキー表に同期されません。このようなテーブルを作成する際には注意が必要です。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    データがFlinkによって処理され、宛先のStarRocksテーブルに書き込まれる前に、ソースと宛先のテーブルスキーマが異なる場合は、テーブル作成文を変更する必要があります。この場合は、宛先テーブルは`product_id`と`product_name`列のみを必要とし、商品の実際の販売のリアルタイムランキングも必要とします。次のテーブル作成文を使用できます。

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
    > v2.5.7以降、StarRocksではテーブルを作成したりパーティションを追加する際にバケット数（BUCKETS）を自動的に設定することができます。バケット数を手動で設定する必要はもはやありません。詳細については[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## データを同期する

Flinkクラスターを実行し、MySQLからStarRocksへのフルおよび増分データを連続して同期するためのFlinkジョブを提出します。

1. Flinkディレクトリに移動し、以下のコマンドを実行して、Flink SQLクライアントで`flink-create.all.sql`ファイルを実行します。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    このSQLファイルは、ダイナミックテーブル`source table`と`sink table`、クエリ文`INSERT INTO SELECT`を定義し、コネクタ、ソースデータベース、宛先データベースを指定します。このファイルを実行すると、FlinkジョブがFlinkクラスターに提出されてデータ同期が開始されます。

    > **注意**
    >
    > - Flinkクラスターが起動していることを確認してください。Flinkクラスターは`flink/bin/start-cluster.sh`を実行して起動できます。
    > - Flinkのバージョンが1.13より前の場合、`flink-create.all.sql`を直接実行することができない場合があります。この場合、このファイル内のSQL文をSQLクライアントのコマンドラインインターフェイス（CLI）で1つずつ実行する必要があります。さらに、`\`文字をエスケープする必要があります。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **データの同期中にデータを処理する**：

    グループ化やJOINなどのデータの処理が必要な場合、データの同期中に`flink-create.all.sql`ファイルを変更することができます。次の例では、COUNT (*)とGROUP BYを実行して商品の販売のリアルタイムランキングを計算します。

    ```Bash
        $ ./bin/sql-client.sh -f flink-create.all.sql
        デフォルトの環境が指定されていません。
        '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'を検索中...見つかりません。
        [INFO] ファイルからSQLを実行中。

        Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
        [INFO] ステートメントが正常に実行されました。
```
        -- MySQLのorderテーブルを基に動的な`source table`を作成します。
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

        -- 動的な`sink table`を作成します。
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

        -- 商品の販売のリアルタイムなランキングを実装し、`source table`のデータ変更を反映するように動的に`sink table`を更新します。
        Flink SQL> 
        INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
        [INFO] クラスターにSQL更新ステートメントを送信しています...
        [INFO] SQL更新ステートメントがクラスターに正常に送信されました:
        ジョブ ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

    支払い日が2021年12月21日より後のデータのようにデータの一部のみを同期する必要がある場合は、`INSERT INTO SELECT`の`WHERE`句を使用して`pay_dt > '2021-12-21'`のようなフィルタ条件を設定できます。この条件を満たさないデータはStarRocksに同期されません。

    以下の結果が返される場合、Flinkジョブは完全および増分同期のために送信されました。

    ```SQL
    [INFO] クラスターにSQL更新ステートメントを送信しています...
    [INFO] SQL更新ステートメントがクラスターに正常に送信されました:
    ジョブ ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

2. [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を使用するか、Flink SQLクライアントで`bin/flink list -running`コマンドを実行して、Flinkクラスターで実行中のFlinkジョブとジョブIDを表示できます。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        応答を待っています...
        ------------------ 実行/再起動中のジョブ -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
        --------------------------------------------------------------
    ```

    > **注意事項**
    >
    > ジョブが異常な場合は、Flink WebUIを使用するか、Flink 1.14.5の`/log`ディレクトリのログファイルを表示してトラブルシューティングを行うことができます。

## FAQ

### 異なるflink-connector-starrocksの設定を使用して異なるテーブルを使用する

データソースの一部のテーブルが頻繁に更新される場合に、flink-connector-starrocksの読み込み速度を加速したい場合は、SMT構成ファイル`config_prod.conf`で各テーブルに対して異なるflink-connector-starrocksの構成を設定する必要があります。

```Bash
[table-rule.1]
# プロパティを設定するためのデータベースのパターン
database = ^order.*$
# プロパティを設定するためのテーブルのパターン
table = ^.*$

############################################
### Flinkシンク構成
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
### Flinkシンク構成
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

### MySQLのシャーディング後、複数のテーブルをStarRocksの1つのテーブルに同期する

シャーディング後、1つのMySQLテーブルのデータが複数のテーブルに分割されるか、複数のデータベースに分散される可能性があります。各テーブルのスキーマは同じです。この場合、`[table-rule]`を設定してこれらのテーブルを1つのStarRocksテーブルに同期することができます。たとえば、MySQLには`edu_db_1`と`edu_db_2`という2つのデータベースがあり、それぞれに2つのテーブル`course_1`と`course_2`があり、すべてのテーブルのスキーマが同じである場合、次の`[table-rule]`構成を使用してすべてのテーブルを1つのStarRocksテーブルに同期できます。

> **注意事項**
>
> StarRocksテーブルの名前はデフォルトで`course__auto_shard`になります。別の名前を使用する必要がある場合は、SQLファイル`starrocks-create.all.sql`および`flink-create.all.sql`で変更できます。

```Bash
[table-rule.1]
# プロパティを設定するためのデータベースのパターン
database = ^edu_db_[0-9]*$
# プロパティを設定するためのテーブルのパターン
table = ^course_[0-9]*$

############################################
### Flinkシンク構成
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

前述の例のデータはCSV形式でインポートされます。適切な区切り文字を選択できない場合は、`[table-rule]`の`flink.starrocks.*`のパラメータを置換する必要があります。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

以下のパラメータが渡された後、データはJSON形式でインポートされます。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意事項**
>
> この方法は、読み込み速度がわずかに低下します。

### 複数のINSERT INTOステートメントを1つのFlinkジョブとして実行する

`flink-create.all.sql`ファイルで[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)構文を使用して、複数のINSERT INTOステートメントを1つのFlinkジョブとして実行できます。これにより、複数のステートメントが多くのFlinkジョブリソースを占有するのを防ぎ、複数のクエリを効率的に実行する効率が向上します。

> **注意事項**
>
```SQL
-- 1.13以降、FlinkはSTATEMENT SET構文をサポートしています。

1. `result/flink-create.all.sql`ファイルを開きます。

2. ファイル内のSQLステートメントを修正します。すべてのINSERT INTOステートメントをファイルの最後に移動します。最初のINSERT INTOステートメントの前に`EXECUTE STATEMENT SET BEGIN`を配置し、最後のINSERT INTOステートメントの後に`END;`を配置します。

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