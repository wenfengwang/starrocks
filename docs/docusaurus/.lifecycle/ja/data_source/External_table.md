---
displayed_sidebar: "Japanese"
---

# 外部テーブル

:::note
v3.0以降、Hive、Iceberg、およびHudiからデータをクエリする場合は、カタログを使用することを推奨します。[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

v3.1以降、MySQLとPostgreSQLからデータをクエリする場合は[JDBCカタログ](../data_source/catalog/jdbc_catalog.md)を使用し、Elasticsearchからデータをクエリする場合は[Elasticsearchカタログ](../data_source/catalog/elasticsearch_catalog.md)を使用することを推奨します。
:::

StarRocksは、外部テーブルを使用して他のデータソースにアクセスできます。外部テーブルは、他のデータソースに格納されているデータテーブルを基に作成されます。StarRocksはデータテーブルのメタデータのみを保存します。外部テーブルを使用して、他のデータソースのデータを直接クエリできます。StarRocksは以下のデータソースをサポートしています: MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg、およびApache Hudi。**現時点では、別のStarRocksクラスタからデータを現在のStarRocksクラスタに書き込むことしかできません。読み取ることはできません。StarRocks以外のデータソースについて、これらのデータソースからの読み取りのみが可能です。**

2.5以降、StarRocksでは、外部データソースのホットデータクエリを高速化するデータキャッシュ機能が提供されます。詳細については、[データキャッシュ](data_cache.md)を参照してください。

## StarRocks外部テーブル

StarRocks 1.19以降、StarRocksには、別のStarRocksクラスタにデータを書き込むStarRocks外部テーブルを使用することができます。これにより、読み書きの分離が実現され、リソースのより良い分離が提供されます。まず、宛先StarRocksクラスタ内で宛先テーブルを作成し、その後、ソースStarRocksクラスタ内で、`PROPERTIES`フィールドに宛先クラスタとテーブルの情報が指定された、宛先テーブルと同じスキーマのStarRocks外部テーブルを作成できます。

StarRocks外部テーブルへのデータは、INSERT INTOステートメントを使用してソースクラスタから宛先クラスタに書き込むことができます。これにより、次の目標が達成できます:

* StarRocksクラスタ間のデータ同期。
* 読み書きの分離。データはソースクラスタに書き込まれ、ソースクラスタからのデータ変更が宛先クラスタに同期され、クエリサービスが提供されます。

次のコードは、宛先テーブルと外部テーブルの作成方法を示しています。

~~~SQL
# 宛先StarRocksクラスタ内で宛先テーブルを作成します。
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

# ソースStarRocksクラスタ内で外部テーブルを作成します。
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

# StarRocks外部テーブルにデータを書き込むことで、ソースクラスタから宛先クラスタにデータを書き込みます。本番環境では、2番目のステートメントが推奨されます。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

パラメータ:

* **EXTERNAL:** このキーワードは、作成するテーブルが外部テーブルであることを示します。
* **host:** このパラメータは、宛先StarRocksクラスタのリーダーFEノードのIPアドレスを指定します。
* **port:**  このパラメータは、宛先StarRocksクラスタのFEノードのRPCポートを指定します。

  :::note

  StarRocks外部テーブルが所属するソースクラスタが宛先StarRocksクラスタにアクセスできるようにするには、次のポートにアクセスできるようにネットワークとファイアウォールを構成する必要があります:

  * FEノードのRPCポート。FE構成ファイル**fe/fe.conf**の`rpc_port`を参照してください。デフォルトのRPCポートは `9020` です。
  * BEノードのbRPCポート。BE構成ファイル**be/be.conf**の`brpc_port`を参照してください。デフォルトのbRPCポートは `8060` です。

  :::

* **user:** このパラメータは、宛先StarRocksクラスタにアクセスする際に使用するユーザー名を指定します。
* **password:** このパラメータは、宛先StarRocksクラスタにアクセスする際に使用するパスワードを指定します。
* **database:** このパラメータは、宛先テーブルが属するデータベースを指定します。
* **table:** このパラメータは、宛先テーブルの名前を指定します。

StarRocks外部テーブルを使用する際の次の制限が適用されます:

* StarRocks外部テーブルでは、INSERT INTOおよびSHOW CREATE TABLEコマンドのみを実行できます。他のデータ書き込み方法はサポートされていません。また、StarRocks外部テーブルからのデータのクエリや外部テーブルへのDDL操作は実行できません。
* 外部テーブルの作成構文は通常のテーブルの作成構文と同じですが、外部テーブルの列名およびその他の情報は宛先テーブルと同じである必要があります。
* 外部テーブルは、宛先テーブルのテーブルメタデータを10秒ごとに同期します。宛先テーブルでDDL操作が実行されると、2つのテーブル間のデータ同期に遅延が発生する場合があります。

## JDBC互換データベース用の外部テーブル

v2.3.0以降、StarRocksでは、JDBC互換データベースをクエリするための外部テーブルが提供されます。これにより、StarRocksにデータをインポートする必要なく、これらのデータベースのデータを驚くほど高速に分析できます。このセクションでは、StarRocksで外部テーブルを作成し、JDBC互換データベースのデータをクエリする方法について説明します。

### 前提条件

JDBC外部テーブルを使用してデータをクエリする前に、FEおよびBEがJDBCドライバのダウンロードURLにアクセスできることを確認してください。ダウンロードURLは、JDBCリソースを作成するために使用されるステートメントで指定された`driver_url`パラメータで指定されます。

### JDBCリソースの作成と管理

#### JDBCリソースの作成

データベースからデータをクエリする外部テーブルを作成する前に、データベースの接続情報を管理するために、StarRocksにJDBCリソースを作成する必要があります。データベースはJDBCドライバをサポートしている必要があり、そのデータベースは"対象データベース"として参照されます。リソースを作成した後は、それを使用して外部テーブルを作成できます。

次のステートメントを実行して、`jdbc0`という名前のJDBCリソースを作成します:

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

`PROPERTIES`内の必須のパラメータは次のとおりです:

* `type`: リソースのタイプ。値を`jdbc`に設定します。

* `user`: ターゲットデータベースに接続する際に使用されるユーザー名。

* `password`: ターゲットデータベースに接続する際に使用されるパスワード。

* `jdbc_uri`: JDBCドライバが対象データベースに接続するために使用するURI。URIの形式は、データベースURI構文を満たす必要があります。一部の一般的なデータベースのURI構文については、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)の公式ウェブサイトを参照してください。

> 注意: URIには対象データベースの名前を含める必要があります。例えば、前のコード例では、`jdbc_test` は接続したい対象データベースの名前です。

* `driver_url`: JDBCドライバのJARパッケージのダウンロードURL。HTTP URLまたはファイルURLがサポートされています。例: `https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar` や `file:///home/disk1/postgresql-42.3.3.jar`。

* `driver_class`: JDBCドライバのクラス名。一般的なデータベースのJDBCドライバクラス名は次のとおりです:
  * MySQL: com.mysql.jdbc.Driver (MySQL 5.x 以前), com.mysql.cj.jdbc.Driver (MySQL 6.x 以降)
  * SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL: org.postgresql.Driver

リソースが作成されると、FEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバJARパッケージをダウンロードし、チェックサムを生成し、BEによってダウンロードされたJDBCドライバのチェックサムを検証します。

> 注：JDBCドライバーJARパッケージのダウンロードが失敗すると、リソースの作成も失敗します。

BE(Business Entity)が初めてJDBC外部テーブルをクエリし、対応するJDBCドライバーJARパッケージが自分たちのマシンに存在しないことを発見した場合、BEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバーJARパッケージをダウンロードし、すべてのJDBCドライバーJARパッケージは`${STARROCKS_HOME}/lib/jdbc_drivers`ディレクトリに保存されます。

#### JDBCリソースを表示

次のステートメントを実行して、StarRocksのすべてのJDBCリソースを表示します:

~~~SQL
SHOW RESOURCES;
~~~

> 注：`ResourceType`列は`jdbc`です。

#### JDBCリソースを削除

次のステートメントを実行して、`jdbc0`という名前のJDBCリソースを削除します:

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> 注：JDBCリソースが削除されると、そのJDBCリソースを使用して作成されたすべてのJDBC外部テーブルが利用できなくなります。ただし、対象データベースのデータは失われません。対象データベースのデータを引き続きStarRocksを使用してクエリする必要がある場合は、JDBCリソースおよびJDBC外部テーブルを再作成できます。

### データベースの作成

次のステートメントを実行して、StarRocksに`jdbc_test`という名前のデータベースを作成してアクセスします:

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> 注：前述のステートメントで指定するデータベース名は、対象データベースの名前と同じである必要はありません。

### JDBC外部テーブルの作成

次のステートメントを実行して、データベース`jdbc_test`に`jdbc_tbl`という名前のJDBC外部テーブルを作成します:

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

`properties`の必須パラメータは次のとおりです:

* `resource`: 外部テーブルを作成するために使用されるJDBCリソースの名前。

* `table`: データベース内の対象表の名前。

StarRocksと対象データベース間のサポートされるデータ型とデータ型のマッピングについては、[データ型のマッピング](External_table.md#Data type mapping)を参照してください。

> 注：
>
> * インデックスはサポートされていません。
> * PARTITION BYまたはDISTRIBUTED BYを使用してデータの分散ルールを指定することはできません。

### JDBC外部テーブルのクエリ

JDBC外部テーブルをクエリする前に、次のステートメントを実行してパイプラインエンジンを有効にする必要があります:

~~~SQL
set enable_pipeline_engine=true;
~~~

> 注：パイプラインエンジンがすでに有効になっている場合は、このステップをスキップできます。

次のステートメントを実行して、JDBC外部テーブルを使用して対象データベースのデータをクエリします。

~~~SQL
select * from JDBC_tbl;
~~~

StarRocksは、フィルタ条件を対象表にプッシュダウンして述語プッシュダウンをサポートしています。データソースにできるだけ近いところでフィルタ条件を実行することで、クエリのパフォーマンスを向上させることができます。現時点で、StarRocksはバイナリ比較演算子 (`>`, `>=`, `=`, `<`, および `<=`), `IN`, `IS NULL`, `BETWEEN ... AND ...` を含む演算子をプッシュダウンできます。ただし、StarRocksは関数をプッシュダウンできません。

### データ型のマッピング

現時点で、StarRocksはNUMBER、STRING、TIME、およびDATEのような対象データベースの基本的な型のデータのみクエリできます。対象データベースのデータ値の範囲がStarRocksでサポートされていない場合、クエリはエラーを報告します。

対象データベースとStarRocks間のマッピングは、対象データベースのタイプに基づいて異なります。

#### **MySQLとStarRocks**

| MySQL        | StarRocks |
| ------------ | --------- |
| BOOLEAN      | BOOLEAN   |
| TINYINT      | TINYINT   |
| SMALLINT     | SMALLINT  |
| MEDIUMINTINT | INT       |
| BIGINT       | BIGINT    |
| FLOAT        | FLOAT     |
| DOUBLE       | DOUBLE    |
| DECIMAL      | DECIMAL   |
| CHAR         | CHAR      |
| VARCHAR      | VARCHAR   |
| DATE         | DATE      |
| DATETIME     | DATETIME  |

#### **OracleとStarRocks**

| Oracle          | StarRocks |
| --------------- | --------- |
| CHAR            | CHAR      |
| VARCHARVARCHAR2 | VARCHAR   |
| DATE            | DATE      |
| SMALLINT        | SMALLINT  |
| INT             | INT       |
| BINARY_FLOAT    | FLOAT     |
| BINARY_DOUBLE   | DOUBLE    |
| DATE            | DATE      |
| DATETIME        | DATETIME  |
| NUMBER          | DECIMAL   |

#### **PostgreSQLとStarRocks**

| PostgreSQL          | StarRocks |
| ------------------- | --------- |
| SMALLINTSMALLSERIAL | SMALLINT  |
| INTEGERSERIAL       | INT       |
| BIGINTBIGSERIAL     | BIGINT    |
| BOOLEAN             | BOOLEAN   |
| REAL                | FLOAT     |
| DOUBLE PRECISION    | DOUBLE    |
| DECIMAL             | DECIMAL   |
| TIMESTAMP           | DATETIME  |
| DATE                | DATE      |
| CHAR                | CHAR      |
| VARCHAR             | VARCHAR   |
| TEXT                | VARCHAR   |

#### **SQL ServerとStarRocks**

| SQL Server        | StarRocks |
| ----------------- | --------- |
| BOOLEAN           | BOOLEAN   |
| TINYINT           | TINYINT   |
| SMALLINT          | SMALLINT  |
| INT               | INT       |
| BIGINT            | BIGINT    |
| FLOAT             | FLOAT     |
| REAL              | DOUBLE    |
| DECIMALNUMERIC    | DECIMAL   |
| CHAR              | CHAR      |
| VARCHAR           | VARCHAR   |
| DATE              | DATE      |
| DATETIMEDATETIME2 | DATETIME  |

### 制限

* JDBC外部テーブルを作成する際、テーブルにインデックスを作成したり、PARTITION BYやDISTRIBUTED BYを使用してテーブルのデータ分散ルールを指定したりすることはできません。

* JDBC外部テーブルをクエリする際、StarRocksは関数をテーブルにプッシュダウンすることはできません。

## (非推奨) Elasticsearch外部テーブル

StarRocksとElasticsearchは2つの人気のある分析システムです。StarRocksは大規模分散コンピューティングに優れており、Elasticsearchはフルテキスト検索に適しています。StarRocksはElasticsearchと組み合わせることで、より完全なOLAPソリューションを提供できます。

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

以下の表はパラメータを説明します。

| **パラメータ**     | **必須** | **デフォルト値** | **説明**                                                   |
| ----------------- | -------- | ---------------- | ----------------------------------------------------------- |
| hosts             | はい     | なし             | Elasticsearchクラスターの接続アドレス。1つまたは複数のアドレスを指定できます。StarRocksはこのアドレスからElasticsearchのバージョンとインデックスのシャード割り当てを解析できます。 StarRocksは`GET /_nodes/http` API操作から返されたアドレスに基づいてElasticsearchクラスターと通信します。したがって、`host`パラメータの値は `GET /_nodes/http` API操作から返されたアドレスと同じでなければなりません。そうでないと、BEはElasticsearchクラスターと通信できない場合があります。 |
| index             | はい     | なし             | StarRocksで作成されたElasticsearchテーブルのElasticsearchインデックスの名前。名前はエイリアスにすることができます。このパラメータはワイルドカード(\*)をサポートします。たとえば、`index`を <code class="language-text">hello*</code>に設定した場合、StarRocksは`hello`で始まるすべてのインデックスを取得します。 |
| user              | いいえ   | 空               | 基本認証が有効になっているElasticsearchクラスターにログインするためのユーザー名。`/*cluster/state/*nodes/http`およびインデックスにアクセスできることを確認してください。 |
| password          | いいえ   | 空               | Elasticsearchクラスターにログインするためのパスワード。 |
| type              | いいえ   | `_doc`           | インデックスのタイプ。デフォルト値:`_doc`。Elasticsearch 8以降のバージョンでデータをクエリしたい場合は、このパラメータを構成する必要はありません。なぜなら、Elasticsearch 8以降のバージョンではマッピングタイプが削除されているからです。 |
| es.nodes.wan.only    | No           | `false`           | Elasticsearchクラスターにアクセスしてデータを取得する際に、StarRocksが`hosts`で指定されたアドレスのみを使用するかどうかを指定します。<ul><li>`true`: StarRocksは`hosts`で指定されたアドレスのみを使用してElasticsearchクラスターにアクセスし、データを取得し、Elasticsearchインデックスのシャードが存在するデータノードをスニッフィングしません。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできない場合は、このパラメータを`true`に設定する必要があります。</li><li>`false`: StarRocksは、Elasticsearchクラスターのインデックスのシャードが存在するデータノードをスニッフィングするために`host`で指定されたアドレスを使用します。StarRocksがクエリ実行プランを生成した後、関連するBEは直接Elasticsearchクラスター内のデータノードにアクセスして、インデックスのシャードからデータを取得します。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできる場合は、デフォルト値である`false`を維持することをお勧めします。</li></ul> |
| es.net.ssl           | No           | `false`           | Elasticsearchクラスターにアクセスする際にHTTPSプロトコルを使用できるかどうかを指定します。StarRocksのバージョン2.4以降のみがこのパラメータを構成することをサポートしています。<ul><li>`true`: HTTPSプロトコルおよびHTTPプロトコルの両方を使用してElasticsearchクラスターにアクセスできます。</li><li>`false`: HTTPプロトコルのみを使用してElasticsearchクラスターにアクセスできます。</li></ul> |
| enable_docvalue_scan | No           | `true`            | Elasticsearchのカラムストレージから対象フィールドの値を取得するかどうかを指定します。ほとんどの場合、カラムストレージからデータを読み取る方がローストレージからデータを読み取るよりもパフォーマンスが向上します。 |
| enable_keyword_sniff | No           | `true`            | KEYWORD型フィールドに基づいてElasticsearchのTEXT型フィールドをスニッフィングするかどうかを指定します。このパラメータを`false`に設定すると、StarRocksはトークン化後に一致を行います。 |

##### より高速なクエリのためのカラムスキャン

`enable_docvalue_scan`を`true`に設定すると、StarRocksはElasticsearchからデータを取得する際に次のルールに従います。

* **トライアンドシー**: StarRocksは自動的に対象フィールドについてカラムストレージが有効かどうかをチェックします。もし有効であれば、StarRocksはすべての値をカラムストレージから取得します。
* **自動ダウングレード**: もし対象フィールドのいずれかがカラムストレージで利用できない場合、StarRocksは対象フィールドのすべての値をローストレージ（`_source`）からパースして取得します。

> **注意**
>
> * ElasticsearchではTEXT型フィールドにはカラムストレージが利用できません。したがって、TEXT型の値を含むフィールドをクエリする場合、StarRocksはフィールドの値を`_source`から取得します。
> * 多くのフィールド（25以上）をクエリする場合、`docvalue`からフィールドの値を読み取ることは、`_source`からフィールドの値を読み取る場合と比較して目立った利点を示しません。

##### KEYWORD型フィールドのスニッフィング

`enable_keyword_sniff`を`true`に設定すると、Elasticsearchはインデックスを自動的に作成するため、インデックスなしでの直接的なデータ取り込みが可能となります。STRING型フィールドに対して、ElasticsearchはTEXT型とKEYWORD型の両方を持つフィールドを作成します。これがElasticsearchのMulti-Field機能が機能する仕組みです。マッピングは以下のようになります：

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

例えば、`k4`に対して"="フィルタリングを行う場合、Elasticsearch上のStarRocksはフィルタリング操作をElasticsearch TermQueryに変換します。

元のSQLフィルタは以下のようになります：

~~~SQL
k4 = "StarRocks On Elasticsearch"
~~~

変換されたElasticsearchクエリDSLは以下のようになります：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

`k4`の最初のフィールドはTEXT型であり、データ取り込み後に`k4`の構成されたアナライザによってトークン化されます（もしくは`k4`の構成されたアナライザが存在しない場合は標準アナライザによって）。その結果、最初のフィールドは`StarRocks`、`On`、`Elasticsearch`の3つの用語にトークン化されます。詳細は以下のようになります：

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

トークン化の結果は以下のようになります：

~~~SQL
{
   "tokens": [
      {
         "token": "starrocks",
         "start_offset": 0,
         "end_offset": 5,
         "type": "<ALPHANUM>",
         "position": 0
      },
      {
         "token": "on",
         "start_offset": 6,
         "end_offset": 8,
         "type": "<ALPHANUM>",
         "position": 1
      },
      {
         "token": "elasticsearch",
         "start_offset": 9,
         "end_offset": 11,
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
~~~

辞書に一致する用語がないため、用語`StarRocks On Elasticsearch`に対して結果は返されません。

しかし、`enable_keyword_sniff`を`true`に設定した場合、StarRocksは`k4 = "StarRocks On Elasticsearch"`を`k4.keyword = "StarRocks On Elasticsearch"`に変換してSQLセマンティクスに一致させます。変換された`StarRocks On Elasticsearch`クエリDSLは以下のようになります：

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword`はKEYWORD型です。したがって、データが完全な用語としてElasticsearchに書き込まれ、成功した一致が可能となります。

#### カラムデータ型のマッピング

外部テーブルを作成する際、Elasticsearchテーブルのカラムデータ型に基づいて外部テーブルのカラムデータ型を指定する必要があります。以下の表はカラムデータ型のマッピングを示しています。

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

> **注意**
>
> * StarRocksはNESTED型のデータをJSON関連の関数を使用して読み取ります。
> * Elasticsearchは自動的に多次元配列を一次元配列にフラット化します。StarRocksも同様に行います。**ElasticsearchからのARRAYデータのクエリサポートはv2.5から追加されました。**

### プレディケートプッシュダウン

StarRocksはプレディケートプッシュダウンをサポートしています。フィルタはElasticsearchにプッシュダウンされて実行され、クエリのパフォーマンスが向上します。次の表にはプレディケートプッシュダウンをサポートする演算子がリストされています。

|   SQL構文  |   ES構文  |
| :---: | :---: |
|  `=`   |  term query   |
|  `in`   |  terms query   |
|  `>=,  <=, >, <`   |  range   |
|  `and`   |  bool.filter   |
|  `or`   |  bool.should   |
|  `not`   |  bool.must_not   |
|  `not in`   |  bool.must_not + terms   |
|  `esquery`   |  ES Query DSL  |

### 例

**esquery function**は、SQLで表現できないクエリ（例: matchやgeoshape）をElasticsearchにフィルタリングするために使用されます。esquery関数の第1パラメータはインデックスを関連付けるために使用されます。第2パラメータは基本的なQuery DSLのJSON式であり、{}で囲まれている必要があります。**JSON式には1つのルートキーのみ**を持っていなければなりません。例: match、geo_shape、またはboolです。

* matchクエリ

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on elasticsearch"
    }
}');
~~~

* ジオ関連クエリ

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

### 使用上の注意
* Elasticsearch 5.xより前のバージョンは、5.x以降と異なる方法でデータをスキャンします。現在、**5.x以降のバージョン**のみがサポートされています。
* HTTP基本認証が有効になっているElasticsearchクラスターはサポートされています。
* StarRocksからデータをクエリする場合、Elasticsearchから直接データをクエリするよりも遅くなることがあります。これは、Elasticsearchが実際のデータをフィルタリングする必要なく、対象ドキュメントのメタデータを直接読み取るため、カウントクエリが高速化されるためです。

## (非推奨) Hive外部テーブル

Hive外部テーブルを使用する前に、サーバーにJDK 1.8がインストールされていることを確認してください。

### Hiveリソースの作成

HiveリソースはHiveクラスターに対応します。StarRocksで使用されるHiveクラスターを構成する必要があります。Hive外部テーブルで使用されるHiveリソースを指定する必要があります。

* `hive0`という名前のHiveリソースを作成します。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://10.10.44.98:9083"
);
~~~

* StarRocksで作成されたリソースを表示します。

~~~sql
SHOW RESOURCES;
~~~

* `hive0`という名前のリソースを削除します。

~~~sql
DROP RESOURCE "hive0";
~~~

StarRocks 2.3以降では、Hiveリソースの`hive.metastore.uris`を変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

### データベースの作成

~~~sql
CREATE DATABASE hive_test;
USE hive_test;
~~~

### Hive外部テーブルの作成

構文

~~~sql
CREATE EXTERNAL TABLE table_name (
  col_name col_type [NULL | NOT NULL] [COMMENT "comment"]
) ENGINE=HIVE
PROPERTIES (
  "key" = "value"
);
~~~

例: `hive0`リソースに対応するHiveクラスターの`rawdata`データベースの下に外部テーブル`profile_parquet_p7`を作成します。

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
  * 列名はHiveテーブル内の列名と同じである必要があります。
  * 列の順序は、Hiveテーブルの列順と**同じである必要はありません**。
  * Hiveテーブルの**一部の列のみを選択**できますが、すべての**パーティションキー列**を選択する必要があります。
  * 外部テーブルのパーティションキー列は`partition by`を使用して指定する必要はありません。他の列と同じ記述リストで定義する必要があります。パーティション情報を指定する必要はありません。StarRocksは自動的にこの情報をHiveテーブルから同期します。
  * `ENGINE`をHIVEに設定します。
* PROPERTIES:
  * **hive.resource**: 使用されるHiveリソース。
  * **database**: Hiveデータベース。
  * **table**: Hive内のテーブル。**ビュー**はサポートされていません。
* 次の表は、HiveとStarRocksの列データ型のマッピングを記載しています。

    |  Hiveの列データ型   |  StarRocksの列データ型   | 説明 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |
    |   TIMESTAMP  | DATETIME    | TIMESTAMPデータをDATETIMEデータに変換する際、精度およびタイムゾーン情報が失われます。タイムゾーンオフセットを持たないDATETIMEデータにTIMESTAMPデータを変換する必要がありますが、セッション変数のタイムゾーンに基づいて行う必要があります。 |
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意:
>
> * 現在、サポートされているHiveストレージ形式はParquet、ORC、CSVです。
ストレージ形式がCSVの場合、エスケープ文字として引用符を使用することはできません。
> * SNAPPYおよびLZ4圧縮形式がサポートされています。
> * クエリ可能なHive文字列列の最大長は1 MBです。文字列列が1 MBを超えると、ヌル列として処理されます。

### Hive外部テーブルの使用

`profile_wos_p7`の総行数をクエリします。

~~~sql
select count(*) from profile_wos_p7;
~~~

### キャッシュされたHiveテーブルのメタデータを更新

* Hiveパーティション情報および関連ファイル情報はStarRocksでキャッシュされています。キャッシュは`hive_meta_cache_refresh_interval_s`で指定された間隔で更新されます。デフォルト値は7200です。`hive_meta_cache_ttl_s`はキャッシュのタイムアウト期間を指定し、デフォルト値は86400です。
  * キャッシュされたデータを手動で更新することもできます。
    1. Hiveのテーブルにパーティションが追加または削除された場合、StarRocksでキャッシュされたテーブルメタデータを更新するために`REFRESH EXTERNAL TABLE hive_t`コマンドを実行する必要があります。`hive_t`はStarRocks内のHive外部テーブルの名前です。
    2. 一部のHiveパーティションのデータが更新された場合、`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')`コマンドを実行して、StarRocksでキャッシュされたデータを更新する必要があります。`hive_t`はStarRocks内のHive外部テーブルの名前です。`'k1=01/k2=02'`および`'k1=03/k2=04'`はデータが更新されたHiveパーティションの名前です。
    3. `REFRESH EXTERNAL TABLE hive_t`を実行すると、StarRocksはまずHive外部テーブルの列情報がHiveメタストアから返されるHiveテーブルの列情報と同じかどうかを確認します。Hiveテーブルのスキーマが変更された場合（列の追加や削除など）、StarRocksは変更をHive外部テーブルに同期させます。同期後、Hive外部テーブルの列順はHiveテーブルの列順と同じままで、パーティション列が最後の列となります。
* HiveデータがParquet、ORC、およびCSV形式で格納されている場合、Hiveテーブルのスキーマ変更（COLUMNの追加やREPLACE COLUMNなど）をStarRocks 2.3およびそれ以降のバージョンでHive外部テーブルに同期できます。

### オブジェクトストレージのアクセス

   * FE設定ファイルのパスは`fe/conf`です。Hadoopクラスターをカスタマイズする必要がある場合は、設定ファイルを追加できます。例: HDFSクラスターが高可用性名前サービスを使用している場合は、`fe/conf`配下に`hdfs-site.xml`を配置する必要があります。HDFSがViewFsで構成されている場合は、`fe/conf`に`core-site.xml`を配置する必要があります。
   * BE設定ファイルのパスは`be/conf`です。Hadoopクラスターをカスタマイズする必要がある場合は、設定ファイルを追加できます。例: HDFSクラスターが高可用性名前サービスを使用している場合は、`be/conf`配下に`hdfs-site.xml`を配置する必要があります。HDFSがViewFsで構成されている場合は、`be/conf`に`core-site.xml`を配置する必要があります。
   * BEが配置されているマシンで、BE **起動スクリプト** `bin/start_be.sh`の中でJAVA_HOMEをJDK環境として設定します。例: `export JAVA_HOME = <JDKのパス>`。この設定をスクリプトの先頭に追加し、設定が有効になるようにBEを再起動する必要があります。
   * Kerberosサポートの設定:
      1. `kinit -kt keytab_path principal`を使用してすべてのFE/BEマシンにログインするには、HiveとHDFSにアクセスできる必要があります。`kinit`コマンドのログインは一定期間有効であり、定期的に実行するためにcrontabに配置する必要があります。
      2. `fe/conf`に`hive-site.xml/core-site.xml/hdfs-site.xml`を配置し、`be/conf`に`core-site.xml/hdfs-site.xml`を配置します。
      3. **$FE_HOME/conf/fe.conf**ファイルの`JAVA_OPTS`オプションの値に`-Djava.security.krb5.conf=/etc/krb5.conf`を追加します。**/etc/krb5.conf**は**krb5.conf**ファイルの保存パスです。このパスはOSに基づいて変更できます。
4. **$BE_HOME/conf/be.conf**ファイルに直接`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加してください。**/etc/krb5.conf**は**krb5.conf**ファイルの保存パスです。お使いのオペレーティング システムに合わせてパスを変更できます。
5. Hiveリソースを追加する場合は、`hive.metastore.uris`にドメイン名を渡す必要があります。また、Hive/HDFSドメイン名とIPアドレスのマッピングを**/etc/hosts**ファイルに追加する必要があります。

AWS S3のサポートを構成する場合: `fe/conf/core-site.xml`および`be/conf/core-site.xml`に次の構成を追加してください。

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

   1. `fs.s3a.access.key`: AWSアクセスキーIDです。
   2. `fs.s3a.secret.key`: AWSシークレットキーです。
   3. `fs.s3a.endpoint`: 接続するAWS S3のエンドポイントです。
   4. `fs.s3a.connection.maximum`: StarRocksからS3への並行接続の最大数です。クエリ中に`Timeout waiting for connection from poll`のエラーが発生した場合は、このパラメータを大きな値に設定できます。

## (非推奨) Iceberg外部テーブル

v2.1.0以降、StarRocksでは外部テーブルを使用してApache Icebergからデータをクエリできます。Icebergのデータをクエリするには、StarRocksでIceberg外部テーブルを作成する必要があります。テーブルを作成する際は、外部テーブルとクエリしたいIcebergテーブルとの間でマッピングを確立する必要があります。

### 開始する前に

StarRocksがApache Icebergで使用するメタデータサービス（Hiveメタストアなど）、ファイルシステム（HDFSなど）、およびオブジェクトストレージシステム（Amazon S3およびAlibaba Cloudオブジェクトストレージサービス）にアクセスする権限があることを確認してください。

### 注意事項

* StarRocksのIceberg外部テーブルは、次のタイプのデータのみをクエリできます。
  * Iceberg v1テーブル（分析データテーブル）。Iceberg v2（行レベルの削除）のORC形式テーブルはv3.0以降、およびIceberg v2のParquet形式テーブルはv3.1以降でサポートされます。Iceberg v1テーブルとIceberg v2テーブルの違いについては、[Iceberg Table Spec](https://iceberg.apache.org/spec/)を参照してください。
  * gzip（デフォルトフォーマット）、Zstd、LZ4、またはSnappy形式で圧縮されたテーブル。
  * ParquetまたはORC形式で保存されたファイル。

* StarRocksの2.3およびそれ以降のバージョンのIceberg外部テーブルでは、Icebergテーブルのスキーマ変更を同期できます。ただし、StarRocksの2.3より古いバージョンのIceberg外部テーブルでは同期できません。Icebergテーブルのスキーマが変更された場合は、対応する外部テーブルを削除し、新しい外部テーブルを作成する必要があります。

### 手順

#### ステップ 1: Icebergリソースを作成

Iceberg外部テーブルを作成する前に、StarRocksでIcebergリソースを作成する必要があります。このリソースはIcebergアクセス情報を管理するために使用されます。さらに、このリソースを、外部テーブルを作成する文で指定する必要があります。ビジネス要件に合わせてリソースを作成できます。

* IcebergテーブルのメタデータがHiveメタストアから取得される場合は、リソースを作成し、カタログタイプを`HIVE`に設定できます。

* Icebergテーブルのメタデータが他のサービスから取得される場合は、カスタムカタログを作成する必要があります。その後、リソースを作成し、カタログタイプを`CUSTOM`に設定できます。

##### カタログタイプが`HIVE`のリソースを作成

例えば、`iceberg0`という名前のリソースを作成し、カタログタイプを`HIVE`に設定してください。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://192.168.0.81:9083" 
);
~~~

以下の表に、関連するパラメータを示します。

| **パラメータ**                     | **説明**                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| type                                | リソースのタイプ。値を`iceberg`に設定してください。         |
| iceberg.catalog.type                | リソースのカタログのタイプ。Hiveカタログとカスタムカタログの両方がサポートされています。Hiveカタログを指定する場合は、値を`HIVE`に設定してください。カスタムカタログを指定する場合は、値を `CUSTOM`に設定してください。 |
| iceberg.catalog.hive.metastore.uris | HiveメタストアのURIです。パラメータ値は、次の形式にしてください： `thrift://<IcebergメタデータのIPアドレス>:<ポート番号>`。ポート番号のデフォルト値は9083です。Apache Icebergは、Hiveカタログを使用してHiveメタストアにアクセスし、その後Icebergテーブルのメタデータをクエリします。 |

##### カタログタイプが`CUSTOM`のリソースを作成

カスタムカタログは、抽象クラスBaseMetastoreCatalogを継承する必要があり、IcebergCatalogインターフェースを実装する必要があります。さらに、カスタムカタログのクラス名は、StarRocksにすでに存在するクラス名と重複してはいけません。カタログが作成されたら、カタログと関連ファイルをパッケージ化し、それらを各フロントエンド（FE）の**fe/lib**パスに配置し、各FEを再起動してください。これらの操作を完了した後、カスタムカタログのリソースを作成できます。

例えば、`iceberg1`という名前のリソースを作成し、カタログタイプを`CUSTOM`に設定してください。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 
);
~~~

以下の表に、関連するパラメータを示します。

| **パラメータ**            | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| type                   | リソースのタイプ。値を`iceberg`に設定してください。         |
| iceberg.catalog.type | リソースのカタログのタイプ。Hiveカタログとカスタムカタログの両方がサポートされています。Hiveカタログを指定する場合は、値を`HIVE`に設定してください。カスタムカタログを指定する場合は、値を `CUSTOM`に設定してください。 |
| iceberg.catalog-impl   | カスタムカタログの完全修飾クラス名です。FEはこの名前に基づいてカタログを検索します。カタログにカスタム設定項目がある場合は、Iceberg外部テーブルを作成する際に`PROPERTIES`パラメータにキーと値のペアとして追加する必要があります。 |

Icebergリソースの`hive.metastore.uris`および`iceberg.catalog-impl`パラメータは、StarRocksの2.3およびそれ以降のバージョンで変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

##### Icebergリソースを表示

~~~SQL
SHOW RESOURCES;
~~~

##### Icebergリソースの削除

例えば、`iceberg0`という名前のリソースを削除してください。

~~~SQL
DROP RESOURCE "iceberg0";
~~~

Icebergリソースを削除すると、このリソースを参照するすべての外部テーブルが利用できなくなります。ただし、Apache Icebergの対応するデータは削除されません。Apache Icebergでデータを引き続きクエリする必要がある場合は、新しいリソースと新しい外部テーブルを作成してください。

#### ステップ 2: (オプション) データベースを作成

例えば、StarRocksで`iceberg_test`という名前のデータベースを作成してください。

~~~SQL
CREATE DATABASE iceberg_test; 
USE iceberg_test; 
~~~

> 注意: StarRocksのデータベース名は、Apache Icebergのデータベース名と異なる場合があります。

#### ステップ 3: Iceberg外部テーブルを作成

例えば、データベース`iceberg_test`に`iceberg_tbl`という名前のIceberg外部テーブルを作成してください。

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

以下の表に、関連するパラメータを示します。

| **パラメータ** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| ENGINE        | エンジン名。値を`ICEBERG`に設定してください。                 |
| resource      | 外部テーブルが参照するIcebergリソースの名前です。 |
| database      | Icebergテーブルが属するデータベースの名前です。 |
| table         | Icebergテーブルの名前です。                               |

> 注意:
   >
   > * 外部テーブルの名前は、Icebergテーブルの名前と異なる場合があります。
* 外部テーブルの列名は、Icebergテーブルと同じでなければなりません。 2つのテーブルの列順序は異なる場合があります。

クエリデータ時にカスタムカタログに構成項目を定義し、構成項目を有効にしたい場合は、外部テーブルを作成する際に`PROPERTIES`パラメータにキー値ペアとして構成項目を追加できます。 例えば、カスタムカタログで `custom-catalog.properties` という構成項目を定義した場合、次のコマンドを実行して外部テーブルを作成できます。

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

外部テーブルを作成する際は、Icebergテーブルの列のデータ型に基づいて外部テーブルの列のデータ型を指定する必要があります。次の表は、列のデータ型のマッピングを示しています。

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

StarRocksは、TIMESTAMPTZ、STRUCT、およびMAPのデータ型を持つIcebergデータをクエリすることをサポートしていません。

#### ステップ4: Apache Icebergでデータをクエリする

外部テーブルが作成されたら、外部テーブルを使用してApache Icebergのデータをクエリできます。

~~~SQL
select count(*) from iceberg_tbl;
~~~

## (非推奨) Hudi外部テーブル

v2.2.0から、StarRocksではHudi外部テーブルを使用してHudiデータレイクからデータをクエリすることができます。これにより、高速なデータレイク分析が可能となります。このトピックでは、StarRocksクラスターにHudi外部テーブルを作成し、Hudi外部テーブルを使用してHudiデータレイクからデータをクエリする方法について説明します。

### 開始する前に

StarRocksクラスターがHiveメタストア、HDFSクラスター、またはHudiテーブルを登録できるバケットへのアクセスを許可されていることを確認してください。

### 注意事項

* Hudiの外部テーブルは読み取り専用であり、クエリのみに使用できます。
* StarRocksはCopy on WriteおよびMerge On Readテーブル（MORテーブルはv2.5からサポートされています）のクエリをサポートしています。これら2種類のテーブルの違いについては、[Table & Query Types](https://hudi.apache.org/docs/table_types/)を参照してください。
* StarRocksはHudiの次の2つのクエリタイプをサポートしています: Snapshot QueriesおよびRead Optimized Queries（HudiはMerge On ReadテーブルでのRead Optimized Queriesのみを実行できます）。Incremental Queriesはサポートされていません。Hudiのクエリタイプについての詳細については、[Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types)を参照してください。
* StarRocksはHudiファイルの圧縮形式としてgzip、zstd、LZ4、およびSnappyをサポートしています。Hudiファイルのデフォルト圧縮形式はgzipです。
* StarRocksは、Hudi管理テーブルからのスキーマ変更を同期することができません。詳細については、[Schema Evolution](https://hudi.apache.org/docs/schema_evolution/)を参照してください。Hudi管理テーブルのスキーマが変更された場合は、関連するHudi外部テーブルをStarRocksクラスターから削除し、その外部テーブルを再作成する必要があります。

### 手順

#### ステップ1: Hudiリソースを作成および管理する

StarRocksクラスターにHudiリソースを作成する必要があります。Hudiリソースは、StarRocksクラスターに作成するHudiデータベースおよび外部テーブルを管理するために使用されます。

##### Hudiリソースを作成する

次の文を実行して、`hudi0`という名前のHudiリソースを作成します:

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://192.168.7.251:9083"
);
~~~

次の表にパラメータの詳細を示します。

| パラメータ            | 説明                                           |
| ------------------- | --------------------------------------------- |
| type                | Hudiリソースのタイプ。値を`hudi`に設定します。 |
| hive.metastore.uris | Hudiリソースが接続するHiveメタストアのThrift URI。HudiリソースをHiveに接続した後、Hiveを使用してHudiテーブルを作成および管理できます。Thrift URIは`<HiveメタストアのIPアドレス>:<Hiveメタストアのポート番号>`の形式です。デフォルトのポート番号は9083です。 |

v2.3以降、StarRocksではHudiリソースの`hive.metastore.uris`の値を変更することができます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

##### Hudiリソースを表示する

次の文を実行して、StarRocksクラスターに作成されたすべてのHudiリソースを表示します:

~~~SQL
SHOW RESOURCES;
~~~

##### Hudiリソースを削除する

次の文を実行して、`hudi0`という名前のHudiリソースを削除します:

~~~SQL
DROP RESOURCE "hudi0";
~~~

> 注意:
>
> Hudiリソースを削除すると、そのHudiリソースを使用して作成されたすべてのHudi外部テーブルは利用できなくなりますが、Hudiに格納されているデータには影響しません。StarRocksから引き続きHudiのデータをクエリする場合は、Hudiリソース、Hudiデータベース、およびHudi外部テーブルを再作成する必要があります。

#### ステップ2: Hudiデータベースを作成する

次の文を実行して、StarRocksクラスターに`hudi_test`という名前のHudiデータベースを作成およびオープンします:

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> 注意:
>
> StarRocksクラスターにおけるHudiデータベースの名前は、関連するHudi内部データベースと同じである必要はありません。

#### ステップ3: Hudi外部テーブルを作成する

次の文を実行して、`hudi_test`のHudiデータベースに`hudi_tbl`という名前のHudi外部テーブルを作成します:

~~~SQL
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

次の表にパラメータの詳細を示します。

| パラメータ | 説明                                           |
| --------- | --------------------------------------------- |
| ENGINE    | Hudi外部テーブルのクエリエンジン。値を`HUDI`に設定します。 |
| resource  | StarRocksクラスター内のHudiリソースの名前。   |
| database  | StarRocksクラスター内のHudi外部テーブルが属するHudiデータベースの名前。 |
| table     | Hudi外部テーブルが関連付けられているHudi管理テーブル。 |

> 注意:
>
> * Hudi外部テーブルの名前は、関連するHudi管理テーブルと同じである必要はありません。
>
> * Hudi外部テーブルの列は、それに関連するHudi管理テーブルの列と同じ名前である必要がありますが、別の順序である場合があります。
>
> * 関連するHudi管理テーブルから一部またはすべての列を選択し、Hudi外部テーブルにのみ選択した列を作成することができます。次の表には、Hudiでサポートされているデータ型とStarRocksでサポートされているデータ型のマッピングがリストされています。

| Hudiでサポートされているデータ型   | StarRocksでサポートされているデータ型 |
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

> **注意**
>
> StarRocksは、STRUCTまたはMAP型のデータのクエリをサポートしておらず、Merge On ReadテーブルでのARRAY型のデータのクエリもサポートしていません。

#### ステップ4: Hudi外部テーブルからデータをクエリする

特定のHudi管理テーブルに関連付けられたHudi外部テーブルを作成したら、Hudi外部テーブルにデータをロードする必要はありません。Hudiからデータをクエリするためには、次の文を実行します:

~~~SQL
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (非推奨) MySQL外部テーブル
```markdown
+ In the star schema, data is generally divided into dimension tables and fact tables. Dimension tables have less data but involve UPDATE operations. Currently, StarRocks does not support direct UPDATE operations (update can be implemented by using the Unique Key table). In some scenarios, you can store dimension tables in MySQL for direct data read.

To query MySQL data, you must create an external table in StarRocks and map it to the table in your MySQL database. You need to specify the MySQL connection information when creating the table.

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

Parameters:

* **host**: the connection address of the MySQL database
* **port**: the port number of the MySQL database
* **user**: the username to log in to MySQL
* **password**: the password to log in to MySQL
* **database**: the name of the MySQL database
* **table**: the name of the table in the MySQL database
```
```markdown
+ スタースキーマでは、データは一般的にディメンジョンテーブルとファクトテーブルに分割されます。ディメンジョンテーブルはデータが少ないですが、UPDATE操作が関わります。現時点で、StarRocksは直接のUPDATE操作をサポートしていません（Unique Keyテーブルを使用して実装することができます）。一部のシナリオでは、ディメンジョンテーブルをMySQLに保存して直接データを読むことができます。

MySQLのデータをクエリするには、StarRocksで外部テーブルを作成し、MySQLデータベースのテーブルにマップする必要があります。テーブルを作成する際には、MySQL接続情報を指定する必要があります。

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

パラメータ：

* **host**: MySQLデータベースの接続アドレス
* **port**: MySQLデータベースのポート番号
* **user**: MySQLにログインするユーザー名
* **password**: MySQLにログインするパスワード
* **database**: MySQLデータベースの名前
* **table**: MySQLデータベースのテーブルの名前
```