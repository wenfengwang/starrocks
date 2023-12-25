---
displayed_sidebar: Chinese
---

# DROP STATS

## 機能

CBO統計情報を削除します。

StarRocksは統計情報の手動削除をサポートしています。統計情報を手動で削除すると、統計情報データと統計情報メタデータが削除され、期限切れの統計情報キャッシュもメモリから削除されます。自動収集タスクが現在存在する場合、削除された統計情報が再収集される可能性があることに注意してください。`SHOW ANALYZE STATUS`を使用して、統計情報の収集履歴を確認できます。このステートメントはバージョン2.4からサポートされています。

基本統計情報を削除するにはDROP STATSを使用し、ヒストグラム統計情報を削除するにはANALYZE TABLE DROP HISTOGRAMを使用します。

### 基本統計情報の削除

#### 構文

```SQL
DROP STATS <tbl_name>
```

### ヒストグラム統計情報の削除

#### 構文

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON <col_name> [, <col_name>]
```

## 関連文書

CBO統計情報の収集についての詳細は、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
