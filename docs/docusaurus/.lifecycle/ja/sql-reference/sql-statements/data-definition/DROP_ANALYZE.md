---
displayed_sidebar: "Japanese"
---

# DROP ANALYZE

## 説明

カスタムの収集タスクを削除します。

デフォルトでは、StarRocksはテーブルの完全な統計情報を自動的に収集します。データの更新を5分ごとにチェックします。データ変更が検出された場合、データ収集が自動的にトリガーされます。自動完全収集を使用したくない場合は、FE構成項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズすることができます。

この文はv2.4からサポートされています。

## 構文

```SQL
DROP ANALYZE <ID>
```

タスクIDは、SHOW ANALYZE JOB文を使用して取得できます。

## 例

```SQL
DROP ANALYZE 266030;
```

## 参照

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動収集タスクをカスタマイズします。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): 自動収集タスクのステータスを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計情報の収集方法についての詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)をご覧ください。