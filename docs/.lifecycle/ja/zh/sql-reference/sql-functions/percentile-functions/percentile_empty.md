---
displayed_sidebar: Chinese
---

# percentile_empty

## 機能

`percentile` 型の数値を構築し、INSERT や Stream Load のインポート時にデフォルト値として使用します。

## 文法

```Haskell
PERCENTILE_EMPTY();
```

## パラメータ説明

なし

## 戻り値の説明

戻り値のデータ型は PERCENTILE です。

## 例

テーブルを作成します。

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
PROPERTIES ("replication_num" = "3");
```

データを挿入します。

```sql
INSERT INTO aggregate_tbl VALUES
(5, '2020-02-23', 'city_code', 555, percentile_empty());
```
