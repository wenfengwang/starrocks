---
displayed_sidebar: English
---

# 外部表

:::注意
从 v3.0 开始，我们建议您使用 Catalog 来查询 Hive、Iceberg 和 Hudi 中的数据。请参阅 [Hive 目录](../data_source/catalog/hive_catalog.md)、[Iceberg 目录](../data_source/catalog/iceberg_catalog.md) 和 [Hudi 目录](../data_source/catalog/hudi_catalog.md)。

从 v3.1 开始，我们建议您使用 [JDBC catalog](../data_source/catalog/jdbc_catalog.md) 来查询 MySQL 和 PostgreSQL 的数据，并使用 [Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md) 来查询 Elasticsearch 的数据。
:::

StarRocks 支持通过外部表访问其他数据源。外部表是基于存储在其他数据源中的数据表创建的。StarRocks 仅存储数据表的元数据。您可以使用外部表直接查询其他数据源中的数据。StarRocks 支持以下数据源：MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg 和 Apache Hudi。**目前，您只能将另一个 StarRocks 集群中的数据写入当前 StarRocks 集群。无法从当前 StarRocks 集群读取数据。对于 StarRocks 以外的数据源，只能从这些数据源读取数据。**

从 2.5 版本开始，StarRocks 提供了 Data Cache 功能，可以加速外部数据源上的热数据查询。有关详细信息，请参阅 [数据缓存](data_cache.md)。

## StarRocks 外部表

从 StarRocks 1.19 版本开始，StarRocks 允许您使用 StarRocks 外部表将数据从一个 StarRocks 集群写入另一个集群。这样可以实现读写分离，并提供更好的资源隔离。您可以先在目标 StarRocks 集群中创建目标表。然后，在源 StarRocks 集群中，您可以创建一个与目标表具有相同 Schema 的 StarRocks 外部表，并在 `PROPERTIES` 字段中指定目标集群和表的信息。

通过使用 INSERT INTO 语句将数据从源集群写入目标集群的 StarRocks 外部表。它可以帮助实现以下目标：

* StarRocks 集群之间的数据同步。
* 读写分离。数据写入源集群，源集群的数据变化同步到目标集群，目标集群提供查询服务。

以下代码显示了如何创建目标表和外部表。

~~~SQL
# 在目标 StarRocks 集群中创建目标表。
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

# 在源 StarRocks 集群中创建外部表。
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

# 通过将数据写入 StarRocks 外部表，将数据从源集群写入目标集群。建议在生产环境中使用第二条语句。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

参数：

* **EXTERNAL：** 该关键字表示要创建的表是外部表。
* **host：** 指定目标 StarRocks 集群的 leader FE 节点的 IP 地址。
* **port：** 指定目标 StarRocks 集群的 FE 节点的 RPC 端口。

  :::注意

  为确保 StarRocks 外部表所属的源集群能够访问目标 StarRocks 集群，您需要配置网络和防火墙，允许访问以下端口：

  * FE 节点的 RPC 端口。请参阅 FE 配置文件 **fe/fe.conf** 中的 `rpc_port`。默认的 RPC 端口是 `9020`。
  * BE 节点的 bRPC 端口。请参阅 BE 配置文件 **be/be.conf** 中的 `brpc_port`。默认的 bRPC 端口是 `8060`。

  :::

* **user：** 用于访问目标 StarRocks 集群的用户名。
* **password：** 指定访问目标 StarRocks 集群的密码。
* **database：** 指定目标表所属的数据库。
* **table：** 指定目标表的名称。

使用 StarRocks 外部表时，存在以下限制：

* 您只能对 StarRocks 外部表执行 INSERT INTO 和 SHOW CREATE TABLE 命令。不支持其他数据写入方法。此外，您不支持查询 StarRocks 外部表的数据，也无法对外部表进行 DDL 操作。
* 创建外部表的语法与创建普通表的语法相同，但外部表中的列名和其他信息必须与目标表相同。
* 外部表每 10 秒同步一次目标表中的表元数据。如果目标表执行 DDL 操作，则两表之间的数据同步可能会有延迟。

## JDBC 兼容数据库的外部表

从 v2.3.0 版本开始，StarRocks 提供了外部表来查询兼容 JDBC 的数据库。这样一来，您就可以快速分析此类数据库的数据，而无需将数据导入 StarRocks。本节介绍如何在 StarRocks 中创建外部表，并在兼容 JDBC 的数据库中查询数据。

### 先决条件

使用 JDBC 外部表查询数据前，请确保 FE 和 BE 能够访问 JDBC 驱动下载地址。下载 URL 由 `driver_url` 参数在创建 JDBC 资源的语句中指定。

### 创建和管理 JDBC 资源

#### 创建 JDBC 资源

在创建外部表查询数据库数据之前，您需要在 StarRocks 中创建 JDBC 资源来管理数据库的连接信息。数据库必须支持 JDBC 驱动程序，称为“目标数据库”。创建资源后，您可以使用它来创建外部表。

执行以下语句，创建一个名为 `jdbc0` 的 JDBC 资源：

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

`PROPERTIES` 中所需的参数如下：

* `type`：资源的类型。将值设置为 `jdbc`。

* `user`：用于连接目标数据库的用户名。

* `password`：用于连接到目标数据库的密码。

* `jdbc_uri`：JDBC 驱动程序用于连接到目标数据库的 URI。URI 格式必须满足数据库 URI 语法。关于一些常用数据库的 URI 语法，请访问 [Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server 的官网](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)。

> 注意：URI 必须包含目标数据库的名称。例如，在前面的代码示例中，`jdbc_test` 是要连接的目标数据库的名称。

* `driver_url`：JDBC 驱动 JAR 包的下载 URL。例如，支持 HTTP URL 或文件 URL，`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar` 或者 `file:///home/disk1/postgresql-42.3.3.jar`。

* `driver_class`：JDBC 驱动程序的类名。常用数据库的 JDBC 驱动类名如下：
  * MySQL：com.mysql.jdbc.Driver（MySQL 5.x 及更早版本）、com.mysql.cj.jdbc.Driver（MySQL 6.x 及以上版本）
  * SQL Server：com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle：oracle.jdbc.driver.OracleDriver
  * PostgreSQL：org.postgresql.Driver

创建资源时，FE 会使用参数中指定的 URL 下载 JDBC 驱动 JAR 包，`driver_url` 生成校验和，并使用校验和验证 BE 下载的 JDBC 驱动。

> 注意：如果 JDBC 驱动 JAR 包下载失败，那么资源的创建也会失败。

当 BE 首次查询 JDBC 外部表时，发现机器上不存在对应的 JDBC 驱动 JAR 包时，BE 会使用参数中指定的 URL 下载 JDBC 驱动 JAR 包，并将 JDBC 驱动 JAR 包 `driver_url` 保存在 `${STARROCKS_HOME}/lib/jdbc_drivers` 该目录下。

#### 查看 JDBC 资源

执行以下语句，查看 StarRocks 中的所有 JDBC 资源：

~~~SQL
SHOW RESOURCES;
~~~

> 注意：`ResourceType` 列为 `jdbc`。

#### 删除 JDBC 资源

执行以下语句删除名为 `jdbc0` 的 JDBC 资源：

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> 注意：删除 JDBC 资源后，使用该 JDBC 资源创建的所有 JDBC 外部表都将不可用。但是，目标数据库中的数据不会丢失。如果您仍然需要使用 StarRocks 查询目标数据库中的数据，可以重新创建 JDBC 资源和 JDBC 外部表。

### 创建数据库

执行以下语句，创建并访问名为 `jdbc_test` 的数据库：

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> 注意：在上述语句中指定的数据库名称不需要与目标数据库的名称相同。

### 创建 JDBC 外部表

执行以下语句，创建名为 `jdbc_tbl` 的 JDBC 外部表在数据库 `jdbc_test` 中：

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

`properties` 中所需的参数如下：

* `resource`：用于创建外部表的 JDBC 资源的名称。

* `table`：数据库中的目标表名。

要查看 StarRocks 和目标数据库之间支持的数据类型以及数据类型映射，请参见 [数据类型映射](External_table.md#数据类型映射)。

> 注意：
>
> * 不支持索引。
> * 无法使用 PARTITION BY 或 DISTRIBUTED BY 指定数据分发规则。

### 查询 JDBC 外部表

在查询 JDBC 外部表之前，您必须执行以下语句以启用 Pipeline 引擎：

~~~SQL
set enable_pipeline_engine=true;
~~~

> 注意：如果 Pipeline 引擎已经启用，您可以跳过此步骤。

执行以下语句以通过 JDBC 外部表查询目标数据库中的数据。

~~~SQL
select * from JDBC_tbl;
~~~

StarRocks 支持通过将筛选条件下推到目标表来支持谓词下推。在尽可能靠近数据源的位置执行筛选条件可以提高查询性能。目前，StarRocks 可以下推运算符，包括二元比较运算符（`>`, `>=`, `=`, `<`, 和 `<=`）、`IN`、`IS NULL` 和 `BETWEEN ... AND ...`。但是，StarRocks 无法下推函数。

### 数据类型映射

目前，StarRocks 只能查询目标数据库中的基本数据类型，例如 NUMBER、STRING、TIME 和 DATE。如果目标数据库中的数据值范围不受 StarRocks 支持，则查询将报错。

目标数据库和 StarRocks 之间的映射因目标数据库类型而异。

#### **MySQL 和 StarRocks**

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

#### **Oracle 和 StarRocks**

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

#### **PostgreSQL 和 StarRocks**

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

#### **SQL Server 和 StarRocks**

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

### 限制

* 创建 JDBC 外部表时，无法在表上创建索引，也无法使用 PARTITION BY 和 DISTRIBUTED BY 指定表的数据分布规则。

* 查询 JDBC 外部表时，StarRocks 无法将函数下推到表中。

## （已弃用）Elasticsearch 外部表

StarRocks 和 Elasticsearch 是两个流行的分析系统。StarRocks 在大规模分布式计算方面表现出色。Elasticsearch 是全文搜索的理想选择。StarRocks 结合 Elasticsearch，可以提供更完整的 OLAP 解决方案。

### 创建 Elasticsearch 外部表示例

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
PROPERTIES (
    "hosts" = "http://192.168.0.1:9200,http://192.168.0.2:9200",
    "user" = "root",
    "password" = "root",
    "index" = "tindex",
    "type" = "_doc",
    "es.net.ssl" = "true"
);
~~~

下表描述了参数。

| **参数**        | **必填** | **默认值** | **描述**                                              |
| -------------------- | ------------ | ----------------- | ------------------------------------------------------------ |
| hosts                | 是          | 无              | Elasticsearch 集群的连接地址。您可以指定一个或多个地址。StarRocks 可以解析 Elasticsearch 版本和索引分片分配。StarRocks 根据 `GET /_nodes/http` API 操作返回的地址与您的 Elasticsearch 集群进行通信。因此，`host` 参数的值必须与 `GET /_nodes/http` API 操作返回的地址相同。否则，BE 可能无法与您的 Elasticsearch 集群通信。 |
| index                | 是          | 无              | 在 StarRocks 上创建的 Elasticsearch 索引的名称。名称可以是别名。此参数支持通配符 (\*)。例如，如果将 `index` 设置为 <code class="language-text">hello*</code>，StarRocks 将检索所有名称以 `hello` 开头的索引。 |
| user                 | 否           | 空             | 用于启用基本认证登录到 Elasticsearch 集群的用户名。确保您有权访问 `/*cluster/state/*nodes/http` 和索引。 |
| password             | 否           | 空             | 用于登录 Elasticsearch 集群的密码。 |
| type                 | 否           | `_doc`            | 索引的类型。默认值： `_doc`。如果您需要查询 Elasticsearch 8 及更高版本的数据，则无需配置此参数，因为 Elasticsearch 8 及更高版本已删除了映射类型。 |
| es.nodes.wan.only    | 否           | `false`           | 指定 StarRocks 是否仅使用 `hosts` 指定的地址访问 Elasticsearch 集群并获取数据。<ul><li>`true`：StarRocks 仅使用 `hosts` 指定的地址访问 Elasticsearch 集群并获取数据，并且不会嗅探 Elasticsearch 索引分片所在的数据节点。如果 StarRocks 无法访问 Elasticsearch 集群内数据节点的地址，需要将此参数设置为 `true`。</li><li>`false`：StarRocks 使用 `host` 指定的地址嗅探 Elasticsearch 集群索引的分片所在的数据节点。StarRocks 生成查询执行计划后，相关 BE 直接访问 Elasticsearch 集群内的数据节点，从索引分片中获取数据。如果 StarRocks 可以访问 Elasticsearch 集群内数据节点的地址，建议保留默认值 `false`。</li></ul> |
| es.net.ssl           | 否           | `false`           | 指定是否可以使用 HTTPS 协议访问您的 Elasticsearch 集群。仅 StarRocks 2.4 及更高版本支持配置此参数。<ul><li>`true`：可以使用 HTTPS 和 HTTP 协议访问您的 Elasticsearch 集群。</li><li>`false`：只能使用 HTTP 协议访问您的 Elasticsearch 集群。</li></ul> |
| enable_docvalue_scan | 否           | `true`            | 指定是否从 Elasticsearch 列式存储中获取目标字段的值。在大多数情况下，从列式存储读取数据优于从行存储读取数据。 |
| enable_keyword_sniff | 否           | `true`            | 指定是否根据 KEYWORD 类型字段在 Elasticsearch 中进行 TEXT 类型字段的嗅探。如果将此参数设置为 `false`，StarRocks 将在标记化后进行匹配。 |

##### 列式扫描以加快查询速度

如果将 `enable_docvalue_scan` 设置为 `true`，StarRocks 在从 Elasticsearch 获取数据时遵循以下规则：

* **尝试并查看**：StarRocks 会自动检查目标字段是否启用了列式存储。如果是，则 StarRocks 会从列式存储中获取目标字段中的所有值。
* **自动降级**：如果列式存储中任何一个目标字段不可用，StarRocks 会解析并从行存储（_source）中获取目标字段的所有值。

> **注意**
>
> * 列式存储不适用于 Elasticsearch 中的 TEXT 类型字段。因此，如果查询包含 TEXT 类型值的字段，StarRocks 会从 _source 中获取这些字段的值。
> * 如果查询大量（大于或等于 25）个字段，则与从 _source 中读取字段值相比，从 docvalue 中读取字段值不会显示出明显的好处。

##### 嗅探 KEYWORD 类型字段

如果将 `enable_keyword_sniff` 设置为 `true`，Elasticsearch 允许在没有索引的情况下直接进行数据摄取，因为它会在摄取后自动创建索引。对于 STRING 类型的字段，Elasticsearch 将创建一个同时具有 TEXT 和 KEYWORD 类型的字段。这就是 Elasticsearch 的多字段功能的工作原理。映射关系如下：

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

例如，在 `k4` 上进行 "=" 过滤，StarRocks 会将过滤操作转换为 Elasticsearch TermQuery。

原始 SQL 过滤器如下：

~~~SQL

k4 = "StarRocks On Elasticsearch"
~~~

转换后的 Elasticsearch 查询 DSL 如下：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"

}
~~~

`k4`的第一个字段是TEXT，在数据摄入后，它将由为`k4`配置的分析器（如果未为`k4`配置分析器，则由标准分析器）进行标记。因此，第一个字段将被标记为三个项：`StarRocks`、`On`和`Elasticsearch`。具体如下：

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

代币化结果如下：

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

假设您进行以下查询：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

字典中没有与术语`StarRocks On Elasticsearch`匹配的术语，因此不会返回任何结果。

但是，如果您已将`enable_keyword_sniff`设置为`true`，StarRocks将转换`k4 = "StarRocks On Elasticsearch"`为`k4.keyword = "StarRocks On Elasticsearch"`以匹配SQL语义。转换后的`StarRocks On Elasticsearch`查询 DSL 如下所示：

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword`属于KEYWORD类型。因此，数据将作为完整术语写入Elasticsearch，从而实现成功匹配。

#### 列数据类型映射

创建外部表时，需要根据Elasticsearch表中列的数据类型，指定外部表中列的数据类型。下表显示了列数据类型的映射。

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
> * StarRocks通过与JSON相关的函数读取NESTED类型的数据。
> * Elasticsearch会自动将多维数组展平为一维数组。StarRocks也是如此。从v2.5开始，新增了对从Elasticsearch查询ARRAY数据的支持。

### 谓词下推

StarRocks支持谓词下推。可以将过滤器下推到Elasticsearch执行，从而提高查询性能。下表列出了支持谓词下推的运算符。

|   SQL语法  |   ES语法  |
| :---: | :---: |
|  `=`   |  term查询   |
|  `in`   |  terms查询   |
|  `>=,  <=, >, <`   |  范围   |
|  `and`   |  bool.filter   |
|  `or`   |  bool.should   |
|  `not`   |  bool.must_not   |
|  `not in`   |  bool.must_not + terms   |
|  `esquery`   |  ES查询DSL  |

### 例子

**esquery函数**用于将**无法用SQL表示**的查询（如match和geoshape）下推到Elasticsearch进行过滤。esquery函数中的第一个参数用于关联索引。第二个参数是基本查询DSL的JSON表达式，它用括号{}括起来。**JSON表达式只能有一个根键**，例如match、geo_shape或bool。

* 匹配查询

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on elasticsearch"
    }
}');
~~~

* 与地理位置相关的查询

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

* 布尔查询

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

### 使用说明

* Elasticsearch 5.x之前的版本扫描数据的方式与5.x之后的版本不同，目前**仅支持**5.x之后的版本。
* 支持启用了HTTP基本身份验证的Elasticsearch集群。
* 从StarRocks查询数据可能不如直接从Elasticsearch查询数据，例如与计数相关的查询。原因是Elasticsearch直接读取目标文档的元数据，无需过滤真实数据，加速了计数查询。

## （已弃用）Hive外部表

在使用Hive外部表之前，请确保您的服务器上已安装JDK 1.8。

### 创建Hive资源

Hive资源对应一个Hive集群。您需要配置StarRocks使用的Hive集群，例如Hive元存储地址。您必须指定Hive外部表使用的Hive资源。

* 创建名为hive0的Hive资源。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
~~~

* 查看在StarRocks中创建的资源。

~~~sql
SHOW RESOURCES;
~~~

* 删除名为`hive0`的资源。

~~~sql
DROP RESOURCE "hive0";
~~~

您可以在StarRocks 2.3及以后版本中修改Hive资源的`hive.metastore.uris`。有关详细信息，请参阅[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

### 创建数据库

~~~sql
CREATE DATABASE hive_test;
USE hive_test;
~~~

### 创建Hive外部表

语法

~~~sql
CREATE EXTERNAL TABLE table_name (
  col_name col_type [NULL | NOT NULL] [COMMENT "comment"]
) ENGINE=HIVE
PROPERTIES (
  "key" = "value"
);
~~~

示例：在与资源对应的Hive集群中，在`rawdata`数据库下创建外部表`profile_parquet_p7`。

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

描述：

* 外部表中的列
  * 列名必须与Hive表中的列名相同。
  * 列顺序**不需要**与Hive表中的列顺序相同。
  * 只能选择**Hive表中的某些列**，但必须选择所有**分区键列**。
  * 外部表的分区键列不需要使用`partition by`指定。它们必须在与其他列相同的说明列表中定义。您不需要指定分区信息。StarRocks会自动从Hive表中同步这些信息。
  * 设置为`ENGINE` HIVE。
* 性能：
  * **hive.resource**：使用的Hive资源。
  * **database**：Hive数据库。
  * **table**：Hive中的表。**不支持视图**。
* Hive和StarRocks的列数据类型对应关系如下表所示。

    |  Hive的列类型   |  StarRocks的列类型   | 描述 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |
    |   TIMESTAMP  | DATETIME    | 将TIMESTAMP数据转换为DATETIME数据时，将丢失精度和时区信息。您需要根据sessionVariable中的时区将TIMESTAMP数据转换为没有时区偏移量的DATETIME数据。 |
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意：
>
> * 目前，支持的Hive存储格式为Parquet、ORC和CSV。
如果存储格式为CSV，则不能使用引号作为转义字符。
> * 支持SNAPPY和LZ4压缩格式。
> * 可查询的Hive字符串列的最大长度为1MB。如果字符串列超过1MB，将处理为null列。

### 使用Hive外部表

查询`profile_wos_p7`的总行数。

~~~sql
select count(*) from profile_wos_p7;
~~~

### 更新缓存的Hive表元数据


* Hive 分区信息和相关文件信息都会被缓存在 StarRocks 中。缓存会按照 `hive_meta_cache_refresh_interval_s` 指定的时间间隔进行刷新，默认值为 7200。`hive_meta_cache_ttl_s` 指定了缓存的超时持续时间，默认值为 86400。
  * 缓存的数据也可以手动进行刷新。
    1. 如果在 Hive 中的某个表中添加或删除了分区，您必须运行 `REFRESH EXTERNAL TABLE hive_t` 命令来刷新 StarRocks 中缓存的表元数据。`hive_t` 是 StarRocks 中 Hive 外部表的名称。
    2. 如果某些 Hive 分区中的数据发生了更新，您必须通过运行 `REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')` 命令来刷新 StarRocks 中的缓存数据。`hive_t` 是 StarRocks 中 Hive 外部表的名称。`'k1=01/k2=02'` 和 `'k1=03/k2=04'` 是数据发生更新的 Hive 分区的名称。
    3. 当您运行 `REFRESH EXTERNAL TABLE hive_t` 时，StarRocks 首先会检查 Hive 外部表的列信息是否与 Hive Metastore 返回的 Hive 表的列信息一致。如果 Hive 表的结构发生变化，例如增加列或移除列，StarRocks 会将这些更改同步到 Hive 外部表。同步完成后，Hive 外部表的列顺序将与 Hive 表的列顺序相同，分区列将成为最后一列。
* 当 Hive 数据以 Parquet、ORC 和 CSV 格式存储时，在 StarRocks 2.3 及更高版本中，您可以将 Hive 表的 Schema 变更（例如 ADD COLUMN 和 REPLACE COLUMN）同步到 Hive 外部表。

### 访问对象存储

* FE 配置文件的路径为 `fe/conf`，如果您需要自定义 Hadoop 集群，可以向该路径添加配置文件。例如：如果 HDFS 集群使用高可用 nameservice，则需要将 `hdfs-site.xml` 放置在 `fe/conf`。如果 HDFS 配置了 ViewFs，则需要将 `core-site.xml` 放置在 `fe/conf`。
* BE 配置文件的路径为 `be/conf`，如果您需要自定义 Hadoop 集群，可以向该路径添加配置文件。例如，如果 HDFS 集群使用高可用性 nameservice，则需要将 `hdfs-site.xml` 放置在 `be/conf`。如果 HDFS 配置了 ViewFs，则需要将 `core-site.xml` 放置在 `be/conf`。
* 在 BE 所在的机器上，在 BE 的 **启动脚本** `bin/start_be.sh` 中，将 JAVA_HOME 配置为 JDK 环境，而不是 JRE 环境，例如，`export JAVA_HOME = <JDK 路径>`。您必须在脚本的开头添加此配置，并重新启动 BE 以使配置生效。
* 配置 Kerberos 支持：
  1. 要登录到所有 FE/BE 机器，您需要使用 `kinit -kt keytab_path principal`，并且需要访问 Hive 和 HDFS。kinit 命令的登录有效期有限，需要将其定期执行放入 crontab 中。
  2. 将 `hive-site.xml/core-site.xml/hdfs-site.xml` 放置在 `fe/conf` 下，然后将 `core-site.xml/hdfs-site.xml` 放置在 `be/conf` 下。
  3. 将 `-Djava.security.krb5.conf=/etc/krb5.conf` 添加到 **$FE_HOME/conf/fe.conf** 文件中的 `JAVA_OPTS` 选项的值中。**/etc/krb5.conf** 是 **krb5.conf** 文件的保存路径。您可以根据操作系统更改路径。
  4. 直接将 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` 添加到 **$BE_HOME/conf/be.conf** 文件中。**/etc/krb5.conf** 是 **krb5.conf** 文件的保存路径。您可以根据操作系统更改路径。
  5. 当添加 Hive 资源时，必须向 `hive.metastore.uris` 传递域名。此外，您需要在 **/etc/hosts** 文件中添加 Hive/HDFS 域名和 IP 地址之间的映射关系。

* 配置对 AWS S3 的支持：将以下配置添加到 `fe/conf/core-site.xml` 和 `be/conf/core-site.xml`。

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

   1. `fs.s3a.access.key`：AWS 访问密钥 ID。
   2. `fs.s3a.secret.key`：AWS 私有密钥。
   3. `fs.s3a.endpoint`：要连接的 AWS S3 终端节点。
   4. `fs.s3a.connection.maximum`：StarRocks 到 S3 的最大并发连接数。如果在查询过程中出现 `Timeout waiting for connection from poll` 错误，可以将此参数设置为较大的值。

## （已弃用）Iceberg 外部表

从 v2.1.0 开始，StarRocks 允许您使用外部表从 Apache Iceberg 查询数据。要查询 Iceberg 中的数据，您需要在 StarRocks 中创建一个 Iceberg 外部表。创建表时，您需要建立外部表和要查询的 Iceberg 表之间的映射。

### 开始之前

请确保 StarRocks 有权访问 Apache Iceberg 使用的元数据服务（如 Hive 元存储）、文件系统（如 HDFS）和对象存储系统（如 Amazon S3 和阿里云对象存储服务）。

### 注意事项

* Iceberg 外部表只能用于查询以下类型的数据：
  * Iceberg v1 表（分析数据表）。从 v3.0 开始支持 ORC 格式的 Iceberg v2（行级删除）表，从 v3.1 开始支持 Parquet 格式的 Iceberg v2 表。有关 Iceberg v1 表和 Iceberg v2 表的区别，请参见 [Iceberg 表规格](https://iceberg.apache.org/spec/)。
  * 以 gzip（默认格式）、Zstd、LZ4 或 Snappy 格式压缩的表。
  * 以 Parquet 或 ORC 格式存储的文件。

* StarRocks 2.3 及更高版本的 Iceberg 外部表支持同步 Iceberg 表的 Schema 变更，而 StarRocks 2.3 之前版本的 Iceberg 外部表不支持。如果 Iceberg 表的 Schema 发生变更，您必须删除对应的外部表并创建一个新的外部表。

### 过程

#### 步骤 1：创建 Iceberg 资源

在创建 Iceberg 外部表之前，您必须在 StarRocks 中创建 Iceberg 资源。该资源用于管理 Iceberg 访问信息。此外，您还需要在用于创建外部表的语句中指定此资源。您可以根据业务需求创建资源：

* 如果 Iceberg 表的元数据是从 Hive 元存储中获取的，则可以创建资源，并将目录类型设置为 `HIVE`。

* 如果 Iceberg 表的元数据是从其他服务获取的，则需要创建自定义目录。然后创建资源并将目录类型设置为 `CUSTOM`。

##### 创建目录类型为 `HIVE`

例如，创建一个名为 `iceberg0` 的资源，并将目录类型设置为 `HIVE`。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",
   "iceberg.catalog.hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083" 
);
~~~

相关参数说明如下表所示。

| **参数**                       | **描述**                                              |
| ----------------------------------- | ------------------------------------------------------------ |
| 类型                                | 资源类型。将值设置为 `iceberg`。               |
| iceberg.catalog.type              | 资源的目录类型。支持 Hive 目录和自定义目录。如果指定 Hive 目录，请将值设置为 `HIVE`。如果指定自定义目录，请将值设置为 `CUSTOM`。 |
| iceberg.catalog.hive.metastore.uris | Hive 元存储的 URI。参数值采用以下格式： `thrift://< Iceberg 元数据的 IP 地址 >:< 端口号 >`。端口号默认为 9083。Apache Iceberg 使用 Hive 目录访问 Hive 元存储，然后查询 Iceberg 表的元数据。 |

##### 创建目录类型为 `CUSTOM`

自定义目录需要继承抽象类 BaseMetastoreCatalog，并且需要实现 IcebergCatalog 接口。此外，自定义目录的类名不能与 StarRocks 中已有的类名重复。目录创建完成后，将目录及其相关文件打包，并放置在每个前端（FE）的 **fe/lib** 路径下。然后重新启动每个 FE。完成上述操作后，您可以创建目录为自定义目录的资源。

例如，创建一个名为 `iceberg1` 的资源，并将目录类型设置为 `CUSTOM`。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg1" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 
);
~~~

相关参数说明如下表所示。

| **参数**          | **描述**                                              |
| ---------------------- | ------------------------------------------------------------ |
| 类型                   | 资源类型。将值设置为 `iceberg`。               |
| iceberg.catalog.type | 资源的目录类型。支持 Hive 目录和自定义目录。如果指定 Hive 目录，请将值设置为 `HIVE`。如果指定自定义目录，请将值设置为 `CUSTOM`。 |
| iceberg.catalog-impl   | 自定义目录的完全限定类名。FE 根据此名称搜索目录。如果目录包含自定义配置项，您必须在创建 Iceberg 外部表时将其作为键值对添加到 `PROPERTIES` 参数中。 |

您可以在 StarRocks 2.3 及更高版本中修改 Iceberg 资源的 `hive.metastore.uris` 和 `iceberg.catalog-impl`。有关详细信息，请参阅 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

##### 查看 Iceberg 资源

~~~SQL
SHOW RESOURCES;
~~~

##### 删除 Iceberg 资源

例如，删除名为 `iceberg0` 的资源。

~~~SQL
DROP RESOURCE "iceberg0";
~~~


删除 Iceberg 资源会导致所有引用该资源的外部表不可用。但是，Apache Iceberg 中的相应数据不会被删除。如果您仍然需要查询 Apache Iceberg 中的数据，请创建一个新的资源和一个新的外部表。

#### 步骤 2：（可选）创建数据库

例如，在 StarRocks 中创建一个名为 `iceberg_test` 的数据库。

~~~SQL
CREATE DATABASE iceberg_test; 
USE iceberg_test; 
~~~

> 注意：StarRocks 中的数据库名称可以与 Apache Iceberg 中的数据库名称不同。

#### 步骤 3：创建 Iceberg 外部表

例如，在 `iceberg_test` 数据库中创建一个名为 `iceberg_tbl` 的 Iceberg 外部表。

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

下表描述了相关参数。

| **参数** | **描述**                                              |
| ------------- | ------------------------------------------------------------ |
| ENGINE        | 引擎名称。将值设置为 `ICEBERG`。                 |
| resource      | 外部表引用的 Iceberg 资源的名称。 |
| database      | Iceberg 表所属的数据库的名称。 |
| table         | Iceberg 表的名称。                               |

> 注意：
   >
   > * 外部表的名称可以与 Iceberg 表的名称不同。
   >
   > * 外部表的列名必须与 Iceberg 表中的列名相同。两个表的列顺序可以不同。

如果您在自定义目录中定义了配置项，并希望在查询数据时生效，可以在创建外部表时将配置项作为键值对添加到 `PROPERTIES` 参数中。例如，如果您在自定义目录中定义了配置项 `custom-catalog.properties`，则可以执行以下命令创建外部表。

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

创建外部表时，需要根据 Iceberg 表中列的数据类型，指定外部表中列的数据类型。下表显示了列数据类型的映射。

| **Iceberg 表** | **Iceberg 外部表** |
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

StarRocks 不支持查询数据类型为 TIMESTAMPTZ、STRUCT 和 MAP 的 Iceberg 数据。

#### 步骤 4：查询 Apache Iceberg 中的数据

创建外部表后，您可以使用外部表查询 Apache Iceberg 中的数据。

~~~SQL
select count(*) from iceberg_tbl;
~~~

## （已弃用）Hudi 外部表

从 v2.2.0 开始，StarRocks 允许您使用 Hudi 外部表从 Hudi 数据湖中查询数据，从而实现快速的数据湖分析。本主题描述了如何在 StarRocks 集群中创建 Hudi 外部表，并使用 Hudi 外部表从 Hudi 数据湖中查询数据。

### 开始之前

确保您的 StarRocks 集群已被授予对 Hive 元存储、HDFS 集群或注册 Hudi 表的存储桶的访问权限。

### 注意事项

* Hudi 的 Hudi 外部表是只读的，仅可用于查询。
* StarRocks 支持查询写入复制和合并读取表（从 v2.5 开始支持 MOR 表）。有关这两种表的差异，请参见 [表和查询类型](https://hudi.apache.org/docs/table_types/)。
* StarRocks 支持 Hudi 的以下两种查询类型：快照查询和读取优化查询（Hudi 仅支持对合并读取表执行读取优化查询）。不支持增量查询。有关 Hudi 查询类型的更多信息，请参见 [表和查询类型](https://hudi.apache.org/docs/next/table_types/#query-types)。
* StarRocks 支持对 Hudi 文件使用 gzip、zstd、LZ4 和 Snappy 等压缩格式。Hudi 文件的默认压缩格式为 gzip。
* StarRocks 无法同步 Hudi 托管表的模式更改。有关更多信息，请参见 [模式演变](https://hudi.apache.org/docs/schema_evolution/)。如果更改了 Hudi 托管表的模式，您必须从 StarRocks 集群中删除关联的 Hudi 外部表，然后重新创建该外部表。

### 过程

#### 步骤 1：创建和管理 Hudi 资源

您必须在 StarRocks 集群中创建 Hudi 资源。Hudi 资源用于管理您在 StarRocks 集群中创建的 Hudi 数据库和外部表。

##### 创建 Hudi 资源

执行以下语句，创建名为 `hudi0` 的 Hudi 资源：

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
~~~

下表描述了参数。

| 参数           | 描述                                                  |
| ------------------- | ------------------------------------------------------------ |
| 类型                | Hudi 资源的类型。将值设置为 hudi。         |
| hive.metastore.uris | Hudi 资源连接的 Hive 元存储的 Thrift URI。连接 Hudi 资源到 Hive 元存储后，您可以使用 Hive 创建和管理 Hudi 表。Thrift URI 格式为 `<Hive 元存储的 IP 地址>:<Hive 元存储的端口号>`。默认端口号为 9083。 |

从 v2.3 开始，StarRocks 允许更改 Hudi 资源的 `hive.metastore.uris` 值。有关更多信息，请参见 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

##### 查看 Hudi 资源

执行以下语句，查看在 StarRocks 集群中创建的所有 Hudi 资源：

~~~SQL
SHOW RESOURCES;
~~~

##### 删除 Hudi 资源

执行以下语句，删除名为 `hudi0` 的 Hudi 资源：

~~~SQL
DROP RESOURCE "hudi0";
~~~

> 注意：
>
> 删除 Hudi 资源会导致使用该 Hudi 资源创建的所有 Hudi 外部表不可用。但是，删除不会影响存储在 Hudi 中的数据。如果您仍想使用 StarRocks 查询 Hudi 数据，您需要在 StarRocks 集群中重新创建 Hudi 资源、Hudi 数据库和 Hudi 外部表。

#### 步骤 2：创建 Hudi 数据库

执行以下语句，在 StarRocks 集群中创建并打开名为 `hudi_test` 的 Hudi 数据库：

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 
~~~

> 注意：
>
> 您在 StarRocks 集群中指定的 Hudi 数据库名称不需要与 Hudi 中的关联数据库名称相同。

#### 步骤 3：创建 Hudi 外部表

执行以下语句，在 `hudi_test` Hudi 数据库中创建名为 `hudi_tbl` 的 Hudi 外部表：

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

下表描述了参数。

| 参数 | 描述                                                  |
| --------- | ------------------------------------------------------------ |
| ENGINE    | Hudi 外部表的查询引擎。将值设置为 `HUDI`。 |
| resource  | StarRocks 集群中的 Hudi 资源名称。     |
| database  | StarRocks 集群中的 Hudi 外部表所属的 Hudi 数据库名称。 |
| table     | 与 Hudi 外部表关联的 Hudi 托管表。 |

> 注意：
>
> * 您为 Hudi 外部表指定的名称不需要与关联的 Hudi 托管表名称相同。
>
> * Hudi 外部表中的列必须具有相同的名称，但与关联的 Hudi 托管表中的对应列相比，其顺序可能不同。
>
> * 您可以从关联的 Hudi 托管表中选择部分或全部列，并在 Hudi 外部表中仅创建选定的列。以下表列出了 Hudi 支持的数据类型与 StarRocks 支持的数据类型之间的映射关系。

| Hudi支持的数据类型   | StarRocks 支持的数据类型 |
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
> StarRocks 不支持查询 STRUCT 或 MAP 类型的数据，也不支持在 Merge On Read 表中查询 ARRAY 类型的数据。

#### 步骤 4：从 Hudi 外部表查询数据

创建与特定 Hudi 托管表关联的 Hudi 外部表后，无需将数据加载到 Hudi 外部表中。如需从 Hudi 查询数据，请执行以下语句：

~~~SQL
SELECT COUNT(*) FROM hudi_tbl;
~~~

## (已弃用) MySQL 外部表

在星型架构中，数据通常被划分为维度表和事实表。维度表的数据较少，但涉及到 UPDATE 操作。目前，StarRocks 不支持直接的 UPDATE 操作（可以通过使用唯一键表来实现更新）。在某些情况下，您可以将维度表存储在 MySQL 中，以便直接读取数据。

要查询 MySQL 数据，您需要在 StarRocks 中创建一个外部表，并将其映射到您的 MySQL 数据库中的表。在创建表时，您需要指定 MySQL 连接信息。

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

参数：

* **host**：MySQL 数据库的连接地址
* **port**：MySQL 数据库的端口号
* **user**：登录 MySQL 的用户名
* **password**：登录 MySQL 的密码
* **database**：MySQL 数据库的名称
* **table**：MySQL 数据库中的表名