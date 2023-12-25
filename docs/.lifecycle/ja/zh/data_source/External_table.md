---
displayed_sidebar: Chinese
---

# (廃止予定) 外部テーブル

StarRocks は外部テーブル (External Table) の形式で他のデータソースに接続することをサポートしています。外部テーブルとは、他のデータソースに保存されているデータテーブルのことで、StarRocks はテーブルに対応するメタデータのみを保存し、直接外部テーブルが存在するデータソースにクエリを発行します。現在 StarRocks がサポートしているサードパーティのデータソースには、MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg、Apache Hudi が含まれます。**StarRocks データソースについては、現段階では Insert による書き込みのみをサポートし、読み取りはサポートしていません。他のデータソースについては、現段階では読み取りのみをサポートし、書き込みはサポートしていません**。

> **注意**
>
> * バージョン 3.0 以降、Hive、Iceberg、Hudi データソースのクエリには Catalog の使用を推奨します。詳細は [Hive catalog](../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../data_source/catalog/hudi_catalog.md) を参照してください。
> * バージョン 3.1 以降、MySQL、PostgreSQL のクエリには [JDBC catalog](../data_source/catalog/jdbc_catalog.md) の使用を推奨し、Elasticsearch のクエリには [Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md) の使用を推奨します。

バージョン 2.5 から、外部データソースのクエリ時に Data Cache をサポートし、ホットデータのクエリパフォーマンスを向上させます。詳細は [Data Cache](data_cache.md) を参照してください。

## StarRocks 外部テーブル

バージョン 1.19 から、StarRocks は外部テーブルを介して別の StarRocks クラスタのテーブルにデータを書き込むことをサポートしています。これにより、ユーザーの読み書き分離のニーズを満たし、より良いリソースの分離を提供します。ユーザーはまずターゲットクラスタにターゲットテーブルを作成し、次にソース StarRocks クラスタにスキーマ情報が一致する外部テーブルを作成し、プロパティでターゲットクラスタとテーブルの情報を指定する必要があります。

INSERT INTO を使用して StarRocks 外部テーブルにデータを書き込むことで、ソースクラスタのデータをターゲットクラスタに書き込むことができます。この機能を利用して、以下の目的を実現できます：

* クラスタ間のデータ同期。
* 読み書き分離。ソースクラスタにデータを書き込み、ソースクラスタのデータ変更をターゲットクラスタに同期し、ターゲットクラスタがクエリサービスを提供します。

以下はターゲットテーブルと外部テーブルを作成する例です：

~~~sql
# ターゲットクラスタで実行
CREATE TABLE t
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=olap
DISTRIBUTED BY HASH(k1);

# 外部テーブルクラスタで実行
CREATE EXTERNAL TABLE external_t
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=olap
DISTRIBUTED BY HASH(k1)
PROPERTIES
(
    "host" = "127.0.0.1",
    "port" = "9020",
    "user" = "user",
    "password" = "passwd",
    "database" = "db_test",
    "table" = "t"
);

# StarRocks 外部テーブルにデータを書き込み、ソースクラスタのデータをターゲットクラスタに書き込む。本番環境では第二の方法を推奨します。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');

insert into external_t select * from other_table;
~~~

その中で：

* **EXTERNAL**：このキーワードは作成されるのが StarRocks 外部テーブルであることを指定します。
* **host**：このプロパティはターゲットテーブルが属する StarRocks クラスタの Leader FE の IP アドレスを記述します。
* **port**：このプロパティはターゲットテーブルが属する StarRocks クラスタの FE の RPC アクセスポートを記述します。

  :::note

  外部テーブルが属するクラスタがターゲットテーブルが属する StarRocks クラスタに正常にアクセスできるようにするためには、以下のポートへのアクセスを許可するネットワークポリシーとファイアウォールの設定を確認する必要があります：

  * FE の RPC アクセスポートは、設定ファイル **fe/fe.conf** の `rpc_port` の設定値を参照してください。デフォルトは `9020` です。
  * BE の bRPC アクセスポートは、設定ファイル **be/be.conf** の `brpc_port` の設定値を参照してください。デフォルトは `8060` です。

  :::

* **user**：このプロパティはターゲットテーブルが属する StarRocks クラスタのアクセスユーザー名を記述します。
* **password**：このプロパティはターゲットテーブルが属する StarRocks クラスタのアクセスパスワードを記述します。
* **database**：このプロパティはターゲットテーブルが属するデータベース名を記述します。
* **table**：このプロパティはターゲットテーブル名を記述します。

現在 StarRocks 外部テーブルの使用には以下の制限があります：

* 外部テーブルでは insert into と show create table の操作のみを実行でき、他のデータ書き込み方法はサポートされておらず、クエリや DDL もサポートされていません。
* 外部テーブルの作成構文は通常のテーブルの作成と同じですが、列名などの情報は対応するターゲットテーブルと一致させてください。
* 外部テーブルは定期的にターゲットテーブルからメタデータを同期します（同期周期は 10 秒）。ターゲットテーブルで実行された DDL 操作は、外部テーブルに反映されるまでに時間がかかる場合があります。

## JDBC 対応のデータベースの外部テーブル

バージョン 2.3.0 から、StarRocks は JDBC 対応のデータベースを外部テーブルを介してクエリすることをサポートしています。データを StarRocks にインポートすることなく、これらのデータベースの高速分析を実現できます。この記事では、StarRocks で外部テーブルを作成し、JDBC 対応のデータベースのデータをクエリする方法について説明します。

### 前提条件

JDBC 外部テーブルを使用する場合、FE および BE ノードは JDBC ドライバをダウンロードするため、FE および BE ノードが配置されているマシンは JDBC ドライバ JAR パッケージをダウンロードするための URL にアクセスできる必要があります。この URL は JDBC リソースを作成する際の `driver_url` 設定項目で指定されます。

### JDBC リソースの作成と管理

#### JDBC リソースの作成

JDBC リソースを事前に StarRocks に作成し、データベースの関連接続情報を管理する必要があります。ここで言うデータベースは JDBC をサポートするデータベースを指し、以下では「ターゲットデータベース」と呼びます。リソースを作成した後、そのリソースを使用して外部テーブルを作成できます。

例えば、ターゲットデータベースが PostgreSQL の場合、以下のステートメントを実行して `jdbc0` という名前の JDBC リソースを作成し、PostgreSQL にアクセスできます：

~~~SQL
CREATE EXTERNAL RESOURCE jdbc0
PROPERTIES (
    "type" = "jdbc",
    "user" = "postgres",
    "password" = "changeme",
    "jdbc_uri" = "jdbc:postgresql://127.0.0.1:5432/jdbc_test",
    "driver_url" = "https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar",
    "driver_class" = "org.postgresql.Driver"
);
~~~

`PROPERTIES` の必須設定項目：

* `type`：リソースのタイプで、固定値は `jdbc` です。

* `user`：ターゲットデータベースのユーザー名。

* `password`：ターゲットデータベースのユーザーログインパスワード。

* `jdbc_uri`：JDBC ドライバがターゲットデータベースに接続するための URI で、ターゲットデータベースの URI の構文を満たす必要があります。一般的なターゲットデータベースの URI については、[MySQL](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html)、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/use/#connecting-to-the-database)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16) の公式ドキュメントを参照してください。

    > **説明**
    >
    > ターゲットデータベース URI には、上の例の `jdbc_test` のように具体的なデータベース名を指定する必要があります。

* `driver_url`：JDBC ドライバ JAR パッケージをダウンロードするための URL で、HTTP プロトコルまたは file プロトコルを使用できます。例えば `https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`、`file:///home/disk1/postgresql-42.3.3.jar` です。

    > **説明**
    >
    > 異なるターゲットデータベースは異なる JDBC ドライバを使用しており、他のデータベースの JDBC ドライバを使用すると互換性の問題が発生する可能性があります。ターゲットデータベースの公式ウェブサイトにアクセスし、サポートされている JDBC ドライバを検索して使用することをお勧めします。一般的なターゲットデータベースの JDBC ドライバのダウンロードアドレスについては、[MySQL](https://dev.mysql.com/downloads/connector/j/)、[Oracle](https://www.oracle.com/database/technologies/maven-central-guide.html)、[PostgreSQL](https://jdbc.postgresql.org/download/)、[SQL Server](https://learn.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver16) を参照してください。

* `driver_class`：JDBC ドライバのクラス名。

  以下は一般的な JDBC ドライバのクラス名を列挙しています：

  * MySQL: com.mysql.jdbc.Driver（MySQL 5.x 以下のバージョン）、com.mysql.cj.jdbc.Driver（MySQL 8.x 以上のバージョン）
  * SQL Server：com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL：org.postgresql.Driver

リソースを作成する際、FE は `driver_url` を通じて JDBC ドライバ JAR パッケージをダウンロードし、checksum を生成して保存します。これは BE がダウンロードした JDBC ドライバ JAR パッケージの正確性を検証するために使用されます。

> **説明**
>
> JDBC ドライバのダウンロードに失敗すると、リソースの作成も失敗します。

BE ノードが初めて JDBC 外部テーブルをクエリする際、該当する JDBC ドライバ JAR パッケージがマシン上に存在しないことが判明した場合、`driver_url` を通じてダウンロードを行います。すべての JDBC ドライバ JAR パッケージは **`${STARROCKS_HOME}/lib/jdbc_drivers`** ディレクトリに保存されます。

#### JDBC リソースの表示

以下のステートメントを実行して、StarRocks にあるすべての JDBC リソースを表示します：

> **説明**
>
> `ResourceType` 列は `jdbc` です。

~~~SQL
SHOW RESOURCES;
~~~

#### JDBC リソースの削除

以下のステートメントを実行して、`jdbc0` という名前の JDBC リソースを削除します：

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> **説明**
>

> JDBCリソースの削除は、そのJDBCリソースを使用して作成されたJDBC外部テーブルを使用不可にしますが、対象データベースのデータは失われません。StarRocksを使用して対象データベースのデータを引き続きクエリする必要がある場合は、JDBCリソースとJDBC外部テーブルを再作成できます。

### データベースの作成

以下のステートメントを実行して、StarRocksに`jdbc_test`という名前のデータベースを作成し、使用します：

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> **説明**
>
> データベース名は対象データベースの名前と一致する必要はありません。

### JDBC外部テーブルの作成

以下のステートメントを実行して、`jdbc_test`データベースに`jdbc_tbl`という名前のJDBC外部テーブルを作成します：

~~~SQL
CREATE EXTERNAL TABLE jdbc_tbl (
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=jdbc 
PROPERTIES (
    "resource" = "jdbc0",
    "table" = "dest_tbl"
);
~~~

`PROPERTIES`の設定項目：

* `resource`：使用するJDBCリソースの名前。必須項目です。

* `table`：対象データベースのテーブル名。必須項目です。

サポートされるデータ型とStarRocksのデータ型とのマッピング関係については、[データ型マッピング](#データ型マッピング)を参照してください。

> **説明**
>
> * インデックスはサポートされていません。
> * PARTITION BYやDISTRIBUTED BYを使用してデータ分布ルールを指定することはサポートされていません。

### JDBC外部テーブルのクエリ

JDBC外部テーブルをクエリする前に、Pipelineエンジンを有効にする必要があります。

> **説明**
>
> すでにPipelineエンジンが有効になっている場合は、このステップをスキップできます。

~~~SQL
set enable_pipeline_engine=true;
~~~

以下のステートメントを実行して、JDBC外部テーブルを介して対象データベースのデータをクエリします：

~~~SQL
select * from jdbc_tbl;
~~~

StarRocksは対象テーブルに対する述語プッシュダウンをサポートし、フィルタ条件を対象テーブルにプッシュして実行させ、できるだけデータソースに近いところで実行させることで、クエリパフォーマンスを向上させます。現在サポートされているプッシュダウン演算子には、二項比較演算子（`>`、`>=`、`=`、`<`、`<=`）、`IN`、`IS NULL`、`BETWEEN ... AND ...`が含まれますが、関数のプッシュダウンはサポートされていません。

### データ型マッピング

現在、対象データベースの数値、文字列、時間、日付などの基本的なデータ型のみのクエリがサポートされています。対象データベースのデータがStarRocksのデータ型の表現範囲を超える場合、クエリはエラーになります。

以下に、対象データベースMySQL、Oracle、PostgreSQL、SQL Serverを例に挙げ、サポートされているクエリデータ型とStarRocksのデータ型とのマッピング関係を説明します。

#### 対象データベースがMySQLの場合

| MySQL        | StarRocks |
| ------------ | --------- |
| BOOLEAN      | BOOLEAN   |
| TINYINT      | TINYINT   |
| SMALLINT     | SMALLINT  |
| MEDIUMINT    | INT       |
| BIGINT       | BIGINT    |
| FLOAT        | FLOAT     |
| DOUBLE       | DOUBLE    |
| DECIMAL      | DECIMAL   |
| CHAR         | CHAR      |
| VARCHAR      | VARCHAR   |
| DATE         | DATE      |
| DATETIME     | DATETIME  |

#### 対象データベースがOracleの場合

| Oracle          | StarRocks |
| --------------- | --------- |
| CHAR            | CHAR      |
| VARCHAR/VARCHAR2 | VARCHAR   |
| DATE            | DATE      |
| SMALLINT        | SMALLINT  |
| INT             | INT       |
| DATE            | DATETIME  |
| NUMBER          | DECIMAL   |

#### 対象データベースがPostgreSQLの場合

| PostgreSQL          | StarRocks |
| ------------------- | --------- |
| SMALLINT/SERIAL     | SMALLINT  |
| INTEGER/SERIAL      | INT       |
| BIGINT/SERIAL       | BIGINT    |
| BOOLEAN             | BOOLEAN   |
| REAL                | FLOAT     |
| DOUBLE PRECISION    | DOUBLE    |
| DECIMAL             | DECIMAL   |
| TIMESTAMP           | DATETIME  |
| DATE                | DATE      |
| CHAR                | CHAR      |
| VARCHAR             | VARCHAR   |
| TEXT                | VARCHAR   |

#### 対象データベースがSQL Serverの場合

| SQL Server        | StarRocks |
| ----------------- | --------- |
| BIT               | BOOLEAN   |
| TINYINT           | TINYINT   |
| SMALLINT          | SMALLINT  |
| INT               | INT       |
| BIGINT            | BIGINT    |
| FLOAT             | FLOAT/DOUBLE     |
| REAL              | FLOAT     |
| DECIMAL/NUMERIC   | DECIMAL   |
| CHAR              | CHAR      |
| VARCHAR           | VARCHAR   |
| DATE              | DATE      |
| DATETIME/DATETIME2 | DATETIME  |

### 使用制限

* JDBC外部テーブルの作成時には、インデックスはサポートされておらず、PARTITION BYやDISTRIBUTED BYを使用してデータ分布ルールを指定することもサポートされていません。
* JDBC外部テーブルのクエリ時には、関数のプッシュダウンはサポートされていません。

## (非推奨)Elasticsearch外部テーブル

Elasticsearchのデータをクエリするには、StarRocksにElasticsearch外部テーブルを作成し、クエリ対象のElasticsearchテーブルとのマッピングを行う必要があります。StarRocksとElasticsearchは現在人気のある分析システムです。StarRocksは大規模分散計算を得意とし、Elasticsearchの外部テーブルを介してクエリすることをサポートしています。Elasticsearchは全文検索を得意としています。両者の組み合わせにより、より完全なOLAPソリューションを提供します。

### テーブル作成例

#### 構文

~~~sql
CREATE EXTERNAL TABLE elastic_search_external_table
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=ELASTICSEARCH
PROPERTIES 
(
    "hosts" = "http://192.168.0.1:9200,http://192.168.0.2:9200",
    "user" = "root",
    "password" = "root",
    "index" = "tindex",
    "type" = "_doc",
    "es.net.ssl" = "true"
);
~~~

#### パラメータ説明

| **パラメータ**             | **必須** | **デフォルト値** | **説明**                                                     |
| -------------------- | ------------ | ---------- | ------------------------------------------------------------ |
| hosts                | はい           | なし         | Elasticsearchクラスタの接続アドレス。Elasticsearchのバージョン番号とインデックスのシャード分布情報を取得するために使用されます。一つ以上指定可能です。StarRocksは`GET /_nodes/http` APIが返すアドレスを使用してElasticsearchクラスタと通信するため、`hosts`パラメータの値は`GET /_nodes/http`が返すアドレスと一致する必要があります。そうでない場合、BEがElasticsearchクラスタと正常に通信できない可能性があります。|
| index                | はい           | なし         | StarRocks内のテーブルに対応するElasticsearchのインデックス名。インデックスのエイリアスも可能です。ワイルドカードマッチングをサポートしており、例えば`index`に`hello*`を設定すると、`hello`で始まるすべてのインデックスにマッチします。 |
| user                 | いいえ           | 空         | Basic認証を有効にしたElasticsearchクラスタのユーザー名。このユーザーには`/*cluster/state/* nodes/http`などのパスへのアクセス権とインデックスの読み取り権限が必要です。|
| password             | いいえ           | 空         | 対応するユーザーのパスワード情報。                                         |
| type                 | いいえ           | `_doc`     | インデックスのタイプを指定します。Elasticsearch 8以上のバージョンでデータをクエリする場合、StarRocksで外部テーブルを作成する際にこのパラメータを設定する必要はありません。Elasticsearch 8以降のバージョンではmapping typesが削除されています。 |
| es.nodes.wan.only    | いいえ           | `false`    | StarRocksが`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データを取得するかどうかを示します。バージョン2.3.0から、StarRocksはこのパラメータの設定をサポートしています。<ul><li>`true`：StarRocksは`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データを取得します。Elasticsearchクラスタの各シャードが配置されているデータノードのアドレスを検出しません。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできない場合は、`true`に設定する必要があります。</li><li>`false`：StarRocksは`hosts`のアドレスを使用してElasticsearchクラスタの各シャードが配置されているデータノードのアドレスを検出します。クエリプランニング後、関連するBEノードはElasticsearchクラスタ内のデータノードに直接リクエストを送り、インデックスのシャードデータを取得します。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできる場合は、デフォルト値の`false`を維持することをお勧めします。</li></ul> |
| es.net.ssl           | いいえ           | `false`    | HTTPSプロトコルを使用してElasticsearchクラスタにアクセスすることを許可するかどうか。バージョン2.4から、StarRocksはこのパラメータの設定をサポートしています。<ul><li>`true`：許可します。HTTPプロトコルとHTTPSプロトコルの両方がアクセス可能です。</li><li>`false`：許可しません。HTTPプロトコルのみがアクセス可能です。</li></ul> |
| enable_docvalue_scan | いいえ           | `true`     | Elasticsearchの列指向ストレージからクエリフィールドの値を取得するかどうか。通常、列指向ストレージからデータを読み取るパフォーマンスは、行指向ストレージから読み取るパフォーマンスよりも優れています。 |
| enable_keyword_sniff | いいえ           | `true`     | Elasticsearch内のTEXT型フィールドを検出し、KEYWORD型フィールドを使用してクエリするかどうか。`false`に設定すると、分割された内容に基づいてマッチングが行われます。デフォルト値は`true`です。 |

##### 列指向スキャンを有効にしてクエリ速度を向上させる

`enable_docvalue_scan`を`true`に設定すると、StarRocksはElasticsearchからデータを取得する際に以下の2つの原則に従います：

* **尽力而为**: 自動検出するかどうか、読み取るフィールドが列式ストレージを有効にしているかどうか。すべての取得フィールドに列存がある場合、StarRocksは列式ストレージからすべてのフィールド値を取得します。
* **自動降格**: 取得するフィールドの中に列存がないフィールドがある場合、StarRocksは行存の `_source` からすべてのフィールド値を解析して取得します。

> **説明**
>
> * TEXT型のフィールドはElasticsearchでは列式ストレージがありません。したがって、取得するフィールド値にTEXT型のフィールドが含まれている場合、自動的に `_source` から取得するように降格します。
> * 取得するフィールドの数が多すぎる場合（25以上）、`docvalue` からフィールド値を取得するパフォーマンスは `_source` から取得するのとほぼ同じになります。

##### KEYWORD型フィールドの検出

`enable_keyword_sniff` を `true` に設定すると、Elasticsearchではインデックスを作成せずにデータのインポートが直接行えます。Elasticsearchはデータインポート完了後に自動的に新しいインデックスを作成します。文字列型のフィールドに対して、ElasticsearchはTEXT型とKEYWORD型の両方を持つフィールドを作成します。これはElasticsearchのMulti-Field特性です。Mappingは以下の通りです：

~~~sql
"k4": {
   "type": "text",
   "fields": {
      "keyword": {   
         "type": "keyword",
         "ignore_above": 256
      }
   }
}
~~~

`k4` に対する条件フィルタ（例えば `=` 条件）を行うとき、StarRocks On ElasticsearchはクエリをElasticsearchのTermQueryに変換します。

元のSQLフィルタ条件は以下の通りです：

~~~sql
k4 = "StarRocks On Elasticsearch"
~~~

ElasticsearchのクエリDSLに変換されたものは以下の通りです：

~~~sql
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

`k4` の最初のフィールドタイプがTEXTであるため、データインポート時にStarRocksは `k4` に設定されたトークナイザー（設定されていない場合はデフォルトで `standard` トークナイザーを使用）を使用してトークン処理を行い、`StarRocks`、`On`、`Elasticsearch` の3つの `term` を得ます。以下に示します：

~~~sql
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

トークン化の結果は以下の通りです：

~~~sql
{
   "tokens": [
      {
         "token": "starrocks",
         "start_offset": 0,
         "end_offset": 10,
         "type": "<ALPHANUM>",
         "position": 0
      },
      {
         "token": "on",
         "start_offset": 11,
         "end_offset": 13,
         "type": "<ALPHANUM>",
         "position": 1
      },
      {
         "token": "elasticsearch",
         "start_offset": 14,
         "end_offset": 27,
         "type": "<ALPHANUM>",
         "position": 2
      }
   ]
}
~~~

以下のクエリを実行するとします：

~~~sql
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

この `term` は辞書内のどの `term` とも一致しないため、結果は返されません。しかし、`enable_keyword_sniff` を `true` に設定すると、StarRocksは自動的に `k4 = "StarRocks On Elasticsearch"` を `k4.keyword = "StarRocks On Elasticsearch"` に変換してSQLの意味を完全に一致させます。変換後のElasticsearchクエリDSLは以下の通りです：

~~~sql
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword` のタイプはKEYWORDで、データがElasticsearchに書き込まれるときには完全な `term` として扱われるため、辞書で一致する結果を見つけることができます。

#### マッピング関係

外部テーブルを作成する際には、Elasticsearchのフィールドタイプに基づいてStarRocks内の外部テーブルの列タイプを指定する必要があります。具体的なマッピング関係は以下の通りです：

| **Elasticsearch**          | **StarRocks**                   |
| -------------------------- | --------------------------------|
| BOOLEAN                    | BOOLEAN                         |
| BYTE                       | TINYINT/SMALLINT/INT/BIGINT     |
| SHORT                      | SMALLINT/INT/BIGINT             |
| INTEGER                    | INT/BIGINT                      |
| LONG                       | BIGINT                          |
| FLOAT                      | FLOAT                           |
| DOUBLE                     | DOUBLE                          |
| KEYWORD                    | CHAR/VARCHAR                    |
| TEXT                       | CHAR/VARCHAR                    |
| DATE                       | DATE/DATETIME                   |
| NESTED                     | CHAR/VARCHAR                    |
| OBJECT                     | CHAR/VARCHAR                    |
| ARRAY                      | ARRAY                           |

> **説明**
>
> * StarRocksはJSON関連関数を使用してネストされたフィールドを読み取ります。
> * ARRAYタイプについては、Elasticsearchでは多次元配列が自動的に1次元配列に平坦化されるため、StarRocksも同様の変換を行います。**バージョン2.5から、Elasticsearch内のARRAYデータのクエリがサポートされています。**

### 谓词下推

StarRocksはElasticsearchテーブルに対して谓词下推をサポートし、フィルタ条件をElasticsearchにプッシュして実行させ、実行をできるだけストレージに近づけてクエリパフォーマンスを向上させます。現在サポートされているオペレータは以下の表の通りです：

| **SQL構文**           | **Elasticsearch構文**   |
| ------------------------ | ---------------------------|
| `=`                      | term query                 |
| `in`                     | terms query                |
| `>=`,  `<=`, `>`, `<`   | range                      |
| `and`                    | bool.filter                |
| `or`                     | bool.should                |
| `not`                    | bool.must_not              |
| `not in`                 | bool.must_not + terms      |
| `esquery`                | ES Query DSL               |

### クエリ例

esquery関数を使用して、SQLでは表現できないElasticsearchクエリ（例：matchやgeoshapeなど）をElasticsearchにプッシュしてフィルタ処理を行います。esqueryの最初の列名パラメータはインデックスに関連付けるために使用され、2番目のパラメータはElasticsearchの基本Query DSLのjson表現で、波括弧（`{}`）で囲まれます。**jsonのroot keyは1つだけでなければなりません**。例えばmatch、geo_shape、boolなどです。

* matchクエリ

   ~~~sql
   select * from es_table where esquery(k4, '{
      "match": {
         "k4": "StarRocks on elasticsearch"
      }
   }');
   ~~~

* geo関連クエリ

   ~~~sql
   select * from es_table where esquery(k4, '{
   "geo_shape": {
      "location": {
         "shape": {
            "type": "envelope",
            "coordinates": [
               [
                  13,
                  53
               ],
               [
                  14,
                  52
               ]
            ]
         },
         "relation": "within"
      }
   }
   }');
   ~~~

* boolクエリ

   ~~~sql
   select * from es_table where esquery(k4, ' {
      "bool": {
         "must": [
            {
               "terms": {
                  "k1": [
                     11,
                     12
                  ]
               }
            },
            {
               "terms": {
                  "k2": [
                     100
                  ]
               }
            }
         ]
      }
   }');
   ~~~

### 注意事項

* Elasticsearch 5.x以前と以後のデータスキャン方法が異なり、現在StarRocksは5.x以後のバージョンのみをサポートしています。
* HTTP Basic認証を使用するElasticsearchクラスターのクエリがサポートされています。
* StarRocksを通じた一部のクエリは、直接Elasticsearchにリクエストするよりも遅くなることがあります。例えばcount関連のクエリです。これは、Elasticsearchが内部で条件に合致するドキュメント数に関連するメタデータを直接読み取り、実際のデータに対するフィルタ処理を行わないため、countの速度が非常に速いためです。

## (非推奨) Hive外部テーブル

Hive外部テーブルを使用する前に、サーバーにJDK 1.8がインストールされていることを確認してください。

### Hiveリソースの作成

StarRocksはHiveリソースを使用して、使用されるHiveクラスターの関連設定（例：Hive Metastoreのアドレスなど）を管理します。1つのHiveリソースは1つのHiveクラスターに対応します。Hive外部テーブルを作成する際には、どのHiveリソースを使用するかを指定する必要があります。

~~~sql
-- hive0という名前のHiveリソースを作成します。
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);

-- StarRocksに作成されたリソースを表示します。
SHOW RESOURCES;

-- hive0という名前のリソースを削除します。
DROP RESOURCE "hive0";
~~~

StarRocks 2.3以降のバージョンでは、Hiveリソースの `hive.metastore.uris` を変更することがサポートされています。詳細は [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

### データベースの作成

~~~sql
CREATE DATABASE hive_test;
USE hive_test;
~~~

<br/>

### Hive外部テーブルの作成

~~~sql
-- 構文
CREATE EXTERNAL TABLE table_name (
  col_name col_type [NULL | NOT NULL] [COMMENT "comment"]
) ENGINE=HIVE
PROPERTIES (
  "key" = "value"
);

-- 例：hive0リソースに対応するHiveクラスターのrawdataデータベースにあるprofile_parquet_p7テーブルの外部テーブルを作成します。
CREATE EXTERNAL TABLE `profile_wos_p7` (
  `id` bigint NULL,
  `first_id` varchar(200) NULL,
  `second_id` varchar(200) NULL,
  `p__device_id_list` varchar(200) NULL,
  `p__is_deleted` bigint NULL,
  `p_channel` varchar(200) NULL,
  `p_platform` varchar(200) NULL,
  `p_source` varchar(200) NULL,
  `p__city` varchar(200) NULL,
  `p__province` varchar(200) NULL,
  `p__update_time` bigint NULL,
  `p__first_visit_time` bigint NULL,
  `p__last_seen_time` bigint NULL
) ENGINE=HIVE
PROPERTIES (
  "resource" = "hive0",
  "database" = "rawdata",
  "table" = "profile_parquet_p7"
);
~~~

説明：

* 外部テーブルの列：
  * 列名はHiveテーブルと一致する必要があります。
  * 列の順序とHiveテーブルとの関係：Hiveテーブルのストレージ形式がParquetまたはORCの場合、列の順序はHiveテーブルと一致する必要は **ありません**。CSV形式の場合、列の順序はHiveテーブルと一致する必要が **あります**。

  * Hive 表中可以只选择 **部分列**，但 **分区列** 必须全部包含。
  * 外部表的分区列无需通过 partition by 语句指定，需要像普通列一样定义到描述列表中。不需要指定分区信息，StarRocks 会自动从 Hive 同步。
  * ENGINE 设为 HIVE。
* PROPERTIES 属性：
  * **hive.resource**：指定使用的 Hive 资源。
  * **database**：指定 Hive 中的数据库。
  * **table**：指定 Hive 中的表，**不支持视图（view）**。
* 创建外部表时，需要根据 Hive 表的列类型来指定 StarRocks 中外部表的列类型，具体映射关系如下：

| **Hive**      | **StarRocks**                                                |
| ------------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT        | BIGINT                                                       |
| TIMESTAMP     | DATETIME <br />注意：TIMESTAMP 转成 DATETIME 会损失精度和时区信息，并根据 sessionVariable 中的时区转成无时区 DATETIME。 |
| STRING        | VARCHAR                                                      |
| VARCHAR       | VARCHAR                                                      |
| CHAR          | CHAR                                                         |
| DOUBLE        | DOUBLE                                                       |
| FLOAT         | FLOAT                                                        |
| DECIMAL       | DECIMAL                                                      |
| ARRAY         | ARRAY                                                        |

说明：

* 支持 Hive 的存储格式为 Parquet、ORC 和 CSV 格式。如果为 CSV 格式，则暂不支持使用引号作为转义字符。
* 压缩格式支持 Snappy 和 LZ4。
* Hive 外表可查询的最大字符串长度为 1 MB。超过 1 MB 时，查询结果设置为 Null。

<br/>

### 查询 Hive 外表

~~~sql
-- 查询 profile_wos_p7 的总行数。
select count(*) from profile_wos_p7;
~~~

<br/>

### 更新缓存的 Hive 表元数据

Hive 表（Hive Table）的分区统计信息及分区下的文件信息可以缓存到 StarRocks FE 中，缓存的内存结构为 Guava LoadingCache。您可以在 `fe.conf` 文件中通过设置 `hive_meta_cache_refresh_interval_s` 参数来修改缓存自动刷新的间隔时间（默认值为 `7200`，单位：秒），也可以通过设置 `hive_meta_cache_ttl_s` 参数来修改缓存的失效时间（默认值为 `86400`，单位：秒）。修改后需要重启 FE 以生效。

#### 手动更新元数据缓存

* 手动刷新元数据信息：
  1. 当 Hive 中新增或删除分区时，需要刷新 **表** 的元数据信息：`REFRESH EXTERNAL TABLE hive_t`，其中 `hive_t` 是 StarRocks 中的外部表名称。
  2. 当 Hive 中向某些分区新增数据时，需要 **指定分区** 进行刷新：`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')`，其中 `hive_t` 是 StarRocks 中的外部表名称，'k1=01/k2=02'、'k1=03/k2=04' 是 Hive 中的分区名称。
  3. 执行 `REFRESH EXTERNAL TABLE hive_t` 命令时，StarRocks 会先检查 Hive 外部表中的列信息与 Hive Metastore 返回的 Hive 表中的列信息是否一致。如果发现 Hive 表的 schema 发生了变化，如增加或减少列，StarRocks 会将变化的信息同步到 Hive 外部表。同步后，Hive 外部表的列顺序与 Hive 表的列顺序保持一致，且分区列位于最后。
  
#### 自动增量更新元数据缓存

自动增量更新元数据缓存主要通过定期消费 Hive Metastore 的事件来实现，新增分区及分区新增数据无需手动执行 refresh 来更新。用户需要在 Hive Metastore 端开启元数据事件机制。与 Loading Cache 的自动刷新机制相比，自动增量更新性能更好，建议用户开启此功能。开启后，Loading Cache 的自动刷新机制将不再生效。

* 开启 Hive Metastore 事件机制

   用户需要在 `$HiveMetastore/conf/hive-site.xml` 中添加如下配置，并重启 Hive Metastore。以下配置适用于 Hive Metastore 3.1.2 版本，用户可以将配置复制到 `hive-site.xml` 中进行验证，因为在 Hive Metastore 中配置不存在的参数只会提示 WARN 信息，不会抛出异常。

~~~xml
<property>
    <name>hive.metastore.event.db.notification.api.auth</name>
    <value>false</value>
  </property>
  <property>
    <name>hive.metastore.notifications.add.thrift.objects</name>
    <value>true</value>
  </property>
  <property>
    <name>hive.metastore.alter.notifications.basic</name>
    <value>false</value>
  </property>
  <property>
    <name>hive.metastore.dml.events</name>
    <value>true</value>
  </property>
  <property>
    <name>hive.metastore.transactional.event.listeners</name>
    <value>org.apache.hive.hcatalog.listener.DbNotificationListener</value>
  </property>
  <property>
    <name>hive.metastore.event.db.listener.timetolive</name>
    <value>172800s</value>
  </property>
  <property>
    <name>hive.metastore.server.max.message.size</name>
    <value>858993459</value>
  </property>
~~~

* StarRocks 开启自动增量元数据同步

    用户需要在 `$FE_HOME/conf/fe.conf` 中添加如下配置并重启 FE。
     `enable_hms_events_incremental_sync=true`
    自动增量元数据同步的相关配置如下，如无特殊需求，无需修改。

   | 参数名                             | 说明                                      | 默认值 |
   | --- | --- | ---|
   | enable_hms_events_incremental_sync | 是否开启元数据自动增量同步功能            | false |
   | hms_events_polling_interval_ms     | StarRocks 拉取 Hive Metastore 事件的间隔时间 | 5000 毫秒 |
   | hms_events_batch_size_per_rpc      | StarRocks 每次拉取事件的最大数量          | 500 |
   | enable_hms_parallel_process_events | 是否并行处理接收到的事件                  | true |
   | hms_process_events_parallel_num    | 处理事件的并发数                          | 4 |

* 注意事项
  * 不同版本的 Hive Metastore 事件可能不同，且上述开启 Hive Metastore 事件机制的配置在不同版本中也可能有所不同。使用时，相关配置可根据实际版本进行适当调整。已验证可以开启 Hive Metastore 事件机制的版本包括 2.X 和 3.X。用户可以在 FE 日志中搜索 "event id" 来验证事件是否成功开启，如果没有成功，event id 将始终为 0。如果无法判断是否成功开启事件机制，请在 StarRocks 用户交流群中联系值班同学进行排查。
  * 当前 Hive 元数据缓存模式为懒加载，即：如果 Hive 新增了分区，StarRocks 只会缓存新增分区的 partition key，不会立即缓存该分区的文件信息。只有在查询该分区或用户手动执行 refresh 分区操作时，才会加载该分区的文件信息。StarRocks 首次缓存该分区统计信息后，该分区后续的元数据变更就会自动同步到 StarRocks 中。
  * 手动执行缓存的效率较低，相比之下自动增量更新性能开销较小，建议用户开启该功能进行缓存更新。
  * 当 Hive 数据存储为 Parquet、ORC、CSV 格式时，StarRocks 2.3 及以上版本支持 Hive 外部表同步 ADD COLUMN、REPLACE COLUMN 等表结构变更（Schema Change）。

### 访问对象存储

* FE 的配置文件路径为 `$FE_HOME/conf`。如果需要自定义 Hadoop 集群的配置，可以在该目录下添加配置文件，例如：如果 HDFS 集群采用了高可用的 Nameservice，需要将 Hadoop 集群中的 `hdfs-site.xml` 放到该目录下；如果 HDFS 配置了 ViewFs，需要将 `core-site.xml` 放到该目录下。
* BE 的配置文件路径为 `$BE_HOME/conf`。如果需要自定义 Hadoop 集群的配置，可以在该目录下添加配置文件，例如：如果 HDFS 集群采用了高可用的 Nameservice，需要将 Hadoop 集群中的 `hdfs-site.xml` 放到该目录下；如果 HDFS 配置了 ViewFs，需要将 `core-site.xml` 放到该目录下。
* BE 所在机器的**启动脚本** `$BE_HOME/bin/start_be.sh` 中需要配置 `JAVA_HOME`，应配置为 JDK 环境，不能配置为 JRE 环境，例如 `export JAVA_HOME=<JDK 的绝对路径>`。注意，需要将该配置添加在 BE 启动脚本的最开始处，添加完成后需重启 BE。
* Kerberos 支持
  1. 在所有的 FE/BE 机器上使用 `kinit -kt keytab_path principal` 登录，该用户需要有访问 Hive 和 HDFS 的权限。kinit 命令登录是有时效性的，需要将其放入 crontab 中定期执行。
  2. 将 Hadoop 集群中的 `hive-site.xml`、`core-site.xml`、`hdfs-site.xml` 放到 `$FE_HOME/conf` 下，将 `core-site.xml`、`hdfs-site.xml` 放到 `$BE_HOME/conf` 下。
  3. 在 `$FE_HOME/conf/fe.conf` 文件中的 `JAVA_OPTS` 选项中添加 `-Djava.security.krb5.conf=/etc/krb5.conf`，其中 `/etc/krb5.conf` 是 `krb5.conf` 文件的路径，可根据系统进行调整。
  4. 在 `$BE_HOME/conf/be.conf` 文件中增加选项 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`，其中 `/etc/krb5.conf` 是 `krb5.conf` 文件的路径，可根据系统进行调整。
  5. Resource 中的 URI 地址必须使用域名，并且相应的 Hive 和 HDFS 的域名与 IP 的映射都需要配置到 `/etc/hosts` 中。

#### AWS S3/Tencent Cloud COS 支持

1. 在 `$FE_HOME/conf/core-site.xml` 中加入如下配置：

   ~~~xml
   <configuration>
      <property>
         <name>fs.s3a.impl</name>
         <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
      </property>
      <property>
         <name>fs.AbstractFileSystem.s3a.impl</name>
         <value>org.apache.hadoop.fs.s3a.S3A</value>
      </property>
      <property>
         <name>fs.s3a.access.key</name>
         <value>******</value>
      </property>
      <property>
         <name>fs.s3a.secret.key</name>
         <value>******</value>
      </property>
      <property>
         <name>fs.s3a.endpoint</name>
         <value>s3.us-west-2.amazonaws.com</value>
      </property>
     <property>
        <name>fs.s3a.connection.maximum</name>
        <value>500</value>
      </property>
   </configuration>
   ~~~

   * `fs.s3a.access.key` AWSのアクセスキーIDを指定します
   * `fs.s3a.secret.key` AWSのシークレットアクセスキーを指定します
   * `fs.s3a.endpoint` AWSのリージョンを指定します
   * `fs.s3a.connection.maximum` 最大接続数を設定します。クエリ中に `Timeout waiting for connection from pool` のエラーが発生した場合は、このパラメータを適切に増やしてください

2. `$BE_HOME/conf/be.conf` に以下の設定を追加します。

   * `object_storage_access_key_id` FE側の `core-site.xml` の `fs.s3a.access.key` と同じ
   * `object_storage_secret_access_key` FE側の `core-site.xml` の `fs.s3a.secret.key` と同じ
   * `object_storage_endpoint` FE側の `core-site.xml` の `fs.s3a.endpoint` と同じ
   * `object_storage_region` は腾讯COSのみ追加が必要です。例：ap-beijing****

3. FEとBEを再起動します。

#### Aliyun OSSのサポート

1. `$FE_HOME/conf/core-site.xml` に以下の設定を追加します。

   ~~~xml
   <configuration>
      <property>
         <name>fs.oss.impl</name>
         <value>com.aliyun.jindodata.oss.JindoOssFileSystem</value>
      </property>
      <property>
         <name>fs.AbstractFileSystem.oss.impl</name>
         <value>com.aliyun.jindodata.oss.OSS</value>
      </property>
      <property>
         <name>fs.oss.accessKeyId</name>
         <value>xxx</value>
      </property>
      <property>
         <name>fs.oss.accessKeySecret</name>
         <value>xxx</value>
      </property>
      <property>
         <name>fs.oss.endpoint</name>
         <!-- 以下は北京リージョンを例にしています。他のリージョンは実際の状況に応じて置き換えてください。 -->
         <value>oss-cn-beijing.aliyuncs.com</value>
      </property>
   </configuration>
   ~~~

   * `fs.oss.accessKeyId` 阿里云アカウントまたはRAMユーザーのAccessKey IDを指定します。取得方法は[AccessKeyの取得](https://help.aliyun.com/document_detail/53045.htm?spm=a2c4g.11186623.0.0.128b4b7896DD4W#task968)を参照してください。
   * `fs.oss.accessKeySecret` 阿里云アカウントまたはRAMユーザーのAccessKey Secretを指定します。取得方法は[AccessKeyの取得](https://help.aliyun.com/document_detail/53045.htm?spm=a2c4g.11186623.0.0.128b4b7896DD4W#task968)を参照してください。
   * `fs.oss.endpoint` OSS Bucketが位置するリージョンのEndpointを指定します。
    Endpointの検索方法は以下の通りです：

     * Endpointとリージョンの関係を参照して検索してください。詳細は[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.htm#concept-zt4-cvy-5db)をご覧ください。
     * [阿里云OSS管理コンソール](https://oss.console.aliyun.com/index?spm=a2c4g.11186623.0.0.11d24772leoEEg#/)にログインし、Bucket概要ページに進みます。例えば、Bucketドメイン名examplebucket.oss-cn-hangzhou.aliyuncs.comの後ろの部分oss-cn-hangzhou.aliyuncs.comがそのBucketのパブリックEndpointです。

2. `$BE_HOME/conf/be.conf` に以下の設定を追加します。

   * `object_storage_access_key_id` FE側の `core-site.xml` の `fs.oss.accessKeyId` と同じ
   * `object_storage_secret_access_key` FE側の `core-site.xml` の `fs.oss.accessKeySecret` と同じ
   * `object_storage_endpoint` FE側の `core-site.xml` の `fs.oss.endpoint` と同じ

3. FEとBEを再起動します。

## （非推奨）Iceberg外部テーブル

Icebergデータをクエリするには、StarRocksにIceberg外部テーブルを作成し、クエリするIcebergテーブルに外部テーブルをマッピングする必要があります。

バージョン2.1.0以降、StarRocksは外部テーブルを通じてIcebergデータをクエリすることをサポートしています。

### 前提条件

StarRocksがIcebergが依存するメタデータサービス（例：Hive metastore）、ファイルシステム（例：HDFS）、オブジェクトストレージシステム（例：Amazon S3や阿里云OSS）にアクセスする権限があることを確認してください。

### 注意事項

* Iceberg外部テーブルは以下の形式のデータのみクエリをサポートしています：
  * Iceberg v1テーブル（Analytic Data Tables）。バージョン3.0からはORC形式のIceberg v2テーブル（Row-level Deletes）のクエリをサポートし、バージョン3.1からはParquet形式のv2テーブルのクエリをサポートしています。Iceberg v1テーブルとv2テーブルに関する詳細は[Iceberg Table Spec](https://iceberg.apache.org/spec/)を参照してください。
  * gzip（デフォルトの圧縮形式）、Zstd、LZ4、Snappyで圧縮されたテーブル。
  * ParquetおよびORC形式のファイル。
* StarRocksのバージョン2.3以降はIcebergテーブルの構造を同期することをサポートしていますが、バージョン2.3未満はサポートしていません。Icebergテーブルの構造に変更があった場合、StarRocksで対応する外部テーブルを削除し、再作成する必要があります。

### 操作手順

#### ステップ1：Icebergリソースの作成

外部テーブルを作成する前に、Icebergのアクセス情報を管理するためのIcebergリソースを先に作成する必要があります。また、Iceberg外部テーブルを作成する際には、参照するIcebergリソースを指定する必要があります。ビジネスニーズに応じて異なるCatalogタイプのリソースを作成できます：

* Hive metastoreをIcebergのメタデータサービスとして使用する場合は、`HIVE`タイプのCatalogリソースを作成できます。
* Icebergのメタデータサービスをカスタマイズしたい場合は、カスタムカタログ（custom catalog）を開発し、`CUSTOM`タイプのCatalogリソースを作成できます。

> **注記**
>
> `CUSTOM`タイプのCatalogリソースの作成はStarRocksのバージョン2.3以降のみサポートされています。

**`HIVE`タイプのCatalogリソースの作成**

例えば、`iceberg0`という名前のリソースを作成し、そのCatalogタイプを`HIVE`に指定します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083" 
);
~~~

パラメータ説明：

| **パラメータ**                            | **説明**                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| type                                | リソースタイプ、`iceberg`としてください。                                 |
| iceberg.catalog.type              | IcebergのCatalogタイプ。Hive catalogとcustom catalogがサポートされています。Hive catalogを使用する場合は、このパラメータを`HIVE`に設定してください。カスタムカタログを使用する場合は、このパラメータを`CUSTOM`に設定してください。 |
| iceberg.catalog.hive.metastore.uris | Hive MetastoreのURI。形式は`thrift://<IcebergメタデータのIPアドレス>:<ポート番号>`で、ポート番号はデフォルトで9083です。Apache IcebergはHive catalogを介してHive metastoreに接続し、Icebergテーブルのメタデータをクエリします。 |

**`CUSTOM`タイプのCatalogリソースの作成**

Custom catalogは、抽象クラスBaseMetastoreCatalogを継承し、IcebergCatalogインターフェースを実装する必要があります。また、custom catalogのクラス名はStarRocksに既に存在するクラス名と重複してはいけません。開発が完了したら、custom catalogと関連ファイルをパッケージ化して、すべてのFEノードの**fe/lib**パスに配置し、すべてのFEノードを再起動してFEがこのクラスを認識できるようにします。これらの操作が完了したら、リソースを作成できます。

例えば、`iceberg1`という名前のリソースを作成し、そのCatalogタイプを`CUSTOM`に指定します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 
);
~~~

パラメータ説明：

| **パラメータ**               | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| type                   | リソースタイプ、`iceberg`としてください。                                 |
| iceberg.catalog.type | IcebergのCatalogタイプ。Hive catalogとcustom catalogがサポートされています。Hive catalogを使用する場合は、このパラメータを`HIVE`に設定してください。カスタムカタログを使用する場合は、このパラメータを`CUSTOM`に設定してください。 |
| iceberg.catalog-impl   | 開発したcustom catalogの完全修飾クラス名。FEはこのクラス名に基づいて開発したcustom catalogを検索します。custom catalogにカスタム設定項目が含まれている場合は、Iceberg外部テーブルを作成する際にSQL文の`PROPERTIES`にキーと値の形式で追加する必要があります。 |

StarRocksのバージョン2.3以降は、Icebergリソースの`hive.metastore.uris`と`iceberg.catalog-impl`を変更することがサポートされています。詳細は[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

**Icebergリソースの表示**

~~~SQL
SHOW RESOURCES;
~~~

**Icebergリソースの削除**

例えば、`iceberg0`という名前のリソースを削除します。

~~~SQL
DROP RESOURCE "iceberg0";
~~~

リソースを削除すると、そのリソースを参照するすべての外部テーブルが使用できなくなりますが、対応するIcebergテーブルのデータは削除されません。削除後もStarRocksを通じてIcebergデータをクエリしたい場合は、IcebergリソースとIceberg外部テーブルを再作成する必要があります。

#### （オプション）ステップ2：データベースの作成

外部テーブルを格納するために新しいデータベースを作成することも、既存のデータベースに外部テーブルを作成することもできます。


たとえば、StarRocksで`iceberg_test`という名前のデータベースを作成します。構文は以下の通りです：

~~~SQL
CREATE DATABASE iceberg_test; 
~~~

> **説明**
>
> このデータベース名は、クエリ対象のIcebergデータベース名と一致する必要はありません。

#### ステップ3：Iceberg外部テーブルの作成

たとえば、データベース`iceberg_test`に`iceberg_tbl`という名前のIceberg外部テーブルを作成します。構文は以下の通りです：

~~~SQL
CREATE EXTERNAL TABLE `iceberg_tbl` ( 
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=ICEBERG 
PROPERTIES ( 
    "resource" = "iceberg0", 
    "database" = "iceberg", 
    "table" = "iceberg_table" 
); 
~~~

パラメータ説明：

| **パラメータ** | **説明**                            |
| ------------ | ----------------------------------- |
| ENGINE       | 値は`ICEBERG`です。                 |
| resource     | 外部テーブルが参照するIcebergリソースの名前。 |
| database     | Icebergテーブルが属するデータベースの名前。    |
| table        | Icebergテーブルの名前。                  |

> 説明：
 >
 > * テーブル名はIcebergの実際のテーブル名と一致する必要はありません。
 > * 列名はIcebergの実際の列名と一致する必要がありますが、列の順序は一致する必要はありません。

カスタムカタログで設定項目をカスタマイズし、クエリ時にこれらの設定が有効になるようにしたい場合は、これらの設定をキーと値のペアとしてテーブル作成ステートメントの`PROPERTIES`に追加できます。たとえば、カスタムカタログで`custom-catalog.properties`という設定項目を定義した場合、Iceberg外部テーブルの作成構文は以下の通りです：

~~~sql
CREATE EXTERNAL TABLE `iceberg_tbl` ( 
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=ICEBERG 
PROPERTIES ( 
    "resource" = "iceberg0", 
    "database" = "iceberg", 
    "table" = "iceberg_table",
    "custom-catalog.properties" = "my_property"
); 
~~~

外部テーブルを作成する際には、Icebergテーブルの列の型に基づいてStarRocksの外部テーブルの列の型を指定する必要があります。具体的なマッピング関係は以下の通りです：

| **Iceberg**   | **StarRocks**        |
|---------------|----------------------|
| BOOLEAN       | BOOLEAN              |
| INT           | TINYINT/SMALLINT/INT |
| LONG          | BIGINT               |
| FLOAT         | FLOAT                |
| DOUBLE        | DOUBLE               |
| DECIMAL(P, S) | DECIMAL              |
| DATE          | DATE/DATETIME        |
| TIME          | BIGINT               |
| TIMESTAMP     | DATETIME             |
| STRING        | STRING/VARCHAR       |
| UUID          | STRING/VARCHAR       |
| FIXED(L)      | CHAR                 |
| BINARY        | VARCHAR              |
| LIST          | ARRAY                |

StarRocksは、TIMESTAMPTZ、STRUCT、MAPのデータタイプをクエリできません。

#### ステップ4：Icebergデータのクエリ

Iceberg外部テーブルを作成した後、外部テーブルを通じてIcebergテーブルのデータをクエリできます。例えば：

~~~SQL
select count(*) from iceberg_tbl;
~~~

## （非推奨）Hudi外部テーブル

バージョン2.2.0から、StarRocksは外部テーブルを通じてHudiデータレイクのデータをクエリし、データレイクの高速分析を実現することをサポートしています。この記事では、StarRocksで外部テーブルを作成し、Hudiのデータをクエリする方法について説明します。

### 前提条件

StarRocksがHudiに対応するHive Metastore、HDFSクラスタ、またはオブジェクトストレージのバケットにアクセスできることを確認してください。

### 注意事項

* Hudi外部テーブルはクエリ操作のみに使用でき、書き込みはサポートされていません。
* 現在サポートされているHudiのテーブルタイプは、Copy on Write（以下COWと略）とMerge on read（以下MORと略、バージョン2.5からサポート）です。COWとMORの詳細な違いについては、[Apache Hudi公式サイト](https://hudi.apache.org/docs/table_types)を参照してください。
* 現在サポートされているHudiのクエリタイプは、Snapshot QueriesとRead Optimized Queries（MORテーブルのみ）で、Incremental Queriesはサポートされていません。Hudiのクエリタイプの説明については、[Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types)を参照してください。
* Hudiファイルのサポートされている圧縮形式は、GZIP（デフォルト値）、ZSTD、LZ4、SNAPPYです。
* StarRocksは現在、Hudiテーブル構造の同期をサポートしていません。Hudiテーブル構造に変更があった場合、StarRocksで対応する外部テーブルを削除し、再作成する必要があります。

### 操作手順

#### ステップ1：Hudiリソースの作成と管理

StarRocksでHudiデータベースと外部テーブルを作成するために、事前にHudiリソースを作成して管理する必要があります。

以下のコマンドを実行して、`hudi0`という名前のHudiリソースを作成します。

~~~sql
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
~~~

| パラメータ | 説明 |
| ---- | ---- |
| type | リソースタイプは**hudi**と固定されています。 |
| hive.metastore.uris | Hive Metastoreのthrift URI。<br />HudiはHive Metastoreに接続してテーブルを作成および管理します。Hive Metastoreのthrift URIを入力する必要があります。形式は**`thrift://<HudiメタデータのIPアドレス>:<ポート番号>`**で、ポート番号はデフォルトで9083です。 |

StarRocksのバージョン2.3以降では、Hudiリソースの`hive.metastore.uris`を変更することができます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

以下のコマンドを実行して、StarRocksにあるすべてのHudiリソースを表示します。

~~~sql
SHOW RESOURCES;
~~~

以下のコマンドを実行して、`hudi0`という名前のHudiリソースを削除します。

~~~sql
DROP RESOURCE "hudi0";
~~~

> Hudiリソースを削除すると、含まれるすべてのHudi外部テーブルが使用できなくなりますが、Hudiのデータは失われません。StarRocksを通じてHudiのデータを引き続きクエリする必要がある場合は、Hudiリソース、Hudiデータベース、および外部テーブルを再作成してください。

#### ステップ2：Hudiデータベースの作成

以下のコマンドを実行して、StarRocksに`hudi_test`という名前のHudiデータベースを作成し、使用します。

~~~sql
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> データベース名はHudiの実際のデータベース名と一致する必要はありません。

#### ステップ3：Hudi外部テーブルの作成

以下のコマンドを実行して、Hudiデータベース`hudi_test`に`hudi_tbl`という名前のHudi外部テーブルを作成します。

~~~sql
CREATE EXTERNAL TABLE `hudi_tbl` ( 
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=HUDI 
PROPERTIES ( 
    "resource" = "hudi0", 
    "database" = "hudi", 
    "table" = "hudi_table" 
); 
~~~

* 関連するパラメータの説明は以下の表を参照してください：

| **パラメータ** | **説明**                       |
| ------------ | ------------------------------ |
| **ENGINE**   | **HUDI**と固定されています。変更する必要はありません。  |
| **resource** | StarRocksのHudiリソースの名前。 |
| **database** | Hudiテーブルがあるデータベースの名前。        |
| **table**    | Hudiテーブルがあるテーブルの名前。        |

* テーブル名はHudiの実際のテーブル名と一致する必要はありません。
* 列名はHudiの実際の列名と一致する必要がありますが、列の順序は一致する必要はありません。
* ビジネスニーズに応じて、Hudiテーブルのすべてまたは一部の列を選択できます。
* 外部テーブルを作成する際には、Hudiテーブルの列の型に基づいてStarRocksの外部テーブルの列の型を指定する必要があります。具体的なマッピング関係は以下の通りです：

| **Hudiタイプ**                    | **StarRocksタイプ**     |
| ----------------------------     | ----------------------- |
| BOOLEAN                          | BOOLEAN                 |
| INT                              | INT                     |
| DATE                             | DATE                    |
| TimeMillis/TimeMicros            | TIME                    |
| TimestampMillis/TimestampMicros  | DATETIME                |
| LONG                             | BIGINT                  |
| FLOAT                            | FLOAT                   |
| DOUBLE                           | DOUBLE                  |
| STRING                           | CHAR/VARCHAR            |
| ARRAY                            | ARRAY                   |
| DECIMAL                          | DECIMAL                 |

> StarRocksは現在、Struct、Mapデータタイプのクエリをサポートしておらず、MORテーブルについてはArrayデータタイプもサポートしていません。

#### ステップ4：Hudi外部テーブルのクエリ

Hudi外部テーブルを作成した後、データをインポートすることなく、以下のコマンドを実行してHudiのデータをクエリできます。

~~~sql
SELECT COUNT(*) FROM hudi_tbl;
~~~

## （非推奨）MySQL外部テーブル

スタースキーマでは、データは一般にディメンションテーブル（dimension table）とファクトテーブル（fact table）に分けられます。ディメンションテーブルはデータ量が少ないですが、UPDATE操作が発生することがあります。現在、StarRocksは直接のUPDATE操作をサポートしていません（Unique/Primaryデータモデルを使用して実現可能です）。一部のシナリオでは、ディメンションテーブルをMySQLに保存し、クエリ時に直接読み取ることができます。

MySQLのデータを使用する前に、StarRocksで外部テーブル（CREATE EXTERNAL TABLE）を作成し、それにマッピングする必要があります。StarRocksでMySQL外部テーブルを作成する際には、以下のようにMySQLの関連する接続情報を指定する必要があります。

~~~sql
CREATE EXTERNAL TABLE mysql_external_table
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=mysql
PROPERTIES
(
    "host" = "127.0.0.1",
    "port" = "3306",
    "user" = "mysql_user",
    "password" = "mysql_passwd",
    "database" = "mysql_db_test",
    "table" = "mysql_table_test"
);
~~~

パラメータ説明：

* **host**: MySQL 接続アドレス
* **port**: MySQL 接続ポート番号
* **user**: MySQL ログインユーザー名
* **password**: MySQL ログインパスワード
* **database**: MySQL データベース名
* **table**: MySQL データベーステーブル名

<br/>

## よくある質問

### StarRocks 外部テーブル同期エラー、どう対処すればいいですか？

**問題のヒント**：

SQLエラー [1064] [42000]: 空のパーティションを持つテーブルにデータを挿入することはできません。`SHOW PARTITIONS FROM external_t;` を使用して、このテーブルの現在のパーティションを確認してください。

パーティションを確認する際に別のエラーが表示されます：SHOW PARTITIONS FROM external_t
SQLエラー [1064] [42000]: テーブル[external_t]はOLAP/ELASTICSEARCH/HIVEテーブルではありません

**解決策**：

外部テーブルを作成する際のポートが間違っています。正しいポートは "port"="9020" です。

**問題のヒント**：

クエリエラー：query_poolのメモリが限界を超えました。ページの読み込みと解凍に使用されたメモリ：49113428144、制限：49111753861。メモリ使用量がクエリプールの制限を超えています

**解決策**：

外部テーブルのクエリで列が多い場合にこの問題が発生する可能性があります。`be.conf` に `buffer_stream_reserve_size=8192` を追加してBEを再起動することで問題を解決できます。
