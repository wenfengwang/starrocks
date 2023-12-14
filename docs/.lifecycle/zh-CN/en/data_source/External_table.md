---
displayed_sidebar: "Chinese"
---

# 外部表

:::note
从v3.0开始，我们建议您使用目录来从Hive、Iceberg和Hudi查询数据。请参阅[Hive目录](../data_source/catalog/hive_catalog.md)、[Iceberg目录](../data_source/catalog/iceberg_catalog.md)和[Hudi目录](../data_source/catalog/hudi_catalog.md)。

从v3.1开始，我们建议您使用[JDBC目录](../data_source/catalog/jdbc_catalog.md)从MySQL和PostgreSQL查询数据，并使用[Elasticsearch目录](../data_source/catalog/elasticsearch_catalog.md)从Elasticsearch查询数据。
:::

StarRocks通过使用外部表支持访问其他数据源。外部表是基于存储在其他数据源中的数据表创建的。StarRocks仅存储数据表的元数据。您可以使用外部表直接查询其他数据源中的数据。StarRocks支持以下数据源：MySQL、StarRocks、Elasticsearch、Apache Hive™、Apache Iceberg和Apache Hudi。**目前，您只能将来自另一个StarRocks集群的数据写入当前StarRocks集群，而无法从中读取。对于StarRocks之外的数据源，您只能从这些数据源中读取数据。**

从2.5开始，StarRocks提供了数据缓存功能，用于加速对外部数据源上的热数据查询。了解更多信息，请参阅[数据缓存](data_cache.md)。

## StarRocks外部表

从StarRocks 1.19开始，StarRocks允许您使用StarRocks外部表将数据从一个StarRocks集群写入另一个。这样实现了读写分离，并提供更好的资源隔离。您可以首先在目标StarRocks集群中创建一个目标表。然后，在源StarRocks集群中，您可以创建一个具有与目标表相同模式的StarRocks外部表，并在`PROPERTIES`字段中指定目标集群和表的信息。 

可以通过使用INSERT INTO语句将数据从源集群写入目标集群，以写入StarRocks外部表。它有助于实现以下目标：

* StarRocks集群之间的数据同步。
* 读写分离。数据写入源集群，并将来自源集群的数据变更同步到目标集群，从而提供查询服务。

以下代码显示了如何创建目标表和外部表。

~~~SQL
# 在目标StarRocks集群中创建目标表。
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

# 在源StarRocks集群中创建外部表。
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

# 通过将数据写入StarRocks外部表将数据从源集群写入目标集群。建议在生产环境中使用第二个语句。
insert into external_t values ('2020-10-11', 1, 1, 'hello', '2020-10-11 10:00:00');
insert into external_t select * from other_table;
~~~

参数：

* **EXTERNAL：** 此关键字表示要创建的表为外部表。
* **host：** 此参数指定目标StarRocks集群的leader FE节点的IP地址。
* **port：** 此参数指定目标StarRocks集群的FE节点的RPC端口。

  :::note

  为确保StarRocks外部表所属的源集群可以访问目标StarRocks集群，您必须配置网络和防火墙以允许对以下端口的访问：

  * FE节点的RPC端口。参见FE配置文件**fe/fe.conf**中的`rpc_port`。默认RPC端口为`9020`。
  * BE节点的bRPC端口。参见BE配置文件**be/be.conf**中的`brpc_port`。默认bRPC端口为`8060`。

  :::

* **user：** 此参数指定用于访问目标StarRocks集群的用户名。
* **password：** 此参数指定用于访问目标StarRocks集群的密码。
* **database：** 此参数指定目标表所属的数据库。
* **table：** 此参数指定目标表的名称。

使用StarRocks外部表时有以下限制：

* 您只能在StarRocks外部表上运行INSERT INTO和SHOW CREATE TABLE命令。不支持其他数据写入方法。此外，您不能从StarRocks外部表查询数据或对外部表执行DDL操作。
* 创建外部表的语法与创建普通表相同，但是外部表中的列名和其他信息必须与目标表相同。
* 外部表每10秒同步目标表的表元数据。如果在目标表上执行DDL操作，则两个表之间的数据同步可能会有延迟。

## 适用于支持JDBC的数据库的外部表

从v2.3.0开始，StarRocks提供了用于查询支持JDBC的数据库的外部表。这样，您可以在StarRocks中快速分析此类数据库的数据，而无需将数据导入StarRocks。本部分描述了在StarRocks中创建外部表并查询支持JDBC的数据库中的数据。

### 先决条件

在使用JDBC外部表查询数据之前，请确保FE和BE能够访问JDBC驱动程序的下载URL。下载URL由用于创建JDBC资源的语句中的`driver_url`参数指定。

### 创建和管理JDBC资源

#### 创建JDBC资源

在创建外部表以从数据库中查询数据之前，您需要在StarRocks中创建一个JDBC资源以管理数据库的连接信息。该数据库必须支持JDBC驱动程序，并被称为“目标数据库”。创建资源后，您可以将其用于创建外部表。

执行以下语句创建名为`jdbc0`的JDBC资源：

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

`PROPERTIES`中的必需参数如下：

* `type`：资源的类型。将值设为`jdbc`。

* `user`：用于连接到目标数据库的用户名。

* `password`：用于连接到目标数据库的密码。

* `jdbc_uri`：JDBC驱动程序用于连接到目标数据库的URI。URI格式必须满足数据库URI语法。有关某些常见数据库的URI语法，请访问[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)的官方网站。

> 注意：URI必须包含目标数据库的名称。例如，在上面的代码示例中，`jdbc_test`是您要连接的目标数据库的名称。

* `driver_url`：JDBC驱动程序JAR包的下载URL。支持HTTP URL或文件URL，例如`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`或`file:///home/disk1/postgresql-42.3.3.jar`。

* `driver_class`：JDBC驱动程序的类名。常见数据库的JDBC驱动程序类名如下：
  * MySQL: com.mysql.jdbc.Driver (MySQL 5.x及更早版本), com.mysql.cj.jdbc.Driver (MySQL 6.x及更新版本)
  * SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
  * Oracle: oracle.jdbc.driver.OracleDriver
  * PostgreSQL: org.postgresql.Driver

资源创建时，FE使用`driver_url`参数中指定的URL下载JDBC驱动程序JAR包，生成校验和，并使用该校验和验证BE下载的JDBC驱动程序。

> 注意: 如果下载JDBC驱动程序JAR包失败，则资源的创建也会失败。

当业务执行首次查询JDBC外部表时，发现对应的JDBC驱动程序JAR包在其计算机上不存在时，业务通过使用“driver_url”参数中指定的URL下载JDBC驱动程序JAR包，并且所有的JDBC驱动程序JAR包都会保存在`${STARROCKS_HOME}/lib/jdbc_drivers`目录中。

#### 查看JDBC资源

执行以下语句来查看StarRocks中的所有JDBC资源：

~~~SQL
SHOW RESOURCES;
~~~

> 注意: “ResourceType”列是`jdbc`。

#### 删除JDBC资源

执行以下语句以删除名为`jdbc0`的JDBC资源：

~~~SQL
DROP RESOURCE "jdbc0";
~~~

> 注意: 删除JDBC资源后，使用该JDBC资源创建的所有JDBC外部表将不可用。但是，目标数据库中的数据不会丢失。如果您仍需使用StarRocks来查询目标数据库中的数据，则可以再次创建JDBC资源和JDBC外部表。

### 创建数据库

执行以下语句在StarRocks中创建和访问名为`jdbc_test`的数据库：

~~~SQL
CREATE DATABASE jdbc_test; 
USE jdbc_test; 
~~~

> 注意: 您在上述语句中指定的数据库名称不需要与目标数据库的名称相同。

### 创建JDBC外部表

执行以下语句在名为`jdbc_test`的数据库中创建名为`jdbc_tbl`的JDBC外部表：

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

在`properties`中的必需参数如下：

* `resource`: 用于创建外部表的JDBC资源的名称。

* `table`: 数据库中的目标表名称。

有关StarRocks和目标数据库之间支持的数据类型以及数据类型映射的详细信息，请参见[数据类型映射](External_table.md#Data type mapping)。

> 注意:
>
> * 不支持索引。
> * 无法使用`PARTITION BY`或`DISTRIBUTED BY`来指定数据分发规则。

### 查询JDBC外部表

在查询JDBC外部表之前，您必须执行以下语句以启用Pipeline引擎：

~~~SQL
set enable_pipeline_engine=true;
~~~

> 注意: 如果Pipeline引擎已经启用，则可以跳过此步骤。

执行以下语句以通过JDBC外部表查询目标数据库中的数据。

~~~SQL
select * from JDBC_tbl;
~~~

StarRocks支持通过将过滤条件下推到目标表来执行谓词下推。尽可能地在数据源附近执行过滤条件可以提高查询性能。目前，StarRocks可以下推操作符，包括二元比较操作符（`>`, `>=`, `=`, `<`, 和 `<=`），`IN`，`IS NULL`和`BETWEEN ... AND ...`。但是，StarRocks无法下推函数。

### 数据类型映射

目前，StarRocks只能查询目标数据库中的基本类型数据，例如NUMBER、STRING、TIME和DATE。如果目标数据库中的数据值范围不受StarRocks支持，则查询会报错。

根据目标数据库的类型，目标数据库与StarRocks之间的映射会有所不同。

#### **MySQL和StarRocks**

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

#### **Oracle和StarRocks**

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

#### **PostgreSQL和StarRocks**

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

#### **SQL Server和StarRocks**

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

* 创建JDBC外部表时，不能在表上创建索引或使用`PARTITION BY`和`DISTRIBUTED BY`来指定表的数据分布规则。

* 查询JDBC外部表时，StarRocks无法将函数下推到表中。

## (已废弃) Elasticsearch外部表

StarRocks和Elasticsearch是两个受欢迎的分析系统。StarRocks在大规模分布式计算方面性能出众，而Elasticsearch则非常适合全文搜索。StarRocks与Elasticsearch结合可以提供更完整的OLAP解决方案。

### 创建Elasticsearch外部表示例

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

以下表格描述了参数。

| **参数**             | **必需** | **默认值** | **描述**                                                                                                                                                                                        |
| -------------------- | -------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| hosts                | 是       | 无         | Elasticsearch集群的连接地址。您可以指定一个或多个地址。StarRocks可以解析Elasticsearch版本和索引分片分配从此地址。StarRocks基于`GET /_nodes/http` API操作返回的地址与Elasticsearch集群通信。因此，“hosts”参数的值必须与`GET /_nodes/http`API操作返回的地址相同。否则，业务可能无法与您的Elasticsearch集群通信。 |
| index                | 是       | 无         | 在StarRocks中创建的Elasticsearch索引的名称。名称可以是别名。此参数支持通配符（\*）。例如，如果您将`index`设置为<code class="language-text">hello*</code>，StarRocks将检索所有以`hello`开头的索引。                  |
| user                 | 否       | 空         | 用于使用基本身份验证登录到Elasticsearch集群的用户名。确保您可以访问`/*cluster/state/*nodes/http`和索引。                                                                                       |
| password             | 否       | 空         | 用于登录到Elasticsearch集群的密码。                                                                                                                                                             |
| type                 | 否       | `_doc`     | 索引的类型。默认值: `_doc`。如果要查询Elasticsearch 8及更高版本中的数据，则不需要配置此参数，因为Elasticsearch 8及更高版本中已移除了映射类型。                                                     |
| es.nodes.wan.only    | 否      | `false`        | 指定StarRocks是否仅使用由`hosts`指定的地址访问Elasticsearch集群并获取数据。<ul><li>`true`: StarRocks仅使用由`hosts`指定的地址访问Elasticsearch集群并获取数据，并且不会对Elasticsearch索引分片所在的数据节点进行嗅探。如果StarRocks无法访问Elasticsearch集群内的数据节点地址，则需要将此参数设置为`true`。</li><li>`false`: StarRocks使用由`host`指定的地址对Elasticsearch集群中的数据节点进行嗅探，确定Elasticsearch集群内的数据节点后，相关BE会直接访问Elasticsearch集群内的数据节点来获取索引分片的数据。如果StarRocks可以访问Elasticsearch集群内的数据节点地址，建议保留默认值`false`。</li></ul> |
| es.net.ssl           | 否      | `false`        | 指定是否可以使用HTTPS协议访问您的Elasticsearch集群。仅支持StarRocks 2.4及更高版本配置此参数。<ul><li>`true`: 可以使用HTTP和HTTPS协议访问您的Elasticsearch集群。</li><li>`false`: 只能使用HTTP协议访问您的Elasticsearch集群。</li></ul> |
| enable_docvalue_scan | 否      | `true`         | 指定是否从Elasticsearch列存储中获取目标字段的值。在大多数情况下，从列存储中读取数据的性能优于从行存储中读取数据。 |
| enable_keyword_sniff | 否      | `true`         | 指定是否基于KEYWORD类型字段在Elasticsearch中嗅探TEXT类型字段。如果该参数设置为`false`，StarRocks将在标记化后执行匹配。 |

##### 用于加快查询的列式扫描

如果将`enable_docvalue_scan`设置为`true`，StarRocks在从Elasticsearch获取数据时会遵循以下规则：

* **尝试并查看**：StarRocks会自动检查目标字段是否已启用列式存储。如果已启用，StarRocks将从列式存储中获取目标字段中的所有值。
* **自动降级**：如果目标字段中有任一字段未在列式存储中，StarRocks将解析并从行存储(`_source`)中获取目标字段中的所有值。

> **注意**
>
> * Elasticsearch中的TEXT类型字段不支持列式存储。因此，如果查询包含TEXT类型值的字段，则StarRocks将从`_source`中获取字段的值。
> * 如果查询大量（大于或等于25个）字段，则从`docvalue`中读取字段值与从`_source`中读取字段值相比并无明显性能优势。

##### 嗅探KEYWORD类型字段

如果将`enable_keyword_sniff`设置为`true`，Elasticsearch允许直接进行数据摄入而无需索引，因为它将在摄入后自动创建索引。对于STRING类型字段，Elasticsearch会创建一个同时具有TEXT和KEYWORD类型的字段。这是Elasticsearch的Multi-Field功能的工作原理。映射如下：

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

例如，对`k4`执行“=”过滤操作，StarRocks在Elasticsearch上将其转换为一个Elasticsearch TermQuery。

原始SQL过滤器如下：

~~~SQL
k4 = "StarRocks On Elasticsearch"
~~~

转换后的Elasticsearch查询DSL如下：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"

}
~~~

`k4`的第一个字段是TEXT，数据摄入后将由为`k4`配置的分析器（如果未为`k4`配置分析器，则由标准分析器）进行标记化。结果，第一个字段将被标记化为三个术语：`StarRocks`、`On`和`Elasticsearch`。详情如下：

~~~SQL
POST /_analyze
{
  "analyzer": "standard",
  "text": "StarRocks On Elasticsearch"
}
~~~

标记化结果如下：

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

假设进行以下查询：

~~~SQL
"term" : {
    "k4": "StarRocks On Elasticsearch"
}
~~~

词典中没有匹配术语`StarRocks On Elasticsearch`的词项，因此不会返回任何结果。

但是，如果将`enable_keyword_sniff`设置为`true`，StarRocks将`k4 = "StarRocks On Elasticsearch"`转换为`k4.keyword = "StarRocks On Elasticsearch"`，以匹配SQL语义。转换后的`StarRocks On Elasticsearch`查询DSL如下：

~~~SQL
"term" : {
    "k4.keyword": "StarRocks On Elasticsearch"
}
~~~

`k4.keyword`是KEYWORD类型。因此，数据以完整术语的形式写入Elasticsearch，可以成功匹配。

#### 列数据类型的映射

创建外部表时，需要基于Elasticsearch表中的列数据类型指定外部表中列的数据类型。以下表格显示了列数据类型的映射。

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
> * StarRocks使用与JSON相关函数来读取NESTED类型数据。
> * Elasticsearch自动将多维数组扁平化为一维数组。StarRocks也会执行相同操作。**从v2.5开始，支持从Elasticsearch查询ARRAY数据。**

### 谓词下推

StarRocks支持谓词下推。可以将过滤器下推到Elasticsearch进行执行，从而提高查询性能。以下表格列出了支持谓词下推的运算符。

|   SQL语法  |   ES语法  |
| :---: | :---: |
|  `=`   |  term query   |
|  `in`   |  terms query   |
|  `>=,  <=, >, <`   |  range   |
|  `and`   |  bool.filter   |
|  `or`   |  bool.should   |
|  `not`   |  bool.must_not   |
|  `not in`   |  bool.must_not + terms   |
|  `esquery`   |  ES查询DSL  |

### 示例

**esquery函数**用于将**无法用SQL表达的查询**（如match和geoshape）下推到Elasticsearch进行过滤。esquery函数中的第一个参数用于关联一个索引。第二个参数是基本查询DSL的JSON表达式，括号内包含{}。**JSON表达式必须唯一地有一个根键**，如match、geo_shape或bool。

* match查询

~~~sql
select * from es_table where esquery(k4, '{
    "match": {
       "k4": "StarRocks on elasticsearch"
    }
}');
~~~

* 与地理相关的查询

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

### 使用注意事项

* Elasticsearch早于5.x以不同的方式扫描数据，而5.x后的版本扫描方式不同。目前仅支持**5.x后的版本**。
* 支持启用HTTP基本身份验证的Elasticsearch集群。
* 从StarRocks查询数据可能没有直接从Elasticsearch查询数据那么快，比如计数相关的查询。原因是Elasticsearch直接读取目标文档的元数据，无需过滤实际数据，加快了计数查询。


## （已弃用）Hive外部表

在使用Hive外部表之前，请确保您的服务器上已安装了JDK 1.8。

### 创建Hive资源

Hive资源对应Hive集群。您必须配置StarRocks使用的Hive集群，如Hive metastore地址。您必须指定Hive外部表使用的Hive资源。

* 创建名为hive0的Hive资源。

~~~sql
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://10.10.44.98:9083"
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

您可以在StarRocks 2.3及更高版本中修改Hive资源的`hive.metastore.uris`。更多信息，请参见[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

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

示例：在`hive0`资源对应的Hive集群中的`rawdata`数据库下创建外部表`profile_parquet_p7`。

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

说明：

* 外部表中的列
  * 列名必须与Hive表中的列名相同。
  * 列顺序 **无需与** Hive表中的列顺序相同。
  * 您只能选择Hive表中的 **一些列**，但必须选择 **所有的分区键列**。
  * 外部表的分区键列无需使用`partition by`指定。它们必须在与其他列相同的描述列表中定义。您不需要指定分区信息。StarRocks将会从Hive表自动同步这些信息。
  * 将`ENGINE`设置为HIVE。
* PROPERTIES:
  * **hive.resource**：使用的Hive资源。
  * **database**：Hive数据库。
  * **table**：Hive中的表。不支持**view**。
* 下表描述了Hive和StarRocks之间的列数据类型映射。

    |  Hive的列类型   |  StarRocks的列类型   | 描述 |
    | --- | --- | ---|
    |   INT/INTEGER  | INT    |
    |   BIGINT  | BIGINT    |
    |   TIMESTAMP  | DATETIME    | 当您将TIMESTAMP数据转换为在sessionVariable中所基于的时区中没有时间区偏移的DATETIME数据时，精度和时区信息会丢失。您需要将TIMESTAMP数据转换为没有时间区偏移的DATETIME数据。
    |  STRING  | VARCHAR   |
    |  VARCHAR  | VARCHAR   |
    |  CHAR  | CHAR   |
    |  DOUBLE | DOUBLE |
    | FLOAT | FLOAT|
    | DECIMAL | DECIMAL|
    | ARRAY | ARRAY |

> 注意:
>
> * 目前支持的Hive存储格式为Parquet、ORC和CSV。
如果存储格式为CSV，不可使用引号作为转义字符。
> * 支持SNAPPY和LZ4压缩格式。
> * 可查询的Hive字符串列的最大长度为1 MB。如果字符串列的长度超过1 MB，它将被处理为null列。

### 使用Hive外部表

查询`profile_wos_p7`的总行数。

~~~sql
select count(*) from profile_wos_p7;
~~~

### 更新缓存的Hive表元数据

* Hive分区信息及相关文件信息在StarRocks中被缓存。缓存在`hive_meta_cache_refresh_interval_s`指定的间隔时间内被刷新。默认值为7200。 `hive_meta_cache_ttl_s`指定缓存的超时持续时间，默认值为86400。
  * 缓存的数据也可以手动刷新。
    1. 如果在Hive表中的表中添加或删除了分区，您必须运行`REFRESH EXTERNAL TABLE hive_t`命令来刷新StarRocks缓存中的表元数据。`hive_t`是StarRocks中Hive外部表的名称。
    2. 如果某些Hive分区中的数据已更新，您必须通过运行`REFRESH EXTERNAL TABLE hive_t PARTITION ('k1=01/k2=02', 'k1=03/k2=04')`命令来刷新StarRocks中的缓存数据。`hive_t`是StarRocks中Hive外部表的名称。`'k1=01/k2=02'`和`'k1=03/k2=04'`是数据已更新的Hive分区的名称。
    3. 运行`REFRESH EXTERNAL TABLE hive_t`时，StarRocks首先检查Hive外部表的列信息是否与Hive Metastore返回的Hive表的列信息相同。若Hive表的模式发生更改，例如添加列或删除列，StarRocks会将更改同步到Hive外部表。同步后，Hive外部表的列顺序保持与Hive表的列顺序相同，分区列作为最后一列。
* 当Hive数据以Parquet、ORC和CSV格式存储时，您可以在StarRocks 2.3及更高版本中将Hive表的模式更改（例如ADD COLUMN和REPLACE COLUMN）同步到Hive外部表中。

### 访问对象存储

* FE配置文件的路径是`fe/conf`，如果需要自定义Hadoop集群的配置文件，可以将配置文件添加到此路径。例如：如果HDFS集群使用高可用性nameservice，您需要将`hdfs-site.xml`放在`fe/conf`下。如果HDFS配置了ViewFs，您需要将`core-site.xml`放在`fe/conf`下。
* BE配置文件的路径是`be/conf`，如果需要自定义Hadoop集群的配置文件，可以将配置文件添加到此路径。例如：如果HDFS集群使用高可用性nameservice，您需要将`hdfs-site.xml`放在`be/conf`下。如果HDFS配置了ViewFs，您需要将`core-site.xml`放在`be/conf`下。
* 在BE所在的机器上，将JAVA_HOME配置为JDK环境，而不是JRE环境，可以通过BE的**启动脚本** `bin/start_be.sh`进行配置，例如，`export JAVA_HOME = <JDK路径>`。您必须在脚本的开头添加此配置，并重新启动BE使配置生效。
* 配置Kerberos支持：
  1. 要登录到所有FE/BE机器并使用`kinit -kt keytab_path principal`，您需要访问Hive和HDFS。kinit命令登录仅在一段时间内有效，需要将kinit命令放入crontab以定期执行。
  2. 将`hive-site.xml/core-site.xml/hdfs-site.xml`放在`fe/conf`下，并将`core-site.xml/hdfs-site.xml`放在`be/conf`下。
  3. 在**$FE_HOME/conf/fe.conf**文件中的`JAVA_OPTS`选项的值中添加`-Djava.security.krb5.conf=/etc/krb5.conf`。**/etc/krb5.conf**是**krb5.conf**文件的保存路径。您可以根据您的操作系统更改路径。
4. 直接向**$BE_HOME/conf/be.conf**文件添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。**/etc/krb5.conf**是**krb5.conf**文件的保存路径。您可以根据您的操作系统更改路径。


5. 当您添加Hive资源时，必须将域名传递给`hive.metastore.uris`。此外，您还需要在**/etc/hosts**文件中添加Hive/HDFS域名和IP地址之间的映射。

* 配置支持AWS S3：向`fe/conf/core-site.xml`和`be/conf/core-site.xml`中添加以下配置。

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

   1. `fs.s3a.access.key`：AWS访问密钥ID。

   2. `fs.s3a.secret.key`：AWS秘钥。

   3. `fs.s3a.endpoint`：要连接的AWS S3端点。

   4. `fs.s3a.connection.maximum`：StarRocks到S3的并发连接的最大数量。如果在查询期间出现“等待连接超时”错误，可以将此参数设置为较大的值。

## (已弃用) Iceberg外部表

从v2.1.0开始，StarRocks允许您通过使用外部表从Apache Iceberg中查询数据。要查询Iceberg中的数据，您需要在StarRocks中创建Iceberg外部表。创建表时，您需要建立外部表与要查询的Iceberg表之间的映射。

### 开始之前

确保StarRocks有权限访问元数据服务（如Hive metastore）、文件系统（如HDFS）和Apache Iceberg使用的对象存储系统（例如Amazon S3和阿里云对象存储服务）。


### 注意事项

* StarRocks 2.3和更高版本中的Iceberg外部表仅支持查询以下类型的数据：

  * Iceberg v1表（分析数据表）。从v3.0版本开始支持ORC格式的Iceberg v2（基于行的删除）表，从v3.1版本开始支持Parquet格式的Iceberg v2表。有关Iceberg v1表和Iceberg v2表之间的区别，请参见 [Iceberg Table Spec](https://iceberg.apache.org/spec/)。

  * 以gzip（默认格式）、Zstd、LZ4或Snappy格式压缩的表。

  * 存储在Parquet或ORC格式的文件。

* StarRocks 2.3及更高版本中的Iceberg外部表支持同步Iceberg表的模式更改，而早于StarRocks 2.3版本的Iceberg外部表不支持。如果Iceberg表的模式发生更改，则必须删除相应的外部表并创建新表。

### 过程

#### 第1步：创建Iceberg资源

在创建Iceberg外部表之前，您必须在StarRocks中创建Iceberg资源。资源用于管理Iceberg访问信息。此外，还需要在用于创建外部表的语句中指定此资源。您可以根据业务需求创建资源：

* 如果从Hive metastore获取Iceberg表的元数据，则可以创建一个资源，并将目录类型设置为`HIVE`。

* 如果从其他服务获取Iceberg表的元数据，则需要创建自定义目录。然后创建资源，并将目录类型设置为`CUSTOM`。

##### 创建目录类型为`HIVE`的资源


例如，创建一个名为`iceberg0`的资源，并将目录类型设置为`HIVE`。

~~~SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES (
   "type" = "iceberg",
   "iceberg.catalog.type" = "HIVE",

   "iceberg.catalog.hive.metastore.uris" = "thrift://192.168.0.81:9083" 

);

~~~

下表描述了相关参数。

| **参数**                         | **描述**                               |
| ------------------------------- | ------------------------------------- |
| type                          | 资源类型。将值设置为`iceberg`。           |
| iceberg.catalog.type          | 资源的目录类型。支持Hive目录和自定义目录。如果指定Hive目录，请将值设置为`HIVE`。如果指定自定义目录，请将值设置为`CUSTOM`。 |
| iceberg.catalog.hive.metastore.uris | Hive metastore的URI。参数值的格式如下：`thrift://<Iceberg元数据的IP地址>:<端口号>`。端口号默认为9083。Apache Iceberg使用Hive目录访问Hive metastore，然后查询Iceberg表的元数据。 |


##### 创建目录类型为`CUSTOM`的资源

自定义目录需要继承抽象类BaseMetastoreCatalog，并且您需要实现IcebergCatalog接口。此外，自定义目录的类名不能与StarRock中已有类的名称重复。创建目录后，打包目录及其相关文件，并将其放置在每个前端（FE）的**fe/lib**路径下。然后重新启动每个FE。在完成上述操作后，可以创建一个目录类型为自定义目录的资源。

例如，创建一个名为`iceberg1`的资源，并将目录类型设置为`CUSTOM`。

~~~SQL

CREATE EXTERNAL RESOURCE "iceberg1" 

PROPERTIES (

   "type" = "iceberg",
   "iceberg.catalog.type" = "CUSTOM",
   "iceberg.catalog-impl" = "com.starrocks.IcebergCustomCatalog" 

);

~~~

下表描述了相关参数。

| **参数**          | **描述**                                              |

| ------------------ | ---------------------------------------------------- |

| type              | 资源类型。将值设置为`iceberg`。                         |

| iceberg.catalog.type | 资源的目录类型。支持Hive目录和自定义目录。如果指定Hive目录，请将值设置为`HIVE`。 如果指定自定义目录，请将值设置为`CUSTOM`。 |

| iceberg.catalog-impl | 自定义目录的完全限定类名。FE根据此名称搜索目录。如果目录包含自定义配置项，您必须在创建Iceberg外部表时将它们作为键值对添加到`PROPERTIES`参数中。 |

您可以在StarRocks 2.3及更高版本中修改Iceberg资源的`hive.metastore.uris`和`iceberg.catalog-impl`。有关更多信息，请参阅 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。


##### 查看Iceberg资源

~~~SQL

SHOW RESOURCES;

~~~

##### 删除Iceberg资源

例如，删除名为`iceberg0`的资源。

~~~SQL
DROP RESOURCE "iceberg0";
~~~

删除Iceberg资源会使所有引用此资源的外部表不可用。但是，Apache Iceberg中的相应数据不会被删除。如果您仍然需要查询Apache Iceberg中的数据，请创建一个新资源和新外部表。

#### 第2步：（可选）创建数据库

例如，在StarRocks中创建一个名为`iceberg_test`的数据库。

~~~SQL
CREATE DATABASE iceberg_test; 
USE iceberg_test; 
~~~

> 注意：StarRocks中数据库的名称可以与Apache Iceberg中数据库的名称不同。

#### 第3步：创建Iceberg外部表

> *The column names of the external table must be the same as those in the Iceberg table. The column order of the two tables can be different.*

外部表的列名必须与Iceberg表中的列名相同。两个表的列顺序可以不同。

如果您在自定义目录中定义了配置项，并希望在查询数据时配置项生效，可以在创建外部表时将配置项添加到`PROPERTIES`参数中，作为键值对。例如，如果您在自定义目录中定义了配置项`custom-catalog.properties`，则可以运行以下命令来创建外部表。

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

在创建外部表时，需要根据Iceberg表中列的数据类型指定外部表中列的数据类型。下表显示了列数据类型的映射。

| **Iceberg表** | **Iceberg外部表** |
| ------------- | ------------------ |
| BOOLEAN       | BOOLEAN            |
| INT           | TINYINT / SMALLINT / INT |
| LONG          | BIGINT             |
| FLOAT         | FLOAT              |
| DOUBLE        | DOUBLE             |
| DECIMAL(P, S) | DECIMAL            |
| DATE          | DATE / DATETIME    |
| TIME          | BIGINT             |
| TIMESTAMP     | DATETIME           |
| STRING        | STRING / VARCHAR   |

| UUID          | STRING / VARCHAR   |

| FIXED(L)      | CHAR               |

| BINARY        | VARCHAR            |

| LIST          | ARRAY              |

StarRocks不支持查询Iceberg数据的数据类型为TIMESTAMPTZ、STRUCT和MAP。

#### 步骤 4：查询Apache Iceberg中的数据

创建外部表后，您可以使用外部表查询Apache Iceberg中的数据。

~~~SQL

select count(*) from iceberg_tbl;

~~~

## (已弃用) Hudi外部表

从v2.2.0开始，StarRocks允许您使用Hudi外部表查询Hudi数据湖，从而实现快速数据湖分析。本主题介绍如何在StarRocks集群中创建Hudi外部表并使用Hudi外部表查询Hudi数据湖中的数据。

### 开始之前

确保您的StarRocks集群已被授予访问Hive元存储、HDFS集群或注册Hudi表的存储桶的权限。

### 注意事项

* Hudi的外部表只能用于查询，是只读的。

* StarRocks支持查询写时复制（COW）和合并读取（MOR，从v2.5开始支持）表（MOR表的差异见[表和查询类型](https://hudi.apache.org/docs/table_types/)）。Hudi支持的查询类型包括：快照查询和读优化查询（Hudi仅支持在合并读取表上执行读优化查询）。不支持增量查询。有关Hudi的查询类型的更多信息，请参见[表和查询类型](https://hudi.apache.org/docs/next/table_types/#query-types)。

* StarRocks支持Hudi文件的以下压缩格式：gzip、zstd、LZ4和Snappy。Hudi文件的默认压缩格式为gzip。

* StarRocks无法同步Hudi管理表中的架构更改。有关更多信息，请参见[架构演变](https://hudi.apache.org/docs/schema_evolution/)。如果更改了Hudi管理表的架构，则必须从StarRocks集群中删除关联的Hudi外部表，然后重新创建该外部表。

### 步骤

#### 步骤 1：创建和管理Hudi资源

您必须在StarRocks集群中创建Hudi资源。Hudi资源用于管理在StarRocks集群中创建的Hudi数据库和外部表。

##### 创建Hudi资源

执行以下语句以创建名为`hudi0`的Hudi资源：

~~~SQL
CREATE EXTERNAL RESOURCE "hudi0" 

PROPERTIES ( 

    "type" = "hudi", 

    "hive.metastore.uris" = "thrift://192.168.7.251:9083"

);
~~~


以下表描述了参数。

| 参数                   | 描述                               |

| ---------------------- | ---------------------------------- |
| type                   | Hudi资源的类型。将值设置为hudi。   |
| hive.metastore.uris    | Hudi资源连接的Hive元存储的Thrift URI。连接Hive资源后，您可以使用Hive来创建和管理Hudi表。Thrift URI的格式为`<Hive元存储的IP地址>:<Hive元存储的端口号>`。默认端口号为9083。 |

从v2.3开始，StarRocks允许更改Hudi资源的`hive.metastore.uris`值。有关更多信息，请参见[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。

##### 查看Hudi资源

执行以下语句以查看在您的StarRocks集群中创建的所有Hudi资源：

~~~SQL

SHOW RESOURCES;
~~~

##### 删除Hudi资源

执行以下语句以删除名为`hudi0`的Hudi资源：

~~~SQL

DROP RESOURCE "hudi0";

~~~

> 注意：
>
> 删除Hudi资源会导致在该Hudi资源创建的所有Hudi外部表不可用。但是，这不会影响您存储在Hudi中的数据。如果仍要使用StarRocks查询Hudi中的数据，则必须重新在StarRocks集群中创建Hudi资源、Hudi数据库和Hudi外部表。

#### 步骤 2：创建Hudi数据库

执行以下语句以在StarRocks集群中创建并打开名为`hudi_test`的Hudi数据库：

~~~SQL
CREATE DATABASE hudi_test; 
USE hudi_test; 

~~~

> 注意：
>
> 在StarRocks集群中指定的Hudi数据库的名称不需要与Hudi中的关联数据库相同。

#### 步骤 3：创建Hudi外部表


执行以下语句以在名为`hudi_test`的Hudi数据库中创建名为`hudi_tbl`的Hudi外部表：

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

以下表描述了参数。

| 参数     | 描述                                             |
| ---------| ------------------------------------------------- |
| ENGINE   | Hudi外部表的查询引擎。将值设置为`HUDI`。           |
| resource | 您StarRocks集群中的Hudi资源的名称。               |

| database | Hudi外部表在StarRocks集群中所属的Hudi数据库的名称。|
| table    | Hudi外部表关联的Hudi管理表。                     |


> 注意：

>

> * 在Hudi外部表中指定的名称不需要与关联的Hudi管理表中的列名相同，但列的顺序可以不同。
>
> * 您可以从关联的Hudi管理表中选择一些或全部列，并在Hudi外部表中仅创建所选列。以下表列出了Hudi和StarRocks支持的数据类型之间的映射。

| Hudi支持的数据类型   | StarRocks支持的数据类型    |

在星型模式中，数据通常被划分为维度表和事实表。维度表的数据量较小，但涉及到更新操作。目前，StarRocks 不支持直接的更新操作（可以通过使用唯一键表来实现更新）。在某些场景中，你可以将维度表存储在 MySQL 中以进行直接数据读取。

要查询 MySQL 数据，你必须在 StarRocks 中创建一个外部表，并将其映射到你的 MySQL 数据库中的表。创建表时，你需要指定 MySQL 连接信息。

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

* **host**: MySQL 数据库的连接地址
* **port**: MySQL 数据库的端口号
* **user**: 登录 MySQL 的用户名
* **password**: 登录 MySQL 的密码
* **database**: MySQL 数据库的名称
* **table**: MySQL 数据库中表的名称