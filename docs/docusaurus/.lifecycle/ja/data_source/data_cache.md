---
displayed_sidebar: "Japanese"
---

# データキャッシュ

このトピックでは、データキャッシュの動作原理と、外部データのクエリパフォーマンスを向上させるためにデータキャッシュを有効にする方法について説明します。

データレイクアナリティクスでは、StarRocksは外部ストレージシステム（HDFS、Amazon S3など）に格納されたデータファイルをスキャンするOLAPエンジンとして機能します。スキャンするファイルの数が増えるとI/Oオーバーヘッドが増加します。また、いくつかのアドホックシナリオでは、同じデータへの頻繁なアクセスによりI/Oオーバーヘッドが倍増します。

これらのシナリオでクエリパフォーマンスを最適化するために、StarRocks 2.5 ではデータキャッシュ機能が提供されています。この機能は外部ストレージシステムからのデータクエリに対してのみ機能し、外部カタログまたは外部テーブル（JDBC互換データベースの外部テーブルを除く）を使用してデータを問い合わせる場合にのみ機能します。StarRocksネイティブテーブルをクエリする場合は機能しません。

## 動作原理

StarRocksは外部ストレージシステムのデータを複数の同じサイズ（デフォルトでは1 MB）のブロックに分割し、そのデータをBE（バックエンド）にキャッシュします。ブロックはデータキャッシュの最小単位であり、設定可能です。

たとえば、ブロックサイズを1 MBに設定し、Amazon S3 から128 MBのParquetファイルをクエリしたい場合、StarRocksはファイルを128個のブロックに分割します。ブロックは[0, 1 MB)、[1 MB, 2 MB)、[2 MB, 3 MB) ... [127 MB, 128 MB) となります。StarRocksは各ブロックにグローバルに一意なIDであるキャッシュキーを割り当てます。キャッシュキーは次の3つのパーツで構成されます。

```プレーン
hash(filename) + fileModificationTime + blockId
```

次の表は、各パーツの説明を示しています。

| **コンポーネント項目** | **説明**                        |
| ----------------------- | ---------------------------------- |
| filename               | データファイルの名前。          |
| fileModificationTime   | データファイルの最終変更時刻。     |
| blockId                | データファイルを分割する際にStarRocksが割り当てるID。同じデータファイル内で一意でありますが、StarRocksクラスタ内では一意ではありません。    |

クエリが[1 MB, 2 MB) ブロックにヒットした場合、StarRocksは次の操作を行います。

1. ブロックがキャッシュに存在するかどうかをチェックします。
2. ブロックが存在する場合は、StarRocksはブロックをキャッシュから読み込みます。ブロックが存在しない場合は、StarRocksはブロックをAmazon S3から読み込んでBEにキャッシュします。

データキャッシュが有効になると、StarRocksは外部ストレージシステムから読み込んだデータブロックをキャッシュします。このようなデータブロックをキャッシュしたくない場合は、次のコマンドを実行します。

```SQL
SET enable_populate_datacache = false;
```

`enable_populate_datacache`の詳細については、[システム変数](../reference/System_variable.md)を参照してください。

## ブロックのストレージメディア

StarRocksはBEマシンのメモリとディスクを使用してブロックをキャッシュします。メモリ単独またはメモリとディスクの両方でのキャッシュをサポートしています。

ディスクをストレージメディアとして使用する場合、キャッシュの速度はディスクのパフォーマンスに直接影響を受けます。したがって、データキャッシュ用にNVMeディスクなどの高性能ディスクを使用することをお勧めします。高性能ディスクを持っていない場合は、ディスクのI/O負荷を軽減するために、より多くのディスクを追加することができます。

## キャッシュ置換ポリシー

StarRocksは、データをキャッシュするためのキャッシュ置換ポリシーとして[最近最少使用](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))（LRU）ポリシーを使用します。

- StarRocksはまずメモリからデータを読み込みます。メモリにデータが見つからない場合、StarRocksはディスクからデータを読み込み、ディスクから読み込んだデータをメモリにロードしようとします。
- メモリから破棄されたデータはディスクに書き込まれます。ディスクから破棄されたデータは削除されます。

## データキャッシュの有効化

データキャッシュはデフォルトで無効になっています。この機能を有効にするには、StarRocksクラスタのFEおよびBEを構成します。

### FEの構成

FE向けにデータキャッシュを有効にするには、次のいずれかの方法を使用できます。

- 要件に応じてセッションごとにデータキャッシュを有効にする。
  ```SQL
  SET enable_scan_datacache = true;
  ```
- すべてのアクティブなセッションでデータキャッシュを有効にする。
  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BEの構成

各BEの**conf/be.conf**ファイルに次のパラメータを追加します。その後、各BEを再起動して設定を有効にします。

| **パラメータ**         | **説明**                                                                           | **デフォルト値**  |
| --------------------- | ----------------------------------------------------------------------------------- | ------------------ |
| datacache_enable     | データキャッシュを有効にするかどうか。<ul><li>`true`: データキャッシュが有効になります。</li><li>`false`: データキャッシュが無効になります。</li></ul>  | false |
| datacache_disk_path  | ディスクのパス。1台のBEマシンに複数のディスクを構成することができ、ディスクのパスはセミコロン（;）で区切ります。BEの数だけディスクパスを設定することをお勧めします。BEの起動時、StarRocksは自動的にディスクキャッシュディレクトリを作成します（親ディレクトリが存在しない場合は作成に失敗します）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | ブロックメタデータの保存パス。このパラメータは未指定のままにしておくことができます。   | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | メモリにキャッシュできるデータの最大量。パーセンテージ（たとえば、 `10％`）または物理的な制限（たとえば、 `10G`、`21474836480`）として設定できます。このパラメータの値を少なくとも10 GBに設定することをお勧めします。   | `10％` |
| datacache_disk_size  | 1台のディスクにキャッシュできるデータの最大量。パーセンテージ（たとえば、 `80％`）または物理的な制限（たとえば、 `2T`、`500G`）として設定できます。たとえば、 `datacache_disk_path`パラメータに2つのディスクパスを構成し、 `datacache_disk_size`パラメータの値を `21474836480`（20 GB）に設定した場合、これらの2つのディスクには最大40 GBのデータをキャッシュできます。   | `0`（これはメモリのみを使用してデータをキャッシュすることを示します）  |

これらのパラメータの設定例です。

```Plain

# データキャッシュを有効にします。
datacache_enable = true  

# ディスクパスを構成します。BEマシンが2つのディスクに装備されていると仮定します。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# datacache_mem_sizeを2 GBに設定します。
datacache_mem_size = 2147483648

# datacache_disk_sizeを1.2 TBに設定します。
datacache_disk_size = 1288490188800
```

## クエリがデータキャッシュにヒットしたかどうかをチェック

クエリプロファイルの次のメトリクスを分析することで、クエリがデータキャッシュにヒットしたかどうかを調べることができます。

- `DataCacheReadBytes`: StarRocksが直接メモリとディスクから読み込んだデータの量。
- `DataCacheWriteBytes`: 外部ストレージシステムからStarRocksのメモリとディスクに読み込まれたデータの量。
- `BytesRead`: StarRocksが外部ストレージシステム、メモリ、およびディスクから読み込んだデータの合計量。

例1：この例では、StarRocksが外部ストレージシステムから大量のデータ（7.65 GB）を読み込み、メモリとディスクからはほとんどデータ（518.73 MB）を読み込んでいます。これは、データキャッシュがほとんどヒットしなかったことを意味します。

```Plain
 - テーブル: lineorder
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

例2：この例では、StarRocksがデータキャッシュから大量のデータ（46.08 GB）を読み込み、外部ストレージシステムからはデータを読み込んでいないことを意味します。つまり、StarRocksがデータをデータキャッシュからのみ読み込んでいることを示します。

```Plain
テーブル: lineitem
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