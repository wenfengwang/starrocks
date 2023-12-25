---
displayed_sidebar: English
---

# データキャッシュ

このトピックでは、データキャッシュの動作原理と、データキャッシュを有効にして外部データに対するクエリパフォーマンスを向上させる方法について説明します。

データレイク分析では、StarRocksはOLAPエンジンとして機能し、HDFSやAmazon S3などの外部ストレージシステムに保存されたデータファイルをスキャンします。I/Oオーバーヘッドは、スキャンするファイルの数が増えるにつれて増加します。さらに、一部のアドホックシナリオでは、同じデータへの頻繁なアクセスがI/Oオーバーヘッドを倍増させます。

これらのシナリオでクエリパフォーマンスを最適化するために、StarRocks 2.5はデータキャッシュ機能を提供しています。この機能は、事前定義されたポリシーに基づいて外部ストレージシステム内のデータを複数のブロックに分割し、StarRocksバックエンド(BE)にデータをキャッシュします。これにより、アクセス要求ごとに外部システムからデータを取得する必要がなくなり、ホットデータに対するクエリと分析が高速化されます。データキャッシュは、外部カタログまたは外部テーブルを使用して外部ストレージシステムからデータをクエリする場合にのみ機能します（JDBC互換データベースの外部テーブルは除く）。StarRocksのネイティブテーブルをクエリする際には機能しません。

## 仕組み

StarRocksは、外部ストレージシステムのデータを同じサイズ（デフォルトでは1MB）の複数のブロックに分割し、BEにデータをキャッシュします。ブロックは、データキャッシュの最小単位であり、設定可能です。

たとえば、ブロックサイズを1MBに設定し、Amazon S3から128MBのParquetファイルをクエリする場合、StarRocksはファイルを128ブロックに分割します。ブロックは[0, 1MB)、[1MB, 2MB)、[2MB, 3MB)...[127MB, 128MB)です。StarRocksは、キャッシュキーと呼ばれるグローバルに一意のIDを各ブロックに割り当てます。キャッシュキーは、次の3つの部分で構成されます。

```Plain
hash(filename) + fileModificationTime + blockId
```

次の表に、各部分の説明を示します。

| **コンポーネントアイテム** | **説明**                                              |
| ------------------ | ------------------------------------------------------------ |
| filename           | データファイルの名前。                                   |
| fileModificationTime | データファイルの最終変更時刻。                  |
| blockId            | StarRocksがデータファイルを分割する際にブロックに割り当てるID。このIDは、同じデータファイル内では一意ですが、StarRocksクラスタ内では一意ではありません。 |

クエリが[1MB, 2MB)ブロックにヒットする場合、StarRocksは以下の操作を行います：

1. ブロックがキャッシュに存在するかどうかを確認します。
2. ブロックが存在する場合、StarRocksはキャッシュからブロックを読み取ります。ブロックが存在しない場合、StarRocksはAmazon S3からブロックを読み取り、BEにキャッシュします。

データキャッシュが有効になると、StarRocksは外部ストレージシステムから読み取ったデータブロックをキャッシュします。このようなデータブロックをキャッシュしたくない場合は、次のコマンドを実行します：

```SQL
SET enable_populate_datacache = false;
```

`enable_populate_datacache`についての詳細は、[システム変数](../reference/System_variable.md)を参照してください。

## ブロックのストレージメディア

StarRocksは、BEマシンのメモリとディスクを使用してブロックをキャッシュします。メモリのみ、またはメモリとディスクの両方でキャッシュをサポートします。

ディスクをストレージメディアとして使用する場合、キャッシュ速度はディスクのパフォーマンスに直接影響されます。そのため、データキャッシュにはNVMeディスクなどの高性能ディスクを使用することを推奨します。高性能ディスクがない場合は、ディスクを追加してディスクI/Oの圧力を軽減できます。

## キャッシュ置換ポリシー

StarRocksは、[最近最も使用されていない](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))（LRU）ポリシーを使用してデータをキャッシュおよび破棄します。

- StarRocksはまずメモリからデータを読み取ります。データがメモリ内に見つからない場合、StarRocksはディスクからデータを読み取り、ディスクから読み取ったデータをメモリにロードしようとします。
- メモリから破棄されたデータはディスクに書き込まれます。ディスクから破棄されたデータは削除されます。

## データキャッシュの有効化

データキャッシュはデフォルトで無効です。この機能を有効にするには、StarRocksクラスタのFEとBEを設定します。

### FEの設定


FEでデータキャッシュを有効にするには、以下の方法のいずれかを使用します。

- 要件に基づいて、特定のセッションでデータキャッシュを有効にします。

  ```SQL
  SET enable_scan_datacache = true;
  ```

- すべてのアクティブセッションでデータキャッシュを有効にします。

  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BEの設定

各BEの**conf/be.conf**ファイルに以下のパラメータを追加します。その後、各BEを再起動して設定を反映させます。

| **パラメータ**          | **説明**                                              | **デフォルト値** |
| ---------------------- | ------------------------------------------------------------ | -------------------|
| datacache_enable     | データキャッシュを有効にするかどうか。<ul><li>`true`: データキャッシュが有効。</li><li>`false`: データキャッシュが無効。</li></ul> | false |
| datacache_disk_path  | ディスクのパス。複数のディスクを設定し、ディスクパスをセミコロン (;) で区切ることができます。設定するパスの数は、BEマシンのディスク数と同じにすることを推奨します。BEが起動すると、StarRocksは自動的にディスクキャッシュディレクトリを作成します（親ディレクトリが存在しない場合は作成に失敗します）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | ブロックメタデータのストレージパス。このパラメータは未指定でも構いません。 | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | メモリにキャッシュできるデータの最大量。パーセンテージ（例: `10%`）または物理的な制限（例: `10G`、`21474836480`）で設定できます。このパラメータの値は少なくとも10GBに設定することを推奨します。 | `10%` |
| datacache_disk_size  | 単一ディスクにキャッシュできるデータの最大量。パーセンテージ（例: `80%`）または物理的な制限（例: `2T`、`500G`）で設定できます。例えば、`datacache_disk_path`パラメータに2つのディスクパスを設定し、`datacache_disk_size`パラメータの値を`21474836480`（20GB）に設定すると、これら2つのディスクに最大40GBのデータをキャッシュできます。 | `0` これは、データキャッシュにメモリのみを使用することを示します。 |

これらのパラメータの設定例。

```Plain

# データキャッシュを有効にする。
datacache_enable = true  

# ディスクパスを設定する。BEマシンに2つのディスクがあると仮定します。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# datacache_mem_sizeを2GBに設定する。
datacache_mem_size = 2147483648

# datacache_disk_sizeを1.2TBに設定する。
datacache_disk_size = 1288490188800
```

## クエリがデータキャッシュをヒットしたかどうかを確認する

クエリがデータキャッシュをヒットしたかどうかは、クエリプロファイルの以下のメトリクスを分析することで確認できます。

- `DataCacheReadBytes`: StarRocksがメモリとディスクから直接読み取ったデータの量。
- `DataCacheWriteBytes`: 外部ストレージシステムからStarRocksのメモリとディスクへロードされたデータの量。
- `BytesRead`: 読み取られたデータの総量で、StarRocksが外部ストレージシステム、メモリ、およびディスクから読み取ったデータを含みます。

例1: この例では、StarRocksは外部ストレージシステムから大量のデータ（7.65GB）を読み取り、メモリとディスクからはわずかなデータ（518.73MB）のみを読み取っています。これは、データキャッシュのヒットが少ないことを意味します。

```Plain
 - Table: lineorder
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

例2: この例では、StarRocksはデータキャッシュから大量のデータ（46.08GB）を読み取り、外部ストレージシステムからはデータを読み取っていません。これは、StarRocksがデータキャッシュからのみデータを読み取っていることを意味します。

```Plain
Table: lineitem
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
```
- BufferUnplugCount: 1.231K（1231）
 - __MAX_OF_BufferUnplugCount: 81
 - __MIN_OF_BufferUnplugCount: 35
- BytesRead: 46.08 GB
 - __MAX_OF_BytesRead: 194.99 MB
 - __MIN_OF_BytesRead: 81.25 MB