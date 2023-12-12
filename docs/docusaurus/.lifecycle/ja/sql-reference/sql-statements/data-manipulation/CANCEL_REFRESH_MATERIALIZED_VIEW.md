---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューのリフレッシュをキャンセル

## 説明

非同期マテリアライズドビューのリフレッシュタスクをキャンセルします。

## 構文

```SQL
CANCEL REFRESH MATERIALIZED VIEW [<database_name>.]<materialized_view_name>
```

## パラメータ

| **パラメータ**         | **必須**      | **説明**                                                    |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| database_name          | いいえ         | マテリアライズドビューがあるデータベースの名前。このパラメータが指定されていない場合、現在のデータベースが使用されます。 |
| materialized_view_name | はい         | マテリアライズドビューの名前。                               |

## 例

例1: ASYNCリフレッシュマテリアライズドビュー`lo_mv1`のリフレッシュタスクをキャンセルする。

```SQL
CANCEL REFRESH MATERIALIZED VIEW lo_mv1;
```