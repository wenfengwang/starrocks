---
displayed_sidebar: Chinese
---

# SHOW CREATE VIEWの表示

## 機能

指定された論理ビューのCREATE VIEW文を表示します。ビューの作成文は、ビューの定義を理解し、後でビューを変更または再作成する際の参考になります。

> **注意**
>
> そのビューとビューに対応する基本テーブルのSELECT権限を持つユーザーのみが表示できます。

バージョン2.5.4から、MySQLの標準構文との互換性を持たせるために、SHOW CREATE VIEWを使用して非同期マテリアライズドビューの作成文を表示することがサポートされました。この文はマテリアライズドビューを通常のビューとして扱います。

## 语法

```SQL
SHOW CREATE VIEW [<db_name>.]<view_name>
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ   | データベース名。指定しない場合は、現在のデータベース内の指定されたビューの作成文を表示します。 |
| view_name      | はい     | ビュー名。                                                   |

## 戻り値の説明

```plain
+---------+--------------+----------------------+----------------------+
| View    | Create View  | character_set_client | collation_connection |
+---------+--------------+----------------------+----------------------+
```

戻り値のパラメータ説明は以下の通りです：

| **パラメータ**           | **説明**                                  |
| ------------------------ | ----------------------------------------- |
| View                     | ビュー名。                                |
| Create View              | ビューの作成文。                          |
| character_set_client     | クライアントがStarRocksサーバーに接続する際に使用する文字セット。 |
| collation_connection     | 文字セットの照合順序。                    |

## 例

### 論理ビューの作成文を表示

`base`テーブルを作成します。

```SQL
CREATE TABLE base (
    k1 date,
    k2 int,
    v1 int sum)
PARTITION BY RANGE(k1)
(
    PARTITION p1 values less than('2020-02-01'),
    PARTITION p2 values less than('2020-03-01')
)
DISTRIBUTED BY HASH(k2)
PROPERTIES( "replication_num"  = "1");
```

`base`テーブルに`example_view`ビューを作成します。

```SQL
CREATE VIEW example_view (k1, k2, v1)
AS SELECT k1, k2, v1 FROM base;
```

ビュー`example_view`の作成文を表示します。

```Plain
SHOW CREATE VIEW example_view;

MySQL [yn_db]> SHOW CREATE VIEW example_view;
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+
| View         | Create View                                                                                                                                         | character_set_client | collation_connection |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+
| example_view | CREATE VIEW `example_view` (k1, k2, v1) COMMENT "VIEW" AS SELECT `yn_db`.`base`.`k1`, `yn_db`.`base`.`k2`, `yn_db`.`base`.`v1`
FROM `yn_db`.`base`; | utf8                 | utf8_general_ci      |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+

```

### マテリアライズドビューの作成文を表示

`base`テーブルにマテリアライズドビュー`example_mv`を作成します。

```SQL
CREATE MATERIALIZED VIEW example_mv distributed by hash(k1)
AS SELECT k1 FROM base;
```

マテリアライズドビュー`example_mv`の作成文を表示します。

```Plain
SHOW CREATE VIEW example_mv;
+------------+----------------------------------------------------------------------------+----------------------+----------------------+
| View       | Create View                                                                | character_set_client | collation_connection |
+------------+----------------------------------------------------------------------------+----------------------+----------------------+
| example_mv | CREATE VIEW `example_mv` AS SELECT `yn_db`.`base`.`k1`
FROM `yn_db`.`base` | utf8                 | utf8_general_ci      |
+------------+------------------------------------------------
```
