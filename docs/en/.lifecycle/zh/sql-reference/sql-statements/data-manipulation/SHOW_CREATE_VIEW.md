---
displayed_sidebar: English
---

# SHOW CREATE VIEW

返回用于创建指定视图的 CREATE 语句。CREATE VIEW 语句帮助您理解视图的定义，并提供了修改或重建视图的参考。请注意，执行 SHOW CREATE VIEW 语句需要您对视图及其基础表拥有 `SELECT` 权限。

从 v2.5.4 版本开始，您可以使用 SHOW CREATE VIEW 来查询创建**物化视图**的语句。

## 语法

```SQL
SHOW CREATE VIEW [db_name.]view_name
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|db_name|否|数据库名。如果未指定此参数，默认返回当前数据库中指定视图的 CREATE VIEW 语句。|
|view_name|是|视图名。|

## 输出

```SQL
+---------+--------------+----------------------+----------------------+
| View    | Create View  | character_set_client | collation_connection |
+---------+--------------+----------------------+----------------------+
```

下表描述了该语句返回的参数。

|**参数**|**描述**|
|---|---|
|View|视图名。|
|Create View|视图的 CREATE VIEW 语句。|
|character_set_client|客户端用来向 StarRocks 发送语句的字符集。|
|collation_connection|字符集比较规则。|

## 示例

创建一个名为 `example_table` 的表。

```SQL
CREATE TABLE example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1);
```

基于 `example_table` 创建一个名为 `example_view` 的视图。

```SQL
CREATE VIEW example_view (k1, k2, k3, v1)
AS SELECT k1, k2, k3, v1 FROM example_table;
```

显示 `example_view` 的 CREATE VIEW 语句。

```Plain
SHOW CREATE VIEW example_db.example_view;

+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+
| View         | Create View                                                                                                                                                                                                                                                                                                                     | character_set_client | collation_connection |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+
| example_view | CREATE VIEW `example_view` (k1, k2, k3, v1) COMMENT "VIEW" AS SELECT `default_cluster:db1`.`example_table`.`k1` AS `k1`, `default_cluster:db1`.`example_table`.`k2` AS `k2`, `default_cluster:db1`.`example_table`.`k3` AS `k3`, `default_cluster:db1`.`example_table`.`v1` AS `v1` FROM `default_cluster:db1`.`example_table`; | utf8                 | utf8_general_ci      |
+--------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+
```