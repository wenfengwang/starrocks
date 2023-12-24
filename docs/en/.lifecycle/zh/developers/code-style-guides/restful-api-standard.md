---
displayed_sidebar: English
---

# StarRocks Restful API 标准

## API 格式

1. API 格式遵循以下模式：`/api/{version}/{target-object-access-path}/{action}`。 
2. `{version}` 表示为 `v{number}`，例如 v1、v2、v3、v4 等。 
3. `{target-object-access-path}` 是按层次结构组织的，稍后将详细解释。 
4. `{action}` 是可选的，API 实现者应尽可能利用 HTTP 方法来传达操作的含义。只有当 HTTP 方法的语义无法满足时，才应使用该操作。例如，如果没有可用的 HTTP 方法来重命名对象。

## 目标对象访问路径的定义

1. REST API 访问的目标对象需要进行分类并组织成分层访问路径。访问路径的格式如下：
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

以目录、数据库、表、列为例：
```
/catalogs: 表示所有目录。
/catalogs/hive: 表示目录类别下名为“hive”的特定目录对象。
/catalogs/hive/databases: 表示“hive”目录中的所有数据库。
/catalogs/hive/databases/tpch_100g: 表示“hive”目录中名为“tpch_100g”的数据库。
/catalogs/hive/databases/tpch_100g/tables: 表示“tpch_100g”数据库中的所有表。
/catalogs/hive/databases/tpch_100g/tables/lineitem: 表示 tpch_100g.lineitem 表。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: 表示 tpch_100g.lineitem 表中的所有列。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: 表示 tpch_100g.lineitem 表中名为 l_orderkey 的特定列。
```

2. 类别使用蛇形命名法，最后一个单词为复数形式，所有单词均为小写，多个单词用下划线（_）连接。特定对象使用其实际名称进行命名。需要明确定义目标对象的层次关系。

## HTTP 方法的选择

1. GET：使用 GET 方法显示单个对象并列出特定类别的所有对象。GET 方法对对象的访问是只读的，不提供请求正文。
```
# 列出数据库 ssb_100g 中的所有表
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# 显示表 ssb_100g.lineorder
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST：用于创建对象。参数通过请求正文传递。它不是幂等的。如果对象已存在，则重复创建将失败并返回错误消息。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT：用于创建对象。参数通过请求正文传递。它是幂等的。如果该对象已存在，它将返回成功。PUT 方法是 POST 方法的 CREATE IF NOT EXISTS 版本。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE：用于删除对象。它不提供请求正文。如果要删除的对象不存在，则返回成功。DELETE 方法具有 DROP IF EXISTS 语义。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH：用于更新对象。它提供了一个请求体，其中只包含需要修改的部分信息。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 身份验证和授权

1. 身份验证和授权信息在 HTTP 请求标头中传递。

## HTTP 状态代码

1. REST API 返回 HTTP 状态代码，用于指示操作的成功或失败。 
2. 成功操作的状态代码（2xx）如下：

- 200 OK：表示请求已成功完成。用于查看/列出/删除/更新对象和查询待处理任务的状态。
- 201 Created：表示对象创建成功。用于 PUT/POST 方法。响应正文必须包含对象 URI，以便后续查看/列出/删除/更新。
- 202 Accepted：表示任务提交成功，任务处于挂起状态。响应正文必须包含任务 URI，以便后续取消、删除和轮询任务状态。

3. 错误代码（4xx）表示客户端错误。用户需要调整和修改 HTTP 请求并重试。
- 400 Bad Request：请求参数无效。
- 401 Unauthorized：缺少身份验证信息、非法身份验证信息、身份验证失败。
- 403 Forbidden：身份验证成功，但用户的操作未通过授权检查。无访问权限。
- 404 Not Found：API URI 编码错误。不属于已注册的 REST API。
- 405 Method Not Allowed：使用的 HTTP 方法不正确。
- 406 Not Acceptable：响应格式与 Accept 标头中指定的媒体类型不匹配。
- 415 Unsupported Media Type：请求内容的媒体类型与 Content-Type 标头中指定的媒体类型不匹配。

4. 错误代码（5xx）表示服务器错误。用户无需修改请求，可以稍后重试。
- 500 Internal Server Error：内部服务器错误，类似于未知错误。
- 503 Service Unavailable：服务暂时不可用。例如，用户访问频率过高，已达到速率限制；或者服务当前由于内部状态而无法提供服务，例如创建有 3 个副本的表，但只有 2 个 BE 可用；用户查询中涉及的所有 Tablet 副本都不可用。

## HTTP 响应格式

1. 当 API 返回 HTTP 代码 200/201/202 时，HTTP 响应不为空。API 以 JSON 格式返回结果，包括顶级字段“code”、“message”和“result”。所有 JSON 字段都使用驼峰命名法。

2. 在成功的 API 响应中，“code”为“0”，“message”为“OK”，“result”包含实际结果。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 在失败的 API 响应中，“code”不是“0”，“message”是简单的错误消息，“result”可以包含详细的错误信息，例如错误堆栈跟踪。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## 参数传递

1. API 参数按路径、请求正文、查询参数和标头的优先级顺序传递。选择适当的参数传递方法。

2. 路径参数：表示对象层次结构关系的必需参数放置在路径参数中。
```
/api/v2/warehouses/{warehouseName}/backends/{backendId}
/api/v2/warehouses/ware0/backends/10027
```

3. 请求正文：使用 application/json 传递参数。参数可以是必需的，也可以是可选的。

4. 查询参数：不允许同时使用查询参数和请求体参数。对于相同的 API，请选择任一种方法。如果不包括标头参数和路径参数的参数个数不超过 2 个，则可以使用查询参数；否则，请使用请求正文传递参数。

5. 标头参数：标头应该用于传递 HTTP 标准参数，如 Content-type 和 Accept，不应滥用标头传递自定义参数。当使用标头传递用户扩展的参数时，标头名称应采用格式 `x-starrocks-{name}`，其中名称可以包含多个英文单词，每个单词为小写，并由连字符（-）连接。
