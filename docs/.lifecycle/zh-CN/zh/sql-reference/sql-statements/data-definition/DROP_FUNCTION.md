---
displayed_sidebar: "Chinese"
---

# 删除函数

## 功能

删除一个自定义函数。只有函数名称和参数类型完全匹配才能被删除。

执行此命令的用户必须是函数的所有者。

## 语法

```sql
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...])
```

## 参数说明

- `GLOBAL`：表示删除全局函数。StarRocks 从 3.0 版本开始支持创建[全局UDF](../../sql-functions/JAVA_UDF.md)。
- `function_name`: 待删除函数的名称，必填。
- `arg_type`: 待删除函数的参数类型，必填。

## 示例

删除一个函数。

```sql
DROP FUNCTION my_add(INT, INT)
```

## 相关 SQL

- [显示函数](./SHOW_FUNCTIONS.md)
- [Java UDF](../../sql-functions/JAVA_UDF.md)