---
displayed_sidebar: "Japanese"
---

# マテリアライズド・ビューの変更

## 説明

このSQLステートメントは、以下の操作を行うことができます：

- 非同期マテリアライズド・ビューの名前を変更します。
- 非同期マテリアライズド・ビューの更新方法を変更します。
- 非同期マテリアライズド・ビューの状態をアクティブまたはインアクティブに変更します。
- 2つの非同期マテリアライズド・ビュー間でアトミックなスワップを実行します。
- 非同期マテリアライズド・ビューのプロパティを変更します。

  このSQLステートメントを使用して、以下のプロパティを変更できます：

  - `partition_ttl_number`
  - `partition_refresh_number`
  - `resource_group`
  - `auto_refresh_partitions_limit`
  - `excluded_trigger_tables`
  - `mv_rewrite_staleness_second`
  - `unique_constraints`
  - `foreign_key_constraints`
  - `colocate_with`
  - セッション変数関連のすべてのプロパティ。セッション変数については、[システム変数](../../../reference/System_variable.md)を参照してください。

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

| **パラメータ**           | **必須** | **説明**                                              |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| mv_name                 | はい          | 変更するマテリアライズド・ビューの名前。                  |
| new_refresh_scheme_desc | いいえ           | 新しい更新方法。詳細については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。 |
| new_mv_name             | いいえ           | マテリアライズド・ビューの新しい名前。                          |
| ACTIVE                  | いいえ           | マテリアライズド・ビューの状態をアクティブに設定します。StarRocksは、基本テーブルのいずれかが変更された場合（例：削除および再作成）、自動的にマテリアライズド・ビューを非アクティブに設定して、元のメタデータが変更された基本テーブルと一致しない状況を防止します。非アクティブなマテリアライズド・ビューは、クエリの高速化やクエリの書き換えに使用することはできません。これらの基本テーブルを変更した後にマテリアライズド・ビューをアクティブにするためにこのSQLを使用できます。 |
| INACTIVE                | いいえ           | マテリアライズド・ビューの状態を非アクティブに設定します。非アクティブな非同期マテリアライズド・ビューは更新できませんが、テーブルとしてはクエリできます。 |
| SWAP WITH               | いいえ           | 必要な整合性チェックの後、他の非同期マテリアライズド・ビューとアトミックな交換を実行します。 |
| key                     | いいえ           | 変更するプロパティ名。詳細については、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。<br />**注意**<br />マテリアライズド・ビューのセッション変数関連のプロパティを変更する場合、「session.query_timeout」のようにプロパティに「session.」の接頭辞を追加する必要があります。セッションを使用しないプロパティについては、「mv_rewrite_staleness_second」のように接頭辞を指定する必要はありません。 |
| value                   | いいえ           | 変更するプロパティの値。                         |

## 例

例1：マテリアライズド・ビューの名前を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

例2：マテリアライズド・ビューの更新間隔を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

例3：マテリアライズド・ビューのプロパティを変更する。

```SQL
-- mv1のquery_timeoutを40000秒に変更する。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- mv1のmv_rewrite_staleness_secondを600秒に変更する。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

例4：マテリアライズド・ビューの状態をアクティブに変更する。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

例5：マテリアライズド・ビュー`order_mv`と`order_mv1`の間でアトミックな交換を行う。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```