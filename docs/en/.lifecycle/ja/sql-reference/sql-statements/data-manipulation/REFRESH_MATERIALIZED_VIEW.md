---
displayed_sidebar: "Japanese"
---

# REFRESH MATERIALIZED VIEW

## 説明

特定の非同期マテリアライズドビューまたはパーティションを手動でリフレッシュします。

> **注意**
>
> ASYNCまたはMANUALのリフレッシュ戦略を採用しているマテリアライズドビューのみを手動でリフレッシュできます。非同期マテリアライズドビューのリフレッシュ戦略は、[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して確認できます。

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
| mv_name                   | yes          | 手動でリフレッシュするマテリアライズドビューの名前。 |
| PARTITION START () END () | no           | 特定の時間間隔内のパーティションを手動でリフレッシュします。 |
| partition_start_date      | no           | 手動でリフレッシュするパーティションの開始日。  |
| partition_end_date        | no           | 手動でリフレッシュするパーティションの終了日。    |
| FORCE                     | no           | このパラメータを指定すると、StarRocksは対応するマテリアライズドビューまたはパーティションを強制的にリフレッシュします。このパラメータを指定しない場合、StarRocksはデータが更新されたかどうかを自動的に判断し、必要な場合にのみパーティションをリフレッシュします。  |
| WITH ... MODE             | no           | リフレッシュタスクの同期または非同期呼び出しを行います。`SYNC`はリフレッシュタスクの同期呼び出しを行い、StarRocksはタスクが成功または失敗した場合にのみタスクの結果を返します。`ASYNC`はリフレッシュタスクの非同期呼び出しを行い、StarRocksはタスクが送信された直後に成功を返し、タスクはバックグラウンドで非同期に実行されます。非同期マテリアライズドビューのリフレッシュタスクの実行状態は、StarRocksのInformation Schemaの`tasks`および`task_runs`メタデータビューをクエリして確認できます。詳細については、[非同期マテリアライズドビューの実行状態を確認する](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)を参照してください。デフォルト：`ASYNC`。v3.1.0以降でサポートされています。 |

> **注意**
>
> 外部カタログに基づいて作成されたマテリアライズドビューをリフレッシュする場合、StarRocksはマテリアライズドビューのすべてのパーティションをリフレッシュします。

## 例

例1：非同期呼び出しを使用して特定のマテリアライズドビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

例2：特定のマテリアライズドビューの一部のパーティションを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

例3：特定のマテリアライズドビューの一部のパーティションを強制的にリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

例4：同期呼び出しを使用してマテリアライズドビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```
