---
displayed_sidebar: English
---

# get_query_profile

## 描述

通过使用其 `query_id` 获取查询的配置文件。如果 `query_id` 不存在或不正确，则此函数返回空值。

要使用此函数，必须启用性能分析功能，即将会话变量 `enable_profile` 设置为 `true` (`set enable_profile = true;`)。如果未启用此功能，则返回空配置文件。

此函数从 v3.0 版本开始支持。

## 语法

```Haskell
get_query_profile(x)
```

## 参数

`x`：query_id 字符串。支持的数据类型为 VARCHAR。

## 返回值

查询配置文件包含以下字段。有关查询配置文件的详细信息，请参阅 [查询配置文件](../../../administration/query_profile.md)。

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

## 例子

```sql
-- 启用性能分析功能。
set enable_profile = true;

-- 运行简单查询。
select 1;

-- 获取查询的 query_id。
select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| bd3335ce-8dde-11ee-92e4-3269eb8da7d1 |
+--------------------------------------+

-- 获取查询配置文件。
select get_query_profile('502f3c04-8f5c-11ee-a41f-b22a2c00f66b');

-- 使用 regexp_extract 函数获取与指定模式匹配的配置文件中的 QueryPeakMemoryUsage。
select regexp_extract(get_query_profile('bd3335ce-8dde-11ee-92e4-3269eb8da7d1'), 'QueryPeakMemoryUsage: [0-9\.]* [KMGB]*', 0);
+-----------------------------------------------------------------------------------------------------------------------+
| regexp_extract(get_query_profile('bd3335ce-8dde-11ee-92e4-3269eb8da7d1'), 'QueryPeakMemoryUsage: [0-9.]* [KMGB]*', 0) |
+-----------------------------------------------------------------------------------------------------------------------+
| QueryPeakMemoryUsage: 3.828 KB                                                                                        |
+-----------------------------------------------------------------------------------------------------------------------+
```

## 相关功能

- [last_query_id](./last_query_id.md)
- [regexp_extract](../like-predicate-functions/regexp_extract.md)
