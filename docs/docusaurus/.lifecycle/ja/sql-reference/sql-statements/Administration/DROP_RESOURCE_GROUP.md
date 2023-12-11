---
displayed_sidebar: "Japanese"
---

# リソースグループの削除

## 説明

指定されたリソースグループを削除します。

## 構文

```SQL
DROP RESOURCE GROUP <resource_group_name>
```

## パラメータ

| **パラメータ**       | **説明**                           |
| ------------------- | ----------------------------------------- |
| resource_group_name | 削除するリソースグループの名前。 |

## 例

例1：リソースグループ `rg1` を削除する。

```SQL
DROP RESOURCE GROUP rg1;
```