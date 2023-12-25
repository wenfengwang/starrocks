---
displayed_sidebar: Chinese
---

# ロードキャンセル

## 機能

指定された [Broker Load](../data-manipulation/BROKER_LOAD.md)、[Spark Load](../data-manipulation/SPARK_LOAD.md)、または [INSERT](./INSERT.md) のインポートジョブをキャンセルします。PREPARED、CANCELLED、または FINISHED 状態のインポートジョブはキャンセルできません。

CANCEL LOAD は非同期操作であり、実行後に [SHOW LOAD](../data-manipulation/SHOW_LOAD.md) ステートメントを使用してキャンセルが成功したかどうかを確認できます。状態 (`State`) が `CANCELLED` であり、インポートジョブの失敗理由 (`ErrorMsg`) に `USER_CANCEL` という失敗タイプ (`type`) が含まれている場合、インポートジョブが正常にキャンセルされたことを意味します。

## 文法

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                       |
| -------------- | -------- | ---------------------------------------------- |
| db_name        | いいえ   | インポートジョブが存在するデータベースの名前。デフォルトは現在のデータベースです。 |
| label_name     | はい     | インポートジョブのラベル。                     |

## 例

例1：現在のデータベースで`example_label`というラベルのインポートジョブをキャンセルします。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

例2：`example_db`データベースで`example_label`というラベルのインポートジョブをキャンセルします。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```
