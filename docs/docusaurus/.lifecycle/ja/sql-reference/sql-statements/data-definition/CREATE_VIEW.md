---
displayed_sidebar: "Japanese"
---

# ビューを作成

## 説明

ビューを作成します。

ビューまたは論理ビューは、他の既存の物理テーブルに対するクエリから派生したデータを持つ仮想テーブルです。したがって、ビューは物理的なストレージを使用せず、ビューへのすべてのクエリはビューを構築するクエリ文のサブクエリと同等です。

StarRocksでサポートされているマテリアライズド・ビューに関する情報については、[同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view-single_table.md) および [非同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view.md) を参照してください。

> **注意**
>
> 特定のデータベースにおける CREATE VIEW 権限を持つユーザーのみがこの操作を実行できます。

## 構文

```SQL
CREATE [OR REPLACE] VIEW [IF NOT EXISTS]
[<database>.]<view_name>
(
    <column_nam>[ COMMENT 'column comment']
    [, <column_name>[ COMMENT 'column comment'], ...]
)
[COMMENT 'view comment']
AS <query_statement>
```

## パラメータ

| **パラメータ**  | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| OR REPLACE      | 既存のビューを置き換えます。                                |
| database        | ビューが存在するデータベースの名前。                         |
| view_name       | ビューの名前。                                               |
| column_name     | ビュー内の列の名前。ビュー内の列と `query_statement` でクエリされる列は数が一致する必要があります。 |
| COMMENT         | ビュー内の列またはビュー自体についてのコメント。            |
| query_statement | ビューを作成するために使用されるクエリ文。StarRocksでサポートされているクエリ文であれば何でも使用できます。 |

## 使用上の注意

- ビューをクエリするには、そのビューと対応する基本テーブルに対する SELECT 権限が必要です。
- 基本テーブルのスキーマ変更によりビューを作成するクエリ文が実行できない場合、ビューをクエリすると StarRocks はエラーを返します。

## 例

例1: `example_db` にある `example_table` を対象とする集計クエリを使用して、`example_view` という名前のビューを作成します。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

例2: テーブル `example_table` に対する集計クエリを使用して、データベース `example_db` にある `example_view` という名前のビューを作成し、ビューと各列にコメントを指定します。

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

## 関連する SQL

- [SHOW CREATE VIEW](../data-manipulation/SHOW_CREATE_VIEW.md)
- [ALTER VIEW](./ALTER_VIEW.md)
- [DROP VIEW](./DROP_VIEW.md)