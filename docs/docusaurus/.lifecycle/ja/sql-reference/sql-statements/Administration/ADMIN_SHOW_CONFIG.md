---
displayed_sidebar: "Japanese"
---

# ADMIN SHOW CONFIG（管理者用コマンド：「CONFIG表示」）

## 説明

現在のクラスタの構成を表示します（現在、FE構成項目のみ表示可能）。これらの構成項目の詳細については、[構成](../../../administration/Configuration.md#fe-configuration-items)を参照してください。

構成項目を設定または変更する場合は、[ADMIN SET CONFIG](ADMIN_SET_CONFIG.md)を使用してください。

## 構文

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "パターン"]
```

注意：

戻り値のパラメータの説明：

```plain text
1. キー:        構成項目名
2. 値:          構成項目の値
3. タイプ:      構成項目のタイプ
4. 変更可能性:  ADMIN SET CONFIGコマンドを介して設定できるかどうか
5. Masterのみ:  リーダーFEにのみ適用されるかどうか
6. コメント:     構成項目の説明
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
    | キー               | 値    | タイプ  | 変更可能性 | Masterのみ | コメント |
    +--------------------+-------+---------+-----------+------------+---------+
    | check_java_version | true  | boolean | false     | false      |         |
    +--------------------+-------+---------+-----------+------------+---------+
    1 行が選択されました (0.00 sec)
    ```