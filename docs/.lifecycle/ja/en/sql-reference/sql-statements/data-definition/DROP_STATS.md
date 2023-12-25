---
displayed_sidebar: English
---

# DROP STATS

## 説明

CBOの基本統計とヒストグラムを含む統計情報を削除します。詳細については、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

不要な統計情報を削除することができます。統計を削除すると、統計のデータとメタデータが削除されるだけでなく、期限切れのキャッシュ内の統計も削除されます。自動収集タスクが進行中の場合、以前に削除された統計が再び収集される可能性があることに注意してください。`SHOW ANALYZE STATUS`を使用して、収集タスクの履歴を確認することができます。

このステートメントはv2.4からサポートされています。

## 構文

### 基本統計の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参照

CBOの統計収集についての詳細は、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
