---
displayed_sidebar: English
---

# StarRocks Restful API 标准

## API 格式

1. API 的格式遵循以下模式：/api/{version}/{target-object-access-path}/{action}。
2. {version} 表示为 v{number}，例如 v1、v2、v3、v4 等。
3. {target-object-access-path} 以层次化的方式组织，我们将在后面详细说明。
4. {action} 是可选的，API 实现者应尽可能通过 HTTP 方法来表达操作的含义。只有在 HTTP 方法无法满足需求时才使用 action。比如，没有适用于重命名对象的 HTTP 方法时。

## 目标对象访问路径的定义

1. REST API 访问的目标对象需要被分类并组织成层级访问路径。访问路径的格式如下：
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

以目录、数据库、表、列为例：
```
/catalogs: Represents all catalogs.
/catalogs/hive: Represents the specific catalog object named "hive" under the catalog category.
/catalogs/hive/databases: Represents all databases in the "hive" catalog.
/catalogs/hive/databases/tpch_100g: Represents the database named "tpch_100g" in the "hive" catalog.
/catalogs/hive/databases/tpch_100g/tables: Represents all tables in the "tpch_100g" database.
/catalogs/hive/databases/tpch_100g/tables/lineitem: Represents the tpch_100g.lineitem table.
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: Represents all columns in the tpch_100g.lineitem table.
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: Represents the specific column l_orderkey in the tpch_100g.lineitem table.
```

2. 类别使用蛇形命名法，最后一个词为复数形式。所有单词都使用小写，并且多个单词之间用下划线（_）连接。具体对象则使用它们的实际名称。目标对象的层级关系应该清晰定义。

## HTTP 方法的选择

1. GET：使用 GET 方法来显示单个对象或列出某一类别的所有对象。GET 方法对对象的访问是只读的，并且不包含请求体。
```
# list all of the tables in database ssb_100g
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# show the table ssb_100g.lineorder
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST：用于创建对象。参数通过请求体传递。它不是幂等的。如果对象已存在，则重复创建会失败并返回错误信息。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT：用于创建对象。参数通过请求体传递。它是幂等的。如果对象已存在，会返回成功。PUT 方法相当于 POST 方法的“CREATE IF NOT EXISTS”版本。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE：用于删除对象。它不提供请求体。如果待删除的对象不存在，也会返回成功。DELETE 方法具有“DROP IF EXISTS”的含义。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH：用于更新对象。它提供请求体，其中只包含需要修改的部分信息。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 认证与授权

1. 身份验证和授权信息通过 HTTP 请求头传递。

## HTTP 状态码

1. REST API 返回 HTTP 状态码以指示操作的成功或失败。
2. 成功操作的状态码（2xx）如下：

- 200 OK：表示请求已成功处理。用于查看/列出/删除/更新对象，以及查询待处理任务的状态。
- 201 Created：表示对象已成功创建。用于 PUT/POST 方法。响应体必须包含对象的 URI，以供后续的查看/列出/删除/更新操作。
- 202 Accepted：表示任务提交成功，并且任务处于等待状态。响应体必须包含任务的 URI，以便后续的取消、删除和状态查询操作。

3. 错误代码（4xx）表示客户端错误。用户需要调整 HTTP 请求后重试。
- 400 Bad Request：请求参数无效。
- 401 Unauthorized：缺少认证信息，认证信息非法，或认证失败。
- 403 Forbidden：认证成功，但用户的操作未通过授权检查。没有访问权限。
- 404 Not Found：API URI 编码错误，不属于已注册的 REST API。
- 405 Method Not Allowed：使用了错误的 HTTP 方法。
- 406 Not Acceptable：响应格式与 Accept 头指定的媒体类型不匹配。
- 415 Unsupported Media Type：请求内容的媒体类型与 Content-Type 头指定的媒体类型不匹配。

4. 错误代码（5xx）表示服务器错误。用户无需修改请求，可以稍后重试。
- 500 Internal Server Error：内部服务器错误，类似于未知错误。
- 503 Service Unavailable：服务暂时不可用。例如，用户访问频率过高，达到了速率限制；或者服务由于内部状态无法提供服务，如创建了 3 个副本的表，但只有 2 个 BE 可用；或者用户查询涉及的所有 Tablet 副本都不可用。

## HTTP 响应格式

1. 当 API 返回 200/201/202 的 HTTP 状态码时，HTTP 响应不为空。API 以 JSON 格式返回结果，包括顶级字段“code”、“message”和“result”。所有 JSON 字段均使用驼峰命名法。

2. 在成功的 API 响应中，“code”为“0”，“message”为“OK”，“result”包含实际结果。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 在失败的 API 响应中，“code”非“0”，“message”为简单的错误信息，“result”可能包含详细错误信息，如错误堆栈跟踪。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## 参数传递

1. API 参数按照路径、请求体、查询参数和头部的优先级顺序传递。选择合适的参数传递方式。

2. 路径参数：表示对象层次关系的必填参数放在路径参数中。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. 请求体：参数通过 application/json 传递。参数可以是必需的或可选的。

4. 查询参数：不允许同时使用查询参数和请求体参数。对于同一个 API，选择其中一个。如果除头参数和路径参数外的参数数量不超过 2 个，则可以使用查询参数；否则，应使用请求体传递参数。

5. HEADER 参数：应使用头部来传递 HTTP 标准参数，如 Content-Type 和 Accept。实现者不应滥用 HTTP 头部来传递自定义参数。当使用头部传递用户扩展参数时，头部名称应使用 x-starrocks-{name} 的格式，其中 name 可以包含多个英文单词，每个单词都用小写字母，并通过连字符（-）连接。
