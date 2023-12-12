---
displayed_sidebar: "Japanese"
---

# データレイク関連FAQ

このトピックでは、データレイクに関するよくある質問（FAQ）について説明し、これらの問題への解決策を提供します。このトピックで言及されている一部のメトリクスは、SQLクエリのプロファイルからのみ取得できます。SQLクエリのプロファイルを取得するには、 `set enable_profile=true` を指定する必要があります。

## HDFSノードの遅延

### 問題の説明

HDFSクラスタに保存されているデータファイルにアクセスする際、実行したSQLクエリのプロファイルから`__MAX_OF_FSIOTime`と`__MIN_OF_FSIOTime`の値の差が大きい場合、それはHDFSノードの遅延を示しています。以下は、HDFSノードの遅延の問題を示す典型的なプロファイルの例です：

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

この問題を解決するために、以下のいずれかの解決策を使用できます：

- **[推奨]** [データキャッシュ](../data_source/data_cache.md)機能を有効にする。これにより、外部ストレージシステムからデータを自動的にStarRocksクラスタのBEにキャッシュし、遅延しているHDFSノードのクエリへの影響を排除します。
- [Hedged Read](https://hadoop.apache.org/docs/r2.8.3/hadoop-project-dist/hadoop-common/release/2.4.0/RELEASENOTES.2.4.0.html)機能を有効にする。この機能を有効にすると、特定のブロックからの読み出しが遅い場合、StarRocksは他のブロックレプリカに対して並行して新しい読み出しを開始し、元の読み出しと並行して実行します。2つの読み出しがいずれか1つが返ってくると、もう1つの読み出しはキャンセルされます。**Hedged Read機能は読み出しを加速するのに役立ちますが、Java仮想マシン（JVM）のヒープメモリ消費量も大幅に増加させます。そのため、物理マシンのメモリ容量が小さい場合は、Hedged Read機能を有効にしないことをお勧めします。**

#### [推奨] データキャッシュ

[データキャッシュ](../data_source/data_cache.md)を参照してください。

#### Hedged Read

HDFSクラスタでHedged Read機能を有効にして設定するために、BE構成ファイル`be.conf`で以下のパラメータ（v3.0以降でサポート）を使用してください。

| パラメータ                                | デフォルト値 | 説明                                                         |
| ---------------------------------------- | ------------- | ---------------------------------------------------------- |
| hdfs_client_enable_hedged_read           | false         | Hedged Read機能を有効にするかどうかを指定します。                                |
| hdfs_client_hedged_read_threadpool_size  | 128           | HDFSクライアントでHedged Readスレッドプールのサイズを指定します。スレッドプールのサイズは、HDFSクライアントで実行するHedged Readのスレッド数を制限します。このパラメータは、HDFSクラスタの`hdfs-site.xml`ファイルの`dfs.client.hedged.read.threadpool.size`パラメータに相当します。 |
| hdfs_client_hedged_read_threshold_millis | 2500          | Hedged Readを開始するまでのミリ秒数を指定します。たとえば、このパラメータを`30`に設定した場合、特定のブロックからの読み出しが30ミリ秒以内に返ってこない場合、HDFSクライアントは別のブロックレプリカに対してHedged Readを直ちに開始します。このパラメータは、HDFSクラスタの`hdfs-site.xml`ファイルの`dfs.client.hedged.read.threshold.millis`パラメータに相当します。 |

クエリプロファイルの以下のメトリクスのいずれかの値が`0`を超える場合、Hedged Read機能が有効になっています。

| メトリクス                         | 説明                                                   |
| ------------------------------ | ------------------------------------------------------ |
| TotalHedgedReadOps             | 開始されたHedged Readの数。                               |
| TotalHedgedReadOpsInCurThread  | 現在のスレッドでHedged Readを新しいスレッドではなく開始する回数。これは`hdfs_client_hedged_read_threadpool_size`パラメータによって指定された最大サイズに達した場合に行われます。 |
| TotalHedgedReadOpsWin          | Hedged Readが元の読み出しを上回った回数。                  |
