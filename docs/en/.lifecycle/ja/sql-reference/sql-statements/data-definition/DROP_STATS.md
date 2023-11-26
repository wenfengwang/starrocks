---
displayed_sidebar: "Japanese"
---

# DROP STATS

## 説明

基本統計情報とヒストグラムを含むCBOの統計情報を削除します。詳細については、[CBOの統計情報を収集する](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

必要のない統計情報を削除することができます。統計情報を削除すると、統計情報のデータとメタデータ、および期限切れのキャッシュ内の統計情報が削除されます。ただし、自動収集タスクが進行中の場合、以前に削除された統計情報が再収集される可能性があります。`SHOW ANALYZE STATUS`を使用して収集タスクの履歴を表示することができます。

このステートメントはv2.4からサポートされています。

## 構文

### 基本統計情報の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参考

CBOの統計情報の収集についての詳細は、[CBOの統計情報を収集する](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
