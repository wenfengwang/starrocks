```yaml
---
displayed_sidebar: "Japanese"
---

# MATERIALIZED VIEWの変更

## 概要

このSQL文は以下の操作が可能です：

- 非同期マテリアライズド・ビューの名前を変更します。
- 非同期マテリアライズド・ビューのリフレッシュ戦略を変更します。
- 非同期マテリアライズド・ビューの状態を有効または無効に変更します。
- 2つの非同期マテリアライズド・ビュー間でアトミックスワップを実行します。
- 非同期マテリアライズド・ビューのプロパティを変更します。

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
  - すべてのセッション変数関連のプロパティ。セッション変数に関する情報については、[システム変数](../../../reference/System_variable.md)を参照してください。

## 構文

```SQL
ALTER MATERIALIZED VIEW [db_name.]<mv_name> 
    { RENAME [db_name.]<new_mv_name> 
    | REFRESH <new_refresh_scheme_desc> 
    | ACTIVE | INACTIVE 
    | SWAP WITH [db_name.]<mv2_name>
    | SET ( "<key>" = "<value>"[,...]) }
```

角括弧[]のパラメータは省略可能です。

## パラメータ

| **パラメータ**           | **必須** | **説明**                                                     |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| mv_name                 | はい          | 変更するマテリアライズド・ビューの名前。                  |
| new_refresh_scheme_desc | いいえ           | 新しいリフレッシュ戦略。詳細については[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。 |
| new_mv_name             | いいえ           | マテリアライズド・ビューの新しい名前。                          |
| ACTIVE                  | いいえ           | マテリアライズド・ビューの状態を有効に設定します。StarRocksは、基になるテーブルが変更された場合（たとえば削除されて再作成された場合）に、マテリアライズド・ビューを自動的に無効に設定し、元のメタデータが変更された基になるテーブルと不一致になる状況を防止します。非アクティブなマテリアライズド・ビューは、クエリの高速化やクエリの書き換えには使用できません。このSQLを使用して、基になるテーブルを変更した後にマテリアライズド・ビューを有効にすることができます。 |
| INACTIVE                | いいえ           | マテリアライズド・ビューの状態を無効に設定します。非アクティブな非同期マテリアライズド・ビューはリフレッシュできませんが、テーブルとしてクエリを実行することはできます。 |
| SWAP WITH               | いいえ           | 必要な整合性チェックの後、他の非同期マテリアライズド・ビューとのアトミックな交換を行います。 |
| key                     | いいえ           | 変更するプロパティの名前。詳細については[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters)を参照してください。<br />**注意**<br />マテリアライズド・ビューのセッション変数関連のプロパティを変更する場合は、プロパティに`session.`の接頭辞を付ける必要があります。たとえば、`session.query_timeout`のようにです。セッション以外のプロパティ（例：`mv_rewrite_staleness_second`）の場合には、接頭辞を指定する必要はありません。 |
| value                   | いいえ           | 変更するプロパティの値。                         |

## 例

例1：マテリアライズド・ビューの名前を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

例2：マテリアライズド・ビューのリフレッシュ間隔を変更する。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

例3：マテリアライズド・ビューのプロパティを変更する。

```SQL
-- mv1のquery_timeoutを40000秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- mv1のmv_rewrite_staleness_secondを600秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

例4：マテリアライズド・ビューの状態を有効に変更する。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

例5：マテリアライズド・ビューの間でアトミックな交換を実行する。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```