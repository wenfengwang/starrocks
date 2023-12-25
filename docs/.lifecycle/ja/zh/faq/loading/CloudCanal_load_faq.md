---
displayed_sidebar: Chinese
---

# CloudCanalデータインポートに関するよくある質問

## インポート時のタスク異常、エラー close index failed/too many tablet versions

### CloudCanal側の解決方法

- タスクの詳細を開く
- タスクパラメータを開く
- fullBatchWaitTimeMsとincreBatchWaitTimeMsのパラメータを調整し、StarRocksへのデータ書き込み後の待機時間を増やして、頻繁な書き込みによる異常エラーを防ぐ

![image.png](../../assets/8.2.1.9-1.png)

![image.png](../../assets/8.2.1.9-2.png)

### StarRocks側の解決策

compaction戦略を調整し、マージを加速する（調整後はメモリとIOを観察する必要があります）、be.confで以下の内容を変更する

```properties
cumulative_compaction_num_threads_per_disk = 4
base_compaction_num_threads_per_disk = 2
cumulative_compaction_check_interval_seconds = 2
update_compaction_num_threads_per_disk = 2 （このパラメータはプライマリキーモデル専用のcompactionパラメータです）
```
