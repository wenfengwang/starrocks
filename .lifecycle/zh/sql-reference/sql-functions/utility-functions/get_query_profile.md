---
displayed_sidebar: English
---

# 获取查询概要

## 描述

通过使用查询的 query_id 来获取查询的概要信息。如果 query_id 不存在或输入错误，则该函数将返回空值。

要使用此功能，您必须启用性能分析特性，即设置会话变量 enable_profile 为 true（即执行 set enable_profile = true;）。如果未启用该特性，将返回一个空的概要信息。

该功能从 v3.0 版本开始支持。

## 语法

```Haskell
get_query_profile(x)
```

## 参数

x：query_id 字符串。支持的数据类型为 VARCHAR。

## 返回值

查询概要包含以下字段。更多关于查询概要的信息，请参见[查询概要](../../../administration/query_profile.md)文档。

```SQL
Query:
  Summary:
  Planner:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

## 示例

```sql
-- Enable the profiling feature.
set enable_profile = true;

-- Run a simple query.
select 1;

-- Get the query_id of the query.
select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| bd3335ce-8dde-11ee-92e4-3269eb8da7d1 |
+--------------------------------------+

-- Obtain the query profile.
select get_query_profile('502f3c04-8f5c-11ee-a41f-b22a2c00f66b');

-- Use the regexp_extract function to obtain the QueryPeakMemoryUsage in the profile that matches the specified pattern.
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
