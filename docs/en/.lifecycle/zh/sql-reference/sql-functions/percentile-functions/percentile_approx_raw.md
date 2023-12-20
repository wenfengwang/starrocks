---
displayed_sidebar: English
---

# percentile_approx_raw

## 描述

返回与指定百分位数对应的 `x` 的值。

如果 `x` 是一列，该函数首先将 `x` 中的值按升序排序，然后返回与百分位数 `y` 对应的值。

## 语法

```Haskell
PERCENTILE_APPROX_RAW(x, y);
```

## 参数

- `x`：可以是一列或一组值。必须计算为 PERCENTILE 类型。

- `y`：百分位数。支持的数据类型为 DOUBLE。值范围：[0.0,1.0]。

## 返回值

返回 PERCENTILE 类型的值。

## 示例

创建表 `aggregate_tbl`，其中 `percent` 列是 percentile_approx_raw() 函数的输入。

```sql
CREATE TABLE `aggregate_tbl` (
  `site_id` largeint(40) NOT NULL COMMENT "站点的 id",
  `date` date NOT NULL COMMENT "事件的时间",
  `city_code` varchar(20) NULL COMMENT "用户的 city_code",
  `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "总页面浏览量",
  `percent` PERCENTILE PERCENTILE_UNION COMMENT "其他"
) ENGINE=OLAP
AGGREGATE KEY(`site_id`, `date`, `city_code`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`site_id`)
PROPERTIES ("replication_num" = "1");
```

向表中插入数据。

```sql
insert into `aggregate_tbl` values (5, '2020-02-23', 'city_code', 555, percentile_hash(1));
insert into `aggregate_tbl` values (5, '2020-02-23', 'city_code', 555, percentile_hash(2));
insert into `aggregate_tbl` values (5, '2020-02-23', 'city_code', 555, percentile_hash(3));
insert into `aggregate_tbl` values (5, '2020-02-23', 'city_code', 555, percentile_hash(4));
```

计算对应于百分位数 0.5 的值。

```Plain
mysql> select percentile_approx_raw(`percent`, 0.5) from `aggregate_tbl`;
+-------------------------------------+
| percentile_approx_raw(`percent`, 0.5) |
+-------------------------------------+
|                                 2.5 |
+-------------------------------------+
1 row in set (0.03 sec)
```