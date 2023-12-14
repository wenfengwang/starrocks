---
displayed_sidebar: "Chinese"
---

# percentile_approx_raw

## 描述

返回与`x`中指定百分位数相对应的值。

如果`x`是一列，此函数首先按升序排序`x`中的值，并返回与百分位数`y`对应的值。

## 语法

```Haskell
PERCENTILE_APPROX_RAW(x, y);
```

## 参数

- `x`：可以是一列或一组值。它必须计算为百分位数。

- `y`：百分位数。支持的数据类型为DOUBLE。值范围：[0.0, 1.0]。

## 返回值

返回一个百分位数值。

## 示例

创建`aggregate_tbl`表，其中`percent`列是percentile_approx_raw()的输入。

  ```sql
  CREATE TABLE `aggregate_tbl` (
    `site_id` largeint(40) NOT NULL COMMENT "站点编号",
    `date` date NOT NULL COMMENT "事件时间",
    `city_code` varchar(20) NULL COMMENT "用户城市代码",
    `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "总页面访问量",
    `percent` PERCENTILE PERCENTILE_UNION COMMENT "其他"
  ) ENGINE=OLAP
  AGGREGATE KEY(`site_id`, `date`, `city_code`)
  COMMENT "OLAP"
  DISTRIBUTED BY HASH(`site_id`)
  PROPERTIES ("replication_num" = "1");
  ```

向表中插入数据。

  ```sql
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(1));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(2));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(3));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(4));
  ```

计算对应于百分位数0.5的值。

  ```Plain Text
  mysql> select percentile_approx_raw(percent, 0.5) from aggregate_tbl;
  +-------------------------------------+
  | percentile_approx_raw(percent, 0.5) |
  +-------------------------------------+
  |                                 2.5 |
  +-------------------------------------+
  1 行受影响 (0.03 秒)
  ```