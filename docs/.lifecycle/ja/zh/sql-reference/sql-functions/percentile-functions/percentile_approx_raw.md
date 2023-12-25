---
displayed_sidebar: Chinese
---

# percentile_approx_raw

## 機能

指定されたパラメータ `x` のパーセンタイルを計算します。`x` は列または数値にすることができます。`x` が列の場合、まずその列を昇順にソートし、次に正確な `y` 番目のパーセンタイルを取得します。

## 文法

```Haskell
PERCENTILE_APPROX_RAW(x, y);
```

## パラメータ説明

`x`: 列または数値とすることができます。サポートされるデータタイプは PERCENTILE です。

`y`: 指定されたパーセンタイルで、サポートされるデータタイプは DOUBLE で、値の範囲は [0.0,1.0] です。

## 戻り値の説明

戻り値のデータタイプは DOUBLE です。

## 例

`aggregate_tbl` というテーブルを作成します。ここで `percent` 列は percentile_approx_raw() 関数の入力となります。

  ```sql
  CREATE TABLE `aggregate_tbl` (
    `site_id` largeint(40) NOT NULL COMMENT "siteのid",
    `date` date NOT NULL COMMENT "イベントの時間",
    `city_code` varchar(20) NULL COMMENT "ユーザーのcity_code",
    `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "総ページビュー",
    `percent` PERCENTILE PERCENTILE_UNION COMMENT "その他"
  ) ENGINE=OLAP
  AGGREGATE KEY(`site_id`, `date`, `city_code`)
  COMMENT "OLAP"
  DISTRIBUTED BY HASH(`site_id`)
  PROPERTIES ("replication_num" = "3");
  ```

テーブルにデータを挿入します。

  ```sql
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(1));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(2));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(3));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(4));
  ```

0.5番目のパーセンタイルに相当する値を計算します。

  ```Plain Text
  mysql> select percentile_approx_raw(percent, 0.5) from aggregate_tbl;
  +-------------------------------------+
  | percentile_approx_raw(percent, 0.5) |
  +-------------------------------------+
  |                                 2.5 |
  +-------------------------------------+
  1 row in set (0.03 sec)
  ```
