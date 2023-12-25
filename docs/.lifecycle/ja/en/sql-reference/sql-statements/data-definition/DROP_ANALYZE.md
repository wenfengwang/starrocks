---
displayed_sidebar: English
---

# DROP ANALYZE

## 説明

カスタム収集タスクを削除します。

デフォルトでは、StarRocksはテーブルの完全な統計を自動的に収集します。5分ごとにデータの更新をチェックし、データ変更が検出された場合、データ収集が自動的にトリガされます。自動完全収集を使用したくない場合は、FE設定項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズできます。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
DROP ANALYZE <ID>
```

タスクIDは、SHOW ANALYZE JOBステートメントを使用して取得できます。

## 例

```SQL
DROP ANALYZE 266030;
```

## 参照

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動収集タスクをカスタマイズする。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): 自動収集タスクのステータスを表示する。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルする。

CBOの統計収集についての詳細は、[CBOのための統計収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
