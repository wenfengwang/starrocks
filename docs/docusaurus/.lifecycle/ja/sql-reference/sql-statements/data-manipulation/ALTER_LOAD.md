```markdown
---
displayed_sidebar: "Japanese"
---

# LOADの変更

## 説明

**QUEUEING**または**LOADING**の状態にあるBroker Loadのジョブの優先度を変更します。この文はv2.5からサポートされています。

> **注意**
>
> **LOADING**状態にあるBroker Loadのジョブの優先度を変更しても、ジョブの実行には影響しません。

## 構文

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## パラメータ

| **パラメータ** | **必須** | 説明                                                  |
| ------------- | ------------ | ------------------------------------------------------------ |
| label_name    | Yes          | ロードジョブのラベル。フォーマット：`[<database_name>.]<label_name>`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)を参照してください。 |
| priority      | Yes          | ロードジョブに指定する新しい優先度。有効な値：`LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md)を参照してください。 |

## 例

ラベルが`test_db.label1`で**QUEUEING**状態にあるBroker Loadのジョブがあるとします。ジョブをできるだけすぐに実行したい場合は、以下のコマンドを実行してジョブの優先度を`HIGHEST`に変更できます：

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```