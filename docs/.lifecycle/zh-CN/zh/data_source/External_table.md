---
displayed_sidebar: "Chinese"
---

# (即将废弃) 外部表

StarRocks支持通过外部表（External Table）的形式接入其他数据源。外部表指的是保存在其他数据源中的数据表，而StartRocks仅保存表对应的元数据，并直接向外部表所在数据源发起查询。目前StarRocks已支持的第三方数据源包括MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg和Apache Hudi。**对于StarRocks数据源，目前仅支持Insert写入，不支持读取；对于其他数据源，目前仅支持读取，还不支持写入**。

> **注意**
>
> * 从3.0版本开始，对于查询Hive、Iceberg、Hudi数据源的场景，推荐使用Catalog。参见[Hive catalog](../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../data_source/catalog/hudi_catalog.md)。
> * 从3.1版本开始，对于查询MySQL、PostgreSQL的场景推荐使用[JDBC catalog](../data_source/catalog/jdbc_catalog.md)，对于查询Elasticsearch的场景推荐使用[Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md)。

从2.5版本开始，查询外部数据源时支持数据缓存（Data Cache），提升对热数据的查询性能。参见[Data Cache](data_cache.md)。

## StarRocks外部表

自1.19版本开始，StarRocks支持以外表方式将数据写入另一个StarRocks集群的表中。这可以解决用户的读写分离需求，提供更好的资源隔离。用户需要首先在目标集群上创建一张目标表，然后在源StarRocks集群上创建一个Schema信息一致的外表，并在属性中指定目标集群和表的信息。

通过INSERT INTO写入数据至StarRocks外表，可以将源集群的数据写入至目标集群。借助这一能力，可以实现如下目标：

* 集群间的数据同步。
* 读写分离。向源集群中写入数据，并且源集群的数据变更同步至目标集群，目标集群提供查询服务。

以下是创建目标表和外表的示例：

~~~sql
# 在目标集群上执行
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

# 在外表集群上执行
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

# 写入数据至StarRocks外表，实现源集群的数据写入至目标集群。推荐生产环境使用第二种方式。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');

insert into external_t select * from other_table;
~~~

其中：

* **EXTERNAL**：该关键字指定创建的是StarRocks外表。
* **host**：该属性描述目标表所属StarRocks集群Leader FE的IP地址。
* **port**：该属性描述目标表所属StarRocks集群FE的RPC访问端口。

  :::note

  为确保外表所属集群能够正常访问目标表所属StarRocks集群，您需要确保网络策略和防火墙设置允许以下端口的访问：

  * FE的RPC访问端口，可参考配置文件**fe/fe.conf**中的`rpc_port`配置取值，默认为`9020`。
  * BE的bRPC访问端口，可参考配置文件**be/be.conf**中的`brpc_port`配置取值，默认为`8060`。

  :::

* **user**：该属性描述目标表所属StarRocks集群的访问用户名。
* **password**：该属性描述目标表所属StarRocks集群的访问密码。
* **database**：该属性描述目标表所属数据库名称。
* **table**：该属性描述目标表名称。

目前StarRocks外表使用上有以下限制：

* 仅可以在外表上执行insert into和show create table操作，不支持其他数据写入方式，也不支持查询和DDL。
* 创建外表语法和创建普通表一致，但其中的列名等信息请保持同其对应的目标表一致。
* 外表会周期性从目标表同步元信息（同步周期为10秒），在目标表执行的DDL操作可能会延迟一定时间反应在外表上。

## 更多数据库（JDBC）的外部表

自2.3.0版本起，StarRocks支持通过外部表的方式查询支持JDBC的数据库，无需将数据导入至StarRocks，即可实现对这类数据库的极速分析。本文介绍如何在StarRocks创建外部表，查询支持JDBC的数据库中的数据。

### 前提条件

在您使用JDBC外表时，FE、BE节点会下载JDBC驱动程序，因此FE、BE节点所在机器必须能够访问用于下载JDBC驱动程序JAR包的URL，该URL由创建JDBC资源中的配置项`driver_url`指定。

### 创建和管理JDBC资源

#### 创建JDBC资源

您需要提前在StarRocks中创建JDBC资源，用于管理数据库的相关连接信息。这里的数据库是指支持JDBC的数据库，以下简称为“目标数据库”。创建资源后，即可使用该资源创建外部表。

例如目标数据库为PostgreSQL，则可以执行如下语句，创建一个名为`jdbc0`的JDBC资源，用于访问PostgreSQL：

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

`PROPERTIES`的必填配置项：

* `type`：资源类型，固定取值为`jdbc`。

* `user`：目标数据库用户名。

* `password`：目标数据库用户登录密码。

* `jdbc_uri`：JDBC driver连接目标数据库的URI，必须符合目标数据库URI的语法。常见的目标数据库URI，请参见[MySQL](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html)、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/use/#connecting-to-the-database)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)官方文档。

    > **说明**
    >
    > 目标数据库URI中必须指定具体数据库的名称，如上示例中的`jdbc_test`。

* `driver_url`：用于下载JDBC driver JAR包的URL，支持使用HTTP协议或者file协议。例如`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`，`file:///home/disk1/postgresql-42.3.3.jar`。

    > **说明**
    >
    > 不同目标数据库使用不同的JDBC driver，使用其他数据库的JDBC driver会产生不兼容的问题，建议访问目标数据库官方网站，查询并使用其支持的JDBC driver。常见目标数据库的JDBC driver下载地址，请参见[MySQL](https://dev.mysql.com/downloads/connector/j/)、[Oracle](https://www.oracle.com/database/technologies/maven-central-guide.html)、[PostgreSQL](https://jdbc.postgresql.org/download/)、[SQL Server](https://learn.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver16)。

* `driver_class`：JDBC driver的类名称。

  以下列举常见JDBC driver的类名称：

  * MySQL: com.mysql.jdbc.Driver（MySQL 5.x及以下版本）、com.mysql.cj.jdbc.Driver（MySQL 8.x及以上版本）
  * SQL Server：com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL：org.postgresql.Driver

创建资源时，FE通过`driver_url`下载JDBC driver JAR包，生成checksum并保存起来，用于校验BE下载的JDBC driver JAR包的正确性。

> **说明**
>
> 如果下载JDBC driver失败，则创建资源也会失败。

BE节点首次查询JDBC外部表时，如果发现所在机器上不存在相应的JDBC driver JAR包，则会通过`driver_url`进行下载，所有的JDBC driver JAR包都会保存在**`${STARROCKS_HOME}/lib/jdbc_drivers`**目录下。

#### 查看JDBC资源

执行如下语句，查看StarRocks中的所有JDBC资源：

> **说明**
>
> `ResourceType`列为`jdbc`。

~~~SQL
SHOW RESOURCES;
~~~

#### 删除JDBC资源

执行如下语句，删除名为`jdbc0`的JDBC资源：

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> **说明**
>
> 删除JDBC资源会导致使用该JDBC资源创建的JDBC外部表不可用，但目标数据库的数据并不会丢失。如果您仍需要通过StarRocks查询目标数据库的数据，可以重新创建JDBC资源和JDBC外部表。

### 创建数据库

执行如下语句，在StarRocks中创建并进入名为`jdbc_test`的数据库：

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> **说明**
>
> 库名无需与目标数据库的名称保持一致。

### 创建JDBC外部表

执行如下语句，在数据库`jdbc_test`中，创建一张名为`jdbc_tbl`的JDBC外部表：

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

`PROPERTIES`配置项：

* `resource`：所使用JDBC资源的名称，必填项。

* `table`：目标数据库表名，必填项。

支持的数据类型以及与StarRocks的数据类型映射关系，请参见[数据类型映射](#数据类型映射)。

> **说明**
>
> * 不支持索引。
> * 不支持通过PARTITION BY、DISTRIBUTED BY来指定数据分布规则。

### 查询JDBC外部表

查询JDBC外部表前，必须启用Pipeline引擎。

> **说明**
>
> 如果已经启用Pipeline引擎，则可跳过本步骤。

~~~SQL
set enable_pipeline_engine=true;
~~~

执行如下语句，通过JDBC外部表查询目标数据库的数据：

~~~SQL
select * from jdbc_tbl;
~~~

StarRocks支持对目标表进行谓词下推，把过滤条件推给目标表执行，让执行尽量靠近数据源，进而提高查询性能。目前支持下推运算符，包括二元比较运算符（`>`、`>=`、`=`、`<`、`<=`）、`IN`、`IS NULL`和`BETWEEN ... AND ...`，但是不支持下推函数。

### 数据类型映射

目前仅支持查询目标数据库中数字、字符串、时间、日期等基础类型的数据。如果目标数据库中的数据超出StarRocks中数据类型的表示范围，则查询会报错。

如下以目标数据库MySQL、Oracle、PostgreSQL、SQL Server为例，说明支持查询的数据类型，以及与StarRocks数据类型的映射关系。

#### 目标数据库为MySQL

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

#### 目标数据库为Oracle

| Oracle          | StarRocks |
| --------------- | --------- |
| CHAR            | CHAR      |
| VARCHAR/VARCHAR2 | VARCHAR   |
| DATE            | DATE      |
| SMALLINT        | SMALLINT  |
| INT             | INT       |
| DATE            | DATETIME      |
| NUMBER          | DECIMAL   |

#### 目标数据库为PostgreSQL

| PostgreSQL          | StarRocks |
| ------------------- | --------- |
| SMALLINT/SMALLSERIAL | SMALLINT  |
| INTEGER/SERIAL       | INT       |
| BIGINT/BIGSERIAL     | BIGINT    |
| BOOLEAN             | BOOLEAN   |
| REAL                | FLOAT     |
| DOUBLE PRECISION    | DOUBLE    |
| DECIMAL             | DECIMAL   |
| TIMESTAMP           | DATETIME  |
| DATE                | DATE      |
| CHAR                | CHAR      |
| VARCHAR             | VARCHAR   |
| TEXT                | VARCHAR   |

#### 目标数据库为SQL Server

| SQL Server        | StarRocks |
| ----------------- | --------- |
| BIT           | BOOLEAN   |
| TINYINT           | TINYINT   |
| SMALLINT          | SMALLINT  |
| INT               | INT       |
| BIGINT            | BIGINT    |
| FLOAT             | FLOAT/DOUBLE     |
| REAL              | FLOAT    |
| DECIMAL/NUMERIC    | DECIMAL   |
| CHAR              | CHAR      |
| VARCHAR           | VARCHAR   |
| DATE              | DATE      |
| DATETIME/DATETIME2 | DATETIME  |

### 使用限制

* 当创建 JDBC 外部表时，不支持索引功能，也不支持通过 PARTITION BY、DISTRIBUTED BY 来指定数据分布规则。
* 在使用 JDBC 外部表进行查询时，不支持下推函数。

## (已弃用) Elasticsearch 外部表

若要查询 Elasticsearch 中的数据，需要在 StarRocks 中创建 Elasticsearch 外部表，并将外部表与待查询的 Elasticsearch 表建立映射。StarRocks 与 Elasticsearch 都是目前流行的分析系统。StarRocks 擅长大规模分布式计算，且支持通过外部表查询 Elasticsearch。Elasticsearch 擅长全文检索。两者结合提供了一个更完善的 OLAP 解决方案。

### 创建示例

#### 语法

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

#### 参数说明

| **参数**             | **是否必须** | **默认值** | **说明**                                                     |
| -------------------- | ------------ | ---------- | ------------------------------------------------------------ |
| hosts                | 是           | 无         | Elasticsearch 集群连接地址，用于获取 Elasticsearch 版本号以及索引的分片分布信息，可指定一个或多个。StarRocks 是根据 `GET /_nodes/http` API 返回的地址和 Elasticsearch 集群进行通讯，所以 `hosts` 参数值必须和 `GET /_nodes/http` 返回的地址一致，否则可能导致 BE 无法和 Elasticsearch 集群进行正常的通讯。 |
| index                | 是           | 无         | StarRocks 中的表对应的 Elasticsearch 的索引名字，可以是索引的别名。支持通配符匹配，比如设置 `index` 为 `hello*`，则 StarRocks 会匹配到所有以 `hello` 开头的索引。 |
| user                 | 否           | 空         | 开启 basic 认证的 Elasticsearch 集群的用户名，需要确保该用户有访问 /*cluster/state/* nodes/http 等路径权限和对索引的读取权限。 |
| password             | 否           | 空         | 对应用户的密码信息。                                         |
| type                 | 否           | `_doc`     | 指定索引的类型。如果您要查询的是数据是在 Elasticsearch 8 及以上版本，那么在 StarRocks 中创建外部表时就不需要配置该参数，因为 Elasticsearch 8 以及上版本已经移除了 mapping types。 |
| es.nodes.wan.only    | 否           | `false`    | 表示 StarRocks 是否仅使用 `hosts` 指定的地址，去访问 Elasticsearch 集群并获取数据。自 2.3.0 版本起，StarRocks 支持配置该参数。<ul><li>`true`：StarRocks 仅使用 `hosts` 指定的地址去访问 Elasticsearch 集群并获取数据，不会探测 Elasticsearch 集群的索引每个分片所在的数据节点地址。如果 StarRocks 无法访问 Elasticsearch 集群内部数据节点的地址，则需要配置为 `true`。</li><li>`false`：StarRocks 通过 `hosts` 中的地址，探测 Elasticsearch 集群索引各个分片所在数据节点的地址。StarRocks 经过查询规划后，相关 BE 节点会直接去请求 Elasticsearch 集群内部的数据节点，获取索引的分片数据。如果 StarRocks 可以访问 Elasticsearch 集群内部数据节点的地址，则建议保持默认值 `false`。</li></ul> |
| es.net.ssl           | 否           | `false`    | 是否允许使用 HTTPS 协议访问 Elasticsearch 集群。自 2.4 版本起，StarRocks 支持配置该参数。<ul><li>`true`：允许，HTTP 协议和 HTTPS 协议均可访问。</li><li>`false`：不允许，只能使用 HTTP 协议访问。</li></ul> |
| enable_docvalue_scan | 否           | `true`     | 是否从 Elasticsearch 列式存储获取查询字段的值。多数情况下，从列式存储中读取数据的性能要优于从行式存储中读取数据的性能。 |
| enable_keyword_sniff | 否           | `true`     | 是否对 Elasticsearch 中 TEXT 类型的字段进行探测，通过 KEYWORD 类型字段进行查询。设置为 `false` 会按照分词后的内容匹配。默认值：`true`。 |

##### 启用列式扫描优化查询速度

如果将 `enable_docvalue_scan` 设置为 `true`，StarRocks 从 Elasticsearch 中获取数据会遵循以下两条原则：

* **尽力而为**: 自动探测要读取的字段是否开启列式存储。如果要获取的字段全部有列存，StarRocks 会从列式存储中获取所有字段的值。
* **自动降级**: 如果要获取的字段中有任何一个字段没有列存，则 StarRocks 会从行存 `_source` 中解析获取所有字段的值。

> **说明**
>
> * TEXT 类型的字段在 Elasticsearch 中没有列式存储。因此，如果要获取的字段值有 TEXT 类型字段时，会自动降级为从 `_source` 中获取。
> * 在获取的字段数量过多（大于等于 25）的情况下，从 `docvalue` 中获取字段值的性能会和从 `_source` 中获取字段值基本一样。

##### 探测 KEYWORD 类型字段

如果将 `enable_keyword_sniff` 设置为 `true`，在 Elasticsearch 中可以不建立索引直接进行数据导入，因为 Elasticsearch 会在数据导入完成后自动创建一个新的索引。针对字符串类型的字段，Elasticsearch 会创建一个既有 TEXT 类型又有 KEYWORD 类型的字段，这就是 Elasticsearch 的 Multi-Field 特性，Mapping 如下：

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

对 `k4` 进行条件过滤（如 `=` 条件）时，StarRocks On Elasticsearch 会将查询转换为 Elasticsearch 的 TermQuery。

原 SQL 过滤条件如下：

~~~sql
k4 = "StarRocks On Elasticsearch"
~~~

转换成 Elasticsearch 的查询 DSL 如下：

~~~sql
"term" : {
    "k4": "StarRocks On Elasticsearch"

}
~~~

由于 `k4` 的第一字段类型为 TEXT，在数据导入时 StarRocks 会根据 `k4` 设置的分词器（如果没有设置分词器，则默认使用 `standard` 分词器）进行分词处理得到 `StarRocks`、`On`、`Elasticsearch` 三个 `term`，如下所示：

~~~sql
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

分词的结果如下：

~~~sql
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
```json
"end_offset": 11,
         "type": "<ALPHANUM>",
         "position": 2
      }
   ]
}
~~~

Assuming the following query is executed:

```sql
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
```

The `term` `StarRocks On Elasticsearch` does not match any term in the dictionary and will not return any results. By setting `enable_keyword_sniff` to `true`, StarRocks will automatically convert `k4 = "StarRocks On Elasticsearch"` to `k4.keyword = "StarRocks On Elasticsearch"` for a complete match in SQL semantics. The transformed Elasticsearch query DSL is as follows:

```sql
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
```

The type of `k4.keyword` is KEYWORD, and the data written to Elasticsearch is a complete term, so a matching result can be found in the dictionary.

#### Mapping Relationship

When creating an external table, the column type in StarRocks needs to be specified according to the field type in Elasticsearch. The specific mapping relationship is as follows:

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

> **Note**
>
> * StarRocks will use JSON-related functions to read nested fields.
> * Regarding the ARRAY type, because in Elasticsearch, multi-dimensional arrays will be automatically flattened into one-dimensional arrays, StarRocks will also perform the same behavior for conversion. **Starting from version 2.5, querying ARRAY data in Elasticsearch is supported.**

### Predicate Pushdown

StarRocks supports predicate pushdown for Elasticsearch tables, pushing the filter conditions to Elasticsearch for execution, allowing the execution to be as close to storage as possible, thus improving query performance. Currently supported operators for pushdown are as follows:

| **SQL syntax**           | **Elasticsearch syntax**   |
| ------------------------ | ---------------------------|
| `=`                      | term query                 |
| `in`                     | terms query                |
| `\>=`,  `<=`, `>`, `<`   | range                      |
| `and`                    | bool.filter                |
| `or`                     | bool.should                |
| `not`                    | bool.must_not              |
| `not in`                 | bool.must_not + terms      |
| `esquery`                | ES Query DSL               |

### Query Examples

Use the `esquery` function to push some Elasticsearch queries that cannot be expressed using SQL to Elasticsearch for filtering processing. The first column name parameter of `esquery` is used to associate the index, and the second parameter is the JSON representation of the basic Query DSL of Elasticsearch, enclosed in curly brackets (`{}`). **The root key of the JSON can only be one**, such as match, geo_shape, and bool.

* Match query

   ```sql
   select * from es_table where esquery(k4, '{
      "match": {
         "k4": "StarRocks on elasticsearch"
      }
   }');
   ```

* Geo-related query

   ```sql
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
   ```

* Bool query

   ```sql
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
   ```

### Notes

* The data scanning methods before and after Elasticsearch 5.x are different, and currently StarRocks only supports querying versions after 5.x.
* Support querying Elasticsearch clusters that use HTTP Basic authentication.
* Some queries through StarRocks will be much slower than directly querying Elasticsearch, such as count-related queries. This is because Elasticsearch internally directly reads metadata related to the number of documents that meet the conditions, without the need to perform filtering operations on real data, making the count very fast.

## (Deprecated) Hive External Table

Before using a Hive external table, ensure that JDK 1.8 is installed on the server.

### Create Hive Resource

StarRocks uses Hive resources to manage related configurations of the Hive cluster used, such as Hive Metastore address, etc., with one Hive resource corresponding to one Hive cluster. When creating a Hive external table, it is necessary to specify which Hive resource to use.

```sql
-- Create a Hive resource named hive0.
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://10.10.44.98:9083"
);

-- View the resources created in StarRocks.
SHOW RESOURCES;

-- Delete the resource named hive0.
DROP RESOURCE "hive0";
```

StarRocks 2.3 and above support modifying `hive.metastore.uris` for Hive resources. For more information, see [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md).

### Create Database

```sql
CREATE DATABASE hive_test;
USE hive_test;
```

### Create Hive External Table

```sql
-- Syntax
CREATE EXTERNAL TABLE table_name (
  col_name col_type [NULL | NOT NULL] [COMMENT "comment"]
) ENGINE=HIVE
PROPERTIES (
  "key" = "value"
);

-- Example: Create an external table for the profile_parquet_p7 table in the rawdata database in the Hive cluster corresponding to the hive0 resource.
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
```

Explanation:

* External table columns:
  * The column names need to correspond one-to-one with the Hive table.
  * The column order corresponds to the Hive table. If the storage format of the Hive table is Parquet or ORC, the column order **does not need** to be consistent with the Hive table. If the storage format of the Hive table is CSV, the column order **needs** to be consistent with the Hive table.
  * It is possible to select only **part of** the columns from the Hive table, but **partition columns** must all be included.
  * The partition columns of the external table do not need to be specified through the partition by statement, and need to be defined in the description list like regular columns. Partition information does not need to be specified, as StarRocks will automatically synchronize from Hive.
  * ENGINE is specified as HIVE.

* 属性：
  * **hive.resource**：指定使用的 Hive 资源。
  * **database**：指定 Hive 中的数据库。
  * **table**：指定 Hive 中的表，**不支持 view**。
* 创建外部表时，需根据 Hive 表列类型指定 StarRocks 中外部表列类型，具体映射关系如下：

| **Hive**      | **StarRocks**                                                |
| ------------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT        | BIGINT                                                       |
| TIMESTAMP     | DATETIME <br />注意 TIMESTAMP 转成 DATETIME 会损失精度和时区信息，并根据 sessionVariable 中的时区转成无时区 DATETIME。 |
| STRING        | VARCHAR                                                      |
| VARCHAR       | VARCHAR                                                      |
| CHAR          | CHAR                                                         |
| DOUBLE        | DOUBLE                                                       |
| FLOAT         | FLOAT                                                        |
| DECIMAL       | DECIMAL                                                      |
| ARRAY         | ARRAY                                                        |

说明：

* 支持 Hive 的存储格式为 Parquet，ORC 和 CSV 格式。如果为 CSV 格式，则暂不支持使用引号作为转义字符。
* 压缩格式支持 Snappy 和 LZ4。
* Hive 外表可查询的最大字符串长度为 1 MB。超过 1 MB 时，查询设置成 Null。

<br/>

### 查询 Hive 外表

~~~sql
-- 查询 profile_wos_p7 的总行数。
select count(*) from profile_wos_p7;
~~~

<br/>

### 更新缓存的 Hive 表元数据

Hive 表 (Hive Table) 的 Partition 统计信息以及 Partition 下面的文件信息可以缓存到 StarRocks FE 中，缓存的内存结构为 Guava LoadingCache。您可以在 `fe.conf` 文件中通过设置 `hive_meta_cache_refresh_interval_s` 参数修改缓存自动刷新的间隔时间（默认值为 `7200`，单位：秒），也可以通过设置 `hive_meta_cache_ttl_s` 参数修改缓存的失效时间（默认值为 `86400`，单位：秒）。修改后需重启 FE 生效。

#### 手动更新元数据缓存

* 手动刷新元数据信息：
  1. Hive 中新增或者删除分区时，需要刷新 **表** 的元数据信息：`REFRESH EXTERNAL TABLE hive_t`，其中 `hive_t` 是 StarRocks 中的外表名称。
  2. Hive 中向某些 partition 新增数据时，需要 **指定 partition** 进行刷新：`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')`，其中 `hive_t` 是 StarRocks 中的外表名称，'k1 = 01/k2 = 02'、 'k1 = 03/k2 = 04'是 hive 中的 partition 名称。
  3. 在执行 `REFRESH EXTERNAL TABLE hive_t` 命令时，StarRocks 会先检查 Hive 外部表中的列信息和 Hive Metastore 返回的 Hive 表中的列信息是否一致。若发现 Hive 表的 schema 有修改，如增加列或减少列，那么 StarRocks 会将修改的信息同步到 Hive 外部表。同步后，Hive 外部表的列顺序和 Hive 表的列顺序保持一致，且分区列为最后一列。

#### 自动增量更新元数据缓存

自动增量更新元数据缓存主要是通过定期消费 Hive Metastore 的 event 来实现，新增分区以及分区新增数据无需通过手动执行 refresh 来更新。用户需要在 Hive Metastore 端开启元数据 Event 机制。相比 Loading Cache 的自动刷新机制，自动增量更新性能更好，建议用户开启该功能。开启该功能后，Loading Cache 的自动刷新机制将不再生效。

* Hive Metastore 开启 event 机制

   用户需要在 $HiveMetastore/conf/hive-site.xml 中添加如下配置，并重启 Hive Metastore. 以下配置为 Hive Metastore 3.1.2 版本的配置，用户可以将以下配置先拷贝到 hive-site.xml 中进行验证，因为在 Hive Metastore 中配置不存在的参数只会提示 WARN 信息，不会抛出任何异常。

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

    用户需要在 $FE_HOME/conf/fe.conf 中添加如下配置并重启 FE。
     `enable_hms_events_incremental_sync=true`
    自动增量元数据同步相关配置如下，如无特殊需求，无需修改。

   | 参数值                             | 说明                                      | 默认值 |
   | --- | --- | ---|
   | enable_hms_events_incremental_sync | 是否开启元数据自动增量同步功能            | false |
   | hms_events_polling_interval_ms     | StarRocks 拉取 Hive Metastore Event 事件间隔 | 5000 毫秒 |
   | hms_events_batch_size_per_rpc      | StarRocks 每次拉取 Event 事件的最大数量      | 500 |
   | enable_hms_parallel_process_evens  | 对接收的 Events 是否并行处理                | true |
   | hms_process_events_parallel_num    | 处理 Events 事件的并发数                    | 4 |

* 注意事项
  * 不同版本 Hive Metastore 的 Events 事件可能不同，且上述开启 HiveMetastore Event 机制的配置在不同版本也存在不同。使用时相关配置可根据实际版进行适当调整。当前已经验证可以开启 Hive Metastore Event 机制的版本有 2.X 和 3.X。用户可以在 FE 日志中搜索 "event id" 来验证 event 是否开启成功，如果没有开启成功，event id 始终保持为 0。如果无法判断是否成功开启 Event 机制，请在 StarRocks 用户交流群中联系值班同学进行排查。
  * 当前 Hive 元数据缓存模式为懒加载，即：如果 Hive 新增了分区，StarRocks 只会将新增分区的 partition key 进行缓存，不会立即缓存该分区的文件信息。只有当查询该分区时或者用户手动执行 refresh 分区操作时，该分区的文件信息才会被加载。StarRocks 首次缓存该分区统计信息后，该分区后续的元数据变更就会自动同步到 StarRocks 中。
  * 手动执行缓存方式执行效率较低，相比之下自动增量更新性能开销较小，建议用户开启该功能进行更新缓存。
  * 当 Hive 数据存储为 Parquet、ORC、CSV 格式时，StarRocks 2.3及以上版本支持 Hive 外部表同步 ADD COLUMN、REPLACE COLUMN 等表结构变更（Schema Change）。

### 访问对象存储

* FE 配置文件路径为 $FE_HOME/conf。如果需要自定义 Hadoop 集群的配置，可以在该目录下添加配置文件，例如：如果 HDFS 集群采用了高可用的 Nameservice，需要将 Hadoop 集群中的 hdfs-site.xml 放到该目录下；如果 HDFS 配置了 ViewFs，需要将 core-site.xml 放到该目录下。
* BE 配置文件路径为 $BE_HOME/conf。如果需要自定义 Hadoop 集群的配置，可以在该目录下添加配置文件，例如：如果 HDFS 集群采用了高可用的 Nameservice，需要将 Hadoop 集群中的 hdfs-site.xml 放到该目录下；如果 HDFS 配置了 ViewFs，需要将 core-site.xml 放到该目录下。
* BE 所在机器的**启动脚本** $BE_HOME/bin/start_be.sh 中需要配置 JAVA_HOME，要配置成 JDK 环境，不能配置成 JRE 环境，比如 `export JAVA_HOME = <JDK 的绝对路径>`。注意需要将该配置添加在 BE 启动脚本最开头，添加完成后需重启 BE。
* Kerberos 支持
  1. 在所有的 FE/BE 机器上用 `kinit -kt keytab_path principal` 登录，该用户需要有访问 Hive 和 HDFS 的权限。kinit 命令登录是有实效性的，需要将其放入 crontab 中定期执行。
  2. 把 Hadoop 集群中的 hive-site.xml/core-site.xml/hdfs-site.xml 放到 $FE_HOME/conf 下，把 core-site.xml/hdfs-site.xml 放到 $BE_HOME/conf 下。
  3. 在 $FE_HOME/conf/fe.conf 文件中的 JAVA_OPTS 选项取值里添加 -Djava.security.krb5.conf=/etc/krb5.conf，其中 /etc/krb5.conf 是 krb5.conf 文件的路径，可以根据自己的系统调整。
  4. 在 $BE_HOME/conf/be.conf 文件增加选项 JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"，其中 /etc/krb5.conf 是 krb5.conf 文件的路径，可以根据自己的系统调整。
  5. resource 中的 uri 地址一定要使用域名，并且相应的 Hive 和 HDFS 的域名与 IP 的映射都需要配置到 /etc/hosts 中。

#### AWS S3/Tencent Cloud COS 支持

1. 在 $FE_HOME/conf/core-site.xml 中加入如下配置：

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

   * `fs.s3a.access.key` 指定 aws 的 access key id
   * `fs.s3a.secret.key` 指定 aws 的 secret access key
   * `fs.s3a.endpoint` 指定 aws 的区域
   * `fs.s3a.connection.maximum` 配置最大链接数，如果查询过程中有报错 `Timeout waiting for connection from poll`，可以适当调高该参数

2. 在 $BE_HOME/conf/be.conf 中加入如下配置。

   * `object_storage_access_key_id` 与 FE 端 core-site.xml 配置 `fs.s3a.access.key` 相同
   * `object_storage_secret_access_key` 与 FE 端 core-site.xml 配置 `fs.s3a.secret.key` 相同
   * `object_storage_endpoint` 与 FE 端 core-site.xml 配置 `fs.s3a.endpoint` 相同
   * `object_storage_region` 只有腾讯 COS 需要额外添加该配置项。如：ap-beijing****

3. 重启 FE 和 BE。

#### Aliyun OSS 支持

1. 在 $FE_HOME/conf/core-site.xml 中加入如下配置。

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
         <!-- 以下以北京地域为例，其他地域请根据实际情况替换。 -->
         <value>oss-cn-beijing.aliyuncs.com</value>
      </property>
   </configuration>
   ~~~

   * `fs.oss.accessKeyId` 指定阿里云账号或 RAM 用户的 AccessKey ID，获取方式，请参见 [获取 AccessKey](https://help.aliyun.com/document_detail/53045.htm?spm=a2c4g.11186623.0.0.128b4b7896DD4W#task968)。
   * `fs.oss.accessKeySecret` 指定阿里云账号或 RAM 用户的 AccessKey Secret，获取方式，请参见 [获取 AccessKey](https://help.aliyun.com/document_detail/53045.htm?spm=a2c4g.11186623.0.0.128b4b7896DD4W#task968)。
   * `fs.oss.endpoint` 指定相关 OSS Bucket 所在地域对应的 Endpoint。
    您可以通过以下方式查询 Endpoint：

     * 根据 Endpoint 与地域的对应关系进行查找，请参见 [访问域名和数据中心](https://help.aliyun.com/document_detail/31837.htm#concept-zt4-cvy-5db)。
     * 您可以登录 [阿里云 OSS 管理控制台](https://oss.console.aliyun.com/index?spm=a2c4g.11186623.0.0.11d24772leoEEg#/)，进入 Bucket 概览页，Bucket 域名 examplebucket.oss-cn-hangzhou.aliyuncs.com 的后缀部分 oss-cn-hangzhou.aliyuncs.com，即为该 Bucket 的外网 Endpoint。

2. 在 $BE_HOME/conf/be.conf 中加入如下配置。

   * `object_storage_access_key_id` 与 FE 端 core-site.xml 配置 `fs.oss.accessKeyId` 相同
   * `object_storage_secret_access_key` 与 FE 端 core-site.xml 配置 `fs.oss.accessKeySecret` 相同
   * `object_storage_endpoint` 与 FE 端 core-site.xml 配置 `fs.oss.endpoint` 相同

3. 重启 FE，BE。

## (Deprecated) Iceberg 外部表

如果要查询 Iceberg 数据，需要在 StarRocks 中创建 Iceberg 外部表，并将外部表与需要查询的 Iceberg 表建立映射。

自 2.1.0 版本起，StarRocks 支持通过外部表的方式查询 Iceberg 数据。

### 前提条件

确保 StarRocks 有权限访问 Iceberg 依赖的元数据服务（如 Hive metastore）、文件系统（如 HDFS ）和对象存储系统（如 Amazon S3 和阿里云对象存储 OSS）。

### 注意事项

* Iceberg 外部表仅支持查询以下格式的数据：
* Iceberg v1 表（分析数据表）。从3.0版本开始，支持查询 ORC 格式的 Iceberg v2 表（逐行删除）。从3.1版本开始，支持查询 Parquet 格式的 v2 表。更多有关 Iceberg v1 表和 Iceberg v2 表的信息，请参见 [Iceberg Table Spec](https://iceberg.apache.org/spec/)。
* 支持的压缩格式包括 gzip（默认压缩格式）、Zstd、LZ4 和 Snappy。
* 支持的文件格式包括 Parquet 和 ORC。

* StarRocks 2.3 及以上版本支持同步 Iceberg 表结构，但StarRocks 2.3 以下版本不支持。如果 Iceberg 表结构发生变化，您需要在 StarRocks 中删除相应的外部表并重新创建。

### 操作步骤

#### 步骤一：创建 Iceberg 资源

在创建外部表之前，需先创建 Iceberg 资源，以用来管理 Iceberg 的访问信息。此外，在创建 Iceberg 外部表时也需要指定引用的 Iceberg 资源。您可以根据业务需求创建不同 catalog 类型的资源：

* 如果使用 Hive metastore 作为 Iceberg 的元数据服务，则可以创建 catalog 类型为 `HIVE` 的资源。
* 如果想要自定义 Iceberg 的元数据服务，则可以开发一个 custom catalog （即自定义 catalog），然后创建 catalog 类型为 `CUSTOM` 的资源。

> **说明**
>
> 仅 StarRocks 2.3 及以上版本支持创建 catalog 类型为 `CUSTOM` 的资源。

**创建 catalog 类型为 `HIVE` 的资源**

例如，创建一个名为 `iceberg0` 的资源，并指定该资源的 catalog 类型为 `HIVE`。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0"
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://192.168.0.81:9083"
);
~~~

参数说明：

| **参数**                            | **说明**                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| type                                | 资源类型，取值为 `iceberg`。                                 |
| iceberg.catalog.type              | Iceberg 的 catalog 类型。目前支持 Hive catalog 和 custom catalog。如要使用 Hive catalog，设置该参数为 `HIVE`。如要使用 custom catalog，设置该参数为 `CUSTOM`。 |
| iceberg.catalog.hive.metastore.uris | Hive Metastore 的 URI。格式为 `thrift://<Iceberg 元数据的IP地址>:<端口号>`，端口号默认为 9083。Apache Iceberg 通过 Hive catalog 连接 Hive metastore，以查询 Iceberg 表的元数据。 |

**创建 catalog 类型为 `CUSTOM` 的资源**

Custom catalog 需要继承抽象类 BaseMetastoreCatalog，并实现 IcebergCatalog 接口。此外，custom catalog 类名不能与 StarRocks 中已存在的类名重复。开发完成后，您需要将 custom catalog 及其相关文件打包并放到所有 FE 节点的 **fe/lib** 路径下，然后重启所有 FE 节点，以便 FE 识别这个类。以上操作完成后即可创建资源。

例如，创建一个名为 `iceberg1` 的资源，并指定该资源的 catalog 类型为 `CUSTOM`。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1"
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog"
);
~~~

参数说明：

| **参数**               | **说明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| type                   | 资源类型，取值为 `iceberg`。                                 |
| iceberg.catalog.type | Iceberg 的 catalog 类型。目前支持 Hive catalog 和 custom catalog。如要使用 Hive catalog，需指定该参数值为 `HIVE`。如要使用 custom catalog，需指定该参数值为 `CUSTOM`。 |
| iceberg.catalog-impl   | 开发的 custom catalog 的全限定类名。FE 会根据该类名查找开发的 custom catalog。如果 custom catalog 中包含自定义的配置项，需要在创建 Iceberg 外部表时将其以键值对的形式添加到 SQL 语句的 `PROPERTIES` 中。 |

StarRocks 2.3 及以上版本支持修改 Iceberg 资源的 `hive.metastore.uris` 和 `iceberg.catalog-impl`。更多信息，请参见 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

**查看 Iceberg 资源**

~~~SQL
SHOW RESOURCES;
~~~

**删除 Iceberg 资源**
例如，删除一个名为 `iceberg0` 的资源。

~~~SQL
DROP RESOURCE  "iceberg0" ;
~~~

删除一个资源会导致引用该资源的所有外部表不可用，但对应的 Iceberg 表中的数据不会删除。如果删除后仍想通过 StarRocks 查询 Iceberg 数据，需要重新创建 Iceberg 资源和 Iceberg 外部表。

#### （可选）步骤二：创建数据库

您可以创建一个新的数据库用来存放外部表，也可以在已有的数据库中创建外部表。

例如，在 StarRocks 中创建名为 `iceberg_test` 的数据库。语法如下：

~~~SQL
CREATE DATABASE iceberg_test;
~~~

> **说明**
>
> 该数据库名称不需要和待查询的 Iceberg 数据库名称保持一致。

#### 步骤三：创建 Iceberg 外部表

例如，在数据库 `iceberg_test` 中创建名为 `iceberg_tbl` 的 Iceberg 外部表。语法如下：

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

参数说明：

| **参数** | **说明**                          |
| -------- | --------------------------------- |
| ENGINE   | 取值为 `ICEBERG`。                |
| resource | 外部表引用的 Iceberg 资源的名称。 |
| database | Iceberg 表所属的数据库的名称。    |
| table    | Iceberg 表名称。                  |

> **说明**
>
> * 表名无需与 Iceberg 的实际表名保持一致。
> * 列名必须与 Iceberg 的实际列名保持一致，列的顺序无需保持一致。

如果您在 custom catalog 中自定义了配置项，且希望在查询外部表时这些配置项能生效，您可以将这些配置项以键值对的形式添加到建表语句的 `PROPERTIES` 中。例如，在 custom catalog 中定义了一个配置项 `custom-catalog.properties`，那么创建 Iceberg 外部表的语法如下：

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

创建外部表时，需根据 Iceberg 表的列类型指定 StarRocks 中外部表的列类型，具体映射关系如下：

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

StarRocks does not support querying the following data types: TIMESTAMPTZ, STRUCT, and MAP.

#### Step 4: Query Iceberg Data

After creating an Iceberg external table, you can query the data in the Iceberg table through the external table. For example:

~~~SQL
select count(*) from iceberg_tbl;
~~~

## (Deprecated) Hudi External Table

Starting from version 2.2.0, StarRocks supports querying data in the Hudi data lake through external tables, helping you achieve high-speed analysis of the data lake. This article describes how to create an external table in StarRocks and query data in Hudi.

### Prerequisites

Please ensure that StarRocks has permission to access the corresponding Hive Metastore, HDFS cluster, or object storage Bucket for Hudi.

### Notes

* Hudi external tables can only be used for query operations and do not support writing.
* The currently supported table types for Hudi are Copy on Write (referred to as COW below) and Merge on read (referred to as MOR below, supported from version 2.5). For more differences between COW and MOR, please refer to the [Apache Hudi official website](https://hudi.apache.org/docs/table_types).
* Currently supported Hudi query types include Snapshot Queries and Read Optimized Queries (only for MOR tables), Incremental Queries are not supported for the time being. For an explanation of Hudi query types, please refer to [Table & Query Types](https://hudi.apache.org/docs/next/table_types/#query-types).
* The supported compression formats for Hudi files are GZIP (default), ZSTD, LZ4, and SNAPPY.
* StarRocks does not currently support synchronizing the structure of Hudi tables. If the structure of the Hudi table changes, you need to delete the corresponding external table in StarRocks and recreate it.

### Operation Steps

#### Step 1: Create and Manage Hudi Resources

You need to create Hudi resources in StarRocks in advance to manage the Hudi databases and external tables created in StarRocks.

Execute the following command to create a Hudi resource named `hudi0`.

~~~sql
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://192.168.7.251:9083"
);
~~~

|  Parameter   | Description  |
|  ----  | ----  |
| type  | Resource type, fixed value is **hudi**. |
| hive.metastore.uris | Thrift URI of the Hive Metastore.<br /> Hudi connects to the Hive Metastore to create and manage tables. You need to pass in the thrift URI of the Hive Metastore. The format is  **`thrift://<IP address of Hudi metadata>:<port>`**, the default port number is 9083. |

StarRocks 2.3 and above support modifying the `hive.metastore.uris` of Hudi resources. For more information, see [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md).

Execute the following command to view all Hudi resources in StarRocks.

~~~sql
SHOW RESOURCES;
~~~~

Execute the following command to delete the Hudi resource named `hudi0`.

~~~sql
DROP RESOURCE "hudi0";
~~~~

> Deleting Hudi resources will make all the Hudi external tables contained in it unavailable, but the data in Hudi will not be lost. If you still need to query the data in Hudi through StarRocks, please recreate the Hudi resources, Hudi databases, and external tables.

#### Step 2: Create Hudi Database

Execute the following command to create and enter a Hudi database named `hudi_test` in StarRocks.

~~~sql
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> The database name does not need to be consistent with the actual database name of Hudi.

#### Step 3: Create Hudi External Table

Execute the following command to create an external table named `hudi_tbl` in the Hudi database `hudi_test`.

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

* For related parameter explanations, please refer to the table below:

| **Parameter**     | **Description**                       |
| ------------ | ------------------------------ |
| **ENGINE**   | Fixed as **HUDI**, no need to change.  |
| **resource** | Name of the Hudi resource in StarRocks. |
| **database** | Name of the database where the Hudi table is located.        |
| **table**    | Name of the data table where the Hudi table is located.        |

* The table name does not need to be consistent with the actual table name of Hudi.
* The column names need to be consistent with the actual column names of Hudi, the order of the columns does not need to be consistent.
* You can select all or part of the columns in the Hudi table according to business requirements.
* When creating an external table, the column type of the external table in StarRocks needs to be specified according to the column type of the Hudi table, and the specific mapping relationship is as follows:

| **Hudi type**                    | **StarRocks type**     |
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

> StarRocks does not currently support querying Struct and Map data types, and for MOR tables, Array data types are not currently supported.

#### Step 4: Query Hudi External Table

After creating the Hudi external table, without importing data, you can query the Hudi data by executing the following command.

~~~sql
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (Deprecated) MySQL External Table

In a star schema, data is generally divided into dimension tables and fact tables. Dimension tables have a small amount of data, but may involve UPDATE operations. Currently, StarRocks does not directly support UPDATE operations (they can be achieved through a Unique/Primary data model). In some scenarios, dimension tables can be stored in MySQL, and the dimension tables can be read directly when queried.

Before using data in MySQL, an external table (CREATE EXTERNAL TABLE) needs to be created in StarRocks, which maps to it. When creating a MySQL external table in StarRocks, you need to specify the relevant connection information for MySQL, as shown below.

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

Parameter explanation:

* **host**: MySQL connection address
* **port**: MySQL connection port number
* **user**: MySQL login username
* **password**: MySQL login password
* **database**: MySQL database name
* **table**: MySQL database table name

## Frequently Asked Questions

### How to resolve an error when synchronizing StarRocks external tables?

**Problem prompt**:

SQL error [1064] [42000]: data cannot be inserted into table with empty partition.Use `SHOW PARTITIONS FROM external_t;` to see the currently partitions of this table.

查看 Partitions 时提示另一错误：SHOW PARTITIONS FROM external_t
SQL 错误 [1064] [42000]: Table[external_t] is not a OLAP/ELASTICSEARCH/HIVE table

**解决方法**：

建外部表时端口不对，正确的端口是 "port"="9020" 。

**提示问题**：

查询报错: Memory of query_pool exceed limit. read and decompress page Used: 49113428144, Limit: 49111753861. Mem usage has exceed the limit of query pool

**解决方法**：

查询的外表列比较多时可能触发该问题，可通过在 be.conf 中添加参数 `buffer_stream_reserve_size=8192` 后重启 BE 解决该问题。