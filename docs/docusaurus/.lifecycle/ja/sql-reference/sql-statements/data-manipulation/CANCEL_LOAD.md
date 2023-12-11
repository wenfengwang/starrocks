---
displayed_sidebar: "Japanese"
---

# ロードのキャンセル

## 説明

指定されたロードジョブをキャンセルします：[ブローカーロード](../data-manipulation/BROKER_LOAD.md)、[スパークロード](../data-manipulation/SPARK_LOAD.md)、または[INSERT](./INSERT.md)。 `PREPARED`、`CANCELLED`、または`FINISHED`の状態のロードジョブはキャンセルできません。

ロードジョブのキャンセルは非同期プロセスです。[SHOW LOAD](../data-manipulation/SHOW_LOAD.md)ステートメントを使用して、ロードジョブが正常にキャンセルされたかどうかを確認できます。 `State`の値が`CANCELLED`であり、`ErrorMsg`に表示される`type`の値が`USER_CANCEL`である場合、ロードジョブは正常にキャンセルされます。

## 構文

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                    |
| ------------- | ---------- | ------------------------------------------------------------ |
| db_name       | いいえ     | ロードジョブが属するデータベースの名前。このパラメータが指定されていない場合、デフォルトで現在のデータベースのロードジョブがキャンセルされます。 |
| label_name    | はい       | ロードジョブのラベル。                                      |

## 例

例1：現在のデータベースでラベルが`example_label`のロードジョブをキャンセルする。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

例2：`example_db`データベースでラベルが`example_label`のロードジョブをキャンセルする。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```