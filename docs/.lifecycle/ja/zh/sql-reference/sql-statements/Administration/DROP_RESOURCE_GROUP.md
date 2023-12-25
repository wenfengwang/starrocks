---
displayed_sidebar: Chinese
---

# DROP RESOURCE GROUP

## 機能

指定されたリソースグループを削除します。

:::tip

この操作には対応するリソースグループのDROP権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```SQL
DROP RESOURCE GROUP <resource_group_name>
```

## パラメータ説明

| **パラメータ**      | **説明**           |
| ------------------- | ------------------ |
| resource_group_name | 削除するリソースグループ名。 |

## 例

例1：リソースグループ `rg1` を削除します。

```SQL
DROP RESOURCE GROUP rg1;
```
