---
displayed_sidebar: Chinese
---

# REFRESH MATERIALIZED VIEW

## 機能

指定された非同期マテリアライズドビュー、またはその一部のパーティションを手動でリフレッシュします。

> **注意**
>
> ASYNC または MANUAL のリフレッシュ方式である非同期マテリアライズドビューのみ、このコマンドで手動リフレッシュが可能です。リフレッシュ方式は [SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md) で確認できます。
> この操作には対応するマテリアライズドビューの REFRESH 権限が必要です。

## 文法

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## パラメータ

| **パラメータ**             | **必須**     | **説明**                     |
| ------------------------- | ------------ | ---------------------------- |
| mv_name                   | はい         | 手動でリフレッシュする非同期マテリアライズドビューの名前。 |
| PARTITION START () END () | いいえ       | 指定された時間範囲内のパーティションを手動でリフレッシュします。 |
| partition_start_date      | いいえ       | 手動でリフレッシュするパーティションの開始日時。 |
| partition_end_date        | いいえ       | 手動でリフレッシュするパーティションの終了日時。 |
| FORCE                     | いいえ       | このパラメータを指定すると、StarRocks は対応するマテリアライズドビューまたはパーティションを強制的にリフレッシュします。指定しない場合、StarRocks はデータが更新されたかどうかを自動的に判断し、必要に応じてパーティションをリフレッシュします。|
| WITH ... MODE             | いいえ       | リフレッシュタスクを同期または非同期で呼び出します。`SYNC` は同期呼び出しで、SQL ステートメント実行後、リフレッシュタスクが成功または失敗した後に結果が返されます。`ASYNC` は非同期呼び出しで、SQL ステートメント実行後、リフレッシュタスクが提出された直後に成功が返され、実際のリフレッシュタスクはバックグラウンドで非同期に実行されます。非同期マテリアライズドビューのリフレッシュ状態は、StarRocks の Information Schema の `tasks` および `task_runs` メタデータビューを照会することで確認できます。詳細は[非同期マテリアライズドビューの実行状態を確認する](../../../using_starrocks/Materialized_view.md#非同期マテリアライズドビューの実行状態を確認する)を参照してください。デフォルト値は `ASYNC` です。v3.1.0 からサポートされています。|

> **注意**
>
> 外部データカタログ（External Catalog）を基に作成された非同期マテリアライズドビューをリフレッシュする場合、StarRocks はすべてのパーティションをリフレッシュします。

## 例

例1：非同期タスクを呼び出して特定のマテリアライズドビューを手動でリフレッシュします。

```SQL
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

例2：マテリアライズドビューの特定のパーティションを手動でリフレッシュします。

```SQL
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

例3：マテリアライズドビューの特定のパーティションを強制的に手動でリフレッシュします。

```SQL
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

例4：同期タスクを呼び出して特定のマテリアライズドビューを手動でリフレッシュします。

```SQL
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```
