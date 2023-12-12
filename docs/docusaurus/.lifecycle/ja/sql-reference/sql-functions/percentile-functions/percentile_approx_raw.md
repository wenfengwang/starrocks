---
displayed_sidebar: "Japanese"
---

# percentile_approx_raw

## 説明

指定されたパーセンタイルに対応する値を`x`から返します。

`x`が列の場合、この関数はまず`x`内の値を昇順に並べ替え、パーセンタイル`y`に対応する値を返します。

## 構文

```Haskell
PERCENTILE_APPROX_RAW(x, y);
```

## パラメータ

- `x`: 列または一連の値です。PERCENTILEに評価できなければなりません。

- `y`: パーセンタイルです。サポートされているデータ型はDOUBLEです。値の範囲：[0.0,1.0]。

## 戻り値

PERCENTILE値を返します。

## 例

 `aggregate_tbl`テーブルを作成し、`percent`列をpercentile_approx_raw()の入力とします。

  ```sql
  CREATE TABLE `aggregate_tbl` (
    `site_id` largeint(40) NOT NULL COMMENT "サイトID",
    `date` date NOT NULL COMMENT "イベントの日時",
    `city_code` varchar(20) NULL COMMENT "ユーザの都市コード",
    `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "合計ページビュー",
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

パーセンタイル0.5に対応する値を計算します。

  ```Plain Text
  mysql> select percentile_approx_raw(percent, 0.5) from aggregate_tbl;
  +-------------------------------------+
  | percentile_approx_raw(percent, 0.5) |
  +-------------------------------------+
  |                                 2.5 |
  +-------------------------------------+
  1 row in set (0.03 sec)
  ```