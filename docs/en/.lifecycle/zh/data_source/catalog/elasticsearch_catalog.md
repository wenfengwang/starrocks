---
displayed_sidebar: English
---

# Elasticsearch 目录

从 v3.1 开始，StarRocks 支持 Elasticsearch 目录。

StarRocks 和 Elasticsearch 都是受欢迎的分析系统，各自具有独特的优势。StarRocks 擅长大规模分布式计算，并支持通过外部表查询 Elasticsearch 中的数据。Elasticsearch 以其全文搜索功能而闻名。StarRocks 和 Elasticsearch 的结合提供了更全面的 OLAP 解决方案。使用 Elasticsearch 目录，您可以在 StarRocks 上使用 SQL 语句直接分析 Elasticsearch 集群中的所有索引数据，而无需进行数据迁移。

与其他数据源的目录不同，Elasticsearch 目录在创建时只有一个名为 `default` 的数据库。每个 Elasticsearch 索引都会自动映射到数据表并挂载到 `default` 数据库中。

## 创建 Elasticsearch 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### `catalog_name`

Elasticsearch 目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### `comment`

Elasticsearch 目录的描述。此参数是可选的。

#### 属性

Elasticsearch 目录的属性。下表描述了 Elasticsearch 目录支持的属性。

| 参数                   | 必填 | 默认值 | 描述                                                  |
| --------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| 主机                       | 是      | 无          | Elasticsearch 集群的连接地址。可以指定一个或多个地址。StarRocks 可以从该地址解析 Elasticsearch 版本和索引分片分配。StarRocks 根据 `GET /_nodes/http` API 返回的地址与您的 Elasticsearch 集群进行通信。因此，`hosts` 参数的值必须与 `GET /_nodes/http` API 操作返回的地址相同。否则，BE 可能无法与您的 Elasticsearch 集群通信。 |
| 类型                        | 是      | 无          | 数据源的类型。在创建 Elasticsearch 目录时，将此参数设置为 `es`。|
| 用户                        | 否       | 空         | 用于登录启用了 HTTP 基本认证的 Elasticsearch 集群的用户名。确保您有权限访问路径，例如 `/cluster/state/ nodes/http`，并具有读取索引的权限。 |
| 密码                    | 否       | 空         | 用于登录 Elasticsearch 集群的密码。 |
| es.type                     | 否       | _doc          | 索引的类型。如果您需要查询 Elasticsearch 8 及更高版本的数据，则无需配置此参数，因为 Elasticsearch 8 及更高版本已移除了映射类型。 |
| es.nodes.wan.only           | 否       | FALSE         | 指定 StarRocks 是否仅使用 `hosts` 指定的地址访问 Elasticsearch 集群并获取数据。<ul><li>`true`：StarRocks 仅使用 `hosts` 指定的地址访问 Elasticsearch 集群并获取数据，不会嗅探 Elasticsearch 索引分片所在的数据节点。如果 StarRocks 无法访问 Elasticsearch 集群内数据节点的地址，需要将此参数设置为 `true`。</li><li>`false`：StarRocks 使用 `hosts` 指定的地址对 Elasticsearch 集群索引的分片所在的数据节点进行嗅探。StarRocks 生成查询执行计划后，BE 直接访问 Elasticsearch 集群内的数据节点，从索引分片中获取数据。如果 StarRocks 可以访问 Elasticsearch 集群内数据节点的地址，建议保留默认值 `false`。</li></ul> |
| [](http://es.net)es.net.ssl | 否       | FALSE         | 指定是否允许使用 HTTPS 协议访问 Elasticsearch 集群。仅 StarRocks v2.4 及更高版本支持配置此参数。<ul><li>`true`：可以使用 HTTPS 和 HTTP 协议访问 Elasticsearch 集群。</li><li>`false`：只能使用 HTTP 协议访问 Elasticsearch 集群。</li></ul> |
| enable_docvalue_scan        | 否       | TRUE          | 指定是否从 Elasticsearch 列式存储中获取目标字段的值。在大多数情况下，从列式存储读取数据优于从行存储读取数据。 |
| enable_keyword_sniff        | 否       | TRUE          | 指定是否根据 KEYWORD 类型的字段在 Elasticsearch 中嗅探 TEXT 类型字段。如果将此参数设置为 `false`，StarRocks 将在标记化后进行匹配。 |

### 示例

以下示例创建一个名为 `es_test` 的 Elasticsearch 目录：

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

StarRocks 支持将查询中指定的 Elasticsearch 表中指定的谓词推送到 Elasticsearch 执行。这样可以最大程度地缩短查询引擎与存储源之间的距离，并提高查询性能。下表列出了可以下推到 Elasticsearch 的运算符。

| SQL 语法   | Elasticsearch 语法  |
| ------------ | --------------------- |
| `=`            | term 查询            |
| `in`           | terms 查询           |
| `>=, <=, >, <` | 范围                 |
| `and`          | bool.filter           |
| `or`           | bool.should           |
| `not`          | bool.must_not         |
| `not in`       | bool.must_not + terms |
| `esquery`      | ES 查询 DSL          |

## 查询示例

`esquery()` 函数可用于将无法用 SQL 表示的 Elasticsearch 查询（例如匹配查询和 geoshape 查询）推送到 Elasticsearch 进行过滤和处理。在 `esquery()` 函数中，指定列名的第一个参数用于与索引关联，第二个参数是 Elasticsearch 查询的基于 DSL 的 JSON 表示形式，用大括号 `{}` 括起来。JSON 表示形式可以且必须只有一个根键，例如 `match`、 `geo_shape` 或 `bool`。

- 匹配查询

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

- 从 v5.x 开始，Elasticsearch 采用了不同的数据扫描方法。StarRocks 仅支持查询 Elasticsearch v5.x 及更高版本的数据。
- StarRocks 仅支持从启用了 HTTP 基本认证的 Elasticsearch 集群中查询数据。
- 一些查询（例如涉及 `count()` 的查询）在 StarRocks 上运行速度比在 Elasticsearch 上慢得多，因为 Elasticsearch 可以直接读取与查询条件匹配的指定数量的文档相关的元数据，而无需过滤请求的数据。
