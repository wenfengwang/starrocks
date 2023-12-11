---
displayed_sidebar: "Japanese"
---

# ADMIN SHOW CONFIG（表示）

## 概要

現在のクラスタの構成を表示します（現在、FEの構成項目のみ表示可能です）。これらの構成項目の詳細については、[設定](../../../administration/Configuration.md#fe-configuration-items)を参照してください。

構成項目を設定または変更したい場合は、[ADMIN SET CONFIG](ADMIN_SET_CONFIG.md)を使用してください。

## 構文

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"]
```

注意：

返されるパラメータの説明：

```plain text
1. Key:        構成項目名
2. Value:      構成項目値
3. Type:       構成項目タイプ 
4. IsMutable:  ADMIN SET CONFIG コマンドを介して設定できるかどうか
5. MasterOnly: リーダーFEにのみ適用されるかどうか
6. Comment:    構成項目の説明 
```

## 例

1. 現在のFEノードの構成を表示します。

    ```sql
    ADMIN SHOW FRONTEND CONFIG;
    ```

2. `like`述語を使用して現在のFEノードの構成を検索します。

    ```plain text
    mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
    +--------------------+-------+---------+-----------+------------+---------+
    | Key                | Value | Type    | IsMutable | MasterOnly | Comment |
    +--------------------+-------+---------+-----------+------------+---------+
    | check_java_version | true  | boolean | false     | false      |         |
    +--------------------+-------+---------+-----------+------------+---------+
    1 行がセットされました (0.00 sec)
    ```