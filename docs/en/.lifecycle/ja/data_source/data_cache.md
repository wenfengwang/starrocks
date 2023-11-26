---
displayed_sidebar: "Japanese"
---

# データキャッシュ

このトピックでは、データキャッシュの動作原理と、外部データのクエリパフォーマンスを向上させるためにデータキャッシュを有効にする方法について説明します。

データレイクアナリティクスでは、StarRocksはHDFSやAmazon S3などの外部ストレージシステムに格納されたデータファイルをスキャンするためのOLAPエンジンとして機能します。スキャンするファイルの数が増えると、I/Oオーバーヘッドも増加します。また、いくつかのアドホックシナリオでは、同じデータへの頻繁なアクセスによりI/Oオーバーヘッドが倍増します。

これらのシナリオでクエリパフォーマンスを最適化するために、StarRocks 2.5ではデータキャッシュ機能が提供されています。この機能は、外部ストレージシステムのデータを事前に定義されたポリシーに基づいて複数のブロックに分割し、データをStarRocksのバックエンド（BE）にキャッシュします。これにより、各アクセスリクエストごとに外部システムからデータを取得する必要がなくなり、ホットデータ上でのクエリと分析が加速されます。データキャッシュは、外部カタログまたは外部テーブル（JDBC互換データベース用の外部テーブルを除く）を使用して外部ストレージシステムからデータをクエリする場合にのみ機能します。StarRocksのネイティブテーブルをクエリする場合は機能しません。

## 動作原理

StarRocksは、外部ストレージシステムのデータを同じサイズ（デフォルトでは1 MB）の複数のブロックに分割し、BEにデータをキャッシュします。ブロックはデータキャッシュの最小単位であり、設定可能です。

たとえば、ブロックサイズを1 MBに設定し、Amazon S3から128 MBのParquetファイルをクエリしたい場合、StarRocksはファイルを128個のブロックに分割します。ブロックは[0, 1 MB)、[1 MB, 2 MB)、[2 MB, 3 MB) ... [127 MB, 128 MB)のようになります。StarRocksは、ブロックごとにグローバルに一意のIDであるキャッシュキーを割り当てます。キャッシュキーは次の3つの部分で構成されます。

```Plain
hash(filename) + fileModificationTime + blockId
```

以下の表は、各部分の説明を提供します。

| **コンポーネント項目** | **説明**                                                     |
| ------------------ | ------------------------------------------------------------ |
| filename           | データファイルの名前。                                          |
| fileModificationTime | データファイルの最終変更時刻。                                      |
| blockId            | データファイルを分割する際にStarRocksがブロックに割り当てるID。このIDは、同じデータファイル内では一意ですが、StarRocksクラスタ内では一意ではありません。 |

クエリが[1 MB, 2 MB)のブロックにヒットした場合、StarRocksは次の操作を実行します。

1. ブロックがキャッシュに存在するかどうかを確認します。
2. ブロックが存在する場合、StarRocksはキャッシュからブロックを読み取ります。ブロックが存在しない場合、StarRocksはブロックをAmazon S3から読み取り、BEにキャッシュします。

データキャッシュが有効になった後、StarRocksは外部ストレージシステムから読み取ったデータブロックをキャッシュします。このようなデータブロックをキャッシュしたくない場合は、次のコマンドを実行します。

```SQL
SET enable_populate_block_cache = false;
```

`enable_populate_block_cache`に関する詳細情報については、[システム変数](../reference/System_variable.md)を参照してください。

## ブロックのストレージメディア

StarRocksは、BEマシンのメモリとディスクを使用してブロックをキャッシュします。メモリのみまたはメモリとディスクの両方でキャッシュをサポートします。

ディスクをストレージメディアとして使用する場合、キャッシュ速度はディスクのパフォーマンスに直接影響を受けます。したがって、データキャッシュにはNVMeディスクなどの高性能ディスクを使用することをお勧めします。高性能ディスクを持っていない場合は、ディスクI/Oの負荷を軽減するためにディスクを追加することができます。

## キャッシュ置換ポリシー

StarRocksは、[最近使用された](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))（LRU）ポリシーを使用してデータをキャッシュし、破棄します。

- StarRocksはまずメモリからデータを読み取ります。データがメモリに存在しない場合、StarRocksはデータをディスクから読み取り、ディスクから読み取ったデータをメモリにロードしようとします。
- メモリから破棄されたデータはディスクに書き込まれます。ディスクから破棄されたデータは削除されます。

## データキャッシュの有効化

データキャッシュはデフォルトで無効になっています。この機能を有効にするには、StarRocksクラスタのFEとBEを設定します。

### FEの設定

次のいずれかの方法を使用して、FEのデータキャッシュを有効にすることができます。

- 要件に基づいて、特定のセッションでデータキャッシュを有効にします。

  ```SQL
  SET enable_scan_block_cache = true;
  ```

- すべてのアクティブなセッションでデータキャッシュを有効にします。

  ```SQL
  SET GLOBAL enable_scan_block_cache = true;
  ```

### BEの設定

各BEの**conf/be.conf**ファイルに次のパラメータを追加します。その後、各BEを再起動して設定を有効にします。

| **パラメータ**          | **説明**                                                     | **デフォルト値** |
| ---------------------- | ------------------------------------------------------------ | -------------------|
| block_cache_enable     | データキャッシュを有効にするかどうか。<ul><li>`true`：データキャッシュが有効になります。</li><li>`false`：データキャッシュが無効になります。</li></ul> | false |
| block_cache_disk_path  | ディスクのパス。複数のディスクを設定し、セミコロン（;）でディスクのパスを区切ることができます。設定するパスの数は、BEマシンのディスクの数と同じにすることをお勧めします。BEが起動すると、StarRocksはディスクキャッシュディレクトリを自動的に作成します（親ディレクトリが存在しない場合は作成に失敗します）。 | `${STARROCKS_HOME}/block_cache` |
| block_cache_meta_path  | ブロックメタデータの保存パス。このパラメータを指定しない場合は、このパラメータを空白のままにしておくことができます。 | `${STARROCKS_HOME}/block_cache` |
| block_cache_mem_size   | メモリにキャッシュできるデータの最大量。単位：バイト。このパラメータの値を少なくとも20 GBに設定することをお勧めします。データキャッシュが有効になった後、StarRocksがディスクから大量のデータを読み取る場合は、この値を増やすことを検討してください。 | `2147483648`（2 GB） |
| block_cache_disk_size  | 単一のディスクにキャッシュできるデータの最大量。単位：バイト。たとえば、`block_cache_disk_path`パラメータに2つのディスクパスを設定し、`block_cache_disk_size`パラメータの値を`21474836480`（20 GB）に設定した場合、これらの2つのディスクに最大40 GBのデータをキャッシュできます。 | `0`（メモリのみを使用してデータをキャッシュすることを示します） |

これらのパラメータの設定例です。

```Plain

# データキャッシュを有効にします。
block_cache_enable = true  

# ディスクパスを設定します。BEマシンに2つのディスクが搭載されているとします。
block_cache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/

# block_cache_mem_sizeを2 GBに設定します。
block_cache_mem_size = 2147483648

# block_cache_disk_sizeを1.2 TBに設定します。
block_cache_disk_size = 1288490188800
```

## クエリがデータキャッシュにヒットしたかどうかを確認する

クエリプロファイルの次のメトリックを分析することで、クエリがデータキャッシュにヒットしたかどうかを確認することができます。

- `BlockCacheReadBytes`：StarRocksが直接メモリとディスクから読み取ったデータの量。
- `BlockCacheWriteBytes`：外部ストレージシステムからStarRocksのメモリとディスクにロードされたデータの量。
- `BytesRead`：読み取られたデータの総量。これには、StarRocksが外部ストレージシステム、メモリ、ディスクから読み取ったデータが含まれます。

例1：この例では、StarRocksは外部ストレージシステムから大量のデータ（7.65 GB）を読み取り、メモリとディスクからはわずかなデータ（518.73 MB）しか読み取りません。これは、データキャッシュがヒットしたデータが少ないことを意味します。

```Plain
 - テーブル：lineorder
 - BlockCacheReadBytes：518.73 MB
   - __MAX_OF_BlockCacheReadBytes：4.73 MB
   - __MIN_OF_BlockCacheReadBytes：16.00 KB
 - BlockCacheReadCounter：684
   - __MAX_OF_BlockCacheReadCounter：4
   - __MIN_OF_BlockCacheReadCounter：0
 - BlockCacheReadTimer：737.357us
 - BlockCacheWriteBytes：7.65 GB
   - __MAX_OF_BlockCacheWriteBytes：64.39 MB
   - __MIN_OF_BlockCacheWriteBytes：0.00 
 - BlockCacheWriteCounter：7.887K（7887）
   - __MAX_OF_BlockCacheWriteCounter：65
   - __MIN_OF_BlockCacheWriteCounter：0
 - BlockCacheWriteTimer：23.467ms
   - __MAX_OF_BlockCacheWriteTimer：62.280ms
   - __MIN_OF_BlockCacheWriteTimer：0ns
 - BufferUnplugCount：15
   - __MAX_OF_BufferUnplugCount：2
   - __MIN_OF_BufferUnplugCount：0
 - BytesRead：7.65 GB
   - __MAX_OF_BytesRead：64.39 MB
   - __MIN_OF_BytesRead：0.00
```

例2：この例では、StarRocksはデータキャッシュから大量のデータ（46.08 GB）を読み取り、外部ストレージシステムからはデータを読み取りません。つまり、StarRocksはデータキャッシュからのみデータを読み取ります。

```Plain
テーブル：lineitem
- BlockCacheReadBytes：46.08 GB
 - __MAX_OF_BlockCacheReadBytes：194.99 MB
 - __MIN_OF_BlockCacheReadBytes：81.25 MB
- BlockCacheReadCounter：72.237K（72237）
 - __MAX_OF_BlockCacheReadCounter：299
 - __MIN_OF_BlockCacheReadCounter：118
- BlockCacheReadTimer：856.481ms
 - __MAX_OF_BlockCacheReadTimer：1s547ms
 - __MIN_OF_BlockCacheReadTimer：261.824ms
- BlockCacheWriteBytes：0.00 
- BlockCacheWriteCounter：0
- BlockCacheWriteTimer：0ns
- BufferUnplugCount：1.231K（1231）
 - __MAX_OF_BufferUnplugCount：81
 - __MIN_OF_BufferUnplugCount：35
- BytesRead：46.08 GB
 - __MAX_OF_BytesRead：194.99 MB
 - __MIN_OF_BytesRead：81.25 MB
```
