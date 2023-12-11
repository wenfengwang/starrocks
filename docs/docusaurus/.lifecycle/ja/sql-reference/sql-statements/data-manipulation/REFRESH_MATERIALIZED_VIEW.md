---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューのリフレッシュ

## 説明

特定の非同期マテリアライズドビューまたはそのパーティションを手動でリフレッシュします。

> **注意**
>
> ASYNCまたはMANUALのリフレッシュ戦略を採用したマテリアライズドビューのみ手動でリフレッシュできます。非同期マテリアライズドビューのリフレッシュ戦略は、[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して確認できます。

## 構文

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## パラメータ

| **パラメータ**           | **必須** | **説明**                                                   |
| ------------------------- | ------------ | ------------------------------------------------------ |
| mv_name                   | yes          | 手動でリフレッシュするマテリアライズドビューの名前。 |
| PARTITION START () END () | no           | 特定の時間間隔内のパーティションを手動でリフレッシュします。 |
| partition_start_date      | no           | 手動でリフレッシュするパーティションの開始日。  |
| partition_end_date        | no           | 手動でリフレッシュするパーティションの終了日。    |
| FORCE                     | no           | このパラメータを指定すると、StarRocksは対応するマテリアライズドビューまたはパーティションを強制的にリフレッシュします。このパラメータを指定しない場合、StarRocksはデータが更新されたかどうかを自動的に判断し、必要に応じてパーティションをリフレッシュします。  |
| WITH ... MODE             | no           | リフレッシュタスクの同期または非同期の呼び出しを行います。`SYNC`はリフレッシュタスクの同期的な呼び出しを示し、StarRocksはタスクが成功または失敗したときだけタスクの結果を返します。`ASYNC`はリフレッシュタスクの非同期な呼び出しを示し、StarRocksはタスクを提出した直後に成功を返し、タスクをバックグラウンドで非同期に実行します。非同期マテリアライズドビューのリフレッシュタスクのステータスは、StarRocksのInformation Schemaの`tasks`および`task_runs`メタデータビューをクエリすることで確認できます。詳細については、[非同期マテリアライズドビューの実行ステータスを確認する](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)を参照してください。デフォルト値: `ASYNC`。v3.1.0以降でサポートされています。 |

> **注意**
>
> 外部カタログに基づいて作成されたマテリアライズドビューをリフレッシュする場合、StarRocksはすべてのパーティションをリフレッシュします。

## 例

例1: 非同期呼び出しで特定のマテリアライズドビューを手動でリフレッシュする。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

例2: 特定のマテリアライズドビューの特定のパーティションを手動でリフレッシュする。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

例3: 特定のマテリアライズドビューの特定のパーティションを強制的にリフレッシュする。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

例4: 同期呼び出しでマテリアライズドビューを手動でリフレッシュする。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```