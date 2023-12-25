---
displayed_sidebar: Chinese
---

# ALTER MATERIALIZED VIEWの表示

## 機能

同期物化ビューの構築状態を確認します。

:::tip

この操作には権限は必要ありません。

:::

## 文法

```SQL
SHOW ALTER MATERIALIZED VIEW [ { FROM | IN } db_name]
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ   | 確認するデータベース名。このパラメータを指定しない場合、デフォルトで現在のデータベースが使用されます。 |

## 戻り値

| **戻り値**        | **説明**             |
| ----------------- | -------------------- |
| JobId             | ジョブID。            |
| TableName         | テーブル名。          |
| CreateTime        | ジョブの作成時間。    |
| FinishedTime      | ジョブの終了時間。    |
| BaseIndexName     | ベースインデックス名。|
| RollupIndexName   | ロールアップインデックス名。|
| RollupId          | ロールアップID。      |
| TransactionId     | 待機中のトランザクションID。|
| State             | タスクの状態。        |
| Msg               | エラーメッセージ。    |
| Progress          | タスクの進行状況。    |
| Timeout           | タイムアウトの時間。  |

## 例

### 例1：同期物化ビューのリフレッシュタスクを確認する

```Plain
MySQL > SHOW ALTER MATERIALIZED VIEW\G
*************************** 1. row ***************************
          JobId: 475991
      TableName: lineorder
     CreateTime: 2022-08-24 19:46:53
   FinishedTime: 2022-08-24 19:47:15
  BaseIndexName: lineorder
RollupIndexName: lo_mv_sync_1
       RollupId: 475992
  TransactionId: 33067
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
*************************** 2. row ***************************
          JobId: 477337
      TableName: lineorder
     CreateTime: 2022-08-24 19:47:25
   FinishedTime: 2022-08-24 19:47:45
  BaseIndexName: lineorder
RollupIndexName: lo_mv_sync_2
       RollupId: 477338
  TransactionId: 33068
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
2 rows in set (0.00 sec)
```
