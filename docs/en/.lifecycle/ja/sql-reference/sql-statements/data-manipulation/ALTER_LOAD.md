---
displayed_sidebar: "Japanese"
---

# ALTER LOAD

## 説明

**QUEUEING**または**LOADING**状態にあるBroker Loadジョブの優先度を変更します。このステートメントはv2.5以降でサポートされています。

> **注意**
>
> **LOADING**状態にあるBroker Loadジョブの優先度を変更しても、ジョブの実行には影響しません。

## 構文

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## パラメータ

| **パラメータ** | **必須** | 説明                                                         |
| ------------- | ---------- | ------------------------------------------------------------ |
| label_name    | Yes        | ロードジョブのラベル。フォーマット：`[<database_name>.]<label_name>`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)を参照してください。 |
| priority      | Yes        | ロードジョブに指定する新しい優先度。有効な値：`LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md)を参照してください。 |

## 例

ラベルが`test_db.label1`で**QUEUEING**状態にあるBroker Loadジョブがあるとします。ジョブをできるだけ早く実行したい場合、次のコマンドを実行してジョブの優先度を`HIGHEST`に変更することができます。

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```
