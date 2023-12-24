---
displayed_sidebar: English
---

# HTTP SQL API

## 描述

StarRocks v3.2.0 引入了 HTTP SQL API，允许用户使用 HTTP 执行各种类型的查询。目前，该 API 支持 SELECT、SHOW、EXPLAIN 和 KILL 语句。

使用 curl 命令的语法：

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

| 字段                    | 描述                                                  |
| ------------------------ | :----------------------------------------------------------- |
|  fe_ip                   | FE 节点 IP 地址。                                                  |
|  fe_http_port            | FE HTTP 端口。                                           |
|  catalog_name            | 目录名称。目前，该 API 仅支持查询内部表，这意味着 `<catalog_name>` 只能设置为 `default_catalog`. |
|  database_name           | 数据库名称。如果在请求行中未指定数据库名称，并且在 SQL 查询中使用了表名，必须在表名前加上其数据库名称，例如 `database_name.table_name`. |

- 在指定目录中跨数据库查询数据。如果 SQL 查询中使用了表，必须在表名前加上其数据库名称。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- 从指定目录和数据库中查询数据。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 身份验证方法

```shell
Authorization: Basic <credentials>
```

使用基本身份验证，即输入用户名和密码为 `credentials` (`-u '<username>:<password>'`)。如果用户名没有设置密码，只需传入 `<username>:` 并将密码留空。例如，如果使用 root 账号，可以输入 `-u 'root:'`。

### 请求正文

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| 字段                    | 描述                                                  |
| ------------------------ | :----------------------------------------------------------- |
| query                    | SQL 查询，采用 STRING 格式。仅支持 SELECT、SHOW、EXPLAIN 和 KILL 语句。每个 HTTP 请求只能运行一个 SQL 查询。 |
| sessionVariables         | 要为查询设置的[会话变量](../System_variable.md)，采用 JSON 格式。此字段是可选的，默认为空。您设置的会话变量对同一连接生效，在连接关闭时失效。|

### 请求标头

```shell
--header "Content-Type: application/json"
```

此标头表示请求正文为 JSON 字符串。

## 响应消息

### 状态码

- 200：HTTP 请求成功，并在数据发送到客户端之前服务器正常。
- 4xx：HTTP 请求错误，表示客户端错误。
- `500 Internal Server Error`：HTTP 请求成功，但在数据发送到客户端之前服务器遇到错误。
- 503：HTTP 请求成功，但 FE 无法提供服务。

### 响应标头

`content-type` 表示响应正文的格式。使用换行分隔的 JSON，即响应正文由多个 JSON 对象组成，这些对象由 `\n` 分隔。

|                      | 描述                                                  |
| -------------------- | :----------------------------------------------------------- |
| content-type         | 格式为换行分隔的 JSON，默认为 "application/x-ndjson charset=UTF-8"。 |
| X-StarRocks-Query-Id | 查询 ID。                                                          |

### 响应正文

#### 请求发送前失败

客户端请求失败，或服务器在返回数据给客户端之前遇到错误。响应正文采用以下格式，其中 `msg` 是错误信息。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### 请求发送后失败

部分结果返回，HTTP 状态码为 200。数据发送暂停，连接关闭，并记录错误。

#### 成功

响应消息中的每一行都是一个 JSON 对象。JSON 对象由 `\n` 分隔。

- 对于 SELECT 语句，将返回以下 JSON 对象。

| 对象       | 描述                                                  |
| ------------ | :----------------------------------------------------------- |
| `connectionId` | 连接 ID。您可以通过调用 KILL 取消长时间挂起的查询 `<connectionId>`。 |
| `meta`        | 表示一列。键为 `meta`，值为 JSON 数组，数组中的每个对象表示一列。 |
| `data`         | 数据行，键为 `data`，值为包含一行数据的 JSON 数组。 |
| `statistics`   | 查询的统计信息。                                       |

- 对于 SHOW 语句，返回 `meta`、 `data` 和 `statistics`。
- 对于 EXPLAIN 语句，将返回一个 `explain` 对象，显示查询的详细执行计划。

以下示例使用 `\n` 作为分隔符。StarRocks 使用 HTTP 分块模式传输数据。FE 每次获取数据块时，都会将数据块流式传输到客户端。客户端可以按行解析数据，无需数据缓存，也无需等待整个数据，从而减少了客户端的内存消耗。

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

## 例子

### 运行 SELECT 查询

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

若要取消运行时间过长的查询，可以关闭连接。StarRocks 在检测到连接关闭时会取消该查询。

您也可以调用 KILL `connectionId` 来取消此查询。例如：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

您可以从响应正文中获取 `connectionId`，或通过调用 SHOW PROCESSLIST 获取。例如：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### 运行包含为此查询设置的会话变量的查询

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:'  -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}'  --header "Content-Type: application/json"
```
