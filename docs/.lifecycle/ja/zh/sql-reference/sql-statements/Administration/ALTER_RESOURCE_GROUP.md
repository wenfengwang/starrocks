---
displayed_sidebar: Chinese
---

# ALTER RESOURCE GROUP

## 機能

リソースグループの設定を変更します。

:::tip

この操作には、対応するResource GroupのALTER権限が必要です。[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

:::

## 文法

```SQL
ALTER RESOURCE GROUP <resource_group_name>
{  ADD CLASSIFIER1, CLASSIFIER2, ...
 | DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...)
 | DROP ALL
 | WITH resource_limit 
}
```

## パラメータ説明

| **パラメータ**        | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| resource_group_name | 変更するリソースグループの名前です。                         |
| ADD                 | リソースグループに分類器を追加します。分類器の詳細については、[CREATE RESOURCE GROUP - パラメータ説明](../../../sql-reference/sql-statements/Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |
| DROP                | 分類器IDを指定してリソースグループから該当する分類器を削除します。分類器IDは[SHOW RESOURCE GROUP](../../../sql-reference/sql-statements/Administration/SHOW_RESOURCE_GROUP.md)ステートメントで確認できます。 |
| DROP ALL            | リソースグループからすべての分類器を削除します。             |
| WITH                | リソースグループのリソース制限を変更します。リソース制限の詳細については、[CREATE RESOURCE GROUP - パラメータ説明](../../../sql-reference/sql-statements/Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |

## 例

例1: リソースグループ `rg1` に新しい分類器を追加します。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

例2: リソースグループ `rg1` からIDが `300040`、`300041`、`300041` の分類器を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

例3: リソースグループ `rg1` からすべての分類器を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

例4: リソースグループ `rg1` のリソース制限を変更します。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```
