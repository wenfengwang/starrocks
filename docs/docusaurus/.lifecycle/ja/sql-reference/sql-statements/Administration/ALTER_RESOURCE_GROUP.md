---
displayed_sidebar: "Japanese"
---

# リソースグループの変更

## 説明

リソースグループの構成を変更します。

## 構文

```SQL
ALTER RESOURCE GROUP リソースグループ名
{  CLASSIFIER1, CLASSIFIER2, ...を追加
 | DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...)
 | DROP ALL
 | リソース制限で WITH 
};
```

## パラメーター

| **パラメーター**  | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| リソースグループ名 | 変更するリソースグループの名前。                    |
| ADD                 | リソースグループに分類子を追加します。分類子の定義方法については、[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md) を参照してください。 |
| DROP                | クラシファイア ID を使用してリソースグループから分類子を削除します。クラシファイアの ID は [SHOW RESOURCE GROUP](../Administration/SHOW_RESOURCE_GROUP.md) ステートメントで確認できます。 |
| DROP ALL            | リソースグループからすべての分類子を削除します。                |
| WITH                | リソースグループのリソース制限を変更します。リソース制限の設定方法については、[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md) を参照してください。 |

## 例

Example 1: リソースグループ `rg1` に新しい分類子を追加します。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

Example 2: リソースグループ `rg1` から ID `300040`、`300041`、`300041` の分類子を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

Example 3: リソースグループ `rg1` からすべての分類子を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

Example 4: リソースグループ `rg1` のリソース制限を変更します。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```