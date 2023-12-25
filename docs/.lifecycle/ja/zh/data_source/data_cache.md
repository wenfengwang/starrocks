---
displayed_sidebar: Chinese
---

# データキャッシュ

この文書では、Data Cacheの原理と、外部データクエリの高速化のためにData Cacheを有効にする方法について説明します。

データレイク分析シナリオでは、StarRocksはOLAPクエリエンジンとして、HDFSやオブジェクトストレージ（以下「外部ストレージシステム」と略）にあるデータファイルをスキャンする必要があります。クエリで実際に読み取るファイルの数が多いほど、I/Oコストも増加します。さらに、アドホッククエリ（ad-hoc）のシナリオでは、同じデータに頻繁にアクセスすると、繰り返しのI/Oコストが発生します。

このようなシナリオでのクエリパフォーマンスをさらに向上させるために、StarRocksはバージョン2.5からData Cache機能を提供しています。外部ストレージシステムの原データを特定の戦略に従って複数のブロックに分割し、それをStarRocksのローカルBEノードにキャッシュすることで、繰り返しのリモートデータフェッチのコストを避け、ホットデータのクエリ分析パフォーマンスをさらに向上させます。Data Cacheは、外部テーブル（JDBC外部テーブルを含まない）を使用し、External Catalogを使用して外部ストレージシステムのデータをクエリする場合にのみ有効であり、StarRocksのネイティブテーブルをクエリする場合には有効ではありません。

## 原理

StarRocksがリモートストレージファイルをローカルBEノードにキャッシュする際、原ファイルを特定の戦略に従って等しいサイズのブロックに分割します。ブロックはデータキャッシュの最小単位であり、サイズは設定可能です。ブロックサイズを1MBに設定した場合、Amazon S3上の128MBのParquetファイルをクエリすると、StarRocksは1MBのステップでファイルを128個のブロックに分割します。つまり、[0, 1MB)、[1MB, 2MB)、[2MB, 3MB)...[127MB, 128MB)とし、各ブロックにグローバルに一意なID、つまりcache keyを割り当てます。Cache keyは3つの部分で構成されます。

```Plain
hash(filename) + fileModificationTime + blockId
```

以下の説明です。

| **構成項目** | **説明**                                                     |
| ---------- | ------------------------------------------------------------ |
| filename   | データファイルの名前。                                               |
| fileModificationTime  | データファイルの最終変更時刻。                                    |
| blockId    | StarRocksがデータファイルを分割する際に各ブロックに割り当てるID。このIDはファイル内で一意ですが、グローバルには一意ではありません。 |

もしクエリが[1MB, 2MB)のブロックをヒットした場合：

1. StarRocksはキャッシュ内にそのブロックが存在するかをチェックします。
2. 存在する場合は、キャッシュからそのブロックを読み取ります。存在しない場合は、Amazon S3からそのブロックを読み取り、BE上にキャッシュします。

Data Cacheを有効にすると、StarRocksは外部ストレージシステムから読み取ったデータファイルをキャッシュします。特定のデータをキャッシュしたくない場合は、以下の設定を行います。

```SQL
SET enable_populate_datacache = false;
```

`enable_populate_datacache`に関する詳細は、[システム変数](../reference/System_variable.md#支持的变量)を参照してください。

## キャッシュメディア

StarRocksは、BEノードのメモリとディスクをキャッシュのストレージメディアとして使用し、全メモリキャッシュまたはメモリ+ディスクの2レベルキャッシュをサポートします。
ディスクをキャッシュメディアとして使用する場合は、キャッシュの加速効果がディスクの性能に直接関連しているため、高性能なローカルディスク（例えばローカルNVMeディスク）を使用することをお勧めします。ディスクの性能が平凡な場合は、複数のディスクを追加して、ディスクごとのI/Oプレッシャーを減らすことができます。

## キャッシュ排除メカニズム

Data Cacheでは、StarRocksは[LRU](https://baike.baidu.com/item/LRU/1269842)（Least Recently Used）戦略を使用してデータをキャッシュし、排除します。概要は以下の通りです：

- 優先的にメモリからデータを読み取ります。メモリ内で見つからない場合は、ディスクから読み取ります。ディスクから読み取ったデータは、メモリにロードしようとします。
- メモリから排除されたデータは、ディスクに書き込もうとします。ディスクから排除されたデータは、廃棄されます。

## データキャッシュを有効にする

Data Cacheはデフォルトで無効です。有効にするには、FEとBEの両方で以下の設定を行う必要があります。

### FEの設定

FEでData Cacheを有効にするには、以下の方法を使用できます：

- 必要に応じて、個々のセッションでData Cacheを有効にします。

  ```SQL
  SET enable_scan_datacache = true;
  ```

- 現在のすべてのセッションに対して、グローバルData Cacheを有効にします。

  ```SQL
  SET GLOBAL enable_scan_datacache = true;
  ```

### BEの設定

各BEの**conf/be.conf**ファイルに以下のパラメータを追加します。追加後、設定を有効にするために各BEを再起動する必要があります。

| **パラメータ**               | **説明**                                                     |**デフォルト値** |
| ---------------------- | ------------------------------------------------------------ |----------|
| datacache_enable     | Data Cacheを有効にするかどうか。<ul><li>`true`：有効。</li><li>`false`：無効。</li></ul>| false |
| datacache_disk_path  | ディスクのパス。複数のパスを追加することができ、各パスはセミコロン(;)で区切ります。BEマシンにいくつかのディスクがある場合は、それに応じて複数のパスを追加することをお勧めします。BEプロセスが起動すると、設定されたディスクキャッシュディレクトリが自動的に作成されます（親ディレクトリが存在しない場合は作成に失敗します）。 | `${STARROCKS_HOME}/datacache` |
| datacache_meta_path  | ブロックのメタデータが保存されるディレクトリで、通常は設定する必要はありません。 | `${STARROCKS_HOME}/datacache` |
| datacache_mem_size   | メモリキャッシュのデータ量の上限で、割合の上限（例："10%"）または物理的な上限（例："10G"、"21474836480"など）として設定できます。このパラメータの値は10GB以上に設定することをお勧めします。 | 10% |
| datacache_disk_size  | 単一のディスクキャッシュのデータ量の上限で、割合の上限（例："80%"）または物理的な上限（例："2T"、"500G"など）として設定できます。例えば、`datacache_disk_path`に2つのディスクを設定し、`datacache_disk_size`のパラメータ値を`21474836480`（20GB）に設定した場合、最大40GBのディスクデータをキャッシュできます。| 0はメモリのみをキャッシュメディアとして使用し、ディスクは使用しないことを意味します。 |

以下は設定例です：

```Plain
# Data Cacheを有効にします。
datacache_enable = true  

# ディスクのパスを設定します。BEマシンに2つのディスクがあると仮定します。
datacache_disk_path = /home/disk1/sr/dla_cache_data/;/home/disk2/sr/dla_cache_data/ 

# メモリキャッシュのデータ量の上限を2GBに設定します。
datacache_mem_size = 2147483648

# 単一のディスクキャッシュのデータ量の上限を1.2TBに設定します。
datacache_disk_size = 1288490188800
```

## Data Cacheのヒット状況を確認する

クエリプロファイルで現在のクエリのキャッシュヒット状況を観察できます。以下の3つの指標を観察してData Cacheのヒット状況を確認します：

- `DataCacheReadBytes`：メモリとディスクから読み取られたデータ量。
- `DataCacheWriteBytes`：外部ストレージシステムからメモリとディスクにロードされたデータ量。
- `BytesRead`：読み取られた総データ量で、メモリ、ディスク、および外部ストレージからの読み取りを含みます。

例1：StarRocksは外部ストレージシステムから大量のデータ（7.65GB）を読み取り、メモリとディスクからの読み取りデータ量（518.73MB）は少ないため、Data Cacheのヒット率は低いことを意味します。

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

例2：StarRocksはData Cacheから46.08GBのデータを読み取り、外部ストレージシステムから直接読み取られたデータ量は0であるため、Data Cacheが完全にヒットしたことを意味します。

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
- BufferUnplugCount: 1.231K (1231)
 - __MAX_OF_BufferUnplugCount: 81
 - __MIN_OF_BufferUnplugCount: 35
- BytesRead: 46.08 GB
 - __MAX_OF_BytesRead: 194.99 MB
 - __MIN_OF_BytesRead: 81.25 MB
```
