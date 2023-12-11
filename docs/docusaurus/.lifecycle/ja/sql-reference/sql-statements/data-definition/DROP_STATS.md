---
displayed_sidebar: "Japanese"
---

# DROP STATS（統計情報の削除）

## 説明

基本統計情報とヒストグラムを含むCBO（Cost-based optimizer）統計情報を削除します。詳細については、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

不要な統計情報を削除できます。統計情報を削除すると、データと統計情報のメタデータが削除され、有効期限切れのキャッシュ内の統計情報も削除されます。自動収集タスクが進行中の場合、以前に削除された統計情報が再度収集される可能性があります。収集タスクの履歴は、`SHOW ANALYZE STATUS` を使用して表示できます。

このステートメントは v2.4 からサポートされています。

## 構文

### 基本統計情報の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参照

CBOのための統計情報の収集の詳細については、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。