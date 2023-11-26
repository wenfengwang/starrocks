---
displayed_sidebar: "Japanese"
---

# REFRESH MATERIALIZED VIEWのキャンセル

## 説明

非同期マテリアライズドビューのリフレッシュタスクをキャンセルします。

## 構文

```SQL
CANCEL REFRESH MATERIALIZED VIEW [<database_name>.]<materialized_view_name>
```

## パラメータ

| **パラメータ**          | **必須** | **説明**                                                     |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| database_name          | No           | マテリアライズドビューが存在するデータベースの名前。このパラメータが指定されていない場合、現在のデータベースが使用されます。 |
| materialized_view_name | Yes          | マテリアライズドビューの名前。                               |

## 例

例1: ASYNCリフレッシュマテリアライズドビュー `lo_mv1` のリフレッシュタスクをキャンセルする。

```SQL
CANCEL REFRESH MATERIALIZED VIEW lo_mv1;
```
