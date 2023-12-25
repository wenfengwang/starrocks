---
displayed_sidebar: Chinese
---

# MATERIALIZED VIEWのリフレッシュキャンセル

## 機能

非同期マテリアライズドビューのリフレッシュタスクをキャンセルします。

:::tip

この操作には対象のマテリアライズドビューのREFRESH権限が必要です。

:::

## 文法

```SQL
CANCEL REFRESH MATERIALIZED VIEW [database_name.]materialized_view_name
```

## パラメータ説明

| **パラメータ**         | **必須** | **説明**                                                     |
| ---------------------- | -------- | ------------------------------------------------------------ |
| database_name          | いいえ   | キャンセルするリフレッシュタスクが属するマテリアライズドビューのデータベース名。指定しない場合、現在のデータベースがデフォルトで使用されます。 |
| materialized_view_name | はい     | キャンセルするリフレッシュタスクのマテリアライズドビュー名。 |

## 例

例1：非同期マテリアライズドビュー `lo_mv1` のリフレッシュタスクをキャンセル。

```SQL
CANCEL REFRESH MATERIALIZED VIEW lo_mv1;
```
