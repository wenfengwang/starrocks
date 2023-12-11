---
displayed_sidebar: "Japanese"
---

# MySQL からのリアルタイム同期

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、数秒以内にMySQLからのリアルタイムデータ同期をサポートし、スケールで超低レイテンシのリアルタイムアナリティクスを提供し、ユーザーがリアルタイムデータをクエリできるようにします。

このチュートリアルでは、リアルタイムアナリティクスをビジネスとユーザーにもたらす方法を学ぶのに役立ちます。次のツールを使用して、MySQLからStarRocksへのデータ同期をリアルタイムで行う方法を示します: StarRocks Migration Tools (SMT)、Flink、Flink CDC Connector、flink-starrocks-connector。

<InsertPrivNote />

## 仕組み

次の図は、全体の同期プロセスを示しています。

![img](../assets/4.9.2.png)

MySQLからのリアルタイム同期は、データベース＆テーブルスキーマの同期とデータの同期の2つのステージで実装されます。まず、SMTはMySQLのデータベース＆テーブルスキーマをStarRocksのテーブル作成ステートメントに変換します。そして、FlinkクラスタはFlinkジョブを実行して、完全および増分のMySQLデータをStarRocksに同期します。

> **注意**
>
> 同期プロセスでは、厳密に一度だけのセマンティクスが保証されます。

**同期プロセス**:

1. データベース＆テーブルスキーマの同期。

   SMTは、同期するMySQLデータベース＆テーブルのスキーマを読み取り、StarRocksの宛先データベース＆テーブルを作成するためのSQLファイルを生成します。この操作は、SMTの構成ファイルに含まれるMySQLとStarRocksの情報に基づいて行われます。

2. データの同期。

   a. Flink SQLクライアントは、データのローディングステートメント `INSERT INTO SELECT` を実行して、1つまたは複数のFlinkジョブをFlinkクラスタに送信します。

   b. Flinkクラスタは、データを取得するためにFlinkジョブを実行します。[Flink CDC Connector](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md) はまずソースデータベースから完全な履歴データを読み取り、シームレスに増分読み取りに切り替え、そしてデータを flink-starrocks-connector に送信します。

   c. flink-starrocks-connector はデータをミニバッチで蓄積し、それぞれのデータバッチをStarRocksに同期します。

> **注意**
>
> MySQLでのデータ定義言語 (DDL) 操作はStarRocksに同期できません。データ操作言語 (DML) の操作のみが同期されます。

## シナリオ

MySQLからのリアルタイム同期は、データが常に変更される広範なユースケースで使用されます。リアルタイムランキングの商品販売という現実のユースケースを例に取ります。

FlinkはMySQLの元の注文テーブルを使用して商品販売のリアルタイムランキングを計算し、それをStarRocksのプライマリキーテーブルにリアルタイムで同期します。ユーザーは、ビジュアライゼーションツールをStarRocksに接続してリアルタイムでランキングを表示し、オンデマンドの運用洞察を得ることができます。

## 準備

### 同期ツールのダウンロードとインストール

MySQLからデータを同期するには、次のツールをインストールする必要があります: SMT、Flink、Flink CDCコネクタ、および flink-starrocks-connector。

1. Flinkをダウンロードしてインストールし、Flinkクラスタを起動します。または、[Flink公式ドキュメント](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/) の手順に従って実行できます。

   a. Flinkを実行する前に、オペレーティングシステムにJava 8またはJava 11をインストールしてください。次のコマンドを実行してインストールされたJavaバージョンを確認できます。

    ```Bash
        # Javaのバージョンを表示
        java -version
        
        # 以下の出力が返された場合、Java 8がインストールされています。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. [Flinkのインストールパッケージ](https://flink.apache.org/downloads.html) をダウンロードして展開します。Flink 1.14以降を使用することをお勧めします。最小許可バージョンはFlink 1.11です。ここではFlink 1.14.5を使用します。

   ```Bash
      # Flinkのダウンロード
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Flinkの展開  
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Flinkディレクトリに移動
      cd flink-1.14.5
    ```

   c. Flinkクラスタを起動します。

   ```Bash
      # Flinkクラスタを起動
      ./bin/start-cluster.sh
      
      # 以下の出力が返された場合、Flinkクラスタが起動されています。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. [Flink CDCコネクタ](https://github.com/ververica/flink-cdc-connectors/releases) をダウンロードします。ここではMySQLをデータソースとして使用し、`flink-sql-connector-mysql-cdc-x.x.x.jar` をダウンロードします。コネクタのバージョンは[Flink](https://github.com/ververica/flink-cdc-connectors/releases) のバージョンと一致させる必要があります。詳細なバージョンマッピングについては、[サポートされているFlinkバージョン](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions) を参照してください。ここでは、Flink 1.14.5を使用し、`flink-sql-connector-mysql-cdc-2.2.0.jar` をダウンロードします。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks) をダウンロードします。バージョンはFlinkバージョンと一致する必要があります。

    > flink-connector-starrocksパッケージ `x.x.x_flink-y.yy _ z.zz.jar` には3つのバージョン番号が含まれています:
    >
    > - `x.x.x` はflink-connector-starrocksのバージョン番号です。
    > - `y.yy` はサポートされているFlinkバージョンです。
    > - `z.zz` はFlinkがサポートするScalaバージョンです。Flinkのバージョンが1.14.xまたはそれ以前の場合、Scalaバージョンのパッケージをダウンロードする必要があります。
    >
    > ここでは、Flink 1.14.5およびScala 2.11を使用しているため、次のパッケージをダウンロードします: `1.2.3_flink-14_2.11.jar`。

4. Flink CDCコネクタ(`flink-sql-connector-mysql-cdc-2.2.0.jar`)およびflink-connector-starrocks(`1.2.3_flink-1.14_2.11.jar`)のJARパッケージをFlinkの`lib`ディレクトリに移動します。

    > **注意**
    >
    > システムで既にFlinkクラスタが実行されている場合は、Flinkクラスタを停止してからJARパッケージをロードおよび検証するために再起動する必要があります。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. [SMTパッケージ](https://www.starrocks.io/download/community)をダウンロードして展開し、`flink-1.14.5`ディレクトリに配置します。StarRocksはLinux x86およびmacos ARM64向けのSMTパッケージを提供しており、オペレーティングシステムとCPUに基づいて選択できます。

    ```Bash
    # Linux x86向け
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # macOS ARM64向け
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### MySQLバイナリログの有効化

MySQLからデータをリアルタイムで同期するためには、システムがMySQLバイナリログ (binlog) からデータを読み取り、データを解析し、それをStarRocksに同期する必要があります。MySQLバイナリログが有効になっていることを確認してください。

1. MySQLの設定ファイル`my.cnf` (デフォルトパス: `/etc/my.cnf`) を編集して、MySQLバイナリログを有効にします。

    ```Bash
    # MySQLバイナリログを有効にします。
    log_bin = ON
    # Binlogの保存パスを設定します。
    log_bin =/var/lib/mysql/mysql-bin
    # server_idを設定します。
    # MySQL 5.7.3以降の場合、server_idを設定しないとMySQLサービスを使用できません。 
    server_id = 1
    # Binlog形式をROWに設定します。
    binlog_format = ROW
    # Binlogファイルのベース名です。識別子が追加され、各Binlogファイルを識別します。
    log_bin_basename =/var/lib/mysql/mysql-bin
    # Binlogファイルのインデックスファイルです。すべてのBinlogファイルのディレクトリを管理します。  
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 修正した設定ファイルが有効になるように、MySQLを再起動するために次のコマンドを実行します。

    ```Bash
    # MySQLの再起動にはサービスを使用します。
    service mysqld restart
    # MySQLの再起動にはmysqldスクリプトを使用します。
    /etc/init.d/mysqld restart

3. MySQLに接続し、MySQLバイナリログが有効化されているかどうかを確認してください。

    ```Plain
    -- MySQLに接続します。
    mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

    -- MySQLバイナリログが有効化されているかどうかを確認します。
    mysql> SHOW VARIABLES LIKE 'log_bin'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       | ON    |
    +---------------+-------+
    1 row in set (0.00 sec)
    ```

## データベースとテーブルのスキーマを同期

1. SMT構成ファイルを編集します。
   SMTの `conf` ディレクトリに移動し、 `config_prod.conf` の構成ファイルを編集します。これにはMySQL接続情報、同期するデータベースとテーブルのマッチングルール、flink-starrocks-connectorの構成情報などが含まれます。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocksのBE数
    be_num = 3
    # `decimal_v3` はStarRocks-1.18.1以降でサポートされます。
    use_decimal_v3 = true
    # 生成されたDDL SQLを保存するファイル
    output_dir = ./result

    [table-rule.1]
    # プロパティを設定するためのデータベースの一致パターン
    database = ^demo.*$
    # プロパティを設定するためのテーブルの一致パターン
    table = ^.*$

    ############################################
    ### Flinkシンク設定
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

    - `[db]`: MySQL接続情報。
       - `host`: MySQLサーバーのIPアドレス。
       - `port`: MySQLデータベースのポート番号。デフォルトは`3306`。
       - `user`: MySQLデータベースにアクセスするためのユーザー名。
       - `password`: ユーザーのパスワード。

    - `[table-rule]`: データベースとテーブルの一致ルールとそれに対応するflink-connector-starrocksの構成。

       - `Datatabase`、 `table`: MySQLのデータベースおよびテーブルの名前。正規表現がサポートされています。
       - `flink.starrocks.*`: flink-connector-starrocksの構成情報。その他の構成と情報については、[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を参照してください。

       > 異なるテーブルに異なるflink-connector-starrocksの構成を使用する場合は、[異なるテーブルに異なるflink-connector-starrocksの構成を使用する](#use-different-flink-connector-starrocks-configurations-for-different-tables)を参照してください。 MySQLのシャーディングから取得した複数のテーブルを同じStarRocksテーブルに同期する必要がある場合は、[MySQLのシャーディング後の複数のテーブルをStarRocksの1つのテーブルに同期](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)を参照してください。

    - `[other]`: その他の情報
       - `be_num`: StarRocksクラスターのBE数（このパラメータは後続のStarRocksテーブル作成における適切なタブレット数の設定に使用されます）。
       - `use_decimal_v3`: [Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)を有効にするかどうか。Decimal V3を有効にすると、MySQLのdecimalデータがStarRocksにデータを同期する際にDecimal V3データに変換されます。
       - `output_dir`: 生成されるSQLファイルを保存するパス。SQLファイルはStarRocksでデータベースとテーブルを作成し、FlinkジョブをFlinkクラスターに送信するために使用されます。デフォルトのパスは `./result` であり、デフォルトの設定を維持することを推奨します。

2. SMTを実行し、MySQLのデータベースとテーブルのスキーマを読み取り、`./result`ディレクトリにSQLファイルを生成します。`starrocks-create.all.sql`ファイルはStarRocksにデータベースとテーブルを作成するために使用されます。`flink-create.all.sql`ファイルはFlinkジョブをFlinkクラスターに送信するために使用されます。

    ```Bash
    # SMTを実行します。
    ./starrocks-migrate-tool

    # resultディレクトリに移動し、このディレクトリ内のファイルを確認します。
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 次のコマンドを実行してStarRocksに接続し、 `starrocks-create.all.sql`ファイルを実行してStarRocksでデータベースとテーブルを作成します。StarRocksの[プライマリキー表](../table_design/table_types/primary_key_table.md)のデフォルトのテーブル作成ステートメントを使用することをお勧めします。

    > **注意**
    >
    > ビジネスニーズに基づいてテーブル作成ステートメントを変更してプライマリキー表を使用しないテーブルを作成することもできますが、ソースのMySQLデータベースでのDELETE操作は非プライマリキーのテーブルに同期されません。このようなテーブルを作成する際には注意が必要です。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    データをFlinkで処理してから宛先のStarRocksテーブルに書き込む必要がある場合、ソースと宛先のテーブルのスキーマは異なります。この場合、テーブル作成ステートメントを変更する必要があります。この例では、宛先のテーブルには`product_id`と`product_name`の列のみが必要で、商品販売のリアルタイムランキングが必要です。次のテーブル作成ステートメントを使用できます。

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
    > v2.5.7以降、StarRocksはテーブルの作成またはパーティションの追加時にバケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はもはやありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## データの同期

Flinkクラスターを実行し、MySQLからStarRocksに完全および増分データを継続的に同期するためにFlinkジョブを送信します。

1. Flinkディレクトリに移動し、次のコマンドを実行してFlink SQLクライアントで `flink-create.all.sql`ファイルを実行します。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    このSQLファイルは動的テーブル `source table` および `sink table` を定義し、クエリステートメント `INSERT INTO SELECT` を実行し、コネクタ、ソースデータベース、宛先データベースを指定します。このファイルを実行すると、FlinkジョブがFlinkクラスターに送信され、データ同期が開始されます。

    > **注意**
    >
    > - Flinkクラスターが起動していることを確認してください。 Flinkクラスターを起動するには、 `flink/bin/start-cluster.sh` を実行してください。
    > - Flinkのバージョンが1.13より前の場合、SQLファイル `flink-create.all.sql` を直接実行できない場合があります。この場合、このファイル内のSQLステートメントをSQLクライアントのコマンドラインインターフェース（CLI）で1つずつ実行する必要があります。また、`\`文字をエスケープする必要があります。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **同期中のデータ処理**:

    データを同期する際にデータを処理する必要がある場合、データを`GROUP BY`や`JOIN`を実行する必要がある場合、`flink-create.all.sql`ファイルを変更できます。次の例では、COUNT (*)およびGROUP BYを実行して商品販売のリアルタイムランキングを計算します。

    ```Bash
        $ ./bin/sql-client.sh -f flink-create.all.sql
        No default environment is specified.
        Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
        [INFO] Executing SQL from file.

        Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
        [INFO] Execute statement succeed.
```
```japanese
-- MySQLのorderテーブルを基にして、動的な`ソーステーブル`を作成します。
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

-- 動的な`シンクテーブル`を作成します。
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

-- 商品の実績をリアルタイムでランキングする場合、`シンクテーブル`は`ソーステーブル`のデータ変更を反映するように動的に更新されます。
Flink SQL> 
INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
[INFO] SQL更新ステートメントをクラスターに送信しています...
[INFO] SQL更新ステートメントがクラスターに正常に送信されました:
ジョブID: 5ae005c4b3425d8bb13fe660260a35da
```

「支払い期日が2021年12月21日よりも後のデータなど、データの一部のみを同期する必要がある場合は、`INSERT INTO SELECT`句内で`WHERE`句を使用してフィルタ条件を設定することができます。例えば、`WHERE pay_dt > '2021-12-21'`のような条件といった形で、この条件に合わないデータはStarRocksに同期されません。

以下の結果が返された場合、Flinkジョブが完全および増分同期のために送信されています。

```SQL
[INFO] SQL更新ステートメントをクラスターに送信しています...
[INFO] SQL更新ステートメントがクラスターに正常に送信されました:
ジョブID: 5ae005c4b3425d8bb13fe660260a35da
```

2. [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)を使用するか、Flink SQLクライアントで`bin/flink list -running`コマンドを実行して、Flinkクラスターで実行中のFlinkジョブとジョブIDを表示できます。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        応答を待っています...
        ------------------ 実行中/再起動中のジョブ -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
        --------------------------------------------------------------
    ```

    > **注意**
    >
    > ジョブが異常である場合は、Flink WebUIを使用するか、Flink 1.14.5の`/log`ディレクトリ内のログファイルを表示してトラブルシューティングを実行できます。

## FAQ

### 異なるflink-connector-starrocks設定を異なるテーブルに使用する

データソースの一部のテーブルが頻繁に更新され、flink-connector-starrocksの読み込み速度を高速化したい場合は、SMT設定ファイル`config_prod.conf`で各テーブルに対して別々のflink-connector-starrocks設定を行う必要があります。

```Bash
[table-rule.1]
# プロパティを設定するデータベースにマッチするパターン
database = ^order.*$
# プロパティを設定するテーブルにマッチするパターン
table = ^.*$

############################################
### Flinkシンク設定
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
# プロパティを設定するデータベースにマッチするパターン
database = ^order2.*$
# プロパティを設定するテーブルにマッチするパターン
table = ^.*$

############################################
### Flinkシンク設定
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

### StarRocksへのMySQL分散後の複数のテーブルの同期

分散後、1つのMySQLテーブルのデータが複数のテーブルに分割されるか、さらに複数のデータベースに分散される可能性があります。すべてのテーブルのスキーマが同じである場合、`[table-rule]`をセットアップして、これらのテーブルを1つのStarRocksテーブルに同期することが可能です。例えば、MySQLには`edu_db_1`と`edu_db_2`の2つのデータベースがあり、それぞれ`course_1`と`course_2`の2つのテーブルがあり、すべてのテーブルのスキーマは同じであるとします。次のような`[table-rule]`設定で、これらのテーブルをすべて1つのStarRocksテーブルに同期できます。

> **注意**
>
> StarRocksテーブルのデフォルトの名前は`course__auto_shard`です。別の名前を使用する必要がある場合は、SQLファイル`starrocks-create.all.sql`および`flink-create.all.sql`で変更できます。

```Bash
[table-rule.1]
# プロパティを設定するデータベースにマッチするパターン
database = ^edu_db_[0-9]*$
# プロパティを設定するテーブルにマッチするパターン
table = ^course_[0-9]*$

############################################
### Flinkシンク設定
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

### JSON形式でデータをインポート

前述の例ではCSV形式でデータをインポートしています。適切な区切り記号を選択できない場合は、`[table-rule]`内の`flink.starrocks.*`の以下のパラメータを置換する必要があります。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

以下のパラメータを渡した後は、JSON形式でデータがインポートされます。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> この方法では読み込み速度がわずかに遅くなります。

### 複数のINSERT INTOステートメントを1つのFlinkジョブとして実行する

`flink-create.all.sql`ファイル内で[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)構文を使用して、複数のINSERT INTOステートメントを1つのFlinkジョブとして実行できます。これにより、複数のステートメントが多くのFlinkジョブリソースを占有することを防ぎ、複数のクエリを効率的に実行できます。

> **注意**
>
```SQL
1.13 から、Flink は STATEMENT SET 構文をサポートしています。

1. `result/flink-create.all.sql` ファイルを開いてください。

2. ファイル内の SQL ステートメントを修正します。すべての INSERT INTO ステートメントをファイルの末尾に移動します。最初の INSERT INTO ステートメントの前に `EXECUTE STATEMENT SET BEGIN` を配置し、最後の INSERT INTO ステートメントの後に `END;` を配置します。

> **注意**
>
> CREATE DATABASE および CREATE TABLE の位置は変更しません。
```