---
displayed_sidebar: English
---

# percentile_approx_raw

## 説明

`x` から指定されたパーセンタイルに対応する値を返します。

`x` が列の場合、この関数は最初に `x` の値を昇順に並べ替え、パーセンタイル `y` に対応する値を返します。

## 構文

```Haskell
PERCENTILE_APPROX_RAW(x, y);
```

## パラメーター

- `x`: 列または値の集合です。PERCENTILE として評価される必要があります。

- `y`: パーセンタイルです。サポートされるデータ型は DOUBLE。値の範囲は [0.0,1.0] です。

## 戻り値

PERCENTILE 値を返します。

## 例

`percent` 列が percentile_approx_raw() の入力である `aggregate_tbl` テーブルを作成します。

  ```sql
  CREATE TABLE `aggregate_tbl` (
    `site_id` largeint(40) NOT NULL COMMENT "サイトのID",
    `date` date NOT NULL COMMENT "イベントの時間",
    `city_code` varchar(20) NULL COMMENT "ユーザーのcity_code",
    `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "総ページビュー",
    `percent` PERCENTILE PERCENTILE_UNION COMMENT "その他"
  ) ENGINE=OLAP
  AGGREGATE KEY(`site_id`, `date`, `city_code`)
  COMMENT "OLAP"
  DISTRIBUTED BY HASH(`site_id`)
  PROPERTIES ("replication_num" = "1");
  ```

テーブルにデータを挿入します。

  ```sql
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(1));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(2));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(3));
  insert into aggregate_tbl values (5, '2020-02-23', 'city_code', 555, percentile_hash(4));
  ```

パーセンタイル 0.5 に対応する値を計算します。

  ```Plain Text
  mysql> select percentile_approx_raw(percent, 0.5) from aggregate_tbl;
  +-------------------------------------+
  | percentile_approx_raw(percent, 0.5) |
  +-------------------------------------+
  |                                 2.5 |
  +-------------------------------------+
  1 row in set (0.03 sec)
  ```
