---
displayed_sidebar: "Japanese"
---

# SHOW ALTER MATERIALIZED VIEW

## 説明

同期マテリアライズドビューのビルド状態を表示します。

## 構文

```SQL
SHOW ALTER MATERIALIZED VIEW [ { FROM | IN } db_name]
```

角括弧 [] 内のパラメータはオプションです。

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | --------- | ------------------------------------------------------------ |
| db_name        | いいえ    | マテリアライズドビューが存在するデータベースの名前です。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。 |

## 戻り値

| **戻り値**        | **説明**                                         |
| ----------------- | ------------------------------------------------- |
| JobId             | リフレッシュジョブのIDです。                       |
| TableName         | テーブルの名前です。                               |
| CreateTime        | リフレッシュジョブが作成された時刻です。           |
| FinishedTime      | リフレッシュジョブが完了した時刻です。             |
| BaseIndexName     | ベーステーブルの名前です。                         |
| RollupIndexName   | マテリアライズドビューの名前です。                 |
| RollupId          | マテリアライズドビューロールアップのIDです。       |
| TransactionId     | 実行待ちのトランザクションのIDです。               |
| State             | ジョブの状態です。                                 |
| Msg               | エラーメッセージです。                             |
| Progress          | リフレッシュジョブの進行状況です。                 |
| Timeout           | リフレッシュジョブのタイムアウトです。             |

## 例

例1: 同期マテリアライズドビューのビルド状態を表示する

```Plain
MySQL > SHOW ALTER MATERIALIZED VIEW\G
*************************** 1. 行 ***************************
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
*************************** 2. 行 ***************************
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
2 行が返されました (0.00 秒)
```
