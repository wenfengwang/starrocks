---
displayed_sidebar: "Chinese"
---

# HTTP SQL API

## 描述

StarRocks v3.2.0引入了HTTP SQL API，用户可以使用HTTP执行各种类型的查询。目前，该API支持SELECT、SHOW、EXPLAIN和KILL语句。

使用curl命令的语法：

```shell
curl -X POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql' \
   -u '<username>:<password>'  -d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}' \
   --header "Content-Type: application/json"
```

## 请求消息

### 请求行

```shell
POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql'
```

| 字段           | 描述                                                      |
| -------------- | :---------------------------------------------------------- |
| fe_ip          | FE节点IP地址。                                                    |
| fe_http_port   | FE HTTP端口。                                            |
| catalog_name   | 目录名称。当前，该API仅支持查询内部表，这意味着`<catalog_name>`只能设置为`default_catalog`。 |
| database_name  | 数据库名称。如果请求行中未指定数据库名称并且在SQL查询中使用了表名称，则必须使用其数据库名称作为表名称的前缀，例如`database_name.table_name`。 |

- 在指定目录中的多个数据库中查询数据。如果SQL查询中使用了表，则必须使用其数据库名称作为表名的前缀。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- 从指定目录和数据库中查询数据。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 鉴权方式

```shell
Authorization: Basic <credentials>
```

使用基本身份验证，即输入`credentials`的用户名和密码（`-u '<username>:<password>'`）。如果未对用户名设置密码，可以只传入`<username>:`，并将密码留空。例如，如果使用root账户，可以输入`-u 'root:'`。

### 请求体

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| 字段              | 描述                                                      |
| ----------------- | :---------------------------------------------------------- |
| query             | 以字符串格式的SQL查询。仅支持SELECT、SHOW、EXPLAIN和KILL语句。可以为一个HTTP请求运行一个SQL查询。 |
| sessionVariables  | 以JSON格式设置的[会话变量](../System_variable.md)，此字段为可选。默认为空。您设置的会话变量对于同一连接生效，在连接关闭时失效。 |

### 请求头

```shell
--header "Content-Type: application/json"
```

此标头表示请求体为JSON字符串。

## 响应消息

### 状态码

- 200：HTTP请求成功，服务器在将数据发送到客户端之前处于正常状态。
- 4xx：HTTP请求错误，表示客户端错误。
- `500 Internal Server Error`：HTTP请求成功，但服务器在将数据发送到客户端之前遇到错误。
- 503：HTTP请求成功，但FE无法提供服务。

### 响应头

`content-type`表示响应体的格式。使用换行分隔的JSON（Newline delimited JSON），这意味着响应体由多个以`\n`分隔的JSON对象组成。

|                | 描述                                                      |
| -------------- | :---------------------------------------------------------- |
| content-type   | 格式为换行分隔的JSON，默认为“application/x-ndjson charset=UTF-8”。 |
| X-StarRocks-Query-Id | 查询ID。                                                    |

### 响应体

#### 请求发送前失败

请求在客户端端失败或服务器在返回数据给客户端前遇到错误。响应体格式如下，其中`msg`为错误信息。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### 请求发送后失败

返回部分结果且HTTP状态码为200。数据发送被暂停，连接关闭，并记录错误。

#### 请求成功

响应消息中的每行都是一个JSON对象。JSON对象通过`\n`分隔。

- 对于SELECT语句，返回以下JSON对象。

| 对象              | 描述                                                      |
| ---------------- | :---------------------------------------------------------- |
| `connectionId`   | 连接ID。您可以通过取消`<connectionId>`来取消长时间挂起的查询。 |
| `meta`           | 代表列。键为`meta`，值为JSON数组，数组中的每个对象代表一列。 |
| `data`           | 数据行，键为`data`，值为包含数据行的JSON数组。 |
| `statistics`     | 查询的统计信息。                                                |

- 对于SHOW语句，返回`meta`、`data`和`statistics`。
- 对于EXPLAIN语句，返回一个`explain`对象以显示查询的详细执行计划。

以下示例使用`\n`作为分隔符。StarRocks使用HTTP分块模式传输数据。每次FE获取数据块时，都将数据块流式传输到客户端。客户端可以按行解析数据，从而无需进行数据缓存并且无需等待所有数据，降低了客户端的内存消耗。

```json
{"connectionId": 7}\n
{"meta": [
    {
      "name": "stock_symbol",
      "type": "varchar"
    },
    {
      "name": "closing_price",
      "type": "decimal64(8, 2)"
    },
    {
      "name": "closing_date",
      "type": "datetime"
    }
  ]}\n
{"data": ["JDR", 12.86, "2014-10-02 00:00:00"]}\n
{"data": ["JDR",14.8, "2014-10-10 00:00:00"]}\n
...
{"statistics": {"scanRows": 0,"scanBytes": 0,"returnRows": 9}}
```

## 示例

### 运行SELECT查询

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "select * from agg;"}' --header "Content-Type: application/json"
```

结果：

```json
{"connectionId":49}
{"meta":[{"name":"no","type":"int(11)"},{"name":"k","type":"decimal64(10, 2)"},{"name":"v","type":"decimal64(10, 2)"}]}
{"data":[1,"10.00",null]}
{"data":[2,"10.00","11.00"]}
{"data":[2,"20.00","22.00"]}
{"data":[2,"25.00",null]}
{"data":[2,"30.00","35.00"]}
{"statistics":{"scanRows":0,"scanBytes":0,"returnRows":5}}
```

### 取消查询

要取消长时间运行的查询，可以关闭连接。StarRocks将在检测到连接关闭时取消该查询。

您还可以调用KILL `<connectionId>`来取消此查询。例如：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

您可以通过响应体或调用SHOW PROCESSLIST来获得`connectionId`。例如：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### 运行带有为该查询设置的会话变量的查询

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:'  -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}'  --header "Content-Type: application/json"
```