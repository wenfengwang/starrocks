---
displayed_sidebar: English
---

# 外部テーブル

:::note
v3.0 以降、Hive、Iceberg、Hudi からデータをクエリする際はカタログの使用を推奨します。[Hive カタログ](../data_source/catalog/hive_catalog.md)、[Iceberg カタログ](../data_source/catalog/iceberg_catalog.md)、[Hudi カタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

v3.1 以降、MySQL および PostgreSQL からのデータをクエリするには [JDBC カタログ](../data_source/catalog/jdbc_catalog.md) の使用を、Elasticsearch からのデータをクエリするには [Elasticsearch カタログ](../data_source/catalog/elasticsearch_catalog.md) の使用を推奨します。
:::

StarRocks は外部テーブルを使用して他のデータソースへのアクセスをサポートしています。外部テーブルは他のデータソースに格納されたデータテーブルに基づいて作成されます。StarRocks はデータテーブルのメタデータのみを保存します。外部テーブルを使用して他のデータソースのデータを直接クエリすることができます。StarRocks は以下のデータソースをサポートしています：MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg、Apache Hudi。**現在、他の StarRocks クラスターから現在の StarRocks クラスターへのデータの書き込みのみが可能です。読み取りはできません。StarRocks 以外のデータソースについては、これらのデータソースからデータを読み取ることのみが可能です。**

バージョン2.5 以降、StarRocks は Data Cache 機能を提供し、外部データソース上のホットデータクエリを高速化します。詳細は [Data Cache](data_cache.md) を参照してください。

## StarRocks 外部テーブル

StarRocks 1.19 以降、StarRocks は StarRocks 外部テーブルを使用して、一つの StarRocks クラスターから別の StarRocks クラスターへデータを書き込むことを可能にします。これにより、読み書きの分離が実現され、リソースの分離が向上します。まず、宛先 StarRocks クラスターに宛先テーブルを作成します。次に、ソース StarRocks クラスターで、宛先テーブルと同じスキーマを持つ StarRocks 外部テーブルを作成し、`PROPERTIES` フィールドに宛先クラスターとテーブルの情報を指定します。

INSERT INTO ステートメントを使用して StarRocks 外部テーブルにデータを書き込むことで、ソースクラスターから宛先クラスターへデータを書き込むことができます。これは以下の目的を実現するのに役立ちます：

* StarRocks クラスター間のデータ同期。
* 読み書きの分離。データはソースクラスターに書き込まれ、ソースクラスターからのデータ変更はクエリサービスを提供する宛先クラスターに同期されます。

以下のコードは、宛先テーブルと外部テーブルを作成する方法を示しています。

~~~SQL
# 宛先 StarRocks クラスターに宛先テーブルを作成します。
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

# ソース StarRocks クラスターに外部テーブルを作成します。
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

# ソースクラスターから宛先クラスターへ StarRocks 外部テーブルにデータを書き込むことでデータを書き込みます。2番目のステートメントは本番環境での使用を推奨します。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

パラメーター：

* **EXTERNAL:** このキーワードは、作成されるテーブルが外部テーブルであることを示します。
* **host:** このパラメータは、宛先 StarRocks クラスタのリーダー FE ノードの IP アドレスを指定します。
* **port:** このパラメータは、宛先 StarRocks クラスタの FE ノードの RPC ポートを指定します。

  :::note

  StarRocks 外部テーブルが属するソースクラスターが宛先 StarRocks クラスターにアクセスできるように、次のポートへのアクセスを許可するようにネットワークとファイアウォールを構成する必要があります：

  * FE ノードの RPC ポート。FE 設定ファイル **fe.conf** の `rpc_port` を参照してください。デフォルトの RPC ポートは `9020` です。
  * BE ノードの bRPC ポート。BE 設定ファイル **be.conf** の `brpc_port` を参照してください。デフォルトの bRPC ポートは `8060` です。

  :::

* **user:** このパラメータは、宛先 StarRocks クラスタへのアクセスに使用するユーザー名を指定します。
* **password:** このパラメータは、宛先 StarRocks クラスタへのアクセスに使用するパスワードを指定します。
* **database:** このパラメータは、宛先テーブルが属するデータベースを指定します。
* **table:** このパラメータは、宛先テーブルの名前を指定します。

StarRocks 外部テーブルを使用する際の制限事項：

* StarRocks 外部テーブルに対しては INSERT INTO および SHOW CREATE TABLE コマンドのみ実行可能です。その他のデータ書き込み方法はサポートされていません。また、StarRocks 外部テーブルからデータをクエリしたり、外部テーブルに対して DDL 操作を実行したりすることはできません。
* 外部テーブルを作成する構文は通常のテーブルを作成するのと同じですが、外部テーブル内の列名やその他の情報は宛先テーブルと同じでなければなりません。
* 外部テーブルは宛先テーブルからテーブルメタデータを10秒ごとに同期します。宛先テーブルで DDL 操作が実行されると、2つのテーブル間のデータ同期に遅延が生じる可能性があります。

## JDBC 互換データベース用外部テーブル

v2.3.0 以降、StarRocks は JDBC 互換データベースをクエリするための外部テーブルを提供します。これにより、StarRocks にデータをインポートすることなく、そのようなデータベースのデータを非常に高速に分析できます。このセクションでは、StarRocks で外部テーブルを作成し、JDBC 互換データベースのデータをクエリする方法について説明します。

### 前提条件

JDBC 外部テーブルを使用してデータをクエリする前に、FE と BE が JDBC ドライバーのダウンロード URL にアクセスできることを確認してください。ダウンロード URL は、JDBC リソースを作成する際に使用されるステートメントの `driver_url` パラメータで指定されます。

### JDBC リソースの作成と管理

#### JDBC リソースの作成

データベースからデータをクエリする外部テーブルを作成する前に、StarRocks に JDBC リソースを作成してデータベースの接続情報を管理する必要があります。データベースは JDBC ドライバをサポートしており、「ターゲットデータベース」と呼ばれます。リソースを作成した後、それを使用して外部テーブルを作成できます。

次のステートメントを実行して、`jdbc0` という名前の JDBC リソースを作成します：

~~~SQL
CREATE EXTERNAL RESOURCE jdbc0
PROPERTIES (
    "type"="jdbc",
    "user"="postgres",
    "password"="changeme",
    "jdbc_uri"="jdbc:postgresql://127.0.0.1:5432/jdbc_test",
    "driver_url"="https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar",
    "driver_class"="org.postgresql.Driver"
);
~~~

`PROPERTIES` で必要なパラメータは以下の通りです：

* `type`: リソースのタイプです。値を `jdbc` に設定します。

* `user`: ターゲットデータベースに接続するために使用されるユーザー名です。

* `password`: ターゲットデータベースに接続するために使用されるパスワードです。


* `jdbc_uri`: JDBCドライバがターゲットデータベースに接続するために使用するURIです。URI形式はデータベースURIの構文を満たす必要があります。一般的なデータベースのURI構文については、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)の公式ウェブサイトを参照してください。

> 注: URIにはターゲットデータベースの名前が含まれている必要があります。例えば、上記のコード例では、`jdbc_test`は接続したいターゲットデータベースの名前です。

* `driver_url`: JDBCドライバのJARパッケージのダウンロードURLです。HTTP URLまたはファイルURLがサポートされており、例えば `https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar` や `file:///home/disk1/postgresql-42.3.3.jar` があります。

* `driver_class`: JDBCドライバのクラス名です。一般的なデータベースのJDBCドライバクラス名は以下の通りです：
  * MySQL: com.mysql.jdbc.Driver (MySQL 5.x以前)、com.mysql.cj.jdbc.Driver (MySQL 6.x以降)
  * SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL: org.postgresql.Driver

リソースが作成される際、FEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバのJARパッケージをダウンロードし、チェックサムを生成して、BEがダウンロードしたJDBCドライバを検証します。

> 注: JDBCドライバのJARパッケージのダウンロードに失敗すると、リソースの作成も失敗します。

BEが初めてJDBC外部テーブルをクエリする際に、対応するJDBCドライバのJARパッケージがマシン上に存在しないことを検出した場合、BEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバのJARパッケージをダウンロードし、全てのJDBCドライバのJARパッケージは`${STARROCKS_HOME}/lib/jdbc_drivers`ディレクトリに保存されます。

#### JDBCリソースの表示

StarRocks内の全てのJDBCリソースを表示するには、以下のステートメントを実行します：

~~~SQL
SHOW RESOURCES;
~~~

> 注: `ResourceType`列は`jdbc`です。

#### JDBCリソースの削除

`jdbc0`という名前のJDBCリソースを削除するには、以下のステートメントを実行します：

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> 注: JDBCリソースが削除されると、そのJDBCリソースを使用して作成された全てのJDBC外部テーブルが利用できなくなります。しかし、ターゲットデータベースのデータは失われません。ターゲットデータベースのデータをStarRocksでクエリする必要がある場合は、JDBCリソースとJDBC外部テーブルを再作成できます。

### データベースの作成

StarRocksに`jdbc_test`という名前のデータベースを作成してアクセスするには、以下のステートメントを実行します：

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> 注: 上記のステートメントで指定するデータベース名は、ターゲットデータベースの名前と同じである必要はありません。

### JDBC外部テーブルの作成

`jdbc_test`データベース内に`jdbc_tbl`という名前のJDBC外部テーブルを作成するには、以下のステートメントを実行します：

~~~SQL
create external table jdbc_tbl (
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=jdbc 
properties (
    "resource" = "jdbc0",
    "table" = "dest_tbl"
);
~~~

`properties`内の必要なパラメータは以下の通りです：

* `resource`: 外部テーブルを作成するために使用されるJDBCリソースの名前。
* `table`: データベース内のターゲットテーブル名。

サポートされるデータ型とStarRocksとターゲットデータベース間のデータ型マッピングについては、[データ型マッピング](External_table.md#データ型マッピング)を参照してください。

> 注：
>
> * インデックスはサポートされていません。
> * PARTITION BYやDISTRIBUTED BYを使用してデータ分散ルールを指定することはできません。

### JDBC外部テーブルのクエリ

JDBC外部テーブルをクエリする前に、パイプラインエンジンを有効にするために以下のステートメントを実行する必要があります：

~~~SQL
set enable_pipeline_engine=true;
~~~

> 注: パイプラインエンジンが既に有効である場合、この手順は省略できます。

JDBC外部テーブルを使用してターゲットデータベース内のデータをクエリするには、以下のステートメントを実行します：

~~~SQL
select * from jdbc_tbl;
~~~

StarRocksはフィルタ条件をターゲットテーブルにプッシュダウンすることで述語プッシュダウンをサポートしています。データソースに近い場所でフィルタ条件を実行することでクエリパフォーマンスが向上します。現在、StarRocksは二項比較演算子（`>`, `>=`, `=`, `<`, `<=`）、`IN`、`IS NULL`、`BETWEEN ... AND ...`をプッシュダウンできますが、関数はプッシュダウンできません。

### データ型マッピング

現在、StarRocksはターゲットデータベース内の基本的な型のデータのみをクエリできます。例えば、NUMBER、STRING、TIME、DATEなどです。ターゲットデータベース内のデータ値の範囲がStarRocksでサポートされていない場合、クエリはエラーを報告します。

ターゲットデータベースとStarRocks間のマッピングは、ターゲットデータベースのタイプに基づいて異なります。

#### **MySQLとStarRocks**

| MySQL         | StarRocks |
| ------------- | --------- |
| BOOLEAN       | BOOLEAN   |
| TINYINT       | TINYINT   |
| SMALLINT      | SMALLINT  |
| MEDIUMINT     | INT       |
| BIGINT        | BIGINT    |
| FLOAT         | FLOAT     |
| DOUBLE        | DOUBLE    |
| DECIMAL       | DECIMAL   |
| CHAR          | CHAR      |
| VARCHAR       | VARCHAR   |
| DATE          | DATE      |
| DATETIME      | DATETIME  |

#### **OracleとStarRocks**

| Oracle        | StarRocks |
| ------------- | --------- |
| CHAR          | CHAR      |
| VARCHAR2      | VARCHAR   |
| DATE          | DATE      |
| SMALLINT      | SMALLINT  |
| INTEGER       | INT       |
| BINARY_FLOAT  | FLOAT     |
| BINARY_DOUBLE | DOUBLE    |
| NUMBER        | DECIMAL   |

#### **PostgreSQLとStarRocks**

| PostgreSQL    | StarRocks |
| ------------- | --------- |
| SMALLINT      | SMALLINT  |
| INTEGER       | INT       |
| BIGINT        | BIGINT    |
| BOOLEAN       | BOOLEAN   |
| REAL          | FLOAT     |
| DOUBLE PRECISION | DOUBLE    |
| DECIMAL       | DECIMAL   |
| TIMESTAMP     | DATETIME  |
| DATE          | DATE      |
| CHAR          | CHAR      |
| VARCHAR       | VARCHAR   |
| TEXT          | VARCHAR   |

#### **SQL ServerとStarRocks**

| SQL Server    | StarRocks |
| ------------- | --------- |
| BIT           | BOOLEAN   |
| TINYINT       | TINYINT   |
| SMALLINT      | SMALLINT  |
| INT           | INT       |
| BIGINT        | BIGINT    |
| FLOAT         | FLOAT     |
| REAL          | DOUBLE    |
| DECIMAL       | DECIMAL   |
| NUMERIC       | DECIMAL   |
| CHAR          | CHAR      |
| VARCHAR       | VARCHAR   |
| DATE          | DATE      |
| DATETIME      | DATETIME  |
| DATETIME2     | DATETIME  |

### 制限事項

* JDBC外部テーブルを作成する際には、テーブルにインデックスを作成したり、PARTITION BYやDISTRIBUTED BYを使用してテーブルのデータ分散ルールを指定したりすることはできません。

* JDBC外部テーブルをクエリする際、StarRocksは関数をテーブルにプッシュダウンすることはできません。

## (非推奨) Elasticsearch外部テーブル

StarRocksとElasticsearchは、2つの人気のある分析システムです。StarRocksは大規模分散コンピューティングにおいて高性能を発揮します。Elasticsearchは全文検索に理想的です。StarRocksとElasticsearchを組み合わせることで、より完全なOLAPソリューションを提供できます。

### Elasticsearch外部テーブルの作成例

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
PROPERTIES (
    "hosts" = "http://192.168.0.1:9200,http://192.168.0.2:9200",
    "user" = "root",
    "password" = "root",
    "index" = "tindex",
    "type" = "_doc",
    "es.net.ssl" = "true"
);
~~~

以下の表は、パラメーターを説明しています。

| **パラメーター**        | **必須** | **デフォルト値** | **説明**                                              |
| -------------------- | ------------ | ----------------- | ------------------------------------------------------------ |
| hosts                | はい          | なし              | Elasticsearchクラスターの接続アドレスです。一つ以上のアドレスを指定できます。StarRocksは、このアドレスからElasticsearchのバージョンとインデックスシャードの割り当てを解析します。StarRocksは`GET /_nodes/http` API操作によって返されるアドレスに基づいてElasticsearchクラスターと通信します。したがって、`hosts`パラメーターの値は`GET /_nodes/http` API操作によって返されるアドレスと同じでなければなりません。そうでないと、BEはElasticsearchクラスターと通信できない可能性があります。|
| index                | はい          | なし              | StarRocksのテーブルに作成されるElasticsearchインデックスの名前です。名前はエイリアスにすることができます。このパラメーターはワイルドカード(*)をサポートします。例えば、`index`を`hello*`に設定すると、StarRocksは名前が`hello`で始まるすべてのインデックスを取得します。 |
| user                 | いいえ           | 空             | 基本認証が有効なElasticsearchクラスターにログインするために使用するユーザー名です。`/*cluster/state/*nodes/http`とインデックスにアクセスできることを確認してください。|
| password             | いいえ           | 空             | Elasticsearchクラスターにログインするために使用されるパスワードです。 |
| type                 | いいえ           | `_doc`            | インデックスのタイプです。デフォルト値は`_doc`です。Elasticsearch 8以降のバージョンでデータをクエリする場合、このパラメーターを設定する必要はありません。なぜなら、マッピングタイプはElasticsearch 8以降のバージョンで削除されているからです。 |
| es.nodes.wan.only    | いいえ           | `false`           | StarRocksが`hosts`に指定されたアドレスのみを使用してElasticsearchクラスターにアクセスしデータを取得するかどうかを指定します。<ul><li>`true`: StarRocksは`hosts`に指定されたアドレスのみを使用してElasticsearchクラスターにアクセスしデータを取得し、Elasticsearchインデックスのシャードが存在するデータノードをスニッフしません。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできない場合、このパラメーターを`true`に設定する必要があります。</li><li>`false`: StarRocksは`hosts`に指定されたアドレスを使用してElasticsearchクラスターインデックスのシャードが存在するデータノードをスニッフします。StarRocksがクエリ実行プランを生成した後、関連するBEはElasticsearchクラスター内のデータノードに直接アクセスし、インデックスのシャードからデータを取得します。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできる場合、デフォルト値の`false`を保持することをお勧めします。</li></ul> |
| es.net.ssl           | いいえ           | `false`           | HTTPSプロトコルを使用してElasticsearchクラスターにアクセスできるかどうかを指定します。StarRocks 2.4以降のバージョンのみがこのパラメーターの設定をサポートしています。<ul><li>`true`: HTTPSとHTTPプロトコルの両方を使用してElasticsearchクラスターにアクセスできます。</li><li>`false`: HTTPプロトコルのみを使用してElasticsearchクラスターにアクセスできます。</li></ul> |
| enable_docvalue_scan | いいえ           | `true`            | ターゲットフィールドの値をElasticsearchの列指向ストレージから取得するかどうかを指定します。ほとんどの場合、列指向ストレージからのデータ読み取りは行指向ストレージからの読み取りよりも優れています。 |
| enable_keyword_sniff | いいえ           | `true`            | ElasticsearchのTEXTタイプフィールドをKEYWORDタイプフィールドに基づいてスニッフするかどうかを指定します。このパラメーターを`false`に設定すると、StarRocksはトークン化後にマッチングを実行します。 |

##### より速いクエリのための列指向スキャン

`enable_docvalue_scan`を`true`に設定すると、StarRocksはElasticsearchからデータを取得する際に以下のルールに従います：

* **試してみる**: StarRocksは自動的にターゲットフィールドに対して列指向ストレージが有効かどうかをチェックします。有効であれば、StarRocksは列指向ストレージからターゲットフィールドの全ての値を取得します。
* **自動ダウングレード**: ターゲットフィールドのいずれかが列指向ストレージで利用できない場合、StarRocksは行指向ストレージ(`_source`)からターゲットフィールドの全ての値を解析し取得します。

> **注記**
>
> * ElasticsearchのTEXTタイプフィールドでは列指向ストレージは利用できません。したがって、TEXTタイプの値を含むフィールドをクエリする場合、StarRocksは`_source`からフィールドの値を取得します。
> * 多数（25以上）のフィールドをクエリする場合、`docvalue`からフィールド値を読み取ることは、`_source`から読み取ることと比較して顕著な利点はありません。

##### KEYWORDタイプフィールドのスニッフ

`enable_keyword_sniff`を`true`に設定すると、Elasticsearchはインデックスなしで直接データを取り込むことができます。なぜなら、取り込み後に自動的にインデックスを作成するからです。STRINGタイプのフィールドについては、ElasticsearchはTEXTとKEYWORDの両タイプを持つフィールドを作成します。これはElasticsearchのマルチフィールド機能の仕組みです。マッピングは以下の通りです：

~~~SQL
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

例えば、`k4`に対して「=」フィルタリングを行う場合、StarRocksはElasticsearchのTermQueryにフィルタリング操作を変換します。

元のSQLフィルターは以下の通りです：

~~~SQL
k4 = "StarRocks On Elasticsearch"
~~~

変換されたElasticsearchクエリDSLは以下の通りです：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

`k4`の最初のフィールドはTEXTタイプで、データ取り込み後に`k4`に設定されたアナライザー（またはアナライザーが設定されていない場合は標準アナライザー）によってトークン化されます。結果として、最初のフィールドは`StarRocks`、`On`、`Elasticsearch`の3つの用語にトークン化されます。詳細は以下の通りです：

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

トークン化の結果は以下の通りです：

~~~SQL
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
         "end_offset": 28,
         "type": "<ALPHANUM>",
         "position": 2
      }
   ]
}
~~~

以下のようにクエリを実行するとします：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
}
~~~

辞書に `StarRocks On Elasticsearch` に一致する用語がないため、結果は返されません。

しかし、`enable_keyword_sniff` を `true` に設定した場合、StarRocks は `k4 = "StarRocks On Elasticsearch"` を `k4.keyword = "StarRocks On Elasticsearch"` に変換して SQL セマンティクスに一致させます。変換された `StarRocks On Elasticsearch` のクエリ DSL は次のとおりです。

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword` は KEYWORD 型です。そのため、データは Elasticsearch に完全な用語として書き込まれ、マッチングが成功します。

#### 列データ型のマッピング

外部テーブルを作成する際には、Elasticsearch テーブルの列のデータ型に基づいて外部テーブルの列のデータ型を指定する必要があります。以下の表は、列データ型のマッピングを示しています。

| **Elasticsearch** | **StarRocks**               |
| ----------------- | --------------------------- |
| BOOLEAN           | BOOLEAN                     |
| BYTE              | TINYINT/SMALLINT/INT/BIGINT |
| SHORT             | SMALLINT/INT/BIGINT         |
| INTEGER           | INT/BIGINT                  |
| LONG              | BIGINT                      |
| FLOAT             | FLOAT                       |
| DOUBLE            | DOUBLE                      |
| KEYWORD           | CHAR/VARCHAR                |
| TEXT              | CHAR/VARCHAR                |
| DATE              | DATE/DATETIME               |
| NESTED            | CHAR/VARCHAR                |
| OBJECT            | CHAR/VARCHAR                |
| ARRAY             | ARRAY                       |

> **注記**
>
> * StarRocks は JSON 関連の関数を使用して NESTED 型のデータを読み取ります。
> * Elasticsearch は多次元配列を一次元配列に自動的に平坦化します。StarRocks も同様です。**Elasticsearch から ARRAY データをクエリするサポートは v2.5 から追加されました。**

### 述語プッシュダウン

StarRocks は述語プッシュダウンをサポートしています。フィルターを Elasticsearch にプッシュダウンして実行することで、クエリのパフォーマンスが向上します。次の表に、述語プッシュダウンをサポートする演算子を示します。

|   SQL 構文  |   ES 構文  |
| :---: | :---: |
|  `=`   |  term query   |
|  `in`   |  terms query   |
|  `>=,  <=, >, <`   |  range query   |
|  `and`   |  bool.filter   |
|  `or`   |  bool.should   |
|  `not`   |  bool.must_not   |
|  `not in`   |  bool.must_not + terms query   |
|  `esquery`   |  ES Query DSL  |

### 例

**esquery 関数**は、**SQL で表現できない**クエリ（match や geo_shape など）を Elasticsearch にプッシュダウンしてフィルタリングするために使用されます。esquery 関数の最初のパラメータはインデックスを関連付けるために使用されます。2 番目のパラメータは、基本的な Query DSL の JSON 式で、{} で囲まれています。**JSON 式には、match、geo_shape、bool などのルートキーが 1 つだけ含まれている必要があります。**

* match query

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on Elasticsearch"
    }
}');
~~~

* geo_shape query

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

* bool query

~~~sql
select * from es_table where esquery(k4, '{
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

### 使用上の注意

* 5.x より前の Elasticsearch は 5.x 以降とは異なる方法でデータをスキャンします。現在、**5.x 以降のバージョンのみ**がサポートされています。
* HTTP 基本認証が有効になっている Elasticsearch クラスターがサポートされています。
* StarRocks からのデータのクエリは、Elasticsearch からのデータを直接クエリする場合（カウント関連のクエリなど）と比べて速くない場合があります。その理由は、Elasticsearch が実際のデータをフィルタリングすることなくターゲットドキュメントのメタデータを直接読み取るため、カウントクエリが高速化されるからです。

## (非推奨) Hive 外部テーブル

Hive 外部テーブルを使用する前に、サーバーに JDK 1.8 がインストールされていることを確認してください。

### Hive リソースの作成

Hive リソースは Hive クラスターに対応します。StarRocks で使用する Hive クラスターを構成する必要があります。例えば、Hive メタストアのアドレスを指定する必要があります。Hive 外部テーブルで使用する Hive リソースを指定する必要があります。

* `hive0` という名前の Hive リソースを作成します。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
~~~

* StarRocks で作成されたリソースを表示します。

~~~sql
SHOW RESOURCES;
~~~

* `hive0` という名前のリソースを削除します。

~~~sql
DROP RESOURCE "hive0";
~~~

StarRocks 2.3 以降のバージョンでは `hive.metastore.uris` の Hive リソースを変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

### データベースの作成

~~~sql
CREATE DATABASE hive_test;
USE hive_test;
~~~

### Hive 外部テーブルの作成

構文

~~~sql
CREATE EXTERNAL TABLE table_name (
  col_name col_type [NULL | NOT NULL] [COMMENT "comment"]
) ENGINE=HIVE
PROPERTIES (
  "key" = "value"
);
~~~

例: `hive0` リソースに対応する Hive クラスターの `rawdata` データベースの下に外部テーブル `profile_parquet_p7` を作成します。

~~~sql
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

説明:

* 外部テーブルの列
  * 列名は Hive テーブルの列名と同じである必要があります。
  * 列の順序は Hive テーブルの列の順序と同じである必要はありません。
  * Hive テーブルの一部の列のみを選択できますが、パーティションキー列はすべて選択する必要があります。
  * 外部テーブルのパーティションキーカラムは `partition by` を使用して指定する必要はありません。これらは他の列と同じ記述リストに定義する必要があります。パーティション情報を指定する必要はありません。StarRocks はこの情報を Hive テーブルから自動的に同期します。
  * `ENGINE` を HIVE に設定します。
* PROPERTIES:
  * **hive.resource**: 使用する Hive リソース。
  * **database**: Hive データベース。
  * **table**: Hive のテーブル。**view** はサポートされていません。
* Hive と StarRocks の間の列データ型のマッピングについては、以下の表を参照してください。

    |  Hive の列の種類   |  StarRocks の列の種類   | 説明 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |

    |   TIMESTAMP  | DATETIME    | TIMESTAMP データを DATETIME データに変換する際、精度とタイムゾーン情報が失われます。sessionVariable のタイムゾーンを基に、タイムゾーンオフセットを含まない DATETIME データに TIMESTAMP データを変換する必要があります。 |
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意：
>
> * 現在サポートされている Hive ストレージフォーマットは Parquet、ORC、CSV です。
ストレージフォーマットが CSV の場合、クオーテーションマークはエスケープ文字として使用できません。
> * SNAPPY および LZ4 圧縮フォーマットがサポートされています。
> * クエリ可能な Hive の STRING 型の列の最大長は 1 MB です。STRING 型の列が 1 MB を超える場合、NULL 列として処理されます。

### Hive 外部テーブルの使用

`profile_wos_p7` の総行数をクエリします。

~~~sql
select count(*) from profile_wos_p7;
~~~

### キャッシュされた Hive テーブルメタデータの更新

* Hive のパーティション情報と関連ファイル情報は StarRocks にキャッシュされています。キャッシュは `hive_meta_cache_refresh_interval_s` で指定された間隔で更新されます。デフォルト値は 7200 秒です。`hive_meta_cache_ttl_s` はキャッシュのタイムアウト期間を指定し、デフォルト値は 86400 秒です。
  * キャッシュされたデータは手動で更新することもできます。
    1. Hive のテーブルにパーティションが追加または削除された場合、`REFRESH EXTERNAL TABLE hive_t` コマンドを実行して StarRocks にキャッシュされたテーブルメタデータを更新する必要があります。`hive_t` は StarRocks の Hive 外部テーブルの名前です。
    2. Hive パーティションのデータが更新された場合、`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')` コマンドを実行して StarRocks のキャッシュデータを更新する必要があります。`hive_t` は StarRocks の Hive 外部テーブルの名前で、`'k1=01/k2=02'` と `'k1=03/k2=04'` は更新された Hive パーティションの名前です。
    3. `REFRESH EXTERNAL TABLE hive_t` を実行すると、StarRocks は最初に Hive 外部テーブルのカラム情報が Hive Metastore によって返される Hive テーブルのカラム情報と同じかどうかを確認します。Hive テーブルのスキーマに変更があった場合（カラムの追加や削除など）、StarRocks はその変更を Hive 外部テーブルに同期します。同期後、Hive 外部テーブルのカラム順序は Hive テーブルのカラム順序と同じで、パーティションカラムが最後のカラムになります。
* Parquet、ORC、CSV 形式で保存された Hive データについては、StarRocks 2.3 以降のバージョンで Hive テーブルのスキーマ変更（ADD COLUMN や REPLACE COLUMN など）を Hive 外部テーブルに同期することができます。

### オブジェクトストレージへのアクセス

* FE の設定ファイルのパスは `fe/conf` で、Hadoop クラスタをカスタマイズするための設定ファイルを追加できます。例えば、HDFS クラスタが高可用性ネームサービスを使用している場合、`hdfs-site.xml` を `fe/conf` に配置する必要があります。HDFS が ViewFs で設定されている場合は、`core-site.xml` を `fe/conf` に配置します。
* BE の設定ファイルのパスは `be/conf` で、Hadoop クラスタをカスタマイズするための設定ファイルを追加できます。例えば、HDFS クラスタが高可用性ネームサービスを使用している場合、`hdfs-site.xml` を `be/conf` に配置する必要があります。HDFS が ViewFs で設定されている場合は、`core-site.xml` を `be/conf` に配置します。
* BE が配置されているマシンで、BE の**起動スクリプト** `bin/start_be.sh` で JAVA_HOME を JRE 環境ではなく JDK 環境として設定します。例: `export JAVA_HOME=<JDKのパス>`。この設定をスクリプトの先頭に追加し、BE を再起動して設定を有効にする必要があります。
* Kerberos サポートの設定：
  1. すべての FE/BE マシンで `kinit -kt keytab_path principal` を使用してログインし、Hive と HDFS にアクセスできるようにします。kinit コマンドでのログインは一定期間のみ有効であり、定期的に実行するために crontab に設定する必要があります。
  2. `hive-site.xml/core-site.xml/hdfs-site.xml` を `fe/conf` に配置し、`core-site.xml/hdfs-site.xml` を `be/conf` に配置します。
  3. `JAVA_OPTS` オプションの値に `-Djava.security.krb5.conf=/etc/krb5.conf` を追加し、`$FE_HOME/conf/fe.conf` ファイルに設定します。`/etc/krb5.conf` は `krb5.conf` ファイルの保存パスです。オペレーティングシステムに基づいてパスを変更できます。
  4. `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を `$BE_HOME/conf/be.conf` ファイルに直接追加します。`/etc/krb5.conf` は `krb5.conf` ファイルの保存パスです。オペレーティングシステムに基づいてパスを変更できます。
  5. Hive リソースを追加する際には、`hive.metastore.uris` にドメイン名を渡す必要があります。また、Hive/HDFS のドメイン名と IP アドレスのマッピングを `/etc/hosts` ファイルに追加する必要があります。

* AWS S3 のサポートを設定する：`fe/conf/core-site.xml` と `be/conf/core-site.xml` に以下の設定を追加します。

   ~~~XML
   <configuration>
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

   1. `fs.s3a.access.key`: AWS アクセスキー ID。
   2. `fs.s3a.secret.key`: AWS シークレットアクセスキー。
   3. `fs.s3a.endpoint`: 接続する AWS S3 のエンドポイント。
   4. `fs.s3a.connection.maximum`: StarRocks から S3 への同時接続の最大数。クエリ中に `Timeout waiting for connection from pool` のエラーが発生した場合、このパラメーターを大きな値に設定できます。

## (非推奨) Iceberg 外部テーブル

v2.1.0 から、StarRocks は外部テーブルを使用して Apache Iceberg からのデータクエリをサポートしています。Iceberg のデータをクエリするには、StarRocks に Iceberg 外部テーブルを作成する必要があります。テーブルを作成する際には、外部テーブルとクエリしたい Iceberg テーブルとの間にマッピングを確立する必要があります。

### 始める前に

StarRocks が Apache Iceberg で使用されるメタデータサービス（例：Hive メタストア）、ファイルシステム（例：HDFS）、オブジェクトストレージシステム（例：Amazon S3 や Alibaba Cloud Object Storage Service）にアクセスする権限があることを確認してください。

### 注意事項

* Iceberg 外部テーブルは、以下のタイプのデータのクエリにのみ使用できます：
  * Iceberg v1 テーブル（分析データテーブル）。ORC 形式の Iceberg v2（行レベル削除）テーブルは v3.0 から、Parquet 形式の Iceberg v2 テーブルは v3.1 からサポートされています。Iceberg v1 テーブルと Iceberg v2 テーブルの違いについては、[Iceberg テーブルスペック](https://iceberg.apache.org/spec/)を参照してください。
  * gzip（デフォルト形式）、Zstd、LZ4、または Snappy 形式で圧縮されたテーブル。
  * Parquet または ORC 形式で保存されたファイル。


* StarRocks 2.3以降のバージョンではIceberg外部テーブルがIcebergテーブルのスキーマ変更を同期することをサポートしていますが、StarRocks 2.3より前のバージョンではサポートされていません。Icebergテーブルのスキーマが変更された場合、対応する外部テーブルを削除し、新しい外部テーブルを作成する必要があります。

### 手順

#### ステップ 1: Icebergリソースを作成する

Iceberg外部テーブルを作成する前に、StarRocksでIcebergリソースを作成する必要があります。このリソースはIcebergへのアクセス情報を管理するために使用されます。また、外部テーブルを作成する際にこのリソースを指定する必要があります。リソースはビジネス要件に基づいて作成できます：

* IcebergテーブルのメタデータがHiveメタストアから取得される場合、リソースを作成し、カタログタイプを `HIVE` に設定します。

* Icebergテーブルのメタデータが他のサービスから取得される場合、カスタムカタログを作成する必要があります。その後、リソースを作成し、カタログタイプを `CUSTOM` に設定します。

##### カタログタイプが `HIVE` のリソースを作成する

例えば、`iceberg0` という名前のリソースを作成し、カタログタイプを `HIVE` に設定します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083" 
);
~~~

以下の表は関連するパラメーターを説明しています。

| **パラメーター**                       | **説明**                                              |
| ----------------------------------- | ------------------------------------------------------------ |
| type                                | リソースタイプ。`iceberg` に設定します。               |
| iceberg.catalog.type              | リソースのカタログタイプ。Hiveカタログとカスタムカタログがサポートされています。Hiveカタログを指定する場合は、`HIVE` に設定します。カスタムカタログを指定する場合は、`CUSTOM` に設定します。 |
| iceberg.catalog.hive.metastore.uris | HiveメタストアのURI。パラメータ値は次の形式です：`thrift://< IcebergメタデータのIPアドレス >:< ポート番号 >`。ポート番号はデフォルトで9083です。Apache IcebergはHiveカタログを使用してHiveメタストアにアクセスし、Icebergテーブルのメタデータをクエリします。 |

##### カタログタイプが `CUSTOM` のリソースを作成する

カスタムカタログは抽象クラスBaseMetastoreCatalogを継承し、IcebergCatalogインターフェースを実装する必要があります。さらに、カスタムカタログのクラス名はStarRocksに既に存在するクラスの名前と重複してはいけません。カタログが作成されたら、カタログと関連ファイルをパッケージ化し、各フロントエンド(FE)の **fe/lib** パスに配置し、各FEを再起動します。これらの操作を完了した後、カスタムカタログのリソースを作成できます。

例えば、`iceberg1` という名前のリソースを作成し、カタログタイプを `CUSTOM` に設定します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 
);
~~~

以下の表は関連するパラメーターを説明しています。

| **パラメーター**          | **説明**                                              |
| ---------------------- | ------------------------------------------------------------ |
| type                   | リソースタイプ。`iceberg` に設定します。               |
| iceberg.catalog.type | リソースのカタログタイプ。Hiveカタログとカスタムカタログがサポートされています。Hiveカタログを指定する場合は、`HIVE` に設定します。カスタムカタログを指定する場合は、`CUSTOM` に設定します。 |
| iceberg.catalog-impl   | カスタムカタログの完全修飾クラス名。FEはこの名前を基にカタログを検索します。カタログにカスタム設定項目が含まれている場合、Iceberg外部テーブルを作成する際に、それらをキーと値のペアとして `PROPERTIES` パラメータに追加する必要があります。 |

StarRocks 2.3以降のバージョンでは、`hive.metastore.uris` と `iceberg.catalog-impl` のIcebergリソースを変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

##### Icebergリソースを表示する

~~~SQL
SHOW RESOURCES;
~~~

##### Icebergリソースを削除する

例えば、`iceberg0` という名前のリソースを削除します。

~~~SQL
DROP RESOURCE "iceberg0";
~~~

Icebergリソースを削除すると、このリソースを参照しているすべての外部テーブルが利用できなくなります。しかし、Apache Iceberg内の対応するデータは削除されません。Apache Iceberg内のデータを引き続きクエリする必要がある場合は、新しいリソースと新しい外部テーブルを作成します。

#### ステップ 2: (オプション) データベースを作成する

例えば、StarRocksに `iceberg_test` という名前のデータベースを作成します。

~~~SQL
CREATE DATABASE iceberg_test; 
USE iceberg_test; 
~~~

> 注: StarRocksのデータベース名はApache Icebergのデータベース名と異なっていても構いません。

#### ステップ 3: Iceberg外部テーブルを作成する

例えば、`iceberg_test` データベースに `iceberg_tbl` という名前のIceberg外部テーブルを作成します。

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

以下の表は関連するパラメーターを説明しています。

| **パラメーター** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| ENGINE        | エンジン名。`ICEBERG` に設定します。                 |
| resource      | 外部テーブルが参照するIcebergリソースの名前。 |
| database      | Icebergテーブルが属するデータベースの名前。 |
| table         | Icebergテーブルの名前。                               |

> 注記：
   >
   > * 外部テーブルの名前はIcebergテーブルの名前と異なっていても構いません。
   >
   > * 外部テーブルの列名はIcebergテーブルの列名と同じでなければなりません。両テーブルの列の順序は異なっていても構いません。

カスタムカタログで設定項目を定義し、データクエリ時に設定項目を有効にしたい場合、外部テーブルを作成する際に `PROPERTIES` パラメータにキーと値のペアとして設定項目を追加できます。例えば、カスタムカタログで `custom-catalog.properties` という設定項目を定義した場合、以下のコマンドを実行して外部テーブルを作成できます。

~~~SQL
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

外部テーブルを作成する際には、Icebergテーブルの列のデータ型に基づいて外部テーブルの列のデータ型を指定する必要があります。以下の表は列のデータ型のマッピングを示しています。

| **Icebergテーブル** | **Iceberg外部テーブル** |
| ----------------- | -------------------------- |
| BOOLEAN           | BOOLEAN                    |
| INT               | TINYINT / SMALLINT / INT   |
| LONG              | BIGINT                     |
| FLOAT             | FLOAT                      |
| DOUBLE            | DOUBLE                     |
| DECIMAL(P, S)     | DECIMAL                    |
| DATE              | DATE / DATETIME            |
| TIME              | BIGINT                     |
| TIMESTAMP         | DATETIME                   |
| STRING            | STRING / VARCHAR           |
| UUID              | STRING / VARCHAR           |
| FIXED(L)          | CHAR                       |
| BINARY            | VARCHAR                    |
| LIST              | ARRAY                      |

StarRocks は、データ型が TIMESTAMPTZ、STRUCT、MAP の Iceberg データのクエリをサポートしていません。

#### ステップ 4: Apache Iceberg のデータをクエリする

外部テーブルを作成したら、その外部テーブルを使用して Apache Iceberg のデータをクエリできます。

~~~SQL
SELECT COUNT(*) FROM iceberg_tbl;
~~~

## (非推奨) Hudi 外部テーブル

v2.2.0 以降、StarRocks は Hudi 外部テーブルを使用して Hudi データレイクからデータをクエリすることをサポートし、迅速なデータレイク分析を容易にしています。このトピックでは、StarRocks クラスターに Hudi 外部テーブルを作成し、Hudi 外部テーブルを使用して Hudi データレイクからデータをクエリする方法について説明します。

### 始める前に

StarRocks クラスターが Hive メタストア、HDFS クラスター、または Hudi テーブルを登録できるバケットへのアクセス権を持っていることを確認してください。

### 注意事項

* Hudi 外部テーブルは読み取り専用で、クエリのみに使用できます。
* StarRocks は Copy on Write と Merge On Read テーブルのクエリをサポートしています（MOR テーブルは v2.5 からサポート）。これらのテーブルタイプの違いについては、[Table & Query Types](https://hudi.apache.org/docs/table_types/) を参照してください。
* StarRocks は Hudi のスナップショットクエリと読み取り最適化クエリの 2 種類をサポートしています（Hudi は Merge On Read テーブルでの読み取り最適化クエリのみをサポート）。インクリメンタルクエリはサポートされていません。Hudi のクエリタイプについての詳細は、[Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types) を参照してください。
* StarRocks は Hudi ファイルの gzip、zstd、LZ4、Snappy の圧縮フォーマットをサポートしています。Hudi ファイルのデフォルトの圧縮フォーマットは gzip です。
* StarRocks は Hudi 管理テーブルからのスキーマ変更を同期できません。詳細は [Schema Evolution](https://hudi.apache.org/docs/schema_evolution/) を参照してください。Hudi 管理テーブルのスキーマが変更された場合、関連する Hudi 外部テーブルを StarRocks クラスターから削除し、その外部テーブルを再作成する必要があります。

### 手順

#### ステップ 1: Hudi リソースの作成と管理

StarRocks クラスターに Hudi リソースを作成する必要があります。Hudi リソースは、StarRocks クラスターで作成する Hudi データベースと外部テーブルを管理するために使用されます。

##### Hudi リソースの作成

次のステートメントを実行して、`hudi0` という名前の Hudi リソースを作成します：

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
~~~

以下の表は、パラメーターについて説明しています。

| パラメーター           | 説明                                                  |
| ------------------- | ------------------------------------------------------------ |
| type                | Hudi リソースのタイプ。値を `hudi` に設定します。         |
| hive.metastore.uris | Hudi リソースが接続する Hive メタストアの Thrift URI。Hudi リソースを Hive メタストアに接続すると、Hive を使用して Hudi テーブルを作成および管理できます。Thrift URI は `<IP address of the Hive metastore>:<Port number of the Hive metastore>` の形式です。デフォルトのポート番号は 9083 です。 |

v2.3 以降、StarRocks は Hudi リソースの `hive.metastore.uris` の値を変更することができます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

##### Hudi リソースの表示

次のステートメントを実行して、StarRocks クラスターに作成されたすべての Hudi リソースを表示します：

~~~SQL
SHOW RESOURCES;
~~~

##### Hudi リソースの削除

次のステートメントを実行して、`hudi0` という名前の Hudi リソースを削除します：

~~~SQL
DROP RESOURCE "hudi0";
~~~

> 注記：
>
> Hudi リソースを削除すると、その Hudi リソースを使用して作成されたすべての Hudi 外部テーブルが利用できなくなります。しかし、削除は Hudi に保存されているデータには影響しません。StarRocks を使用して Hudi からデータをクエリする場合は、StarRocks クラスターに Hudi リソース、Hudi データベース、および Hudi 外部テーブルを再作成する必要があります。

#### ステップ 2: Hudi データベースの作成

次のステートメントを実行して、StarRocks クラスターに `hudi_test` という名前の Hudi データベースを作成し、使用します：

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> 注記：
>
> StarRocks クラスターで指定する Hudi データベースの名前は、Hudi にある関連データベースの名前と同じである必要はありません。

#### ステップ 3: Hudi 外部テーブルの作成

次のステートメントを実行して、`hudi_test` Hudi データベースに `hudi_tbl` という名前の Hudi 外部テーブルを作成します：

~~~SQL
CREATE EXTERNAL TABLE `hudi_tbl` ( 
    `id` BIGINT NULL, 
    `data` VARCHAR(200) NULL 
) ENGINE=HUDI 
PROPERTIES ( 
    "resource" = "hudi0", 
    "database" = "hudi", 
    "table" = "hudi_table" 
); 
~~~

以下の表は、パラメーターについて説明しています。

| パラメーター | 説明                                                  |
| --------- | ------------------------------------------------------------ |
| ENGINE    | Hudi 外部テーブルのクエリエンジン。値を `HUDI` に設定します。 |
| resource  | StarRocks クラスターの Hudi リソースの名前。     |
| database  | StarRocks クラスター内の Hudi 外部テーブルが属する Hudi データベースの名前。 |
| table     | Hudi 外部テーブルが関連付けられている Hudi 管理テーブル。 |

> 注記：
>
> * Hudi 外部テーブルに指定する名前は、関連する Hudi 管理テーブルの名前と同じである必要はありません。
>
> * Hudi 外部テーブルの列は、関連する Hudi 管理テーブルの列と同じ名前である必要がありますが、順序は異なることができます。
>
> * 関連する Hudi 管理テーブルから一部または全ての列を選択し、選択した列のみを Hudi 外部テーブルに作成することができます。以下の表は、Hudi がサポートするデータ型と StarRocks がサポートするデータ型のマッピングを示しています。

| Hudi がサポートするデータ型   | StarRocks がサポートするデータ型 |
| ----------------------------   | --------------------------------- |
| BOOLEAN                        | BOOLEAN                           |
| INT                            | TINYINT/SMALLINT/INT              |
| DATE                           | DATE                              |
| TimeMillis/TimeMicros          | TIME                              |
| TimestampMillis/TimestampMicros| DATETIME                          |
| LONG                           | BIGINT                            |
| FLOAT                          | FLOAT                             |
| DOUBLE                         | DOUBLE                            |
| STRING                         | CHAR/VARCHAR                      |
| ARRAY                          | ARRAY                             |
| DECIMAL                        | DECIMAL                           |

> **注記**
>
> StarRocks は STRUCT や MAP 型のデータ、および Merge On Read テーブルの ARRAY 型のデータのクエリをサポートしていません。

#### ステップ 4: Hudi 外部テーブルからデータをクエリする

特定の Hudi 管理テーブルに関連付けられた Hudi 外部テーブルを作成した後、Hudi 外部テーブルにデータをロードする必要はありません。Hudi からデータをクエリするには、次のステートメントを実行します：

~~~SQL
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (非推奨) MySQL 外部テーブル

スタースキーマでは、データは通常、ディメンションテーブルとファクトテーブルに分けられます。ディメンションテーブルはデータ量が少ないですが、UPDATE操作が必要になることがあります。現在、StarRocksは直接のUPDATE操作には対応していません（更新はUnique Keyテーブルを利用して実装可能です）。一部のシナリオでは、ディメンションテーブルをMySQLに保存し、直接データを読み込むことができます。

MySQLデータをクエリするためには、StarRocksに外部テーブルを作成し、それをMySQLデータベースのテーブルにマッピングする必要があります。テーブルを作成する際には、MySQLの接続情報を指定する必要があります。

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

パラメータ:

* **host**: MySQLデータベースの接続アドレス
* **port**: MySQLデータベースのポート番号
* **user**: MySQLにログインするユーザー名
* **password**: MySQLにログインするパスワード
* **database**: MySQLデータベースの名前
* **table**: MySQLデータベースのテーブル名
