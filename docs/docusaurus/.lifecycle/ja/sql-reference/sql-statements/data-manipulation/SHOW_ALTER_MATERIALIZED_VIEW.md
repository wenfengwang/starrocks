---
displayed_sidebar: "英語"
---

# MATERIALIZED VIEW の変更状態の表示

## 説明

同期物化ビューの構築状態を表示します。

## 構文

```SQL
SHOW ALTER MATERIALIZED VIEW [ { FROM | IN } db_name]
```

角括弧 [] 内のパラメータはオプションです。

## パラメータ

| **パラメータ** | **必須** | **説明**                                                      |
| --------------- | -------- | ------------------------------------------------------------- |
| db_name         | いいえ   | 物化ビューが存在するデータベースの名前。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。 |

## 戻り値

| **戻り値**        | **説明**                                        |
| ----------------- | ---------------------------------------------- |
| JobId             | リフレッシュジョブのID。                         |
| TableName         | テーブルの名前。                                |
| CreateTime        | リフレッシュジョブが作成された時刻。              |
| FinishedTime      | リフレッシュジョブが完了した時刻。                |
| BaseIndexName     | ベーステーブルの名前。                          |
| RollupIndexName   | 物化ビューの名前。                             |
| RollupId          | 物化ビューロールアップのID。                   |
| TransactionId     | 実行を待っているトランザクションのID。           |
| State             | ジョブの状態。                                  |
| Msg               | エラーメッセージ。                              |
| Progress          | リフレッシュジョブの進行状況。                   |
| Timeout           | リフレッシュジョブのタイムアウト。               |

## 例

例1: 同期物化ビューの構築状態

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