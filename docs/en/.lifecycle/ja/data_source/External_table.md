---
displayed_sidebar: "Japanese"
---

# 外部テーブル

:::note
v3.0以降、Hive、Iceberg、およびHudiからデータをクエリする場合は、カタログを使用することをお勧めします。[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、および[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

v3.1以降、MySQLおよびPostgreSQLからデータをクエリする場合は、[JDBCカタログ](../data_source/catalog/jdbc_catalog.md)を使用し、Elasticsearchからデータをクエリする場合は[Elasticsearchカタログ](../data_source/catalog/elasticsearch_catalog.md)を使用することをお勧めします。
:::

StarRocksは、外部テーブルを使用して他のデータソースにアクセスすることができます。外部テーブルは、他のデータソースに格納されているデータテーブルを基に作成されます。StarRocksはデータテーブルのメタデータのみを保存します。外部テーブルを使用して他のデータソースのデータを直接クエリすることができます。StarRocksは、以下のデータソースをサポートしています：MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg、およびApache Hudi。**現在、別のStarRocksクラスタからデータを現在のStarRocksクラスタに書き込むことしかできません。データソースがStarRocks以外の場合、これらのデータソースからのデータの読み取りのみが可能です。**

2.5以降、StarRocksはデータキャッシュ機能を提供しており、外部データソース上のホットデータクエリを高速化することができます。詳細については、[データキャッシュ](data_cache.md)を参照してください。

## StarRocks外部テーブル

StarRocks 1.19以降、StarRocks外部テーブルを使用して、1つのStarRocksクラスタから別のStarRocksクラスタにデータを書き込むことができます。これにより、読み書きの分離が実現され、リソースの分離が向上します。まず、宛先StarRocksクラスタに宛先テーブルを作成し、次に、ソースStarRocksクラスタで、宛先テーブルと同じスキーマを持つStarRocks外部テーブルを作成し、`PROPERTIES`フィールドに宛先クラスタとテーブルの情報を指定します。

INSERT INTOステートメントを使用して、ソースクラスタから宛先クラスタにデータを書き込むことができます。これにより、次の目標を実現できます。

* StarRocksクラスタ間のデータ同期。
* 読み書きの分離。データはソースクラスタに書き込まれ、ソースクラスタのデータ変更が宛先クラスタに同期され、クエリサービスが提供されます。

以下のコードは、宛先テーブルと外部テーブルを作成する方法を示しています。

~~~SQL
# 宛先StarRocksクラスタに宛先テーブルを作成します。
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

# ソースStarRocksクラスタに外部テーブルを作成します。
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

# ソースクラスタから宛先クラスタにデータを書き込むために、StarRocks外部テーブルにデータを書き込みます。本番環境では、2番目のステートメントが推奨されます。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

パラメータ：

* **EXTERNAL:** このキーワードは、作成するテーブルが外部テーブルであることを示します。
* **host:** このパラメータは、宛先StarRocksクラスタのリーダーFEノードのIPアドレスを指定します。
* **port:** このパラメータは、宛先StarRocksクラスタのFEノードのRPCポートを指定します。

  :::note

  StarRocks外部テーブルが所属するソースクラスタが宛先StarRocksクラスタにアクセスできるようにするには、ネットワークとファイアウォールの設定を行い、次のポートへのアクセスを許可する必要があります。

  * FEノードのRPCポート。FEの設定ファイル**fe/fe.conf**の`rpc_port`を参照してください。デフォルトのRPCポートは`9020`です。
  * BEノードのbRPCポート。BEの設定ファイル**be/be.conf**の`brpc_port`を参照してください。デフォルトのbRPCポートは`8060`です。

  :::

* **user:** このパラメータは、宛先StarRocksクラスタにアクセスするために使用するユーザー名を指定します。
* **password:** このパラメータは、宛先StarRocksクラスタにアクセスするために使用するパスワードを指定します。
* **database:** このパラメータは、宛先テーブルが所属するデータベースを指定します。
* **table:** このパラメータは、宛先テーブルの名前を指定します。

StarRocks外部テーブルを使用する場合、次の制限が適用されます。

* StarRocks外部テーブルでは、INSERT INTOおよびSHOW CREATE TABLEコマンドのみを実行できます。他のデータ書き込み方法はサポートされていません。また、StarRocks外部テーブルからデータをクエリしたり、外部テーブルでDDL操作を実行したりすることはできません。
* 外部テーブルの作成構文は通常のテーブルの作成構文と同じですが、外部テーブルの列名およびその他の情報は宛先テーブルと同じである必要があります。
* 外部テーブルは、宛先テーブルのテーブルメタデータを10秒ごとに同期します。宛先テーブルでDDL操作が実行された場合、2つのテーブル間のデータ同期に遅延が発生する場合があります。

## JDBC互換データベースの外部テーブル

v2.3.0以降、StarRocksはJDBC互換データベースをクエリするための外部テーブルを提供しています。これにより、StarRocksにデータをインポートする必要なく、このようなデータベースのデータを高速に分析することができます。このセクションでは、StarRocksで外部テーブルを作成し、JDBC互換データベースのデータをクエリする方法について説明します。

### 前提条件

データをクエリする前に、FEおよびBEがJDBCドライバのダウンロードURLにアクセスできることを確認してください。ダウンロードURLは、JDBCリソースの作成に使用されるステートメントで指定されます。

### JDBCリソースの作成と管理

#### JDBCリソースの作成

データベースからデータをクエリするための外部テーブルを作成する前に、StarRocksでデータベースの接続情報を管理するためのJDBCリソースを作成する必要があります。データベースはJDBCドライバをサポートする必要があり、"ターゲットデータベース"として参照されます。リソースを作成した後、それを使用して外部テーブルを作成することができます。

次のステートメントを実行して、`jdbc0`という名前のJDBCリソースを作成します。

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

`PROPERTIES`内の必須パラメータは次のとおりです。

* `type`: リソースのタイプ。値を`jdbc`に設定します。

* `user`: ターゲットデータベースに接続するために使用されるユーザー名。

* `password`: ターゲットデータベースに接続するために使用されるパスワード。

* `jdbc_uri`: JDBCドライバがターゲットデータベースに接続するために使用するURI。URIの形式は、データベースURIの構文を満たす必要があります。一部の一般的なデータベースのURI構文については、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)の公式ウェブサイトを参照してください。

> 注意: URIにはターゲットデータベースの名前を含める必要があります。たとえば、前のコード例では、`jdbc_test`は接続するターゲットデータベースの名前です。

* `driver_url`: JDBCドライバJARパッケージのダウンロードURLです。HTTP URLまたはファイルURLがサポートされています。たとえば、`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`または`file:///home/disk1/postgresql-42.3.3.jar`です。

* `driver_class`: JDBCドライバのクラス名です。一般的なデータベースのJDBCドライバのクラス名は次のとおりです：
  * MySQL: com.mysql.jdbc.Driver (MySQL 5.xおよびそれ以前)、com.mysql.cj.jdbc.Driver (MySQL 6.xおよびそれ以降)
  * SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL: org.postgresql.Driver

リソースが作成されている間、FEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバJARパッケージをダウンロードし、チェックサムを生成し、BEがダウンロードしたJDBCドライバを検証するためにチェックサムを使用します。

> 注意: JDBCドライバJARパッケージのダウンロードに失敗した場合、リソースの作成も失敗します。

BEがJDBC外部テーブルを初めてクエリし、対応するJDBCドライバJARパッケージが自分のマシンに存在しないことを検出した場合、BEは`driver_url`パラメータで指定されたURLを使用してJDBCドライバJARパッケージをダウンロードし、すべてのJDBCドライバJARパッケージは`${STARROCKS_HOME}/lib/jdbc_drivers`ディレクトリに保存されます。

#### JDBCリソースの表示

次のステートメントを実行して、StarRocksのすべてのJDBCリソースを表示します。

~~~SQL
SHOW RESOURCES;
~~~

> 注意: `ResourceType`列は`jdbc`です。

#### JDBCリソースの削除

次のステートメントを実行して、`jdbc0`という名前のJDBCリソースを削除します。

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> 注意: JDBCリソースが削除されると、そのJDBCリソースを使用して作成されたすべてのJDBC外部テーブルが使用できなくなります。ただし、ターゲットデータベースのデータは失われません。ターゲットデータベースでデータをStarRocksでクエリする必要がある場合は、JDBCリソースとJDBC外部テーブルを再作成することができます。

### データベースの作成

次のステートメントを実行して、StarRocksで`jdbc_test`という名前のデータベースを作成し、アクセスします。

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> 注意: 前のステートメントで指定するデータベース名は、ターゲットデータベースの名前と同じである必要はありません。

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

`properties`内の必須パラメータは次のとおりです。

* `resource`: 外部テーブルの作成に使用するJDBCリソースの名前。

* `table`: データベース内のターゲットテーブル名。

サポートされるデータ型とStarRocksとターゲットデータベース間のデータ型マッピングについては、[データ型マッピング](External_table.md#データ型マッピング)を参照してください。

> 注意：
>
> * インデックスはサポートされていません。
> * PARTITION BYまたはDISTRIBUTED BYを使用してデータの分散ルールを指定することはできません。

### JDBC外部テーブルのクエリ

JDBC外部テーブルをクエリする前に、次のステートメントを実行してパイプラインエンジンを有効にする必要があります。

~~~SQL
set enable_pipeline_engine=true;
~~~

> 注意: パイプラインエンジンがすでに有効になっている場合は、このステップをスキップできます。

次のステートメントを実行して、JDBC外部テーブル内のデータをクエリします。

~~~SQL
select * from JDBC_tbl;
~~~

StarRocksは、フィルタ条件をElasticsearchにプッシュダウンして実行することで、プレディケートプッシュダウンをサポートしています。データソースにできるだけ近い場所でフィルタ条件を実行することで、クエリのパフォーマンスを向上させることができます。現在、StarRocksは、バイナリ比較演算子（`>`, `>=`, `=`, `<`, `<=`）、`IN`、`IS NULL`、および`BETWEEN ... AND ...`を含む演算子をプッシュダウンすることができます。ただし、関数はプッシュダウンすることはできません。

### データ型マッピング

現在、StarRocksは、NUMBER、STRING、TIME、DATEなどの基本的なタイプのデータのみをターゲットデータベースからクエリすることができます。ターゲットデータベースのデータ値の範囲がStarRocksでサポートされていない場合、クエリはエラーが発生します。

ターゲットデータベースとStarRocksの間のデータ型のマッピングは、ターゲットデータベースのタイプに基づいて異なります。

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

### 制限事項

* JDBC外部テーブルを作成する場合、テーブルにインデックスを作成したり、PARTITION BYやDISTRIBUTED BYを使用してテーブルのデータ分散ルールを指定したりすることはできません。

* JDBC外部テーブルをクエリする場合、StarRocksは関数をテーブルにプッシュダウンすることはできません。

## (非推奨) Elasticsearch外部テーブル

StarRocksとElasticsearchは2つの人気のある分析システムです。StarRocksは大規模な分散コンピューティングで高いパフォーマンスを発揮します。Elasticsearchはフルテキスト検索に最適です。StarRocksとElasticsearchを組み合わせることで、より完全なOLAPソリューションを提供することができます。

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

次の表に、パラメータの説明を示します。

| **パラメータ**        | **必須** | **デフォルト値** | **説明**                                              |
| -------------------- | ------------ | ----------------- | ------------------------------------------------------------ |
| hosts                | Yes          | None              | Elasticsearchクラスタの接続アドレスです。1つまたは複数のアドレスを指定できます。StarRocksはこのアドレスからElasticsearchのバージョンとインデックスシャードの割り当てを解析することができます。StarRocksは、`GET /_nodes/http` API操作で返されるアドレスに基づいてElasticsearchクラスタと通信します。したがって、`host`パラメータの値は、`GET /_nodes/http` API操作で返されるアドレスと同じである必要があります。そうでない場合、BEはElasticsearchクラスタと通信できない場合があります。 |
| index                | Yes          | None              | StarRocksのテーブルに作成されるElasticsearchインデックスの名前です。名前はエイリアスにすることもできます。このパラメータはワイルドカード（\*）をサポートします。たとえば、`index`を<code class="language-text">hello*</code>に設定すると、`hello`で始まるすべてのインデックスが取得されます。 |
| user                 | No           | Empty             | 基本認証が有効になっているElasticsearchクラスタにログインするために使用されるユーザー名です。`/*cluster/state/*nodes/http`およびインデックスにアクセスできることを確認してください。 |
| password             | No           | Empty             | Elasticsearchクラスタにログインするために使用されるパスワードです。 |
| type                 | No           | `_doc`            | インデックスのタイプです。デフォルト値は`_doc`です。Elasticsearch 8以降のバージョンでは、マッピングタイプが削除されているため、このパラメータを設定する必要はありません。 |
| es.nodes.wan.only    | No           | `false`           | StarRocksは、`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データをフェッチするかどうかを指定します。<ul><li>`true`：StarRocksは、`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データノードのスニッフィングを行いません。Elasticsearchクラスタのインデックスのシャードが存在するデータノードのアドレスにアクセスできない場合は、このパラメータを`true`に設定する必要があります。</li><li>`false`：StarRocksは、`host`で指定されたアドレスを使用してElasticsearchクラスタのデータノードをスニッフィングします。クエリの実行計画が生成された後、関連するBEはインデックスのシャードからデータをフェッチするため、Elasticsearchクラスタのデータノードのアドレスにアクセスできる場合は、デフォルト値`false`を保持することをお勧めします。</li></ul> |
| es.net.ssl           | No           | `false`           | HTTPSプロトコルを使用してElasticsearchクラスタにアクセスできるかどうかを指定します。StarRocks 2.4以降のバージョンのみがこのパラメータの設定をサポートしています。<ul><li>`true`：HTTPSおよびHTTPの両方のプロトコルを使用してElasticsearchクラスタにアクセスできます。</li><li>`false`：ElasticsearchクラスタにアクセスするためにHTTPプロトコルのみを使用できます。</li></ul> |
| enable_docvalue_scan | No           | `true`            | Elasticsearchのカラムストアから対象フィールドの値を取得するかどうかを指定します。ほとんどの場合、カラムストアからデータを読み取る方が行ストアからデータを読み取るよりもパフォーマンスが向上します。 |
| enable_keyword_sniff | No           | `true`            | KEYWORDタイプのフィールドをベースにElasticsearchのTEXTタイプのフィールドをスニッフィングするかどうかを指定します。このパラメータを`false`に設定する場合、StarRocksはトークン化後にマッチングを実行します。 |

##### より高速なクエリのためのカラムスキャン

`enable_docvalue_scan`を`true`に設定すると、StarRocksはElasticsearchからデータを取得する際に次のルールに従います。

* **試してみる**: StarRocksは自動的に対象フィールドのカラムストアが有効かどうかをチェックします。有効な場合、StarRocksはカラムストアから対象フィールドのすべての値を取得します。
* **自動ダウングレード**: 対象フィールドのいずれかがカラムストアで利用できない場合、StarRocksは行ストア（`_source`）から対象フィールドのすべての値を解析して取得します。

> **注意**
>
> * Elasticsearchでは、TEXTタイプのフィールドではカラムストアは使用できません。したがって、TEXTタイプの値を含むフィールドをクエリする場合、StarRocksはフィールドの値を`_source`から取得します。
> * 大量（25以上）のフィールドをクエリする場合、`docvalue`からフィールドの値を読み取ることは、`_source`からフィールドの値を読み取ることと比較して明らかな利点はありません。

##### KEYWORDタイプのフィールドのスニッフィング

`enable_keyword_sniff`を`true`に設定すると、Elasticsearchはインデックスなしで直接データをインジェストできます。インデックスは、インジェスト後に自動的に作成されます。STRINGタイプのフィールドについては、ElasticsearchはTEXTタイプとKEYWORDタイプの両方を持つフィールドを作成します。これは、ElasticsearchのMulti-Field機能の動作方法です。マッピングは次のようになります。

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

たとえば、`k4`に対して"="フィルタリングを行う場合、StarRocksはElasticsearch上でフィルタリング操作をElasticsearchのTermQueryに変換します。

元のSQLフィルタは次のようになります。

~~~SQL
k4 = "StarRocks On Elasticsearch"
~~~

変換されたElasticsearchのクエリDSLは次のようになります。

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"

}
~~~

`k4`の最初のフィールドはTEXTであり、データインジェスト後に`k4`に設定されたアナライザー（または`k4`にアナライザーが設定されていない場合は標準アナライザー）によってトークン化されます。その結果、最初のフィールドは3つのトークンにトークン化されます：`StarRocks`、`On`、および`Elasticsearch`。詳細は次のとおりです。

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

トークン化の結果は次のとおりです。

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

次のようなクエリを実行するとします。

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

辞書には、`StarRocks On Elasticsearch`に一致する用語はありませんので、結果は返されません。

ただし、`enable_keyword_sniff`を`true`に設定している場合、StarRocksは`k4 = "StarRocks On Elasticsearch"`を`k4.keyword = "StarRocks On Elasticsearch"`に変換し、SQLのセマンティクスに一致するようにします。変換された`StarRocks On Elasticsearch`クエリDSLは次のようになります。

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword`はKEYWORDタイプです。したがって、データは完全な用語としてElasticsearchに書き込まれ、一致が可能になります。

#### カラムデータ型のマッピング

外部テーブルを作成する際には、Elasticsearchテーブルのカラムデータ型に基づいて外部テーブルのカラムデータ型を指定する必要があります。次の表は、カラムデータ型のマッピングを示しています。

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
> * StarRocksは、JSON関連の関数を使用してNESTEDタイプのデータを読み取ります。
> * Elasticsearchは、多次元配列を1次元配列に自動的にフラット化します。StarRocksも同様に動作します。**ElasticsearchからARRAYデータをクエリするサポートはv2.5から追加されました。**

### プレディケートプッシュダウン

StarRocksはプレディケートプッシュダウンをサポートしています。フィルタをElasticsearchにプッシュダウンして実行することができ、クエリのパフォーマンスが向上します。次の表に、プレディケートプッシュダウンをサポートする演算子を示します。

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

**esquery関数**は、SQLで表現できないクエリ（matchやgeoshapeなど）をElasticsearchにフィルタリングするためにプッシュダウンするために使用されます。esquery関数の最初のパラメータはインデックスを関連付けるために使用されます。2番目のパラメータは、基本的なQuery DSLのJSON式であり、中括弧{}で囲まれています。**JSON式にはただ1つのルートキー**が必要です。match、geo_shape、またはboolなどです。

* matchクエリ

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on elasticsearch"
    }
}');
~~~

* 地理関連クエリ

~~~sql
select * from es_table where esquery(k4, '{
  "geo_shape": {
     "location": {
        "shape": {
           "type": "envelope",
           "coordinates": [
[
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

* Elasticsearch 5.x より前のバージョンでは、データのスキャン方法が異なるため、現在は **5.x より新しいバージョンのみ** サポートされています。
* HTTP 基本認証が有効になっている Elasticsearch クラスタもサポートされています。
* StarRocks からデータをクエリする場合、Elasticsearch から直接データをクエリする場合と比較して、パフォーマンスが低下することがあります。特に count 関連のクエリでは、Elasticsearch は実際のデータをフィルタリングする必要がないため、対象ドキュメントのメタデータを直接読み取るだけで済むため、クエリの高速化が図られます。

## (非推奨) Hive 外部テーブル

Hive 外部テーブルを使用する前に、サーバーに JDK 1.8 がインストールされていることを確認してください。

### Hive リソースの作成

Hive リソースは Hive クラスタに対応します。StarRocks で使用する Hive クラスタを構成する必要があります。たとえば、Hive メタストアのアドレスを指定する必要があります。Hive 外部テーブルで使用する Hive リソースを指定する必要があります。

* hive0 という名前の Hive リソースを作成します。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://10.10.44.98:9083"
);
~~~

* StarRocks で作成したリソースを表示します。

~~~sql
SHOW RESOURCES;
~~~

* hive0 という名前のリソースを削除します。

~~~sql
DROP RESOURCE "hive0";
~~~

StarRocks 2.3 以降では、Hive リソースの `hive.metastore.uris` を変更することができます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

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

例: `hive0` リソースに対応する Hive クラスタの `rawdata` データベースの下に `profile_parquet_p7` という外部テーブルを作成します。

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
  * 列の順序は、Hive テーブルの列の順序と **同じである必要はありません**。
  * Hive テーブルの列の一部のみを選択することができますが、すべての **パーティションキーの列** を選択する必要があります。
  * 外部テーブルのパーティションキーの列は `partition by` を使用して指定する必要はありません。他の列と同じ記述リストで定義する必要があります。パーティション情報を指定する必要はありません。StarRocks は Hive テーブルからこの情報を自動的に同期します。
  * `ENGINE` を HIVE に設定します。
* PROPERTIES:
  * **hive.resource**: 使用する Hive リソースです。
  * **database**: Hive データベースです。
  * **table**: Hive のテーブルです。**ビュー** はサポートされていません。
* 次の表は、Hive と StarRocks の列データ型のマッピングを示しています。

    |  Hive の列データ型  |  StarRocks の列データ型   | 説明 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |
    |   TIMESTAMP  | DATETIME    | TIMESTAMP データを DATETIME データに変換する際に、精度とタイムゾーン情報が失われます。タイムゾーンはセッション変数のタイムゾーンに基づいて、タイムゾーンオフセットを持たない DATETIME データに TIMESTAMP データを変換する必要があります。 |
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意:
>
> * 現在、サポートされている Hive のストレージ形式は Parquet、ORC、CSV のみです。
ストレージ形式が CSV の場合、エスケープ文字として引用符を使用することはできません。
> * SNAPPY および LZ4 圧縮形式がサポートされています。
> * クエリの対象となる Hive の文字列列の最大長は 1 MB です。文字列列が 1 MB を超える場合、null 列として処理されます。

### Hive 外部テーブルの使用

`profile_wos_p7` の総行数をクエリします。

~~~sql
select count(*) from profile_wos_p7;
~~~

### キャッシュされた Hive テーブルのメタデータを更新する

* Hive パーティション情報および関連するファイル情報は、StarRocks でキャッシュされます。キャッシュは `hive_meta_cache_refresh_interval_s` で指定された間隔で更新されます。デフォルト値は 7200 です。`hive_meta_cache_ttl_s` はキャッシュのタイムアウト期間を指定します。デフォルト値は 86400 です。
  * キャッシュされたデータは手動で更新することもできます。
    1. Hive のテーブルにパーティションが追加または削除された場合、StarRocks でキャッシュされたテーブルのメタデータを更新するために `REFRESH EXTERNAL TABLE hive_t` コマンドを実行する必要があります。`hive_t` は StarRocks での Hive 外部テーブルの名前です。
    2. 一部の Hive パーティションのデータが更新された場合、`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')` コマンドを実行して、StarRocks でキャッシュされたデータを更新する必要があります。`hive_t` は StarRocks での Hive 外部テーブルの名前です。`'k1=01/k2=02'` および `'k1=03/k2=04'` はデータが更新された Hive パーティションの名前です。
    3. `REFRESH EXTERNAL TABLE hive_t` を実行すると、StarRocks はまず Hive 外部テーブルの列情報が Hive メタストアから返される Hive テーブルの列情報と同じかどうかをチェックします。Hive テーブルのスキーマが変更された場合（列の追加や削除など）、StarRocks は変更を Hive 外部テーブルに同期します。同期後、Hive 外部テーブルの列順序は Hive テーブルの列順序と同じままで、パーティション列は最後の列になります。
* Hive データが Parquet、ORC、CSV 形式で保存されている場合、StarRocks 2.3 以降では、Hive テーブルのスキーマ変更（列の追加や置換など）を Hive 外部テーブルに同期することができます。

### オブジェクトストレージへのアクセス

* FE の設定ファイルのパスは `fe/conf` であり、Hadoop クラスタをカスタマイズする必要がある場合は、設定ファイルを追加できます。たとえば: HDFS クラスタが高可用名前サービスを使用している場合、`hdfs-site.xml` を `fe/conf` の下に配置する必要があります。HDFS が ViewFs で構成されている場合、`core-site.xml` を `fe/conf` の下に配置する必要があります。
* BE の設定ファイルのパスは `be/conf` であり、Hadoop クラスタをカスタマイズする必要がある場合は、設定ファイルを追加できます。たとえば、HDFS クラスタが高可用名前サービスを使用している場合、`hdfs-site.xml` を `be/conf` の下に配置する必要があります。HDFS が ViewFs で構成されている場合、`core-site.xml` を `be/conf` の下に配置する必要があります。
* BE が配置されているマシンで、BE **起動スクリプト** `bin/start_be.sh` に対して JRE 環境ではなく JDK 環境として JAVA_HOME を設定します。たとえば、`export JAVA_HOME = <JDK パス>` のように設定します。この設定はスクリプトの先頭に追加し、BE を再起動して設定が有効になるようにする必要があります。
* Kerberos サポートの設定:
  1. `kinit -kt keytab_path principal` をすべての FE/BE マシンで実行してログインするには、Hive と HDFS にアクセスする権限が必要です。kinit コマンドのログインは一定期間有効であり、定期的に実行するために crontab に追加する必要があります。
  2. `fe/conf` に `hive-site.xml/core-site.xml/hdfs-site.xml` を配置し、`be/conf` に `core-site.xml/hdfs-site.xml` を配置します。
  3. **$FE_HOME/conf/fe.conf** ファイルの `JAVA_OPTS` オプションの値に `-Djava.security.krb5.conf=/etc/krb5.conf` を追加します。**/etc/krb5.conf** は **krb5.conf** ファイルの保存パスです。オペレーティングシステムに基づいてパスを変更できます。
  4. **$BE_HOME/conf/be.conf** ファイルに直接 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。**/etc/krb5.conf** は **krb5.conf** ファイルの保存パスです。オペレーティングシステムに基づいてパスを変更できます。
  5. Hive リソースを追加する場合、`hive.metastore.uris` にドメイン名を渡す必要があります。さらに、Hive/HDFS ドメイン名と IP アドレスのマッピングを **/etc/hosts** ファイルに追加する必要があります。

* AWS S3 のサポートの設定: `fe/conf/core-site.xml` および `be/conf/core-site.xml` に次の設定を追加します。

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

   1. `fs.s3a.access.key`: AWS のアクセスキー ID。
   2. `fs.s3a.secret.key`: AWS のシークレットキー。
   3. `fs.s3a.endpoint`: 接続する AWS S3 のエンドポイント。
   4. `fs.s3a.connection.maximu``m`: StarRocks から S3 への同時接続数の最大値。クエリ中に `Timeout waiting for connection from poll` エラーが発生した場合は、このパラメータを大きな値に設定できます。

## (非推奨) Iceberg 外部テーブル

StarRocks では、v2.1.0 以降、Apache Iceberg のデータレイクからデータをクエリするための Iceberg 外部テーブルを作成できます。Iceberg データをクエリするためには、StarRocks で Iceberg 外部テーブルを作成する必要があります。テーブルを作成する際には、外部テーブルとクエリする Iceberg テーブルとの間のマッピングを確立する必要があります。

### 事前準備

StarRocks が Apache Iceberg にアクセスするためのメタデータサービス（Hive メタストアなど）、ファイルシステム（HDFS など）、およびオブジェクトストレージシステム（Amazon S3 や Alibaba Cloud Object Storage Service など）へのアクセス権限があることを確認してください。

### 注意事項

* StarRocks では、Hudi データレイクからデータをクエリするための Hudi 外部テーブルを作成できます。Hudi 外部テーブルは読み取り専用であり、クエリにのみ使用できます。
* StarRocks は、Copy on Write テーブルと Merge On Read テーブルのクエリをサポートしています（Merge On Read テーブルは v2.5 以降でサポートされています）。これらの 2 つのテーブルの違いについては、[Table & Query Types](https://hudi.apache.org/docs/table_types/) を参照してください。
* StarRocks は、Hudi の次の 2 つのクエリタイプをサポートしています: スナップショットクエリと Read Optimized クエリ（Merge On Read テーブルでは Read Optimized クエリのみサポートされます）。インクリメンタルクエリはサポートされていません。Hudi のクエリタイプの詳細については、[Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types) を参照してください。
* StarRocks は、Hudi ファイルの次の圧縮形式をサポートしています: gzip、zstd、LZ4、Snappy。Hudi ファイルのデフォルトの圧縮形式は gzip です。
* StarRocks は、Hudi 管理テーブルからスキーマ変更を同期することはできません。詳細については、[Schema Evolution](https://hudi.apache.org/docs/schema_evolution/) を参照してください。Hudi 管理テーブルのスキーマが変更された場合、StarRocks で関連する Hudi 外部テーブルを削除し、その外部テーブルを再作成する必要があります。

### 手順

#### ステップ 1: Iceberg リソースの作成

Iceberg リソースを StarRocks に作成する必要があります。リソースは、StarRocks で作成する Iceberg データベースと外部テーブルを管理するために使用されます。

##### Iceberg リソースの作成

次のステートメントを実行して、`hudi0` という名前の Iceberg リソースを作成します。

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "iceberg", 
    "hive.metastore.uris" = "thrift://192.168.0.81:9083" 
);
~~~

次の表に、関連するパラメータの説明を示します。

| パラメータ | 説明 |
| --------- | ------------------------------------------------------------ |
| type      | Iceberg リソースのタイプです。値を `iceberg` に設定します。 |
| hive.metastore.uris | Iceberg リソースが接続する Hive メタストアの Thrift URI です。Hive メタストアに接続した後、Hive を使用して Iceberg テーブルを作成および管理できます。Thrift URI は `<Hive メタストアの IP アドレス>:<Hive メタストアのポート番号>` の形式です。デフォルトのポート番号は 9083 です。 |

StarRocks 2.3 以降では、Iceberg リソースの `hive.metastore.uris` の値を変更することができます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md) を参照してください。

##### Iceberg リソースの表示

次のステートメントを実行して、StarRocks クラスタで作成されたすべての Iceberg リソースを表示します。

~~~SQL
SHOW RESOURCES;
~~~

##### Iceberg リソースの削除

次のステートメントを実行して、`hudi0` という名前の Iceberg リソースを削除します。

~~~SQL
DROP RESOURCE "hudi0";
~~~

> 注意:
>
> Iceberg リソースを削除すると、そのリソースを参照するすべての Iceberg 外部テーブルが使用できなくなります。ただし、Apache Iceberg に格納されているデータは削除されません。StarRocks でデータをクエリする必要がある場合は、新しいリソースと新しい外部テーブルを作成する必要があります。

#### ステップ 2: データベースの作成

次のステートメントを実行して、StarRocks クラスタに `hudi_test` という名前のデータベースを作成し、開きます。

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> 注意:
>
> StarRocks クラスタの Hudi データベースの名前は、Hudi の関連するデータベースと同じである必要はありません。

#### ステップ 3: Iceberg 外部テーブルの作成

次のステートメントを実行して、`hudi_test` データベースに `hudi_tbl` という名前の Iceberg 外部テーブルを作成します。

~~~SQL
CREATE EXTERNAL TABLE `hudi_tbl` ( 
    `id` bigint NULL, 
    `data` varchar(200) NULL 
) ENGINE=ICEBERG 
PROPERTIES ( 
    "resource" = "hudi0", 
    "database" = "hudi", 
    "table" = "hudi_table" 
); 
~~~

次の表に、関連するパラメータの説明を示します。

| パラメータ | 説明 |
| --------- | ------------------------------------------------------------ |
| ENGINE    | Iceberg 外部テーブルのクエリエンジンです。値を `ICEBERG` に設定します。 |
| resource  | StarRocks クラスタの Iceberg リソースの名前です。 |
| database  | StarRocks クラスタの Iceberg 外部テーブルが所属する Iceberg データベースの名前です。 |
| テーブル | Hudiの外部テーブルと関連付けられるHudiの管理テーブルです。 |

> 注意：
>
> * Hudiの外部テーブルに指定する名前は、関連付けられるHudiの管理テーブルと同じである必要はありません。
>
> * Hudiの外部テーブルの列は、関連するHudiの管理テーブルの対応する列と同じ名前である必要がありますが、異なる順序である場合があります。
>
> * 関連するHudiの管理テーブルからいくつかまたはすべての列を選択し、Hudiの外部テーブルには選択した列のみを作成することができます。次の表には、Hudiがサポートするデータ型とStarRocksがサポートするデータ型のマッピングが示されています。

| Hudiがサポートするデータ型 | StarRocksがサポートするデータ型 |
| ---------------------------- | --------------------------------- |
| BOOLEAN                      | BOOLEAN                           |
| INT                          | TINYINT/SMALLINT/INT              |
| DATE                         | DATE                              |
| TimeMillis/TimeMicros        | TIME                              |
| TimestampMillis/TimestampMicros | DATETIME                          |
| LONG                         | BIGINT                            |
| FLOAT                        | FLOAT                             |
| DOUBLE                       | DOUBLE                            |
| STRING                       | CHAR/VARCHAR                      |
| ARRAY                        | ARRAY                             |
| DECIMAL                      | DECIMAL                           |

> **注意**
>
> StarRocksはSTRUCT型やMAP型のデータのクエリをサポートしておらず、Merge On ReadテーブルのARRAY型のデータのクエリもサポートしていません。

#### ステップ4: Hudiの外部テーブルからデータをクエリする

特定のHudiの管理テーブルと関連付けられたHudiの外部テーブルを作成した後、Hudiからデータをクエリするためには、次のステートメントを実行します。

~~~SQL
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (非推奨) MySQLの外部テーブル

スターシェマでは、データは一般的に次元テーブルとファクトテーブルに分割されます。次元テーブルはデータが少ないですが、UPDATE操作が関与します。現在、StarRocksは直接のUPDATE操作をサポートしていません（ユニークキーテーブルを使用して実装することができます）。一部のシナリオでは、MySQLに次元テーブルを保存して直接データを読み取ることができます。

MySQLのデータをクエリするには、StarRocksで外部テーブルを作成し、それをMySQLデータベースのテーブルにマッピングする必要があります。テーブルを作成する際に、MySQLの接続情報を指定する必要があります。

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

パラメーター:

* **host**: MySQLデータベースの接続アドレス
* **port**: MySQLデータベースのポート番号
* **user**: MySQLにログインするためのユーザー名
* **password**: MySQLにログインするためのパスワード
* **database**: MySQLデータベースの名前
* **table**: MySQLデータベースのテーブルの名前
