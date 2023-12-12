```
---
displayed_sidebar: "Japanese"
---

# MATERIALIZED VIEWのリフレッシュ

## 説明

特定の非同期マテリアライズド・ビューまたはそのパーティションを手動でリフレッシュします。

> **注意**
>
> ASYNCまたはMANUALのリフレッシュ戦略を採用するマテリアライズド・ビューのみを手動でリフレッシュできます。非同期マテリアライズド・ビューのリフレッシュ戦略は、[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して確認できます。

## 構文

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## パラメータ

| **パラメータ**             | **必須** | **説明**                                        |
| ------------------------- | ------------ | ------------------------------------------------------ |
| mv_name                   | yes          | 手動でリフレッシュするマテリアライズド・ビューの名前。 |
| PARTITION START () END () | no           | 特定の時間間隔内のパーティションを手動でリフレッシュします。 |
| partition_start_date      | no           | 手動でリフレッシュするパーティションの開始日。  |
| partition_end_date        | no           | 手動でリフレッシュするパーティションの終了日。    |
| FORCE                     | no           | このパラメータを指定すると、StarRocksは対応するマテリアライズド・ビューまたはパーティションを強制的にリフレッシュします。このパラメータを指定しない場合、StarRocksはデータが更新されたかどうかを自動的に判断し、必要に応じてパーティションのみをリフレッシュします。  |
| WITH ... MODE             | no           | リフレッシュタスクを同期または非同期で実行します。`SYNC`はリフレッシュタスクを同期的に実行し、StarRocksはタスクが成功または失敗した場合のみタスク結果を返します。`ASYNC`はリフレッシュタスクを非同期的に実行し、StarRocksはタスクが提出されるとすぐに成功を返し、タスクはバックグラウンドで非同期に実行されます。非同期マテリアライズド・ビューのリフレッシュタスクの状態は、StarRocksのInformation Schemaの`tasks`および`task_runs`メタデータビューをクエリして確認できます。詳細については、[非同期マテリアライズド・ビューの実行状態を確認する](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)を参照してください。デフォルト：`ASYNC`。v3.1.0以降でサポートされています。 |

> **注意**
>
> 外部カタログに基づいて作成されたマテリアライズド・ビューをリフレッシュする場合、StarRocksはすべてのパーティションをリフレッシュします。

## 例

Example 1: 非同期コールを使用して特定のマテリアライズド・ビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

Example 2: 特定のマテリアライズド・ビューの特定のパーティションを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

Example 3: 特定のマテリアライズド・ビューの特定のパーティションを強制的にリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

Example 4: 同期コールを使用してマテリアライズド・ビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```