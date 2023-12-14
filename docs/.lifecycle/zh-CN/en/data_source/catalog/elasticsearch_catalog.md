---
displayed_sidebar: "Chinese"
---

# Elasticsearch目录

StarRocks支持从v3.1开始使用Elasticsearch目录。

StarRocks和Elasticsearch都是受欢迎的分析系统，各有其独特的优势。StarRocks在大规模分布式计算方面表现出色，并支持通过外部表查询Elasticsearch中的数据。Elasticsearch以其全文搜索功能而闻名。StarRocks和Elasticsearch的结合提供了更全面的OLAP解决方案。使用Elasticsearch目录，您可以直接通过StarRocks上的SQL语句分析Elasticsearch集群中的所有索引数据，无需进行数据迁移。

与其他数据源的目录不同，创建Elasticsearch目录时该目录中只有一个名为`default`的数据库。每个Elasticsearch索引会自动映射到一个数据表并挂载到`default`数据库中。

## 创建Elasticsearch目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### `catalog_name`

Elasticsearch目录的名称。名称命名规范如下：

- 名称可包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，且不能超过1023个字符。

#### `comment`

Elasticsearch目录的描述。此参数为可选参数。

#### PROPERTIES

Elasticsearch目录的属性。以下表格描述了Elasticsearch目录支持的属性。

| 参数                        | 必填     | 默认值         | 描述                                                     |
| --------------------------- | -------- | ------------- | --------------------------------------------------------- |
| hosts                       | 是       | 无            | Elasticsearch集群的连接地址。您可以指定一个或多个地址。StarRocks可以从该地址解析Elasticsearch版本和索引分片分配情况。StarRocks根据`GET /_nodes/http` API操作返回的地址与您的Elasticsearch集群进行通信。因此，`hosts`参数的值必须与`GET /_nodes/http` API操作返回的地址相同。否则，BEs可能无法与您的Elasticsearch集群通信。 |
| type                        | 是       | 无            | 数据源的类型。在创建Elasticsearch目录时，将此参数设置为`es`。 |
| user                        | 否       | 空            | 用于登录已启用HTTP基本认证的Elasticsearch集群的用户名。请确保您具有访问路径（如`/cluster/state/ nodes/http`）权限并具有读取索引的权限。 |
| password                    | 否       | 空           | 用于登录到Elasticsearch集群的密码。 |
| es.type                     | 否       | _doc          | 索引的类型。如果要查询Elasticsearch 8及更高版本中的数据，则无需配置此参数，因为Elasticsearch 8及更高版本已移除映射类型。 |
| es.nodes.wan.only           | 否       | FALSE         | 指示StarRocks是否仅使用`hosts`指定的地址访问Elasticsearch集群并提取数据。<ul><li>`true`:StarRocks仅使用`hosts`指定的地址访问Elasticsearch集群并提取数据，并不会获取Elasticsearch索引分片所在的数据节点。如果StarRocks无法访问Elasticsearch集群内的数据节点地址，则需要将此参数设置为`true`。</li><li>`false`:StarRocks使用`hosts`指定的地址获取Elasticsearch集群内的数据节点地址。在StarRocks生成查询执行计划后，BEs直接访问Elasticsearch集群内的数据节点，以从索引的分片中获取数据。如果StarRocks能够访问Elasticsearch集群内数据节点的地址，则建议保留默认值`false`。</li></ul> |
| [es.net](http://es.net).ssl | 否       | FALSE         | 指示是否可以使用HTTPS协议访问Elasticsearch集群。仅StarRocks v2.4及更高版本支持配置此参数。<ul><li>`true`:可以同时使用HTTPS和HTTP协议访问您的Elasticsearch集群。</li><li>`false`:只能使用HTTP协议访问您的Elasticsearch集群。</li></ul> |
| enable_docvalue_scan        | 否       | TRUE          | 指示是否从Elasticsearch列式存储中获取目标字段的值。在大多数情况下，从列式存储读取数据的性能优于从行式存储读取数据。 |
| enable_keyword_sniff        | 否       | TRUE          | 指示是否根据KEYWORD类型字段在Elasticsearch中嗅探TEXT类型字段。如果此参数设置为`false`，StarRocks将在标记化后执行匹配。 |

### 示例

以下示例创建了名为`es_test`的Elasticsearch目录：

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

StarRocks支持将查询中指定的谓词推送至Elasticsearch以进行执行。这样可以最大限度地减少查询引擎和存储源之间的距离，并提高查询性能。以下表格列出了可以推送至Elasticsearch的运算符。

| SQL语法        | Elasticsearch语法  |
| -------------- | ------------------- |
| `=`            | 术语查询            |
| `in`           | 术语查询            |
| `>=, <=, >, <` | 范围               |
| `and`          | 布尔过滤           |
| `or`           | 布尔 应              |
| `not`          | 布尔必不            |
| `not in`       | 布尔必不+术语        |
| `esquery`      | ES查询DSL           |

## 查询示例

`esquery()`函数可用于推送Elasticsearch查询（如不能在SQL中表示的匹配查询和地理形状查询）至Elasticsearch以进行过滤和处理。在`esquery()`函数中，指定列名的第一个参数用于与索引相关联，第二个参数是基于Elasticsearch查询DSL的JSON表示，用大括号（`{}`）括起来。JSON表示只能且必须有一个根键，如`match`、`geo_shape`或`bool`。

- 匹配查询

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
     }
  }');
  ```

- 地理形状查询

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

- 布尔查询

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

## 使用注意事项

- 从v5.x开始，Elasticsearch采用了不同的数据扫描方法。StarRocks仅支持从Elasticsearch v5.x及更高版本查询数据。
- StarRocks仅支持从启用HTTP基本认证的Elasticsearch集群查询数据。
- 一些查询（例如涉及`count()`的查询）在StarRocks上运行速度比在Elasticsearch上慢得多，因为Elasticsearch可以直接读取符合查询条件的文档数量相关的元数据，无需过滤请求的数据。