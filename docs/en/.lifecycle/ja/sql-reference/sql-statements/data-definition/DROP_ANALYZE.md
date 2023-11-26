---
displayed_sidebar: "Japanese"
---

# DROP ANALYZE

## 説明

カスタムのコレクションタスクを削除します。

デフォルトでは、StarRocksはテーブルの完全な統計情報を自動的に収集します。データの更新を5分ごとにチェックします。データの変更が検出されると、データの収集が自動的にトリガされます。自動的な完全な収集を使用したくない場合は、FEの設定項目`enable_collect_full_statistic`を`false`に設定し、コレクションタスクをカスタマイズすることができます。

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

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動コレクションタスクをカスタマイズします。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): 自動コレクションタスクのステータスを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細は、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
