```yaml
---
displayed_sidebar: "Japanese"
---

# データキャッシュ

このトピックでは、データキャッシュの動作原理と、外部データのクエリパフォーマンスを向上させるためにデータキャッシュを有効にする方法について説明します。

データレイクアナリティクスでは、StarRocksは外部ストレージシステム（HDFSやAmazon S3など）に保存されているデータファイルをスキャンするためのOLAPエンジンとして機能します。スキャンするファイルの数が増えるとI/Oオーバーヘッドが増加します。さらに、特定のアドホックなシナリオでは、同じデータに頻繁にアクセスすることでI/Oオーバーヘッドが倍増します。

これらのシナリオにおけるクエリパフォーマンスを最適化するために、StarRocks 2.5ではデータキャッシュ機能が提供されています。この機能は、あらかじめ定義されたポリシーに基づいて外部ストレージシステムのデータを複数のブロックに分割し、StarRocksバックエンド（BEs）にデータをキャッシュします。これにより、各アクセス要求ごとに外部システムからデータを取得する必要がなくなり、ホットデータのクエリや分析が加速されます。データキャッシュは、外部カタログや外部テーブルを使用して外部ストレージシステムからデータをクエリする場合にのみ機能し、JDBC互換データベースの外部テーブルを除きます。StarRocksネイティブテーブルをクエリする場合は機能しません。

## 動作方法

StarRocksは外部ストレージシステムのデータをデフォルトで1 MBという同じサイズの複数のブロックに分割し、BEにデータをキャッシュします。ブロックはデータキャッシュの最小単位であり、設定可能です。

たとえば、ブロックサイズを1 MBに設定し、Amazon S3から128 MBのParquetファイルをクエリしたい場合、StarRocksはファイルを128ブロックに分割します。ブロックは[0, 1 MB)、[1 MB, 2 MB)、[2 MB, 3 MB) ... [127 MB, 128 MB)となります。StarRocksは各ブロックにグローバルにユニークなIDであるキャッシュキーを割り当てます。キャッシュキーは次の3つの部分から構成されます。

```Plain
hash(filename) + fileModificationTime + blockId
```

以下の表は各部分の説明を示しています。

| **コンポーネント項目** | **説明**                                              |
| ------------------ | ------------------------------------------------------------ |
| filename           | データファイルの名前。                                   |
| fileModificationTime | データファイルの最終変更時刻。                  |
| blockId            | データファイルを分割する際にStarRocksがブロックに割り当てるID。このIDは同じデータファイル内ではユニークですが、StarRocksクラスタ内ではユニークではありません。 |

クエリが[1 MB, 2 MB)ブロックにヒットする場合、StarRocksは以下の操作を行います。

1. ブロックがキャッシュに存在するかどうかをチェックします。
2. ブロックが存在する場合、StarRocksはブロックをキャッシュから読み込みます。ブロックが存在しない場合、StarRocksはAmazon S3からブロックを読み込み、BEにキャッシュします。

データキャッシュを有効にした後、StarRocksは外部ストレージシステムから読み込まれたデータブロックをキャッシュします。このようなデータブロックをキャッシュしたくない場合は、次のコマンドを実行します。

```SQL
SET enable_populate_datacache = false;
```

`enable_populate_datacache`に関する詳細は、[システム変数](../reference/System_variable.md)を参照してください。

## ブロックのストレージメディア

StarRocksはBEマシンのメモリとディスクを使用してブロックをキャッシュします。メモリのみまたはメモリとディスクの両方でキャッシュをサポートしています。

ディスクをストレージメディアとして使用する場合、キャッシュ速度はディスクのパフォーマンスに直接影響されます。したがって、データキャッシュ用にNVMeディスクなどの高性能ディスクを使用することをお勧めします。高性能ディスクがない場合は、ディスクのI/O負荷を和らげるためにより多くのディスクを追加できます。

## キャッシュ置換ポリシー

StarRocksは、データをキャッシュして破棄するために[最近最少使用](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))（LRU）ポリシーを使用します。

- StarRocksはまずメモリからデータを読み込みます。データがメモリに見つからない場合、StarRocksはデータをディスクから読み込み、ディスクから読み込んだデータをメモリにロードしようとします。
- メモリから破棄されたデータはディスクに書き込まれます。ディスクから破棄されたデータは削除されます。

## データキャッシュの有効化

データキャッシュはデフォルトで無効になっています。この機能を有効にするには、StarRocksクラスタのFEおよびBEを構成してください。

### FEの構成

FEでデータキャッシュを有効にするには、次のいずれかの方法を使用できます。

- 要件に応じて、特定のセッションでデータキャッシュを有効にします。

  ```SQL
  SET enable_scan_datacache = true;
  ```

- すべてのアクティブなセッションでデータキャッシュを有効にします。

  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BEの構成

各BEの**conf/be.conf**ファイルに次のパラメータを追加します。その後、各BEを再起動して設定を有効にします。

| **パラメータ**          | **説明**                                              | **デフォルト値** |
| ---------------------- | ------------------------------------------------------------ | -------------------|
| datacache_enable     | データキャッシュを有効にするかどうか。<ul><li>`true`: データキャッシュは有効です。</li><li>`false`: データキャッシュは無効です。</li></ul> | false |
| datacache_disk_path  | ディスクのパス。複数のディスクを構成し、セミコロン（;）でディスクのパスを区切ることができます。BEマシンのディスク数と同じ数のパスを構成することをお勧めします。BEが起動すると、StarRocksは自動的にディスクキャッシュディレクトリを作成します（親ディレクトリが存在しない場合は作成に失敗します）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | ブロックメタデータの保存パス。このパラメータを指定しない場合があります。 | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | メモリにキャッシュできる最大データ量。パーセンテージ（たとえば、`10%`）または物理的な制限（たとえば、`10G`、`21474836480`）として設定できます。このパラメータの値を少なくとも10 GBに設定することをお勧めします。 | `10%` |
| datacache_disk_size  | 単一のディスクにキャッシュできる最大データ量。パーセンテージ（たとえば、`80%`）または物理的な制限（たとえば、`2T`、`500G`）として設定できます。たとえば、`datacache_disk_path`パラメータに2つのディスクパスを構成し、`datacache_disk_size`パラメータの値を`21474836480`（20 GB）に設定した場合、これらの2つのディスクに最大40 GBのデータをキャッシュできます。  | `0`（これはデータをキャッシュするにはメモリのみを使用することを示します。） |

これらのパラメータの設定例です。
```Plain

# データキャッシュを有効にする
datacache_enable = true  

# ディスクパスの構成。BEマシンに2つのディスクを搭載していると仮定します。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# datacache_mem_sizeを2 GBに設定します。
datacache_mem_size = 2147483648

# datacache_disk_sizeを1.2 TBに設定します。
datacache_disk_size = 1288490188800
```

## クエリがデータキャッシュにヒットしたかどうかを確認

クエリプロファイルの次のメトリクスを分析することで、クエリがデータキャッシュにヒットしたかどうかを確認できます。

- `DataCacheReadBytes`: StarRocksがメモリおよびディスクから直接読み取るデータの量。
- `DataCacheWriteBytes`: 外部ストレージシステムからStarRocksのメモリおよびディスクに読み込まれたデータの量。
- `BytesRead`: StarRocksが外部ストレージシステムおよび自身のメモリおよびディスクから読み取る合計のデータ量。

例 1：この例では、StarRocksは外部ストレージシステムから多くのデータ（7.65 GB）を読み取り、わずかなデータ（518.73 MB）だけをメモリおよびディスクから読み取っています。つまり、ほとんどのデータキャッシュがヒットしていないことを意味します。

```Plain
 - テーブル：lineorder
 - DataCacheReadBytes: 518.73 MB
   - __MAX_OF_DataCacheReadBytes: 4.73 MB
   - __MIN_OF_DataCacheReadBytes: 16.00 KB
 - DataCacheReadCounter: 684
   - __MAX_OF_DataCacheReadCounter: 4
   - __MIN_OF_DataCacheReadCounter: 0
 - DataCacheReadTimer: 737.357us
 - DataCacheWriteBytes: 7.65 GB
   - __MAX_OF_DataCacheWriteBytes: 64.39 MB
   - __MIN_OF_DataCacheWriteBytes: 0.00 
 - DataCacheWriteCounter: 7.887K (7887)
   - __MAX_OF_DataCacheWriteCounter: 65
   - __MIN_OF_DataCacheWriteCounter: 0
 - DataCacheWriteTimer: 23.467ms
   - __MAX_OF_DataCacheWriteTimer: 62.280ms
   - __MIN_OF_DataCacheWriteTimer: 0ns
 - BufferUnplugCount: 15
   - __MAX_OF_BufferUnplugCount: 2
   - __MIN_OF_BufferUnplugCount: 0
 - BytesRead: 7.65 GB
   - __MAX_OF_BytesRead: 64.39 MB
   - __MIN_OF_BytesRead: 0.00
```

例 2：この例では、StarRocksはデータキャッシュから大量のデータ（46.08 GB）を読み取り、外部ストレージシステムからはデータを読み取っていないため、StarRocksはデータをデータキャッシュからのみ読み取っています。

```Plain
テーブル：lineitem
```
- DataCacheReadBytes: 46.08 GB
  - __MAX_OF_DataCacheReadBytes: 194.99 MB
  - __MIN_OF_DataCacheReadBytes: 81.25 MB
- DataCacheReadCounter: 72.237K (72237)
  - __MAX_OF_DataCacheReadCounter: 299
  - __MIN_OF_DataCacheReadCounter: 118
- DataCacheReadTimer: 856.481ms
  - __MAX_OF_DataCacheReadTimer: 1s547ms
  - __MIN_OF_DataCacheReadTimer: 261.824ms
- DataCacheWriteBytes: 0.00 
- DataCacheWriteCounter: 0
- DataCacheWriteTimer: 0ns
- BufferUnplugCount: 1.231K (1231)
  - __MAX_OF_BufferUnplugCount: 81
  - __MIN_OF_BufferUnplugCount: 35
- BytesRead: 46.08 GB
  - __MAX_OF_BytesRead: 194.99 MB
  - __MIN_OF_BytesRead: 81.25 MB