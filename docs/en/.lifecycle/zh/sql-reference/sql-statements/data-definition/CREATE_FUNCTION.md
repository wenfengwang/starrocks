---
displayed_sidebar: English
---

# 创建函数

## 描述

创建用户定义函数（UDF）。目前，您只能创建Java UDF，包括标量函数、用户定义的聚合函数（UDAF）、用户定义的窗口函数（UDWF）和用户定义的表函数（UDTF）。

**有关如何编译、创建和使用Java UDF的详细信息，请参见[Java UDF](../../sql-functions/JAVA_UDF.md)。**

> **注意**
>
> 要创建全局UDF，您必须具有SYSTEM级别的CREATE GLOBAL FUNCTION特权。要创建数据库范围的UDF，您必须具有DATABASE级别的CREATE FUNCTION特权。

## 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## 参数

| **参数**      | **必填** | **描述**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | 否       | 是否创建全局UDF，从v3.0开始支持。  |
| AGGREGATE     | 否       | 要创建UDAF或UDWF。       |
| TABLE         | 否       | 要创建UDTF。如果未同时指定`AGGREGATE`和`TABLE`，则创建标量函数。               |
| function_name | 是       | 要创建的函数的名称。您可以在此参数中包含数据库的名称，例如`db1.my_func`。如果`function_name`包含数据库名称，则UDF将在该数据库中创建。否则，UDF将在当前数据库中创建。新函数的名称及其参数不能与目标数据库中的现有名称相同。否则，无法创建函数。如果函数名称相同但参数不同，则创建将成功。 |
| arg_type      | 是       | 函数的参数类型。可以使用`, ...`表示添加的参数。有关支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)。|
| return_type   | 是       | 函数的返回类型。有关支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)。 |
| PROPERTIES    | 是       | 函数的属性，具体取决于要创建的UDF的类型。详细信息，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)。 |
