---
displayed_sidebar: English
---

# リソースグループの削除

## 説明

指定されたリソースグループを削除します。

:::tip

この操作には、対象リソースグループに対するDROP権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の説明に従ってください。

:::

## 構文

```SQL
DROP RESOURCE GROUP <resource_group_name>
```

## パラメータ

| **パラメータ**       | **説明**                                   |
| ------------------- | ----------------------------------------- |
| resource_group_name | 削除するリソースグループの名前。           |

## 例

例1: リソースグループ `rg1` を削除します。

```SQL
DROP RESOURCE GROUP rg1;
```
