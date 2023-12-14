---
displayed_sidebar: "Chinese"
---

# 创建函数

## 描述

创建用户定义函数（UDF）。目前，您只能创建Java UDF，包括标量函数、用户定义的聚合函数（UDAFs）、用户定义的窗口函数（UDWFs）和用户定义的表函数（UDTFs）。

**有关如何编译、创建和使用Java UDF的详细信息，请参见[Java UDF](../../sql-functions/JAVA_UDF.md)。**

> **注意**
>
> 要创建全局UDF，您必须具有SYSTEM级别的CREATE GLOBAL FUNCTION权限。 要创建数据库范围的UDF，您必须具有DATABASE级别的CREATE FUNCTION权限。

## 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## 参数

| **参数**      | **必需** | **描述**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | 否       | 是否创建全局UDF，从v3.0开始支持。  |
| AGGREGATE     | 否       | 是否创建UDAF或UDWF。       |
| TABLE         | 否       | 是否创建UDTF。如果未同时指定`AGGREGATE`和`TABLE`，则创建标量函数。               |
| function_name | 是       | 要创建的函数的名称。您可以将数据库名称包含在此参数中，例如，`db1.my_func`。如果`function_name`包含数据库名称，则UDF将在该数据库中创建。否则，UDF将在当前数据库中创建。新函数的名称及其参数不能与目标数据库中现有名称相同。否则，函数将无法创建。如果函数名相同但参数不同，则创建成功。 |
| arg_type      | 是       | 函数的参数类型。添加的参数可以用`, ...`表示。支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#sql-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E4%B8%8EJava%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%98%A0%E5%B0%84)。|
| return_type      | 是       | 函数的返回类型。支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#sql-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E4%B8%8EJava%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%98%A0%E5%B0%84)。 |
| PROPERTIES    | 是       | 函数的属性，根据要创建的UDF类型而异。详情，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)。 |