---
displayed_sidebar: "Japanese"
---

# ALTER LOAD（ロードの変更）

## 説明

**QUEUEING** または **LOADING** の状態にあるブローカー ロード ジョブの優先度を変更します。このステートメントは v2.5 からサポートされています。

> **注意**
>
> **LOADING** の状態にあるブローカー ロード ジョブの優先度を変更しても、ジョブの実行には影響しません。

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
| ------------- | ------------ | ------------------------------------------------------------ |
| label_name    | はい          | ロード ジョブのラベル。形式: `[<database_name>.]<label_name>`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)を参照してください。 |
| priority      | はい          | ロード ジョブの指定する新しい優先度。有効な値: `LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST`。[BROKER LOAD](../data-manipulation/BROKER_LOAD.md)を参照してください。 |

## 例

ラベルが `test_db.label1` で状態が **QUEUEING** であるブローカー ロード ジョブがあるとします。そのジョブをできるだけ早く実行したい場合は、次のコマンドを実行してジョブの優先度を `HIGHEST` に変更できます:

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```