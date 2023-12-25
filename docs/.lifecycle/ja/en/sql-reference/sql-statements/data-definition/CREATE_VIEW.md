---
displayed_sidebar: English
---

# CREATE VIEW

## 説明

ビューを作成します。

ビュー（論理ビュー）は、他の既存の物理テーブルに対するクエリからデータが派生する仮想テーブルです。したがって、ビューは物理ストレージを使用せず、ビューに対するすべてのクエリは、ビューを構築するために使用されたクエリステートメントのサブクエリに相当します。

StarRocksがサポートするマテリアライズドビューについては、[同期マテリアライズドビュー](../../../using_starrocks/Materialized_view-single_table.md)および[非同期マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)を参照してください。

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

## パラメーター

| **パラメーター**   | **説明**                                              |
| --------------- | ------------------------------------------------------------ |
| OR REPLACE      | 既存のビューを置き換えます。                                    |
| database        | ビューが存在するデータベースの名前です。             |
| view_name       | ビューの名前です。                                        |
| column_name     | ビュー内の列の名前です。ビューの列と`query_statement`でクエリされる列は数が一致している必要があります。 |
| COMMENT         | ビューまたはビュー自体の列に対するコメントです。    |
| query_statement | ビューを作成するために使用されるクエリステートメントです。StarRocksがサポートする任意のクエリステートメントが使用できます。 |

## 使用上の注意

- ビューをクエリするには、ビューとそれに対応するベーステーブルに対するSELECT権限が必要です。
- ビューを構築するために使用されたクエリステートメントが、ベーステーブルのスキーマ変更によって実行できない場合、StarRocksはビューをクエリする際にエラーを返します。

## 例

例1: `example_db`データベースに`example_view`という名前のビューを作成し、`example_table`に対する集約クエリを使用します。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

例2: `example_db`データベースに`example_view`という名前のビューを作成し、`example_table`に対する集約クエリを使用し、ビューと各列にコメントを指定します。

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
