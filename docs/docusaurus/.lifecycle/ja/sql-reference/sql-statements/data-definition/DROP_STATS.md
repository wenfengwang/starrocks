---
displayed_sidebar: "Japanese"
---

# DROP STATS（統計情報の削除）

## 説明

基本統計情報やヒストグラムを含むCBO（Cost Based Optimizer）の統計情報を削除します。詳細については、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

必要のない統計情報を削除することができます。統計情報を削除すると、データと統計情報のメタデータ、有効期限が切れたキャッシュ内の統計情報も削除されます。自動収集タスクが進行中である場合、削除された統計情報は以前に収集される場合があります。収集タスクの履歴を表示するには、`SHOW ANALYZE STATUS` を使用できます。

このステートメントはv2.4からサポートされています。

## 構文

### 基本統計情報の削除

```SQL
DROP STATS テーブル名
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE テーブル名 DROP HISTOGRAM ON 列名 [, 列名];
```

## 参照

CBOの統計情報の収集に関する詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。