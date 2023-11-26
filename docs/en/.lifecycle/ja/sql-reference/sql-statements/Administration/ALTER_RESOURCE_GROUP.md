---
displayed_sidebar: "Japanese"
---

# ALTER RESOURCE GROUP（リソースグループの変更）

## 説明

リソースグループの設定を変更します。

## 構文

```SQL
ALTER RESOURCE GROUP リソースグループ名
{  ADD クラシファイア1, クラシファイア2, ...
 | DROP (クラシファイアID1, クラシファイアID2, ...)
 | DROP ALL
 | WITH リソース制限 
};
```

## パラメータ

| **パラメータ**         | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| リソースグループ名      | 変更するリソースグループの名前。                                       |
| ADD                 | リソースグループにクラシファイアを追加します。クラシファイアの定義方法については、[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |
| DROP                | リソースグループからクラシファイアをクラシファイアIDで削除します。クラシファイアのIDは、[SHOW RESOURCE GROUP](../Administration/SHOW_RESOURCE_GROUP.md)ステートメントで確認できます。 |
| DROP ALL            | リソースグループからすべてのクラシファイアを削除します。                             |
| WITH                | リソースグループのリソース制限を変更します。リソース制限の設定方法については、[CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)を参照してください。 |

## 例

例1：リソースグループ `rg1` に新しいクラシファイアを追加します。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

例2：リソースグループ `rg1` からクラシファイアID `300040`、`300041`、`300041` を削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

例3：リソースグループ `rg1` からすべてのクラシファイアを削除します。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

例4：リソースグループ `rg1` のリソース制限を変更します。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```
