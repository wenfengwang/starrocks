---
displayed_sidebar: "Chinese"
---

# StarRocks Restful API 标准

## API 格式

1. API格式遵循模式：`/api/{version}/{target-object-access-path}/{action}`。 
2. `{version}`表示为`v{number}`，例如v1、v2、v3、v4等。 
3. `{target-object-access-path}`以分层方式组织，稍后将详细解释。 
4. `{action}`为可选项，API实现者应尽量利用HTTP方法传达操作的含义。只有当无法满足HTTP方法的语义时，才应使用该操作。例如，如果没有可用的HTTP方法来重命名对象。

## 目标对象访问路径的定义

1. 通过REST API访问的目标对象需进行分类并组织为分层访问路径。访问路径格式如下：
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

以目录、数据库、表、列为例：
```
/catalogs: 表示所有目录。
/catalogs/hive: 表示目录类别下名为"hive"的特定目录对象。
/catalogs/hive/databases: 表示"hive"目录中的所有数据库。
/catalogs/hive/databases/tpch_100g: 表示"hive"目录中名为"tpch_100g"的数据库。
/catalogs/hive/databases/tpch_100g/tables: 表示"tpch_100g"数据库中的所有表。
/catalogs/hive/databases/tpch_100g/tables/lineitem: 表示tpch_100g.lineitem表。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: 表示tpch_100g.lineitem表中的所有列。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: 表示tpch_100g.lineitem表中名为l_orderkey的特定列。
```

2. 类别名称使用蛇形命名法，最后一个单词采用复数形式。所有单词均小写，多个单词用下划线(_)连接。需要清楚定义目标对象的分层关系。

## HTTP方法的选择

1. GET: 使用GET方法显示单个对象并列出特定类别中的所有对象。GET方法对对象的访问是只读的，不提供请求正文。
```
# 列出数据库ssb_100g中的所有表
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# 显示表ssb_100g.lineorder
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST: 用于创建对象。通过请求正文传递参数。这不是幂等的。如果对象已经存在，重复创建将失败并返回错误消息。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT: 用于创建对象。通过请求正文传递参数。这是幂等的。如果对象已经存在，将返回成功。PUT方法是POST方法的CREATE IF NOT EXISTS版本。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE: 用于删除对象。不提供请求正文。如果要删除的对象不存在，将返回成功。DELETE方法具有DROP IF EXISTS语义。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH: 用于更新对象。提供请求正文，其中只包含需要修改的部分信息。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 身份验证和授权

1. 身份验证和授权信息通过HTTP请求头传递。

## HTTP状态码

1. REST API返回HTTP状态码以指示操作的成功或失败。 
2. 成功操作的状态码（2xx）如下：

- 200 OK: 表示请求已成功完成。用于查看/列出/删除/更新对象和查询挂起任务的状态。
- 201 Created: 表示对象已成功创建。用于PUT/POST方法。响应正文必须包含对象URI，以供后续查看/列出/删除/更新。
- 202 Accepted: 表示任务提交成功，任务处于挂起状态。响应正文必须包含任务URI，以供后续取消、删除和轮询任务状态。

3. 错误码（4xx）指示客户端错误。用户需要调整和修改HTTP请求后重试。
- 400 Bad Request: 无效的请求参数。
- 401 Unauthorized: 缺少身份验证信息、非法身份验证信息、身份验证失败。
- 403 Forbidden: 身份验证成功，但用户的操作未通过授权检查。无访问权限。
- 404 Not Found: API URI编码错误。不属于注册的REST API。
- 405 Method Not Allowed: 使用了不正确的HTTP方法。
- 406 Not Acceptable: 响应格式与Accept标题中指定的媒体类型不匹配。
- 415 Not Acceptable: 请求内容的媒体类型与Content-Type标题中指定的媒体类型不匹配。

4. 错误码（5xx）指示服务器错误。用户无需修改请求，可稍后重试。
- 500 Internal Server Error: 内部服务器错误，类似于未知错误。
- 503 Service Unavailable: 服务暂时不可用。例如，用户的访问频率过高，已达到速率限制；或由于内部状态，服务当前无法提供服务，例如创建具有3个副本的表，但只有2个BE可用；用户查询涉及的所有Tablet副本都不可用。

## HTTP响应格式

1. 当API返回HTTP状态码为200/201/202时，HTTP响应不为空。API以JSON格式返回结果，包括顶层字段“code”、“message”和“result”。所有的JSON字段均使用驼峰命名。

2. 在成功的API响应中，“code”为“0”，“message”为“OK”，“result”包含实际结果。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 在失败的API响应中，“code”不为“0”，“message”为简单的错误消息，“result”可包含详细的错误信息，如错误堆栈跟踪。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## 参数传递

1. API参数按路径、请求正文、查询参数和头部的优先顺序进行传递。选择适当的方法进行参数传递。

2. 路径参数：代表对象的分层关系的必需参数放置在路径参数中。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. 请求正文：使用application/json传递参数。参数可以是必需或可选类型。

4. 查询参数：不允许同时使用查询参数和请求正文参数。对于相同的API，选择其中一种。如果除头部参数和路径参数外的参数数量不超过2个，则可使用查询参数；否则，请使用请求正文传递参数。

5. 头部参数：头部应用于传递HTTP标准参数，如Content-type和Accept置于头部中，实现者不应滥用HTTP头部传递自定义参数。在使用头部传递用户扩展的参数时，头部名称应采用格式“x-starrocks-{name}”，其中名称可以包含多个英文单词，且每个单词均小写且由连字符(-)连接。