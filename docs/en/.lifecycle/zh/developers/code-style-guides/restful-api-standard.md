---
displayed_sidebar: English
---

# StarRocks Restful API 标准

## API 格式

1. API 格式遵循模式：`/api/{version}/{target-object-access-path}/{action}`。
2. `{version}` 表示为 `v{number}`，例如 v1、v2、v3、v4 等。
3. `{target-object-access-path}` 以分层方式组织，将在后文详细解释。
4. `{action}` 是可选的，API 实现者应尽可能通过 HTTP 方法传达操作含义。仅当 HTTP 方法的语义无法满足时，才使用 action。例如，没有可用于重命名对象的 HTTP 方法时。

## 目标对象访问路径定义

1. REST API 访问的目标对象需要分类并组织成分层访问路径。访问路径格式如下：
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

以目录、数据库、表、列为例：
```
/catalogs：代表所有目录。
/catalogs/hive：代表目录类别下名为 "hive" 的特定目录对象。
/catalogs/hive/databases：代表 "hive" 目录中的所有数据库。
/catalogs/hive/databases/tpch_100g：代表 "hive" 目录中名为 "tpch_100g" 的数据库。
/catalogs/hive/databases/tpch_100g/tables：代表 "tpch_100g" 数据库中的所有表。
/catalogs/hive/databases/tpch_100g/tables/lineitem：代表 tpch_100g.lineitem 表。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns：代表 tpch_100g.lineitem 表中的所有列。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey：代表 tpch_100g.lineitem 表中的特定列 l_orderkey。
```

2. 类别使用蛇形命名，最后一个词为复数形式。所有词均为小写，多个词之间用下划线（_）连接。特定对象按其实际名称命名。目标对象的层次关系需明确定义。

## HTTP 方法选择

1. GET：使用 GET 方法来展示单个对象和列出某类别的所有对象。GET 方法对对象的访问是只读的，不提供请求体。
```
# 列出数据库 ssb_100g 中的所有表
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# 展示表 ssb_100g.lineorder
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST：用于创建对象。参数通过请求体传递。它不是幂等的。如果对象已存在，重复创建将失败并返回错误消息。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT：用于创建对象。参数通过请求体传递。它是幂等的。如果对象已存在，将返回成功。PUT 方法是 POST 方法的 CREATE IF NOT EXISTS 版本。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE：用于删除对象。不提供请求体。如果要删除的对象不存在，将返回成功。DELETE 方法具有 DROP IF EXISTS 语义。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH：用于更新对象。提供请求体，仅包含需要修改的部分信息。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 身份验证和授权

1. 身份验证和授权信息通过 HTTP 请求头传递。

## HTTP 状态码

1. REST API 返回 HTTP 状态码以指示操作的成功或失败。
2. 成功操作的状态码（2xx）如下：

- 200 OK：表示请求已成功处理。用于查看/列出/删除/更新对象和查询待处理任务状态。
- 201 Created：表示对象已成功创建。用于 PUT/POST 方法。响应体必须包含对象 URI，以供后续查看/列出/删除/更新。
- 202 Accepted：表示任务提交成功，任务处于待处理状态。响应体必须包含任务 URI，以供后续取消、删除和查询任务状态。

3. 客户端错误的错误码（4xx）。用户需调整 HTTP 请求后重试。
- 400 Bad Request：请求参数无效。
- 401 Unauthorized：缺少认证信息、认证信息非法或认证失败。
- 403 Forbidden：认证成功，但用户操作未通过授权检查。无访问权限。
- 404 Not Found：API URI 编码错误，非注册 REST API。
- 405 Method Not Allowed：使用了错误的 HTTP 方法。
- 406 Not Acceptable：响应格式与 Accept 头指定的媒体类型不匹配。
- 415 Unsupported Media Type：请求内容的媒体类型与 Content-Type 头指定的媒体类型不匹配。

4. 服务器错误的错误码（5xx）。用户无需修改请求，可稍后重试。
- 500 Internal Server Error：服务器内部错误，类似于未知错误。
- 503 Service Unavailable：服务暂不可用。例如，用户访问频率过高，达到速率限制；或服务因内部状态无法提供服务，如创建3副本的表但仅2个 BE 可用；或用户查询涉及的所有 Tablet 副本不可用。

## HTTP 响应格式

1. 当 API 返回 HTTP 状态码 200/201/202 时，HTTP 响应非空。API 以 JSON 格式返回结果，包括顶级字段 "code"、"message" 和 "result"。所有 JSON 字段均采用驼峰命名。

2. 成功的 API 响应中，"code" 为 "0"，"message" 为 "OK"，"result" 包含实际结果。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 失败的 API 响应中，"code" 非 "0"，"message" 为简洁的错误消息，"result" 可包含详细错误信息，如错误堆栈跟踪。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## 参数传递

1. API 参数按路径、请求体、查询参数和头部的优先级顺序传递。选择合适的参数传递方式。

2. 路径参数：必需的参数，代表对象层级关系，放在路径参数中。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. 请求体：参数通过 application/json 传递。参数可以是必需或可选类型。

4. 查询参数：不允许同时使用查询参数和请求体参数。对于同一 API，选择其一。如果除头部参数和路径参数外的参数数量不超过2个，可使用查询参数；否则，应使用请求体传递参数。

5. HEADER 参数：应使用 Header 传递 HTTP 标准参数，如 Content-Type 和 Accept 等放在 Header 中，实现者不应滥用 HTTP Header 传递自定义参数。使用 Header 传递用户扩展参数时，Header 名称应采用 `x-starrocks-{name}` 格式，其中 name 可包含多个英文单词，每个单词小写并通过连字符（-）连接。