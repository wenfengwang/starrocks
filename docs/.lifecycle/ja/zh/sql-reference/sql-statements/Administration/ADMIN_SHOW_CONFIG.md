---
displayed_sidebar: Chinese
---

# ADMIN SHOW CONFIG

## 機能

このステートメントは、現在のクラスターの設定を表示するために使用されます（現在はFEの設定項目のみ表示がサポートされています）。

各設定項目の意味については、[FE設定項目](../../../administration/FE_configuration.md)を参照してください。

クラスターの設定項目を動的に設定または変更するには、[ADMIN SET CONFIG](ADMIN_SET_CONFIG.md)を参照してください。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

:::

## 文法

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"]
```

説明：

結果の各列の意味は以下の通りです：

```plain text
1. Key         設定項目名
2. AliasNames  設定項目の別名
3. Value       設定項目の値
4. Type        設定項目のデータ型
5. IsMutable   ADMIN SET CONFIGコマンドで動的に設定可能かどうか
6. Comment     設定項目の説明
```

## 例

1. 現在のFEノードの設定を表示します。

    ```sql
    ADMIN SHOW FRONTEND CONFIG;
    ```

2. like述語を使用して現在のFEノードの設定を検索します。

    ```plain text
    mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
    +--------------------+------------+-------+---------+-----------+---------+
    | Key                | AliasNames | Value | Type    | IsMutable | Comment |
    +--------------------+------------+-------+---------+-----------+---------+
    | check_java_version | []         | true  | boolean | false     |         |
    +--------------------+------------+-------+---------+-----------+---------+
    1 row in set (0.00 sec)

    ```
