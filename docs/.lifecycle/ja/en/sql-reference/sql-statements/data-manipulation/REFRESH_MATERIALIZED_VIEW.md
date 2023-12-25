---
displayed_sidebar: English
---

# MATERIALIZED VIEWのリフレッシュ

## 説明

特定の非同期マテリアライズドビューまたはその中のパーティションを手動でリフレッシュします。

> **注意**
>
> 手動でリフレッシュできるのは、ASYNCまたはMANUALリフレッシュ戦略を採用しているマテリアライズドビューのみです。非同期マテリアライズドビューのリフレッシュ戦略は、[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して確認できます。
> この操作には、対象マテリアライズドビューに対するREFRESH権限が必要です。

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
| mv_name                   | はい          | 手動でリフレッシュするマテリアライズドビューの名前。 |
| PARTITION START () END () | いいえ           | 特定の時間間隔内のパーティションを手動でリフレッシュします。 |
| partition_start_date      | いいえ           | 手動でリフレッシュするパーティションの開始日。  |
| partition_end_date        | いいえ           | 手動でリフレッシュするパーティションの終了日。    |
| FORCE                     | いいえ           | このパラメータを指定すると、StarRocksは対応するマテリアライズドビューまたはパーティションを強制的にリフレッシュします。このパラメータを指定しない場合、StarRocksはデータが更新されているかどうかを自動的に判断し、必要に応じてのみパーティションをリフレッシュします。  |
| WITH ... MODE             | いいえ           | リフレッシュタスクを同期または非同期で呼び出します。`SYNC`はリフレッシュタスクを同期的に呼び出し、StarRocksはタスクが成功または失敗するまで結果を返しません。`ASYNC`はリフレッシュタスクを非同期的に呼び出し、StarRocksはタスクが提出された直後に成功を返し、タスクはバックグラウンドで非同期に実行されます。非同期マテリアライズドビューのリフレッシュタスクのステータスは、StarRocksのInformation Schema内の`tasks`および`task_runs`メタデータビューをクエリすることで確認できます。詳細は[非同期マテリアライズドビューの実行ステータスの確認](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)を参照してください。デフォルト: `ASYNC`。v3.1.0以降でサポート。|

> **注意**
>
> 外部カタログに基づいて作成されたマテリアライズドビューをリフレッシュする場合、StarRocksはマテリアライズドビューのすべてのパーティションをリフレッシュします。

## 例

例1: 非同期呼び出しを使用して特定のマテリアライズドビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

例2: 特定のマテリアライズドビューの特定のパーティションを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

例3: 特定のマテリアライズドビューの特定のパーティションを強制的にリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

例4: 同期呼び出しを使用してマテリアライズドビューを手動でリフレッシュします。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```
