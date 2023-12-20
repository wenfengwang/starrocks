---
displayed_sidebar: English
---

# 创建函数

## 说明

创建用户自定义函数（UDF）。目前，您只能创建 Java UDF，包括标量函数、用户自定义聚合函数（UDAF）、用户自定义窗口函数（UDWF）以及用户自定义表函数（UDTF）。

**有关如何编译、创建和使用 Java UDF 的详细信息，请参阅 [Java UDF](../../sql-functions/JAVA_UDF.md)。**

> **注意**
> 要创建**全局 UDF**，您必须拥有**系统级别**的 CREATE GLOBAL FUNCTION 权限。要创建**数据库范围内**的 UDF，您必须拥有**数据库级别**的 CREATE FUNCTION 权限。

## 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## 参数

|参数|必填|说明|
|---|---|---|
|GLOBAL|否|是否创建全局UDF，从v3.0开始支持。|
|AGGREGATE|否|是否创建 UDAF 或 UDWF。|
|TABLE|否|是否创建UDTF。如果未指定 AGGREGATE 和 TABLE，则创建标量函数。|
|function_name|Yes|您要创建的函数的名称。您可以在此参数中包含数据库的名称，例如db1.my_func。如果 function_name 包含数据库名称，则在该数据库中创建 UDF。否则，将在当前数据库中创建 UDF。新函数的名称及其参数不能与目标数据库中的现有名称相同。否则，无法创建该函数。函数名相同但参数不同则创建成功。|
|arg_type|Yes|函数的参数类型。添加的参数可以用 , ... 表示。支持的数据类型请参见 Java UDF。|
|return_type|是|函数的返回类型。有关支持的数据类型，请参阅 Java UDF。|
|属性|是|函数的属性，根据要创建的 UDF 的类型而有所不同。详细信息请参见Java UDF。|
