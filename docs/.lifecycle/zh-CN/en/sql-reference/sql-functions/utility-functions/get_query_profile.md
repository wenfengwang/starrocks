---
displayed_sidebar: "Chinese"
---

# get_query_profile

## 描述

通过使用其`query_id`获取查询的概要信息。如果`query_id`不存在或不正确，则此功能将返回空。

要使用此功能，您必须启用分析功能，即将会话变量`enable_profile`设置为`true`（`set enable_profile = true;`）。如果未启用此功能，则会返回空白概要信息。

此功能支持v3.0及更高版本。

## 语法

```Haskell
get_query_profile(x)
```

## 参数

`x`：查询的`query_id`字符串。支持的数据类型为VARCHAR。

## 返回值

查询概要信息包含以下字段。有关查询概要信息的更多信息，请参见[Query Profile](../../../administration/query_profile.md)。

```SQL
查询:
  概要:
  计划：
  执行概要 7de16a85-761c-11ed-917d-00163e14d435:
    片段 0:
      流水线（id=2）:
        EXCHANGE_SINK（计划节点id=18）:
        LOCAL_MERGE_SOURCE（计划节点id=17）:
      流水线（id=1）:
        LOCAL_SORT_SINK（计划节点id=17）:
        AGGREGATE_BLOCKING_SOURCE（计划节点id=16）:
      流水线（id=0）:
        AGGREGATE_BLOCKING_SINK（计划节点id=16）:
        EXCHANGE_SOURCE（计划节点id=15）:
    片段 1:
       ...
    片段 2:
       ...
```

## 示例

```sql
-- 启用分析功能。
set enable_profile = true;

-- 运行简单查询。
select 1;

-- 获取查询的query_id。
select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| bd3335ce-8dde-11ee-92e4-3269eb8da7d1 |
+--------------------------------------+

-- 获取查询概要信息。
select get_query_profile('502f3c04-8f5c-11ee-a41f-b22a2c00f66b');

-- 使用regexp_extract函数获取与指定模式匹配的概要信息中的QueryPeakMemoryUsage。
select regexp_extract(get_query_profile('bd3335ce-8dde-11ee-92e4-3269eb8da7d1'), 'QueryPeakMemoryUsage: [0-9\.]* [KMGB]*', 0);
+-----------------------------------------------------------------------------------------------------------------------+
| regexp_extract(get_query_profile('bd3335ce-8dde-11ee-92e4-3269eb8da7d1'), 'QueryPeakMemoryUsage: [0-9.]* [KMGB]*', 0) |
+-----------------------------------------------------------------------------------------------------------------------+
| QueryPeakMemoryUsage: 3.828 KB                                                                                        |
+-----------------------------------------------------------------------------------------------------------------------+
```

## 相关函数

- [last_query_id](./last_query_id.md)
- [regexp_extract](../like-predicate-functions/regexp_extract.md)