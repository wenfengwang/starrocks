---
displayed_sidebar: Chinese
---

# データレイク関連 FAQ

本文では、データレイクに関する一般的な質問とその解決策を紹介します。多くの指標については、`set enable_profile=true` を設定して SQL クエリのプロファイルを収集し分析する必要があります。

## HDFS の遅いノード問題

### 問題の説明

HDFS に保存されたデータファイルにアクセスする際、SQL クエリのプロファイルで `__MAX_OF_FSIOTime` と `__MIN_OF_FSIOTime` の二つの指標の値に大きな差がある場合、HDFS の遅いノードが存在することを示しています。以下に示すプロファイルは、HDFS の遅いノードの典型的なシナリオです：

```plaintext
 - InputStream: 0
   - AppIOBytesRead: 22.72 GB
     - __MAX_OF_AppIOBytesRead: 187.99 MB
     - __MIN_OF_AppIOBytesRead: 64.00 KB
   - AppIOCounter: 964.862K (964862)
     - __MAX_OF_AppIOCounter: 7.795K (7795)
     - __MIN_OF_AppIOCounter: 1
   - AppIOTime: 1s372ms
     - __MAX_OF_AppIOTime: 4s358ms
     - __MIN_OF_AppIOTime: 1.539ms
   - FSBytesRead: 15.40 GB
     - __MAX_OF_FSBytesRead: 127.41 MB
     - __MIN_OF_FSBytesRead: 64.00 KB
   - FSIOCounter: 1.637K (1637)
     - __MAX_OF_FSIOCounter: 12
     - __MIN_OF_FSIOCounter: 1
   - FSIOTime: 9s357ms
     - __MAX_OF_FSIOTime: 60s335ms
     - __MIN_OF_FSIOTime: 1.536ms
```

### 解決策

現在、二つの解決策があります：

- 【推奨】[Data Cache](../data_source/data_cache.md) を有効にする。BE ノードへのリモートデータの自動キャッシュにより、HDFS の遅いノードがクエリに与える影響を排除します。
- [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html) 機能を有効にする。有効にすると、データブロックからのデータ読み取りが遅い場合、StarRocks は新しい Read タスクを開始し、元の Read タスクと並行して、データブロックのレプリカからデータを読み取ります。どちらの Read タスクが先に結果を返しても、もう一方の Read タスクはキャンセルされます。**Hedged Read はデータ読み取り速度を向上させることができますが、Java 仮想マシン（JVM）のヒープメモリ消費が大幅に増加するため、物理メモリが少ない場合は Hedged Read を有効にすることは推奨されません。**

#### 【推奨】Data Cache

[Data Cache](../data_source/data_cache.md) を参照してください。

#### Hedged Read

BE の設定ファイル `be.conf` で以下のパラメーター（バージョン 3.0 からサポート）を使用して、HDFS クラスターの Hedged Read 機能を有効にし、設定します。

| パラメータ名                                 | デフォルト値 | 説明                                                         |
| ---------------------------------------- | ------ | ------------------------------------------------------------ |
| hdfs_client_enable_hedged_read           | false  | Hedged Read 機能を有効にするかどうかを指定します。 |
| hdfs_client_hedged_read_threadpool_size  | 128    | HDFS クライアント側の Hedged Read スレッドプールのサイズを指定します。つまり、HDFS クライアント側で Hedged Read をサービスするために許可されるスレッドの数です。このパラメータは HDFS クラスターの設定ファイル `hdfs-site.xml` の `dfs.client.hedged.read.threadpool.size` パラメータに対応しています。 |
| hdfs_client_hedged_read_threshold_millis | 2500   | Hedged Read リクエストを開始する前に待機するミリ秒数を指定します。例えば、このパラメータが `30` に設定されている場合、Read タスクが 30 ミリ秒以内に結果を返さない場合、HDFS クライアントは直ちに Hedged Read を開始し、データブロックのレプリカからデータを読み取ります。このパラメータは HDFS クラスターの設定ファイル `hdfs-site.xml` の `dfs.client.hedged.read.threshold.millis` パラメータに対応しています。 |

プロファイルで以下のいずれかの指標が `0` より大きい値を示している場合、Hedged Read 機能が正常に有効になっていることを意味します。

| 指標                            | 説明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| TotalHedgedReadOps             | Hedged Read を開始した回数。                                      |
| TotalHedgedReadOpsInCurThread  | Hedged Read スレッドプールのサイズ制限（`hdfs_client_hedged_read_threadpool_size` で設定）により新しいスレッドを開始できず、現在のスレッド内で Hedged Read をトリガーした回数。 |
| TotalHedgedReadOpsWin          | Hedged Read が元の Read よりも早く結果を返した回数。 |
