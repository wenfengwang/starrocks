---
displayed_sidebar: Chinese
---

# ALTER MATERIALIZED VIEW

## 機能

このSQLステートメントにより、以下の操作が可能です:

- マテリアライズドビューの名前を変更する
- マテリアライズドビューのリフレッシュ戦略を変更する
- マテリアライズドビューの状態をActiveまたはInactiveに変更する
- マテリアライズドビューをアトミックに置換する
- マテリアライズドビューの属性を変更する

  以下のマテリアライズドビューの属性をこのSQLステートメントで変更できます:

  - `partition_ttl_number`
  - `partition_refresh_number`
  - `resource_group`
  - `auto_refresh_partitions_limit`
  - `excluded_trigger_tables`
  - `mv_rewrite_staleness_second`
  - `unique_constraints`
  - `foreign_key_constraints`
  - `colocate_with`
  - すべてのセッション変数属性。セッション変数についての詳細は、[システム変数](../../../reference/System_variable.md)を参照してください。

:::tip

この操作には対応するマテリアライズドビューのALTER権限が必要です。権限を付与するには、[GRANT](../account-management/GRANT.md)を参照してください。

:::

## 文法

```SQL
ALTER MATERIALIZED VIEW [db_name.]<mv_name> 
    { RENAME [db_name.]<new_mv_name> 
    | REFRESH <new_refresh_scheme_desc> 
    | ACTIVE | INACTIVE 
    | SWAP WITH [db_name.]<mv2_name>
    | SET ( "<key>" = "<value>"[,...]) }
```

## パラメータ

| **パラメータ**          | **必須** | **説明**                                                     |
| ----------------------- | -------- | ------------------------------------------------------------ |
| mv_name                 | はい     | 変更するマテリアライズドビューの名前。                         |
| new_refresh_scheme_desc | いいえ   | 新しいリフレッシュメカニズムについての詳細は、[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#パラメータ)を参照してください。|
| new_mv_name             | いいえ   | 新しいマテリアライズドビューの名前。                           |
| ACTIVE                  | いいえ   | マテリアライズドビューの状態をActiveに設定します。ベーステーブルに変更があった場合（例えば削除されて再作成された場合）、StarRocksは自動的にそのマテリアライズドビューをInactiveに設定し、元のメタデータが変更後のベーステーブルと一致しない状況を避けます。Inactive状態のマテリアライズドビューはクエリの加速や書き換えには使用できません。ベーステーブルを変更した後、このSQLを使用してマテリアライズドビューをActiveに設定できます。 |
| INACTIVE                | いいえ   | マテリアライズドビューの状態をInactiveに設定します。Inactive状態のマテリアライズドビューはリフレッシュされませんが、テーブルとして直接クエリすることは可能です。 |
| SWAP WITH               | いいえ   | 別のマテリアライズドビューとアトミックに置換します。置換前に、StarRocksは必要な一貫性チェックを行います。|
| key                     | いいえ   | 変更する属性の名前。詳細は[SQLリファレンス - CREATE MATERIALIZED VIEW - パラメータ](../data-definition/CREATE_MATERIALIZED_VIEW.md#パラメータ)を参照してください。<br />**説明**<br />マテリアライズドビューのセッション変数属性を変更する場合は、`session.`プレフィックスをセッション属性に追加する必要があります。例えば、`session.query_timeout`です。セッション属性でない属性にはプレフィックスを指定する必要はありません。例えば、`mv_rewrite_staleness_second`です。|
| value                   | いいえ   | 変更する属性の値。                                           |

## 例

例1: マテリアライズドビューの名前を変更する

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

例2: マテリアライズドビューのリフレッシュ間隔を変更する

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

例3: マテリアライズドビューの属性を変更する

```SQL
-- mv1のquery_timeoutを40000秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- mv1のmv_rewrite_staleness_secondを600秒に変更します。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

例4: マテリアライズドビューの状態をActiveに変更する。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

例5: マテリアライズドビュー`order_mv`と`order_mv1`をアトミックに置換する。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```
