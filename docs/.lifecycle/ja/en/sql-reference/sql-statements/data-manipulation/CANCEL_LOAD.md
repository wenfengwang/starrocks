---
displayed_sidebar: English
---

# CANCEL LOAD

## 説明

指定されたロードジョブをキャンセルします：[Broker Load](../data-manipulation/BROKER_LOAD.md)、[Spark Load](../data-manipulation/SPARK_LOAD.md)、または[INSERT](./INSERT.md)。`PREPARED`、`CANCELLED`、または`FINISHED`状態のロードジョブはキャンセルできません。

ロードジョブのキャンセルは非同期プロセスです。ロードジョブが正常にキャンセルされたかどうかを確認するために、[SHOW LOAD](../data-manipulation/SHOW_LOAD.md)文を使用できます。`State`の値が`CANCELLED`であり、`ErrorMsg`に表示される`type`の値が`USER_CANCEL`である場合、ロードジョブは正常にキャンセルされたとみなされます。

## 構文

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | ロードジョブが属するデータベースの名前です。このパラメーターが指定されていない場合、現在のデータベース内のロードジョブがデフォルトでキャンセルされます。 |
| label_name    | はい          | ロードジョブのラベルです。                                   |

## 例

例 1: 現在のデータベースでラベルが`example_label`のロードジョブをキャンセルします。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

例 2: `example_db`データベースでラベルが`example_label`のロードジョブをキャンセルします。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```
