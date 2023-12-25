---
displayed_sidebar: Chinese
---

# ALTER LOAD の変更

## 機能

**QUEUEING** または **LOADING** 状態の Broker Load ジョブの優先度を変更します。StarRocks は v2.5 バージョンからこのコマンドをサポートしています。

> **注意**
>
> **LOADING** 状態の Broker Load ジョブの優先度を変更しても、ジョブには影響しません。

## 文法

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| label_name     | はい     | インポートジョブのラベルを指定します。形式：`[<database_name>.]<label_name>`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-と-label_name)を参照してください。 |
| priority       | はい     | インポートジョブの優先度を指定します。値の範囲：`LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#opt_properties)を参照してください。 |

## 例

`test_db.label1` というラベルの Broker Load ジョブがあり、そのジョブが現在 **QUEUEING** 状態または **LOADING** 状態にあるとします。そのジョブをできるだけ早く実行したい場合は、以下のコマンドを使用して、ジョブの優先度を `HIGHEST` に変更できます：

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```
