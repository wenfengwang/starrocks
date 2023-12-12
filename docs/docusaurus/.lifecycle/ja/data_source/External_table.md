---
displayed_sidebar: "Japanese"
---

# 外部テーブル

:::note
v3.0以降、Hive、Iceberg、およびHudiからデータをクエリする場合は、カタログを使用することを推奨します。[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、および[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

v3.1以降、MySQLおよびPostgreSQLからデータをクエリする場合は[JDBCカタログ](../data_source/catalog/jdbc_catalog.md)を使用し、Elasticsearchからデータをクエリする場合は[Elasticsearchカタログ](../data_source/catalog/elasticsearch_catalog.md)を使用することを推奨します。
:::

StarRocksは、外部テーブルを使用して他のデータソースにアクセスすることをサポートしています。外部テーブルは、他のデータソースに格納されているデータテーブルに基づいて作成されます。StarRocksはデータテーブルのメタデータのみを保存します。外部テーブルを使用して、他のデータソースのデータを直接クエリできます。StarRocksは、次のデータソースをサポートしています：MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg、およびApache Hudi。**現時点では、他のStarRocksクラスターからのデータを現在のStarRocksクラスターに書き込むことができますが、読み取ることはできません。StarRocks以外のデータソースについては、それらのデータソースからデータを読み取ることしかできません。**

2.5以降、StarRocksは外部データソース上のホットデータクエリを加速するデータキャッシュ機能を提供します。詳細については[Data Cache](data_cache.md)を参照してください。

## StarRocks外部テーブル

StarRocks 1.19以降、StarRocksはStarRocks外部テーブルを使用して、1つのStarRocksクラスターから別のStarRocksクラスターにデータを書き込むことを可能にします。これにより読み取りと書き込みを分離し、リソースをより良く隔離できます。まず、宛先のStarRocksクラスターで宛先テーブルを作成し、次に、ソースのStarRocksクラスターで同じスキーマを持つStarRocks外部テーブルを作成し、`PROPERTIES`フィールドに宛先クラスターおよびテーブルの情報を指定できます。

INSERT INTOステートメントを使用して、ソースクラスターから宛先クラスターにデータを書き込むことで、データをソースクラスターから宛先クラスターに書き込むことができます。これにより、次の目標を実現できます：

* StarRocksクラスター間のデータ同期。
* 読み取りと書き込みの分離。データはソースクラスターに書き込まれ、ソースクラスターからのデータ変更は宛先クラスターに同期され、クエリサービスが提供されます。

次のコードは、宛先テーブルと外部テーブルの作成方法を示しています。

~~~SQL
# 宛先StarRocksクラスターで宛先テーブルを作成します。
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

# ソースStarRocksクラスターで外部テーブルを作成します。
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

# StarRocks外部テーブルにデータを書き込むことで、ソースクラスターから宛先クラスターにデータを書き込みます。本番環境では、2番目のステートメントが推奨されます。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

パラメータ：

* **EXTERNAL:** このキーワードは、作成するテーブルが外部テーブルであることを示します。
* **host:** このパラメータは、宛先StarRocksクラスターのリーダーFEノードのIPアドレスを指定します。
* **port:**  このパラメータは、宛先StarRocksクラスターのFEノードのRPCポートを指定します。

  :::note

  StarRocks外部テーブルが属するソースクラスターが宛先StarRocksクラスターにアクセスできるようにするためには、ネットワークとファイアウォールを構成して、次のポートへのアクセスを許可する必要があります：

  * FEノードのRPCポート。FE設定ファイル **fe/fe.conf** にある`rpc_port`を参照してください。デフォルトのRPCポートは `9020` です。
  * BEノードのbRPCポート。BE設定ファイル **be/be.conf** にある`brpc_port`を参照してください。デフォルトのbRPCポートは `8060` です。

  :::

* **user:** このパラメータは、宛先StarRocksクラスターにアクセスするために使用されるユーザー名を指定します。
* **password:** このパラメータは、宛先StarRocksクラスターにアクセスするために使用されるパスワードを指定します。
* **database:** このパラメータは、宛先テーブルが属するデータベースを指定します。
* **table:** このパラメータは、宛先テーブルの名前を指定します。

次の制限がStarRocks外部テーブルの使用時に適用されます：

* StarRocks外部テーブルに対しては、INSERT INTOおよびSHOW CREATE TABLEコマンドのみを実行できます。その他のデータ書き込みメソッドはサポートされていません。また、StarRocks外部テーブルからデータをクエリしたり、外部テーブルにDDL操作を実行したりすることはできません。
* 外部テーブルを作成する構文は通常のテーブルの作成と同じですが、外部テーブルの列名および他の情報は宛先テーブルと同じである必要があります。
* 外部テーブルは、宛先テーブルのテーブルメタデータを10秒ごとに同期します。宛先テーブルでDDL操作を実行すると、2つのテーブル間のデータ同期に遅延が生じることがあります。

## JDBC互換データベースの外部テーブル

v2.3.0以降、StarRocksはJDBC互換データベースをクエリする外部テーブルを提供しています。これにより、StarRocksにデータを取り込む必要なく、これらのデータベースのデータを高速に分析できます。このセクションでは、StarRocksで外部テーブルを作成し、JDBC互換データベースのデータをクエリする方法について説明します。

### 前提条件

JDBC外部テーブルを使用してデータをクエリする前に、FEおよびBEがJDBCドライバーのダウンロードURLにアクセスできることを確認してください。ダウンロードURLは、JDBCリソースの作成に使用されるステートメントで`driver_url`パラメータで指定されます。

### JDBCリソースの作成と管理

#### JDBCリソースの作成

データベースからデータをクエリするための外部テーブルを作成する前に、データベースの接続情報を管理するためにStarRocksにJDBCリソースを作成する必要があります。データベースはJDBCドライバーをサポートし、"ターゲットデータベース"として参照されます。リソースを作成した後、それを使用して外部テーブルを作成できます。

以下のステートメントを実行して、`jdbc0`という名前のJDBCリソースを作成します：

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

`PROPERTIES`内で必要なパラメータは次のとおりです：

* `type`: リソースのタイプ。値を`jdbc`に設定します。

* `user`: ターゲットデータベースに接続するために使用されるユーザー名。

* `password`: ターゲットデータベースに接続するために使用されるパスワード。

* `jdbc_uri`: JDBCドライバーがターゲットデータベースに接続するために使用するURI。URIの形式はデータベースのURI構文を満たしている必要があります。一部の一般的なデータベースのURI構文については、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)の公式ウェブサイトを参照してください。

> 注意: URIにはターゲットデータベースの名前を含める必要があります。たとえば、前述のコード例では、`jdbc_test`が接続したいターゲットデータベースの名前です。

* `driver_url`: JDBCドライバーJARパッケージのダウンロードURL。HTTP URLまたはファイルURLがサポートされています。たとえば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`または`file:///home/disk1/postgresql-42.3.3.jar`のようなURLがあります。

* `driver_class`: JDBCドライバーのクラス名。一般的なデータベースのJDBCドライバークラス名は次のとおりです：
  * MySQL: com.mysql.jdbc.Driver (MySQL 5.x およびそれ以前), com.mysql.cj.jdbc.Driver (MySQL 6.x およびそれ以降)
  * SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL: org.postgresql.Driver

リソースが作成される際に、FEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバーJARパッケージをダウンロードし、チェックサムを生成し、BEによってダウンロードされたJDBCドライバーを検証するためにチェックサムを使用します。
> ノート：JDBCドライバのJARパッケージのダウンロードに失敗した場合、リソースの作成も失敗します。

BEが初めてJDBC外部テーブルをクエリし、対応するJDBCドライバのJARパッケージが自分たちのマシンに存在しないことを見つけた場合、BEは「driver_url」パラメータで指定されたURLを使用してJDBCドライバのJARパッケージをダウンロードし、「${STARROCKS_HOME}/lib/jdbc_drivers」ディレクトリにすべてのJDBCドライバのJARパッケージを保存します。

#### JDBCリソースの表示

次のステートメントを実行して、StarRocksのすべてのJDBCリソースを表示します。

~~~SQL
SHOW RESOURCES;
~~~

> ノート：`ResourceType`列は`jdbc`です。

#### JDBCリソースの削除

次のステートメントを実行して、`jdbc0`という名前のJDBCリソースを削除します。

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> ノート：JDBCリソースが削除されると、そのJDBCリソースを使用して作成されたすべてのJDBC外部テーブルが利用できなくなります。ただし、対象データベースのデータは失われません。対象データベースのデータを引き続きStarRocksでクエリする必要がある場合は、JDBCリソースとJDBC外部テーブルを再作成できます。

### データベースの作成

次のステートメントを実行して、StarRocksの`jdbc_test`という名前のデータベースを作成しアクセスします。

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> ノート：前述のステートメントで指定するデータベース名は、対象のデータベースの名前と同じである必要はありません。

### JDBC外部テーブルの作成

次のステートメントを実行して、データベース`jdbc_test`に`jdbc_tbl`という名前のJDBC外部テーブルを作成します。

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

`properties`で必要なパラメータは次の通りです。

* `resource`：外部テーブルを作成するために使用されるJDBCリソースの名前。

* `table`：データベース内の対象テーブルの名前。

サポートされるデータ型とStarRocksと対象データベース間のデータ型マッピングについては、「[データ型マッピング](External_table.md#データ型マッピング)」を参照してください。

> ノート：
>
> * インデックスはサポートされていません。
> * PARTITION BYやDISTRIBUTED BYを使用してデータ分配規則を指定することはできません。

### JDBC外部テーブルのクエリ

JDBC外部テーブルをクエリする前に、次のステートメントを実行してPipelineエンジンを有効にする必要があります。

~~~SQL
set enable_pipeline_engine=true;
~~~

> ノート：Pipelineエンジンがすでに有効になっている場合は、このステップをスキップできます。

次のステートメントを実行して、JDBC外部テーブルを使用して対象データベースのデータをクエリします。

~~~SQL
select * from JDBC_tbl;
~~~

StarRocksは、フィルタ条件を対象テーブルにプッシュダウンすることにより、述語のプッシュダウンをサポートします。データソースにできるだけ近い場所でフィルタ条件を実行することで、クエリのパフォーマンスを向上させることができます。現在、StarRocksはバイナリ比較演算子（`>`、`>=`、`=`, `<`, and `<=`）、`IN`、`IS NULL`、および`BETWEEN ... AND ...`を含む演算子をプッシュダウンすることができます。ただし、StarRocksは関数をプッシュダウンすることはできません。

### データ型マッピング

現在、StarRocksは対象データベースの基本的なデータ型（NUMBER、STRING、TIME、DATEなど）のデータのみをクエリできます。対象データベースのデータ値の範囲がStarRocksでサポートされていない場合、クエリでエラーが発生します。

対象データベースとStarRocksとの間のマッピングは、対象データベースのタイプに基づいて異なります。

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

* JDBC外部テーブルを作成する際、テーブルにインデックスを作成したり、PARTITION BYやDISTRIBUTED BYを使用してテーブルのデータ分配規則を指定することはできません。

* JDBC外部テーブルをクエリする際、StarRocksは関数をテーブルにプッシュダウンすることができません。

## （非推奨）Elasticsearch外部テーブル

StarRocksとElasticsearchは2つの人気のある分析システムです。StarRocksは大規模な分散コンピューティングに効果的です。Elasticsearchは全文検索に理想的です。StarRocksとElasticsearchを組み合わせることで、より完全なOLAPソリューションを提供できます。

### Elasticsearch外部テーブルを作成する例

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

以下の表はパラメータを説明しています。

| **パラメータ**       | **必須** | **デフォルト値** | **説明**                                                     |
| -------------------- | -------- | ---------------- | ------------------------------------------------------------ |
| hosts                | Yes      | なし             | Elasticsearchクラスタの接続アドレス。1つまたは複数のアドレスを指定できます。StarRocksはこのアドレスからElasticsearchのバージョンおよびインデックスのシャード割り当てを解析できます。StarRocksは`GET /_nodes/http`API操作の応答に基づいてElasticsearchクラスタと通信します。したがって、`host`パラメータの値は、`GET /_nodes/http`API操作の応答と同じである必要があります。さもないと、BEはElasticsearchクラスタと通信できなくなる可能性があります。 |
| index                | Yes      | なし             | StarRocksのテーブルで作成されたElasticsearchインデックスの名前。名前はエイリアスにすることができます。このパラメータはワイルドカード（\*）をサポートします。たとえば、`index`を`hello*`に設定すると、StarRocksは`hello`で始まるすべてのインデックスを取得します。 |
| user                 | No       | 空               | 基本認証が有効になっているElasticsearchクラスタにログインするためのユーザ名。`/*cluster/state/*nodes/http`およびインデックスへのアクセス権があることを確認してください。 |
| password             | No       | 空               | Elasticsearchクラスタにログインするためのパスワード。 |
| type                 | No       | `_doc`           | インデックスのタイプ。デフォルト値：`_doc`。Elasticsearch 8以降のバージョンでデータをクエリする場合は、このパラメータを構成する必要はありません。なぜなら、Elasticsearch 8以降のバージョンではマッピングのタイプが取り除かれているためです。|
| es.nodes.wan.only    | いいえ           | `false`           | StarRocksがElasticsearchクラスタにアクセスし、データを取得するために`hosts`で指定されたアドレスのみを使用するかどうかを指定します。<ul><li>`true`: StarRocksは`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データを取得し、Elasticsearchインデックスのシャードが存在するデータノードをスニッフィングしません。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできない場合、このパラメータを`true`に設定する必要があります。</li><li>`false`: StarRocksは`hosts`で指定されたアドレスを使用してElasticsearchクラスタ内のインデックスのシャードが存在するデータノードをスニッフィングします。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできる場合は、デフォルト値の`false`を保持することをお勧めします。</li></ul> |
| es.net.ssl           | いいえ           | `false`           | ElasticsearchクラスタにアクセスするためにHTTPSプロトコルを使用できるかどうかを指定します。このパラメータを構成するのは、StarRocks 2.4以降のバージョンのみです。<ul><li>`true`: HTTPSプロトコルとHTTPプロトコルの両方を使用してElasticsearchクラスタにアクセスできます。</li><li>`false`: HTTPプロトコルのみを使用してElasticsearchクラスタにアクセスできます。</li></ul> |
| enable_docvalue_scan | いいえ           | `true`            | Elasticsearchのカラム型ストレージから対象フィールドの値を取得するかどうかを指定します。ほとんどの場合、カラム型ストレージからデータを読み取る方が行型ストレージからデータを読み取るよりもパフォーマンスが向上します。 |
| enable_keyword_sniff | いいえ           | `true`            | KEYWORD型フィールドに基づいてElasticsearchのTEXT型フィールドをスニッフィングするかどうかを指定します。このパラメータを`false`に設定すると、StarRocksはトークン化後に一致を実行します。 |

##### より高速なクエリのためのカラムスキャン

`enable_docvalue_scan`を`true`に設定すると、StarRocksはElasticsearchからデータを取得する際に次のルールに従います。

* **試してみる**: StarRocksは自動的に対象フィールドに対してカラム型ストレージが有効かどうかをチェックします。有効な場合、StarRocksは対象フィールドのすべての値をカラム型ストレージから取得します。
* **自動ダウングレード**: 対象フィールドのいずれか1つがカラム型ストレージで利用できない場合、StarRocksは対象フィールドのすべての値を行型ストレージ（`_source`）から解析および取得します。

> **注意**
>
> * Elasticsearchでは、TEXT型フィールドにはカラム型ストレージが利用できません。そのため、TEXT型の値を含むフィールドをクエリする場合、StarRocksはフィールドの値を`_source`から取得します。
> * フィールドの値を`docvalue`から読み取る場合、フィールドの値を`_source`から読み取る場合と比較して顕著なメリットはありません。

##### KEYWORD型フィールドのスニッフィング

`enable_keyword_sniff`を`true`に設定すると、Elasticsearchはインデックスを自動的に作成するため、インデックスなしで直接データを取り込むことが可能になります。STRING型フィールドの場合、ElasticsearchはTEXT型とKEYWORD型の両方を持つフィールドを作成します。これがElasticsearchのMulti-Field機能の動作方法です。マッピングは次のようになります:

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

例えば、`k4`で`=`フィルタリングを実行する場合、Elasticsearch上のStarRocksはElasticsearchのTermQueryにフィルタリング操作を変換します。

元のSQLフィルタは次のようになります:

~~~SQL
k4 = "StarRocks On Elasticsearch"
~~~

変換されたElasticsearchクエリDSLは次のようになります:

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"

}
~~~

`k4`の最初のフィールドはTEXT型であり、データ取り込み後に`k4`に構成されたアナライザー（または`k4`に構成されたアナライザーがない場合は標準アナライザー）でトークン化されます。その結果、最初のフィールドは`StarRocks`、`On`、`Elasticsearch` の3つの用語にトークン化されます。詳細は次の通りです:

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

トークン化の結果は次のようになります:

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

以下のようにクエリを実行すると:

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

辞書に一致する用語がないため、`StarRocks On Elasticsearch`という用語に一致する結果は返されません。

しかし、`enable_keyword_sniff`を`true`に設定している場合、StarRocksは`k4 = "StarRocks On Elasticsearch"` を `k4.keyword = "StarRocks On Elasticsearch"`に変換し、SQLの意味に一致させるため、変換された`StarRocks On Elasticsearch` クエリDSLは次のようになります:

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword`はKEYWORD型です。そのため、データは完全な用語としてElasticsearchに書き込まれ、成功した一致が可能になります。

#### カラムデータ型のマッピング

外部テーブルを作成する際に、Elasticsearchテーブルのカラムのデータ型に基づいて外部テーブルのカラムのデータ型を指定する必要があります。次の表はカラムデータ型のマッピングを示しています。

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
> * StarRocksはNESTED型データをJSON関連の関数を使用して読み取ります。
> * Elasticsearchは多次元配列を一次元配列に自動的にフラット化します。StarRocksも同様です。 **ElasticsearchからARRAYデータをクエリするサポートはv2.5から追加されています。**

### プレディケートのプッシュダウン

StarRocksはプレディケートのプッシュダウンをサポートしています。フィルタは実行のためにElasticsearchにプッシュダウンされ、クエリのパフォーマンスが向上します。次の表はプレディケートのプッシュダウンをサポートする演算子を示しています。

|   SQL構文  |   ES構文  |
| :---: | :---: |
|  `=`   |  term query   |
|  `in`   |  terms query   |
|  `>=,  <=, >, <`   |  range   |
|  `and`   |  bool.filter   |
|  `or`   |  bool.should   |
|  `not`   |  bool.must_not   |
|  `not in`   |  bool.must_not + terms   |
|  `esquery`   |  ESクエリDSL  |

### 例

**esquery関数**を使用して、SQLで表現できないクエリ（matchやgeoshapeなど）をElasticsearchにフィルタリングするために使用します。esquery関数の最初のパラメーターはインデックスの関連付けに使用されます。2番目のパラメータは基本的なクエリDSLのJSON式であり、{}で囲まれています。**JSON式は1つのルートキーのみを持っている必要があります**。例えば、match、geo_shape、またはboolなどです。

* matchクエリ

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on elasticsearch"
    }
}');
~~~

* 地理関連のクエリ

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
* Elasticsearch の 5.x より前のバージョンは、5.x より後のバージョンとは異なる方法でデータをスキャンします。現在、**5.x より後のバージョンのみ**がサポートされています。
* HTTP 基本認証が有効になっている Elasticsearch クラスターはサポートされています。
* StarRocks からデータを問い合わせる際、Elasticsearch から直接データを問い合わせるよりも速くない場合があります。その理由は、Elasticsearch は実際のデータをフィルタする必要がなく、対象ドキュメントのメタデータを直接読み取るため、カウントクエリが高速化されることです。

## (非推奨) Hive 外部テーブル

Hive 外部テーブルを使用する前に、サーバーに JDK 1.8 がインストールされていることを確認してください。

### Hive リソースの作成

Hive リソースは Hive クラスターに対応します。StarRocks で使用される Hive クラスターを構成する必要があります。Hive 外部テーブルで使用される Hive リソースを指定する必要があります。

* hive0 という名前の Hive リソースを作成します。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://10.10.44.98:9083"
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

Hive リソースの `hive.metastore.uris` を StarRocks 2.3 以降のバージョンで変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

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

例：`hive0` リソースに対応する Hive クラスターの `rawdata` データベース内に、外部テーブル `profile_parquet_p7` を作成します。

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
  * 列名は、Hive テーブル内の列名と同じである必要があります。
  * 列の順序は、Hive テーブル内の列の順序と**同じである必要はありません**。
  * Hive テーブルの**一部の列のみを選択**することができますが、**すべての** **パーティションキー列**を選択する必要があります。
  * 外部テーブルのパーティションキー列は `partition by` を使用して指定する必要はありません。他の列と同じ記述リストで定義する必要があります。パーティション情報は指定する必要はありません。StarRocks は、これらの情報を自動的に Hive テーブルから同期します。
  * `ENGINE` を HIVE に設定します。
* PROPERTIES:
  * **hive.resource**: 使用される Hive リソース。
  * **database**: Hive データベース。
  * **table**: Hive 内のテーブル。**view** はサポートされていません。
* 次の表は、Hive と StarRocks の列データ型のマッピングを示しています。

    |  Hive の列データ型  |  StarRocks の列データ型   | 説明 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |
    |   TIMESTAMP  | DATETIME    | TIMESTAMP データを DATETIME データに変換する際、精度とタイムゾーン情報が失われます。タイムゾーン オフセットを持たない DATETIME データに TIMESTAMP データを変換する必要があります。 |
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意:
>
> * 現在、サポートされている Hive ストレージ形式は、Parquet、ORC、CSV です。
CSV 形式の場合、エスケープ文字として引用符を使用することはできません。
> * SNAPPY および LZ4 圧縮形式がサポートされています。
> * クエリ可能な Hive 文字列列の最大長は 1 MB です。文字列列が 1 MB を超える場合、それは null 列として処理されます。

### Hive 外部テーブルの使用

`profile_wos_p7` の合計行数をクエリします。

~~~sql
select count(*) from profile_wos_p7;
~~~

### キャッシュされた Hive テーブルのメタデータを更新する

* Hive のパーティション情報と関連するファイル情報は StarRocks でキャッシュされます。キャッシュは `hive_meta_cache_refresh_interval_s` で指定された間隔で更新されます。デフォルト値は 7200 です。 `hive_meta_cache_ttl_s` は、キャッシュのタイムアウト期間を指定し、デフォルト値は 86400 です。
  * キャッシュされたデータは、手動で更新することもできます。
    1. Hive のテーブルにパーティションが追加または削除された場合、StarRocks にキャッシュされたテーブルのメタデータを更新するには、`REFRESH EXTERNAL TABLE hive_t` コマンドを実行する必要があります。`hive_t` は StarRocks の Hive 外部テーブルの名前です。
    2. いくつかの Hive パーティションのデータが更新された場合、`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')` コマンドを実行して StarRocks のキャッシュデータを更新する必要があります。`hive_t` は StarRocks の Hive 外部テーブルの名前です。`'k1=01/k2=02'` および `'k1=03/k2=04'` はデータが更新された Hive パーティションの名前です。
    3. `REFRESH EXTERNAL TABLE hive_t` を実行すると、StarRocks はまず、Hive Metastore で返される Hive テーブルの列情報が Hive 外部テーブルの列情報と同じかどうかをチェックします。Hive テーブルのスキーマが変更された場合、列の追加や削除など、StarRocks は変更を Hive 外部テーブルに同期します。同期後、Hive 外部テーブルの列順序は引き続き Hive テーブルの列順序と同じであり、パーティション列が最後の列となります。
* Hive データが Parquet、ORC、および CSV 形式で保存されている場合、StarRocks 2.3 以降のバージョンで、Hive テーブルのスキーマ変更（列の追加や置換など）を Hive 外部テーブルに同期することができます。

### オブジェクトストレージへのアクセス

FE 設定ファイルのパスは `fe/conf` であり、Hadoop クラスターをカスタマイズする必要がある場合、設定ファイルを追加できます。例: HDFS クラスターが高可用性のネームサービスを使用する場合、`hdfs-site.xml` を `fe/conf` に配置する必要があります。HDFS が ViewFs で構成されている場合、`core-site.xml` を `fe/conf` に配置する必要があります。
BE 設定ファイルのパスは `be/conf` であり、Hadoop クラスターをカスタマイズする必要がある場合、設定ファイルを追加できます。例: HDFS クラスターが高可用性のネームサービスを使用する場合、`hdfs-site.xml` を `be/conf` に配置する必要があります。HDFS が ViewFs で構成されている場合、`core-site.xml` を `be/conf` に配置する必要があります。
BE があるマシンで、BE **起動スクリプト** `bin/start_be.sh` に JAVA_HOME を JRE 環境ではなく JDK 環境として構成します。例: `export JAVA_HOME = <JDK パス>`。この構成をスクリプトの最初に追加し、構成が有効になるように BE を再起動する必要があります。
Kerberos サポートの設定:
1. `kinit -kt キータブ_パス principal` で FE/BE マシン全体にログインするには、Hive と HDFS へのアクセス権が必要です。kinit コマンドのログインは一定期間有効であり、定期的に実行されるよう crontab に追加する必要があります。
2. `fe/conf` に `hive-site.xml/core-site.xml/hdfs-site.xml` を配置し、`be/conf` に `core-site.xml/hdfs-site.xml` を配置します。
3. **$FE_HOME/conf/fe.conf** ファイルの `JAVA_OPTS` オプションの値に `-Djava.security.krb5.conf=/etc/krb5.conf` を追加します。**/etc/krb5.conf** は **krb5.conf** ファイルの保存パスです。これはオペレーティング システムに基づいてパスを変更できます。
4. **$BE_HOME/conf/be.conf**ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を直接追加します。 **/etc/krb5.conf** は**krb5.conf**ファイルの保存パスです。操作システムに基づいてパスを変更できます。
   5. Hiveリソースを追加する場合、`hive.metastore.uris`にドメイン名を渡さなければなりません。さらに、Hive/HDFSのドメイン名とIPアドレスのマッピングを**/etc/hosts**ファイルに追加する必要があります。

* AWS S3のサポートを構成します：`fe/conf/core-site.xml`および`be/conf/core-site.xml`に以下の構成を追加します。

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

   1. `fs.s3a.access.key`: AWSアクセスキーID。
   2. `fs.s3a.secret.key`: AWSシークレットキー。
   3. `fs.s3a.endpoint`: 接続先のAWS S3エンドポイント。
   4. `fs.s3a.connection.m``aximum`: StarRocksからS3への同時接続の最大数。クエリ実行中にエラー`Timeout waiting for connection from poll`が発生した場合は、このパラメーターをより大きな値に設定できます。

## (廃止) Iceberg外部テーブル

v2.1.0以降、StarRocksでは外部テーブルを使用してApache Icebergのデータを問い合わせることができます。Icebergデータを問い合わせるには、StarRocksにIceberg外部テーブルを作成する必要があります。テーブルを作成する際は、外部テーブルと問い合わせたいIcebergテーブルとの間のマッピングを確立する必要があります。

### 開始する前に

StarRocksがApache Icebergで使用するメタデータサービス（Hiveメタストアなど）、ファイルシステム（HDFSなど）、およびオブジェクトストレージシステム（Amazon S3およびAlibaba Cloud Object Storage Serviceなど）にアクセスする権限があることを確認してください。

### 注意事項

* StarRocks 2.3およびそれ以降のバージョンでは、Iceberg外部テーブルは次の種類のデータのみを問い合わせるために使用できます：
  * Iceberg v1テーブル（解析データテーブル）。ORC形式のIceberg v2（行レベル削除）テーブルはv3.0以降でサポートされ、Parquet形式のIceberg v2テーブルはv3.1以降でサポートされます。Iceberg v1テーブルとIceberg v2テーブルの違いについては、[Iceberg Table Spec](https://iceberg.apache.org/spec/)を参照してください。
  * gzip（デフォルト形式）、Zstd、LZ4、またはSnappy形式で圧縮されたテーブル。
  * ParquetまたはORC形式で保存されたファイル。

* StarRocks 2.3およびそれ以降のバージョンでは、Iceberg外部テーブルはIcebergテーブルのスキーマ変更を同期でサポートしますが、それ以前のバージョンではサポートしません。Icebergテーブルのスキーマが変更された場合は、対応する外部テーブルを削除して新しいテーブルを作成する必要があります。

### 手順

#### 手順 1: Icebergリソースを作成

Iceberg外部テーブルを作成する前に、StarRocksでIcebergリソースを作成する必要があります。このリソースはIcebergへのアクセス情報を管理するために使用されます。また、このリソースをIceberg外部テーブルを作成する際に指定する必要があります。ビジネス要件に基づいてリソースを作成できます：

* IcebergテーブルのメタデータがHiveメタストアから取得される場合は、リソースを作成し、カタログタイプを`HIVE`に設定することができます。

* Icebergテーブルのメタデータが他のサービスから取得される場合は、カスタムカタログを作成する必要があります。その後、リソースを作成し、カタログタイプを`CUSTOM`に設定する必要があります。

##### カタログタイプが`HIVE`であるリソースを作成

例えば、名前を`iceberg0`として、カタログタイプを`HIVE`に設定したリソースを作成します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://192.168.0.81:9083" 
);
~~~

以下の表は、関連するパラメータについて説明しています。

| **Parameter**                       | **Description**                                              |
| ----------------------------------- | ------------------------------------------------------------ |
| type                                | リソースのタイプ。値を`iceberg`に設定します。               |
| iceberg.catalog.type              | リソースのカタログタイプ。Hiveカタログとカスタムカタログの両方がサポートされています。Hiveカタログを指定する場合は、値を`HIVE`に設定します。カスタムカタログを指定する場合は、値を`CUSTOM`に設定します。 |
| iceberg.catalog.hive.metastore.uris | HiveメタストアのURI。パラメータの値は次の形式です： `thrift://<IcebergメタデータのIPアドレス>:<ポート番号>`。ポート番号はデフォルトで9083になります。Apache IcebergはHiveカタログを使用してHiveメタストアにアクセスし、その後Icebergテーブルのメタデータを問い合わせます。 |

##### カタログタイプが`CUSTOM`であるリソースを作成

カスタムカタログは、BaseMetastoreCatalog抽象クラスを継承する必要があり、IcebergCatalogインターフェースを実装する必要があります。また、カスタムカタログのクラス名はStarRockにすでに存在するクラスの名前と重複しないようにする必要があります。カタログが作成されたら、カタログとそれに関連するファイルをパッケージ化し、それぞれのフロントエンド（FE）の**fe/lib**パスに配置し、それぞれのFEを再起動してください。これらの操作を完了した後に、カスタムカタログであるリソースを作成することができます。

例えば、名前を`iceberg1`として、カタログタイプを`CUSTOM`に設定したリソースを作成します。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 
);
~~~

以下の表は、関連するパラメータについて説明しています。

| **Parameter**          | **Description**                                              |
| ---------------------- | ------------------------------------------------------------ |
| type                   | リソースのタイプ。値を`iceberg`に設定します。               |
| iceberg.catalog.type | リソースのカタログタイプ。Hiveカタログとカスタムカタログの両方がサポートされています。Hiveカタログを指定する場合は、値を`HIVE`に設定します。カスタムカタログを指定する場合は、値を`CUSTOM`に設定します。 |
| iceberg.catalog-impl   | カスタムカタログの完全修飾クラス名。FEはこの名前を基にカタログを検索します。カタログにカスタム構成項目が含まれている場合は、Iceberg外部テーブルを作成する際に`PROPERTIES`パラメータにそれらをキーと値のペアで追加する必要があります。 |

Icebergリソースの`hive.metastore.uris`および`iceberg.catalog-impl`をStarRocks 2.3およびそれ以降のバージョンで変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

##### Icebergリソースを表示

~~~SQL
SHOW RESOURCES;
~~~

##### Icebergリソースを削除

例えば、`iceberg0`という名前のリソースを削除します。

~~~SQL
DROP RESOURCE "iceberg0";
~~~

Icebergリソースを削除すると、このリソースを参照するすべての外部テーブルが利用できなくなります。ただし、Apache Iceberg内の対応するデータは削除されません。Apache Icebergのデータを引き続き問い合わせる必要がある場合は、新しいリソースと新しい外部テーブルを作成してください。

#### 手順 2:（オプション）データベースを作成

例えば、StarRocksで`iceberg_test`という名前のデータベースを作成します。

~~~SQL
CREATE DATABASE iceberg_test; 
USE iceberg_test; 
~~~

> 注意: StarRocksのデータベースの名前は、Apache Icebergのデータベースの名前と異なる場合があります。

#### 手順 3: Iceberg外部テーブルを作成

例えば、データベース`iceberg_test`で`iceberg_tbl`という名前のIceberg外部テーブルを作成します。

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

以下の表は、関連するパラメータについて説明しています。

| **Parameter** | **Description**                                              |
| ------------- | ------------------------------------------------------------ |
| ENGINE        | エンジンの名前。値を`ICEBERG`に設定します。               |
| resource      | 外部テーブルが参照するIcebergリソースの名前。             |
| database      | Icebergテーブルが属するデータベースの名前。                 |
| table         | Icebergテーブルの名前。                                     |

> 注意:
   >
   > * 外部テーブルの名前は、Icebergテーブルの名前と異なる場合があります。
* 外部テーブルの列名はIcebergテーブルの列名と同じでなければなりません。2つのテーブルの列順は異なっていてもかまいません。

カスタムカタログで構成アイテムを定義し、クエリデータを取得する際に構成アイテムを外部テーブルの作成時に`PROPERTIES`パラメータにキーと値のペアとして追加することができます。たとえば、カスタムカタログの`custom-catalog.properties`で構成アイテムを定義した場合、次のコマンドを実行して外部テーブルを作成できます。

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

StarRocksは、TIMESTAMPTZ、STRUCT、MAPのデータ型を持つIcebergデータのクエリをサポートしていません。

#### Step 4: Apache Icebergでデータをクエリする

外部テーブルが作成されたら、外部テーブルを使用してApache Icebergのデータをクエリできます。

~~~SQL
select count(*) from iceberg_tbl;
~~~

## (廃止予定) Hudi外部テーブル

v2.2.0以降、StarRocksではHudi外部テーブルを使用してHudiデータレイクからデータをクエリすることができます。これにより、データレイク分析を高速化できます。このトピックでは、StarRocksクラスターでHudi外部テーブルを作成し、Hudi外部テーブルを使用してHudiデータレイクからデータをクエリする方法について説明します。

### 開始する前に

StarRocksクラスターがHiveメタストア、HDFSクラスター、またはHudiテーブルを登録できるバケットへのアクセス権限を許可されていることを確認してください。

### 注意事項

* HudiのHudi外部テーブルは読み取り専用であり、クエリにのみ使用できます。
* StarRocksはCopy on WriteおよびMerge On Readテーブルをサポートしています（MORテーブルのサポートはv2.5からです）。これら2つのテーブルの違いについては、[Table & Query Types](https://hudi.apache.org/docs/table_types/)を参照してください。
* StarRocksはHudiの次の2つのクエリタイプをサポートしています：SnapshotクエリおよびRead Optimizedクエリ（HudiはMerge On ReadテーブルでのRead Optimizedクエリのみをサポートしています）。Incrementalクエリはサポートされていません。Hudiのクエリタイプについての詳細については、[Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types)を参照してください。
* StarRocksはHudiファイルの圧縮形式としてgzip、zstd、LZ4、およびSnappyをサポートしています。Hudiファイルのデフォルトの圧縮形式はgzipです。
* StarRocksはHudi管理テーブルからスキーマ変更を同期できません。詳細については[Schema Evolution](https://hudi.apache.org/docs/schema_evolution/)を参照してください。Hudi管理テーブルのスキーマが変更された場合、関連するHudi外部テーブルをStarRocksクラスターから削除し、その外部テーブルを再作成する必要があります。

### 手順

#### Step 1: Hudiリソースの作成と管理

StarRocksクラスターでHudiリソースを作成する必要があります。Hudiリソースは、StarRocksクラスターで作成するHudiデータベースおよび外部テーブルを管理するために使用されます。

##### Hudiリソースの作成

次のステートメントを実行して`hudi0`という名前のHudiリソースを作成します：

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://192.168.7.251:9083"
);
~~~

次の表はパラメータを示しています。

| パラメータ             | 説明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| type                   | Hudiリソースのタイプ。値を`hudi`に設定します。               |
| hive.metastore.uris    | Hudiリソースが接続するHiveメタストアのThrift URI。HudiリソースをHiveに接続した後、Hiveを使用してHudiテーブルを作成および管理できます。Thrift URI は`<HiveメタストアのIPアドレス>:<Hiveメタストアのポート番号>`の形式です。デフォルトのポート番号は9083です。 |

v2.3以降、StarRocksはHudiリソースの`hive.metastore.uris`の値を変更できます。詳細については[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。

##### Hudiリソースの表示

次のステートメントを実行して、StarRocksクラスターで作成されたすべてのHudiリソースを表示します：

~~~SQL
SHOW RESOURCES;
~~~

##### Hudiリソースの削除

次のステートメントを実行して、`hudi0`という名前のHudiリソースを削除します：

~~~SQL
DROP RESOURCE "hudi0";
~~~

> 注意：
>
> Hudiリソースを削除すると、そのHudiリソースを使用して作成されたすべてのHudi外部テーブルが使用できなくなります。ただし、削除はHudiに保存されているデータに影響を与えません。StarRocksを使用して引き続きHudiからデータをクエリする場合は、Hudiリソース、Hudiデータベース、およびHudi外部テーブルをStarRocksクラスターで再作成する必要があります。

#### Step 2: Hudiデータベースの作成

次のステートメントを実行して、StarRocksクラスター内の`hudi_test`という名前のHudiデータベースを作成およびオープンします：

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> 注意：
>
> StarRocksクラスター内のHudiデータベースに指定する名前は、関連するHudi内のデータベースと同じである必要はありません。

#### Step 3: Hudi外部テーブルの作成

次のステートメントを実行して、`hudi_test` Hudiデータベース内の`hudi_tbl`という名前のHudi外部テーブルを作成します：

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

次の表はパラメータを示しています。

| パラメータ | 説明                                                     |
| --------- | -------------------------------------------------------- |
| ENGINE    | Hudi外部テーブルのクエリエンジン。値を `HUDI` に設定します。       |
| resource  | StarRocksクラスター内のHudiリソースの名前。                     |
| database  | StarRocksクラスター内のHudi外部テーブルが属するHudiデータベースの名前。 |
| table     | Hudi外部テーブルが関連付けられているHudi管理テーブル。             |

> 注意：
>
> * Hudi外部テーブルに指定する名前は、関連するHudi管理テーブルと同じである必要はありません。
>
> * Hudi外部テーブルの列は、関連するHudi管理テーブルの対応する列と同じ名前でなければなりませんが、異なる順序であってもかまいません。
>
> * 関連するHudi管理テーブルからいくつかまたはすべての列を選択して、Hudi外部テーブルにのみ選択された列を作成することができます。次の表は、Hudiがサポートするデータ型とStarRocksがサポートするデータ型のマッピングを示しています。

| Hudiがサポートするデータ型   | StarRocksがサポートするデータ型 |
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
> StarRocksは、STRUCTまたはMAPタイプのデータ、およびMerge On ReadテーブルのARRAYタイプのデータのクエリをサポートしていません。

#### Step 4: Hudi外部テーブルからデータをクエリする

特定のHudi管理テーブルに関連付けられたHudi外部テーブルが作成されたら、Hudi外部テーブルにデータをロードする必要はありません。Hudiからデータをクエリするには、次のステートメントを実行します：

~~~SQL
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (廃止予定) MySQL外部テーブル
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
+ スター・スキーマでは、データは一般的にディメンションテーブルとファクトテーブルに分割されます。ディメンションテーブルはデータが少ないが、UPDATE操作が含まれます。現時点で、StarRocksは直接のUPDATE操作をサポートしていません（UPDATEはユニークキーテーブルを使用して実装することができます）。一部のシナリオでは、ディメンションテーブルをMySQLに格納し、直接データを読むことができます。

MySQLのデータをクエリするには、StarRocksで外部テーブルを作成し、MySQLデータベースのテーブルにマッピングする必要があります。テーブルを作成する際にMySQL接続情報を指定する必要があります。

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
* **table**: MySQLデータベースのテーブル名
```