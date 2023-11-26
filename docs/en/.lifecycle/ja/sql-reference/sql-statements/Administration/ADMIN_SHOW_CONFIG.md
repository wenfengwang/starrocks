---
displayed_sidebar: "Japanese"
---

# ADMIN SHOW CONFIG

## 説明

現在のクラスタの設定を表示します（現在、FEの設定項目のみ表示できます）。これらの設定項目の詳細な説明については、[Configuration](../../../administration/Configuration.md#fe-configuration-items)を参照してください。

設定項目を設定または変更する場合は、[ADMIN SET CONFIG](ADMIN_SET_CONFIG.md)を使用してください。

## 構文

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"]
```

注意：

戻り値のパラメータの説明：

```plain text
1. Key:        設定項目の名前
2. Value:      設定項目の値
3. Type:       設定項目のタイプ
4. IsMutable:  ADMIN SET CONFIGコマンドを介して設定できるかどうか
5. MasterOnly: リーダーFEにのみ適用されるかどうか
6. Comment:    設定項目の説明
```

## 例

1. 現在のFEノードの設定を表示します。

    ```sql
    ADMIN SHOW FRONTEND CONFIG;
    ```

2. `like`述語を使用して、現在のFEノードの設定を検索します。

    ```plain text
    mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
    +--------------------+-------+---------+-----------+------------+---------+
    | Key                | Value | Type    | IsMutable | MasterOnly | Comment |
    +--------------------+-------+---------+-----------+------------+---------+
    | check_java_version | true  | boolean | false     | false      |         |
    +--------------------+-------+---------+-----------+------------+---------+
    1 row in set (0.00 sec)
    ```
