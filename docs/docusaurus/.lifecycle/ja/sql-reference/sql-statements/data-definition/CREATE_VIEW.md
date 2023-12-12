---
displayed_sidebar: "Japanese"
---

# ビューの作成

## 説明

ビューを作成します。

ビュー、または論理ビューとは、データが他の既存の物理テーブルに対するクエリから派生した仮想テーブルのことです。したがって、ビューは物理的なストレージを使用せず、ビューへのすべてのクエリは、ビューを構築するために使用されたクエリ文のサブクエリと同等です。

StarRocksでサポートされているマテライズド・ビューに関する情報は、[同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view-single_table.md) および [非同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view.md) を参照してください。

> **注意**
>
> 特定のデータベースに対するCREATE VIEW権限を持つユーザーのみがこの操作を実行できます。

## 構文

```SQL
CREATE [OR REPLACE] VIEW [IF NOT EXISTS]
[<database>.]<view_name>
(
    <column_name>[ COMMENT 'column comment']
    [, <column_name>[ COMMENT 'column comment'], ...]
)
[COMMENT 'view comment']
AS <query_statement>
```

## パラメータ

| **パラメータ**  | **説明**                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------------- |
| OR REPLACE      | 既存のビューを置き換えます。                                                                         |
| database        | ビューが存在するデータベースの名前です。                                                            |
| view_name       | ビューの名前です。                                                                                   |
| column_name     | ビュー内の列の名前です。ビュー内の列と `query_statement` でクエリされる列は数が一致していなければなりません。 |
| COMMENT         | ビュー内またはビュー自体の列についてのコメントです。                                               |
| query_statement | ビューを作成するために使用されるクエリ文です。それはStarRocksでサポートされている任意のクエリ文であることができます。 |

## 使用上の注意

- ビューをクエリするには、ビューとそれに対応するベーステーブルに対するSELECT権限が必要です。
- ビューを構築するために使用されるクエリ文がベーステーブルのスキーマ変更のために実行できない場合、ビューをクエリするとStarRocksはエラーを返します。

## 例

例1: `example_table` に対する集計クエリを使用して、データベース `example_db` に `example_view` という名前のビューを作成します。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

例2: データベース `example_db` のテーブル `example_table` に対する集計クエリを使用して、ビューと各列にコメントを指定して、`example_view` という名前のビューを作成します。

```SQL
CREATE VIEW example_db.example_view
(
    k1 COMMENT 'first key',
    k2 COMMENT 'second key',
    k3 COMMENT 'third key',
    v1 COMMENT 'first value'
)
COMMENT 'my first view'
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

## 関連するSQL

- [SHOW CREATE VIEW](../data-manipulation/SHOW_CREATE_VIEW.md)
- [ALTER VIEW](./ALTER_VIEW.md)
- [DROP VIEW](./DROP_VIEW.md)