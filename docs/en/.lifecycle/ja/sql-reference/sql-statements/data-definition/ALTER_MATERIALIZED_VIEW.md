---
displayed_sidebar: "Japanese"
---

# ALTER MATERIALIZED VIEW

## 説明

このSQL文は以下の操作を行うことができます：

- 非同期マテリアライズドビューの名前を変更します。
- 非同期マテリアライズドビューのリフレッシュ戦略を変更します。
- 非同期マテリアライズドビューのステータスをアクティブまたは非アクティブに変更します。
- 2つの非同期マテリアライズドビュー間でアトミックなスワップを実行します。
- 非同期マテリアライズドビューのプロパティを変更します。

  このSQL文を使用して、以下のプロパティを変更できます：

  - `partition_ttl_number`
  - `partition_refresh_number`
  - `resource_group`
  - `auto_refresh_partitions_limit`
  - `excluded_trigger_tables`
  - `mv_rewrite_staleness_second`
  - `unique_constraints`
  - `foreign_key_constraints`
  - `colocate_with`
  - セッション変数に関連するすべてのプロパティ。セッション変数についての詳細は、[システム変数](../../../reference/System_variable.md)を参照してください。

## 構文

```SQL
ALTER MATERIALIZED VIEW [db_name.]<mv_name> 
    { RENAME [db_name.]<new_mv_name> 
    | REFRESH <new_refresh_scheme_desc> 
    | ACTIVE | INACTIVE 
    | SWAP WITH [db_name.]<mv2_name>
    | SET ( "<key>" = "<value>"[,...]) }
```

角括弧[]内のパラメータはオプションです。

## パラメータ

| **パラメータ**           | **必須** | **説明**                                                     |
| ----------------------- | ---------- | ------------------------------------------------------------ |
| mv_name                 | yes          | 変更するマテリアライズドビューの名前。                         |
| new_refresh_scheme_desc | no           | 新しいリフレッシュ戦略。詳細については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。 |
| new_mv_name             | no           | マテリアライズドビューの新しい名前。                           |
| ACTIVE                  | no           | マテリアライズドビューのステータスをアクティブに設定します。StarRocksは、ベーステーブルのいずれかが変更された場合（例：削除および再作成）、元のメタデータが変更されたベーステーブルと一致しない状況を防ぐため、マテリアライズドビューを自動的に非アクティブに設定します。非アクティブなマテリアライズドビューは、クエリの高速化やクエリの書き換えに使用することはできません。ベーステーブルを変更した後、このSQLを使用してマテリアライズドビューをアクティブにすることができます。 |
| INACTIVE                | no           | マテリアライズドビューのステータスを非アクティブに設定します。非アクティブな非同期マテリアライズドビューはリフレッシュできませんが、テーブルとしてクエリすることはできます。 |
| SWAP WITH               | no           | 必要な整合性チェック後に、他の非同期マテリアライズドビューとのアトミックな交換を実行します。 |
| key                     | no           | 変更するプロパティの名前。詳細については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。<br />**注意**<br />マテリアライズドビューのセッション変数に関連するプロパティを変更する場合は、プロパティに `session.` の接頭辞を追加する必要があります。例えば、`session.query_timeout` です。セッションに関連しないプロパティの場合は、接頭辞を指定する必要はありません。例えば、`mv_rewrite_staleness_second` です。 |
| value                   | no           | 変更するプロパティの値。                                     |

## 例

例1：マテリアライズドビューの名前を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

例2：マテリアライズドビューのリフレッシュ間隔を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

例3：マテリアライズドビューのプロパティを変更する。

```SQL
-- mv1のquery_timeoutを40000秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- mv1のmv_rewrite_staleness_secondを600秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

例4：マテリアライズドビューのステータスをアクティブに変更する。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

例5：マテリアライズドビュー `order_mv` と `order_mv1` の間でアトミックな交換を実行する。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```
