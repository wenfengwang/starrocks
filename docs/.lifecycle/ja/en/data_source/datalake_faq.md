---
displayed_sidebar: English
---

# データレイク関連のFAQ

このトピックでは、データレイクに関するよくある質問（FAQ）とそれらの問題に対する解決策について説明します。このトピックで言及される一部のメトリクスは、SQLクエリのプロファイルからのみ取得可能です。SQLクエリのプロファイルを取得するには、`set enable_profile=true`を指定する必要があります。

## 低速のHDFSノード

### 問題の説明

HDFSクラスターに保存されているデータファイルにアクセスする際、実行したSQLクエリのプロファイルから得られる`__MAX_OF_FSIOTime`と`__MIN_OF_FSIOTime`のメトリクスの値に大きな差があることがあり、これはHDFSノードの低速を示しています。以下の例は、HDFSノードの低速問題を示す典型的なプロファイルです：

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

この問題に対しては、以下の解決策のいずれかを使用できます：

- **[推奨]** [データキャッシュ](../data_source/data_cache.md)機能を有効にすることで、外部ストレージシステムからStarRocksクラスタのBEへデータを自動的にキャッシュし、低速なHDFSノードがクエリに与える影響を排除できます。
- [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)機能を有効にします。この機能を有効にすると、ブロックからの読み取りが遅い場合、StarRocksは新しい読み取りを開始し、元の読み取りと並行して異なるブロックレプリカからの読み取りを行います。どちらかの読み取りが完了すると、もう一方はキャンセルされます。**Hedged Read機能は読み取りを加速するのに役立ちますが、Java仮想マシン（JVM）上でのヒープメモリ消費も大幅に増加するため、物理マシンのメモリ容量が少ない場合は、Hedged Read機能を有効にしないことを推奨します。**

#### [推奨] データキャッシュ

[データキャッシュ](../data_source/data_cache.md)を参照してください。

#### Hedged Read

BEの設定ファイル`be.conf`において、以下のパラメーター（v3.0以降でサポート）を使用して、HDFSクラスターでHedged Read機能を有効にし、設定します。

| パラメーター                                | 既定値 | 説明                                                         |
| ---------------------------------------- | ------------- | ------------------------------------------------------------------- |
| hdfs_client_enable_hedged_read           | false         | Hedged Read機能を有効にするかどうかを指定します。                                    |
| hdfs_client_hedged_read_threadpool_size  | 128           | HDFSクライアント上のHedged Readスレッドプールのサイズを指定します。スレッドプールのサイズは、HDFSクライアントで実行されるHedged Readのスレッド数を制限します。このパラメーターは、HDFSクラスタの`hdfs-site.xml`ファイル内の`dfs.client.hedged.read.threadpool.size`パラメーターに相当します。 |
| hdfs_client_hedged_read_threshold_millis | 2500          | Hedged Readを開始する前に待機するミリ秒数を指定します。例えば、このパラメーターを`30`に設定した場合、ブロックからの読み取りが30ミリ秒以内に返されない場合、HDFSクライアントは直ちに異なるブロックレプリカに対するHedged Readを開始します。このパラメーターは、HDFSクラスタの`hdfs-site.xml`ファイル内の`dfs.client.hedged.read.threshold.millis`パラメーターに相当します。|

クエリプロファイルの以下のメトリクスの値が`0`を超える場合、Hedged Read機能が有効になっています。

| メトリック                         | 説明                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| TotalHedgedReadOps             | 開始されたHedged Readの回数です。                 |
| TotalHedgedReadOpsInCurThread  | StarRocksが`hdfs_client_hedged_read_threadpool_size`パラメーターで指定された最大サイズに達したHedged Readスレッドプールのため、新しいスレッドではなく現在のスレッドでHedged Readを開始する必要があった回数です。 |
| TotalHedgedReadOpsWin          | Hedged Readが元の読み取りよりも早く完了した回数です。 |
