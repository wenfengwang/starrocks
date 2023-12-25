---
displayed_sidebar: English
---

# ALTER LOAD（ロードの変更）

## 説明

**QUEUEING** または **LOADING** 状態にあるBroker Loadジョブの優先順位を変更します。このステートメントはv2.5以降でサポートされています。

> **注記**
>
> **LOADING** 状態にあるBroker Loadジョブの優先順位を変更しても、ジョブの実行には影響しません。

## 構文

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## パラメータ

| **パラメータ** | **必須** | 説明                                                          |
| -------------- | -------- | ------------------------------------------------------------- |
| label_name     | はい     | ロードジョブのラベル。形式: `[<database_name>.]<label_name>`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)を参照してください。 |
| priority       | はい     | ロードジョブに指定したい新しい優先順位。有効な値: `LOWEST`, `LOW`, `NORMAL`, `HIGH`, `HIGHEST`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md)を参照してください。 |

## 例

ラベルが `test_db.label1` のBroker Loadジョブがあり、そのジョブが**QUEUEING**状態にあるとします。ジョブをできるだけ早く実行したい場合、次のコマンドを実行してジョブの優先度を`HIGHEST`に変更できます。

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```
