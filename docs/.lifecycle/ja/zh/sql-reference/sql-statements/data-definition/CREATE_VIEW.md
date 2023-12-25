---
displayed_sidebar: Chinese
---

# CREATE VIEW

## 機能

ビューを作成します。

ビュー（または論理ビュー）は仮想テーブルで、そのデータは他の既存の実体テーブルへのクエリ結果から来ています。したがって、ビューは物理的なストレージスペースを占有する必要はありません。ビューに対するすべてのクエリは、そのビューに対応するクエリステートメントのサブクエリとみなされます。

StarRocks がサポートするマテリアライズドビューについては、[同期マテリアライズドビュー](../../../using_starrocks/Materialized_view-single_table.md)と[非同期マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)を参照してください。

> **注意**
>
> この操作には指定されたデータベースの CREATE VIEW 権限が必要です。

## 文法

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

## パラメータ説明

| **パラメータ**  | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| OR REPLACE      | 既存のビューを置き換えます。                                 |
| database        | ビューが属するデータベース名。                               |
| view_name       | ビュー名。                                                   |
| column_name     | ビュー内の列名。ビュー内の列と `query_statement` 内のクエリされた列の数は一致している必要があります。 |
| COMMENT         | ビュー内の列またはビュー自体のコメント。                     |
| query_statement | ビューを作成するためのクエリステートメント。StarRocks がサポートする任意のクエリステートメントが使用できます。 |

## 使用説明

- ビューをクエリするには、そのビューの SELECT 権限と対応するベーステーブルの SELECT 権限が必要です。
- ベーステーブルの変更によりビューを作成するクエリステートメントが実行できなくなった場合、そのビューをクエリするとエラーが発生します。

## 例

例1：テーブル `example_table` の集約クエリに基づいて、データベース `example_db` に `example_view` という名前のビューを作成します。

```SQL
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

例2：テーブル `example_table` の集約クエリに基づいて、データベース `example_db` に `example_view` という名前のビューを作成し、ビューとその列にコメントを設定します。

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
