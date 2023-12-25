---
displayed_sidebar: English
---

# ALTER MATERIALIZED VIEW

## 説明

このSQLステートメントは以下の操作を行うことができます：

- 非同期マテリアライズドビューの名前を変更する。
- 非同期マテリアライズドビューのリフレッシュ戦略を変更する。
- 非同期マテリアライズドビューのステータスをアクティブまたは非アクティブに変更する。
- 二つの非同期マテリアライズドビュー間でアトミックスワップを実行する。
- 非同期マテリアライズドビューのプロパティを変更する。

  このSQLステートメントで変更できるプロパティは以下の通りです：

  - `partition_ttl_number`
  - `partition_refresh_number`
  - `resource_group`
  - `auto_refresh_partitions_limit`
  - `excluded_trigger_tables`
  - `mv_rewrite_staleness_second`
  - `unique_constraints`
  - `foreign_key_constraints`
  - `colocate_with`
  - セッション変数に関連するすべてのプロパティ。セッション変数についての情報は、[システム変数](../../../reference/System_variable.md)を参照してください。

:::tip

この操作には、対象のマテリアライズドビューに対するALTER権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の指示に従ってください。

:::

## 構文

```SQL
ALTER MATERIALIZED VIEW [db_name.]<mv_name> 
    { RENAME [db_name.]<new_mv_name> 
    | REFRESH <new_refresh_scheme_desc> 
    | ACTIVE | INACTIVE 
    | SWAP WITH [db_name.]<mv2_name>
    | SET ( "<key>" = "<value>"[,...]) }
```

角括弧[]内のパラメータはオプショナルです。

## パラメータ

| **パラメータ**           | **必須** | **説明**                                              |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| mv_name                 | はい          | 変更するマテリアライズドビューの名前。                  |
| new_refresh_scheme_desc | いいえ           | 新しいリフレッシュ戦略については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。 |
| new_mv_name             | いいえ           | マテリアライズドビューの新しい名前。                          |
| ACTIVE                  | いいえ           |マテリアライズドビューのステータスをアクティブに設定します。StarRocksは、ベーステーブルが変更された場合（例：削除や再作成など）、元のメタデータが変更されたベーステーブルと一致しない状況を防ぐために、マテリアライズドビューを自動的に非アクティブに設定します。非アクティブなマテリアライズドビューはクエリの高速化やクエリの書き換えには使用できません。このSQLを使用して、ベーステーブルの変更後にマテリアライズドビューをアクティブにすることができます。 |
| INACTIVE                | いいえ           | マテリアライズドビューのステータスを非アクティブに設定します。非アクティブな非同期マテリアライズドビューはリフレッシュできませんが、テーブルとしてクエリすることは可能です。 |
| SWAP WITH               | いいえ           | 必要な整合性チェックの後、別の非同期マテリアライズドビューとアトミック交換を実行します。 |
| key                     | いいえ           | 変更するプロパティの名前については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。<br />**注**<br />マテリアライズドビューのセッション変数関連プロパティを変更する場合は、プロパティ名に`session.`接頭辞を追加する必要があります（例：`session.query_timeout`）。セッション変数でないプロパティの場合は接頭辞を指定する必要はありません（例：`mv_rewrite_staleness_second`）。 |
| value                   | いいえ           | 変更するプロパティの値。                         |

## 例

例 1: マテリアライズドビューの名前を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

例 2: マテリアライズドビューのリフレッシュ間隔を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

例 3: マテリアライズドビューのプロパティを変更する。

```SQL
-- mv1のquery_timeoutを40000秒に変更する。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- mv1のmv_rewrite_staleness_secondを600秒に変更する。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

例 4: マテリアライズドビューのステータスをアクティブに変更する。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

例 5: マテリアライズドビュー`order_mv`と`order_mv1`間でアトミック交換を実行する。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```
