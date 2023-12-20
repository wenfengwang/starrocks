---
displayed_sidebar: English
---

# 创建函数

## 描述

创建用户定义函数（UDF）。目前，只能创建Java UDF，包括标量函数、用户定义聚合函数（UDAF）、用户定义窗口函数（UDWF）以及用户定义表函数（UDTF）。

**有关如何编译、创建和使用Java UDF的详细信息，请参见[Java UDF](../../sql-functions/JAVA_UDF.md)**。

> **注意**
> 要创建**全局UDF**，您必须拥有**系统级**的CREATE GLOBAL FUNCTION权限。要创建**数据库范围内**的UDF，您必须拥有**数据库级别**的CREATE FUNCTION权限。

## 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|GLOBAL|否|是否创建全局UDF，从v3.0版本开始支持。|
|AGGREGATE|否|是否创建UDAF或UDWF。|
|TABLE|否|是否创建UDTF。如果未指定`AGGREGATE`和`TABLE`，则创建标量函数。|
|function_name|是|您想要创建的函数名称。您可以在此参数中包含数据库名称，例如`db1.my_func`。如果`function_name`包含数据库名称，则在该数据库中创建UDF。否则，UDF将在当前数据库中创建。新函数的名称及其参数不能与目标数据库中现有的名称相同，否则无法创建该函数。如果函数名称相同但参数不同，则可以创建成功。|
|arg_type|是|函数的参数类型。可以用`, ...`表示附加的参数。支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)。|
|return_type|是|函数的返回类型。支持的数据类型，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)。|
|PROPERTIES|是|函数的属性，这些属性根据要创建的UDF的类型而有所不同。详细信息，请参见[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)。|