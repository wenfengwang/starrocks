---
displayed_sidebar: "Japanese"
---

# データレイク関連FAQ

このトピックでは、データレイクに関するよくある質問（FAQ）について説明し、これらの問題の解決策を提供します。このトピックで言及されているいくつかのメトリクスは、SQLクエリのプロファイルからのみ取得できます。SQLクエリのプロファイルを取得するには、`set enable_profile=true`を指定する必要があります。

## 遅いHDFSノード

### 問題の説明

HDFSクラスタに保存されているデータファイルにアクセスすると、実行したSQLクエリのプロファイルから`__MAX_OF_FSIOTime`および`__MIN_OF_FSIOTime`メトリクスの値に大きな差がある場合、HDFSノードが遅いことを示しています。以下は、HDFSノードの遅延の問題を示す典型的なプロファイルの例です：

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

この問題を解決するために、以下のいずれかの解決策を利用できます：

- **[推奨]** [データキャッシュ](../data_source/data_cache.md)機能を有効にし、外部ストレージシステムからデータを自動的にStarRocksクラスタのBEにキャッシュすることで、遅いHDFSノードがクエリに与える影響を取り除きます。
- [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)機能を有効にします。この機能を有効にすると、ブロックの読み取りが遅い場合、StarRocksはオリジナルの読み取りと並行して別のブロックレプリカを読み取る新しい読み取りを開始します。2つの読み取りのうちどちらかが返ってきた時点で、もう一方の読み取りはキャンセルされます。**Hedged Read機能は読み取りを加速するのに役立ちますが、Java仮想マシン（JVM）のヒープメモリ消費量を大幅に増加させます。そのため、物理マシンのメモリ容量が小さい場合は、Hedged Read機能を有効にしないことをお勧めします。**

#### [推奨] データキャッシュ

[データキャッシュ](../data_source/data_cache.md)を参照してください。

#### Hedged Read

HDFSクラスタのBE構成ファイル`be.conf`で以下のパラメータ（v3.0以降でサポート）を使用して、HDFSクラスタでHedged Read機能を有効にし、構成できます。

| パラメータ                               | デフォルト値 | 説明                                                                    |
| ---------------------------------------- | ------------- | ----------------------------------------------------------------------- |
| hdfs_client_enable_hedged_read           | false         | Hedged Read機能を有効にするかどうかを指定します。                     |
| hdfs_client_hedged_read_threadpool_size  | 128           | HDFSクライアントでHedged Readスレッドプールのサイズを指定します。スレッドプールサイズは、HDFSクライアントで実行されるHedged Readのスレッド数を制限します。このパラメータは、HDFSクラスタの`hdfs-site.xml`ファイルの`dfs.client.hedged.read.threadpool.size`パラメータに相当します。 |
| hdfs_client_hedged_read_threshold_millis | 2500          | Hedged Readを開始するまでの待機時間（ミリ秒）を指定します。たとえば、このパラメータを`30`に設定した場合、ブロックからの読み取りが30ミリ秒以内に返ってこない場合、HDFSクライアントはすぐに別のブロックレプリカに対してHedged Readを開始します。このパラメータは、HDFSクラスタの`hdfs-site.xml`ファイルの`dfs.client.hedged.read.threshold.millis`パラメータに相当します。 |

クエリプロファイルの以下のメトリクスのいずれかの値が`0`を超える場合、Hedged Read機能が有効になっています。

| メトリクス                      | 説明                                           |
| ------------------------------ | ---------------------------------------------- |
| TotalHedgedReadOps             | 開始されたHedged Readの数。                        |
| TotalHedgedReadOpsInCurThread  | Hedged Readスレッドプールが`hdfs_client_hedged_read_threadpool_size`パラメータで指定された最大サイズに達したため、StarRocksが現在のスレッドでHedged Readを開始する必要がある場合の回数。 |
| TotalHedgedReadOpsWin          | Hedged Readが元の読み取りを打ち負かした回数。                 |