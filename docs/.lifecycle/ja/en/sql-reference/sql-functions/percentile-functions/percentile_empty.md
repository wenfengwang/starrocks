---
displayed_sidebar: English
---

# percentile_empty

## 説明

[Stream Load](../../../loading/StreamLoad.md)または[INSERT INTO](../../../loading/InsertInto.md)を使用してデータロード時にnull値を埋めるために使われるPERCENTILE値を構築します。

## 構文

```Haskell
PERCENTILE_EMPTY();
```

## パラメーター

なし

## 戻り値

PERCENTILE値を返します。

## 例

テーブルを作成します。`percent` 列はPERCENTILE型の列です。

```sql
CREATE TABLE `aggregate_tbl` (
  `site_id` largeint(40) NOT NULL COMMENT "サイトのID",
  `date` date NOT NULL COMMENT "イベントの日時",
  `city_code` varchar(20) NULL COMMENT "ユーザーのcity_code",
  `pv` bigint(20) SUM NULL DEFAULT "0" COMMENT "ページビューの合計",
  `percent` PERCENTILE PERCENTILE_UNION COMMENT "その他"
) ENGINE=OLAP
AGGREGATE KEY(`site_id`, `date`, `city_code`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`site_id`)
PROPERTIES ("replication_num" = "3");
```

テーブルにデータを挿入します。

```sql
INSERT INTO aggregate_tbl VALUES
(5, '2020-02-23', 'city_code', 555, percentile_empty());
```
