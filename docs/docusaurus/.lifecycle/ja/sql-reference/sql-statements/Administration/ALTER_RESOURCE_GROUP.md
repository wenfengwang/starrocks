---
displayed_sidebar: "Japanese"
---

# ALTER RESOURCE GROUP（リソースグループの変更）

## 説明

リソースグループの構成を変更します。

## 構文

```SQL
ALTER RESOURCE GROUP resource_group_name
{  ADD CLASSIFIER1、CLASSIFIER2、...
 | DROP（CLASSIFIER_ID_1、CLASSIFIER_ID_2、...）
 | DROP ALL
 | WITH resource_limit 
};
```

## パラメータ

| **パラメータ**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| resource_group_name | 変更するリソースグループの名前。                    |
| ADD                 | リソースグループに分類子を追加します。詳細については[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |
| DROP                | リソースグループから分類子を削除します。分類子のIDについては[SHOW RESOURCE GROUP](../Administration/SHOW_RESOURCE_GROUP.md)ステートメントでIDを確認できます。 |
| DROP ALL            | リソースグループからすべての分類子を削除します。                |
| WITH                | リソースグループのリソース制限を修正します。リソース制限の設定方法については[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |

## 例

例1：リソースグループ`rg1`に新しい分類子を追加します。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

例2：リソースグループ`rg1`からIDが`300040`、`300041`、`300041`の分類子を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

例3：リソースグループ`rg1`からすべての分類子を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

例4：リソースグループ`rg1`のリソース制限を修正します。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```