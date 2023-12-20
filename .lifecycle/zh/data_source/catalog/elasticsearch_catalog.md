---
displayed_sidebar: English
---

# Elasticsearch 目录

StarRocks 从 v3.1 版本开始支持 Elasticsearch 目录。

StarRocks 和 Elasticsearch 均为广受欢迎的分析系统，各具特色。StarRocks 擅长大规模分布式计算，并支持通过外部表查询 Elasticsearch 中的数据。Elasticsearch 以其全文搜索能力而著称。StarRocks 与 Elasticsearch 的结合提供了更加全面的 OLAP 解决方案。通过 Elasticsearch 目录，您可以直接在 StarRocks 上使用 SQL 语句分析 Elasticsearch 集群中的所有已索引数据，无需数据迁移。

与其他数据源的目录不同，创建 Elasticsearch 目录时，其中只包含一个名为 default 的数据库。每个 Elasticsearch 索引都会自动映射为一个数据表，并挂载到 default 数据库中。

## 创建 Elasticsearch 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### catalog_name

Elasticsearch 目录的名称。命名规则如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不得超过 1023 个字符。

#### comment

Elasticsearch 目录的描述。此参数为可选。

#### PROPERTIES

Elasticsearch 目录的属性。下表描述了 Elasticsearch 目录支持的属性。

|参数|必填|默认值|说明|
|---|---|---|---|
|hosts|是|无|Elasticsearch 集群的连接地址。您可以指定一个或多个地址。 StarRocks 可以从此地址解析 Elasticsearch 版本和索引分片分配。 StarRocks 根据 GET /_nodes/http API 操作返回的地址与您的 Elasticsearch 集群进行通信。因此，hosts参数的值必须与GET /_nodes/http API操作返回的地址相同。否则，BE 可能无法与您的 Elasticsearch 集群通信。|
|type|Yes|None|数据源的类型。创建Elasticsearch目录时，将此参数设置为es。|
|user|否|空|用于登录启用HTTP基本身份验证的Elasticsearch集群的用户名。确保您有权访问 /cluster/state/nodes/http 等路径，并有权读取索引。|
|password|否|空|用于登录Elasticsearch集群的密码。|
|es.type|否|_doc|索引的类型。如果您想在Elasticsearch 8及更高版本中查询数据，则不需要配置该参数，因为Elasticsearch 8及更高版本中已经删除了映射类型。|
|es.nodes.wan.only|No|FALSE|指定StarRocks是否仅使用主机指定的地址访问Elasticsearch集群并获取数据。true：StarRocks仅使用主机指定的地址访问Elasticsearch集群并获取数据并且不会嗅探Elasticsearch索引分片所在的数据节点。如果StarRocks无法访问Elasticsearch集群内部数据节点的地址，则需要将此参数设置为true。false：StarRocks使用主机指定的地址来嗅探Elasticsearch集群索引分片所在的数据节点。 StarRocks生成查询执行计划后，BE直接访问Elasticsearch集群内的数据节点，从索引分片中获取数据。如果StarRocks可以访问Elasticsearch集群内数据节点的地址，建议您保留默认值false。|
|es.net.ssl|否|FALSE|指定是否可以使用HTTPS协议访问Elasticsearch集群。仅StarRocks v2.4及以上版本支持配置此参数。true：HTTPS和HTTP协议均可用于访问Elasticsearch集群。false：仅可使用HTTP协议访问Elasticsearch集群。|
|enable_docvalue_scan|No|TRUE|指定是否从Elasticsearch列式存储中获取目标字段的值。在大多数情况下，从列式存储读取数据的性能优于从行存储读取数据。|
|enable_keyword_sniff|No|TRUE|指定是否根据KEYWORD类型字段嗅探Elasticsearch中的TEXT类型字段。如果此参数设置为 false，StarRocks 将在标记化后执行匹配。|

### 示例

以下示例展示了如何创建一个名为 es_test 的 Elasticsearch 目录：

```SQL
CREATE EXTERNAL CATALOG es_test
COMMENT 'test123'
PROPERTIES
(
    "type" = "es",
    "es.type" = "_doc",
    "hosts" = "https://xxx:9200",
    "es.net.ssl" = "true",
    "user" = "admin",
    "password" = "xxx",
    "es.nodes.wan.only" = "true"
);
```

## 谓词下推

StarRocks 支持将针对 Elasticsearch 表的查询中指定的谓词下推到 Elasticsearch 执行。这样可以最大限度地缩短查询引擎与存储源之间的距离，提高查询性能。下表列出了可以下推到 Elasticsearch 的运算符。

|SQL 语法|Elasticsearch 语法|
|---|---|
|=|词条查询|
|在|条款查询|
|>=、<=、>、<|范围|
|和|bool.filter|
|或|bool.should|
|not|bool.must_not|
|not in|bool.must_not + terms|
|esquery|ES 查询 DSL|

## 查询示例

esquery() 函数可以用于将无法用 SQL 表达的 Elasticsearch 查询，如 match 和 geoshape 查询，下推到 Elasticsearch 进行过滤和处理。在 esquery() 函数中，第一个参数指定列名，用于与索引关联，第二个参数是一个用大括号 ({}) 括起来的 Elasticsearch 查询的基于 Elasticsearch Query DSL 的 JSON 表示形式。这个 JSON 表示形式可以且必须只有一个根键，如 match、geo_shape 或 bool。

- Match 查询

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
     }
  }');
  ```

- Geoshape 查询

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
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

- Boolean 查询

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, ' {
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

## 使用说明

- 从 v5.x 版本开始，Elasticsearch 采用了不同的数据扫描方法。StarRocks 仅支持查询 Elasticsearch v5.x 及更高版本的数据。
- StarRocks 仅支持查询启用了 HTTP 基本认证的 Elasticsearch 集群中的数据。
- 一些查询，如涉及 count() 的查询，在 StarRocks 上的执行速度可能比在 Elasticsearch 上慢，因为 Elasticsearch 可以直接读取与满足查询条件的指定文档数量相关的元数据，而无需过滤请求的数据。
