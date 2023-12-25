---
displayed_sidebar: Chinese
---

# メモリ管理

この記事では、メモリリソースの管理とチューニング方法について説明します。

## メモリ使用状況の確認

以下の方法で BE のメモリ使用状況を確認および分析できます。

* **ブラウザまたは curl コマンドを使用して Metrics インターフェースにアクセスし、メモリ使用状況を分析します。**

Metrics は10秒ごとに更新されます。

```bash
curl -XGET -s http://be_ip:8040/metrics | grep "^starrocks_be_.*_mem_bytes\|^starrocks_be_tcmalloc_bytes_in_use" 
```

> 説明：
>
> * 上記の `be_ip` を BE ノードの実際の IP アドレスに変更してください。
> * BE の `be_http_port` はデフォルトで `8040` です。

対応する指標の意味は [メモリの分類](#メモリの分類) を参照してください。

* **ブラウザまたは curl コマンドを使用して mem_tracker インターフェースにアクセスし、BE のメモリ使用状況を分析します。**

```bash
http://be_ip:8040/mem_tracker
```

> 説明：
>
> * 上記の `be_ip` を BE ノードの実際の IP アドレスに変更してください。
> * BE の `be_http_port` はデフォルトで `8040` です。

![MemTracker](../assets/memory_management_1.png)

指標の説明：

* `level`: MemTracker はツリー構造で、第1レベルは BE が使用する総メモリ、第2レベルは分類されたメモリ使用です。
* `Label`: メモリの分類を示すラベルで、対応する指標の意味は [メモリの分類](#メモリの分類) を参照してください。
* `Parent`: 親ノードの Label。
* `Limit`: メモリ使用の制限で、`-1` は制限なしを意味します。
* `Current Consumption`: 現在のメモリ使用量。
* `Peak Consumption`: ピーク時のメモリ使用量。

* **ブラウザまたは curl コマンドを使用して TCmalloc インターフェースにアクセスし、BE のメモリ使用状況を分析します。**

```bash
http://be_ip:8040/memz
```

> 説明：
>
> * 上記の `be_ip` を BE ノードの実際の IP アドレスに変更してください。
> * BE の `be_http_port` はデフォルトで `8040` です。

例：

```plain text
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) Bytes in use by application
MALLOC: +   8851890176 ( 8441.8 MiB) Bytes in page heap freelist
MALLOC: +    143722232 (  137.1 MiB) Bytes in central cache freelist
MALLOC: +     21869824 (   20.9 MiB) Bytes in transfer cache freelist
MALLOC: +    832509608 (  793.9 MiB) Bytes in thread cache freelists
MALLOC: +     58195968 (   55.5 MiB) Bytes in malloc metadata
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) Actual memory used (physical + swap)
MALLOC: +  25231564800 (24062.7 MiB) Bytes released to OS (aka unmapped)
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) Virtual address space used
MALLOC:
MALLOC:         112388              Spans in use
MALLOC:            335              Thread heaps in use
MALLOC:           8192              Tcmalloc page size
------------------------------------------------
Call ReleaseFreeMemory() to release freelist memory to the OS (via madvise()).
Bytes released to the OS take up virtual address space but no physical memory.
```

指標の説明：

* `Bytes in use by application`: アプリケーションによって実際に使用されているメモリ。
* `Bytes in page heap freelist`: BE が使用しなくなったが、まだ OS に返還されていないメモリ。
* `Actual memory used`: OS が検出した BE の実際のメモリ使用量（BE は一部の空きメモリを予約し、OS に返還しないか、ゆっくりと返還します）。
* `Bytes released to OS`: BE が回収可能と設定したが、OS がまだ回収していないメモリ。

## メモリの分類

StarRocks BE のメモリは以下のカテゴリに分類されます。

| ラベル | Metric 名称 | 説明 | BE 関連設定 |
| --- | --- | --- | --- |
| process | starrocks_be_process_mem_bytes | BE プロセスが実際に使用するメモリ（予約された空きメモリを含まない）。| mem_limit |
| query_pool | starrocks_be_query_mem_bytes | BE クエリ層が使用する総メモリ。 | |
| load | starrocks_be_load_mem_bytes | インポートに使用される総メモリ。 | load_process_max_memory_limit_bytes, load_process_max_memory_limit_percent |
| table_meta | starrocks_be_tablet_meta_mem_bytes | メタデータの総メモリ。 | |
| compaction | starrocks_be_compaction_mem_bytes | バージョン統合に使用される総メモリ。 | compaction_max_memory_limit, compaction_max_memory_limit_percent |
| column_pool | starrocks_be_column_pool_mem_bytes | column pool メモリプール。ストレージ層のデータ読み取りを加速するための Column Cache。 | |
| page_cache | starrocks_be_storage_page_cache_mem_bytes | BE ストレージ層の page キャッシュ。 | disable_storage_page_cache, storage_page_cache_limit |
| chunk_allocator | starrocks_be_chunk_allocator_mem_bytes | CPU コアごとのキャッシュ。小さなメモリブロックの割り当てを加速するための Cache。 | chunk_reserved_bytes_limit |
| consistency | starrocks_be_consistency_mem_bytes | 定期的な一貫性チェックに使用されるメモリ。 | consistency_max_memory_limit_percent, consistency_max_memory_limit |
| schema_change | starrocks_be_schema_change_mem_bytes | Schema Change タスクに使用される総メモリ。 | memory_limitation_per_thread_for_schema_change |
| clone | starrocks_be_clone_mem_bytes | Tablet Clone タスクに使用される総メモリ。 | |
| update | starrocks_be_update_mem_bytes | プライマリキーモデルに使用される総メモリ。 | |

## メモリ関連の設定項目

### BE 設定項目

| 名称 | デフォルト値 | 説明|  
| --- | --- | --- |
| mem_limit | 90% | BE プロセスのメモリ上限。デフォルトでは、BE が配置されているマシンのメモリの 90% がハード上限、80% がソフト上限です。BE が単独でデプロイされている場合は設定不要ですが、他のメモリを多く使用するサービスと混在してデプロイされている場合は、適切に設定する必要があります。|
| load_process_max_memory_limit_bytes | 107374182400 | 単一ノード上のすべてのインポートスレッドが占有するメモリ上限。mem_limit * load_process_max_memory_limit_percent / 100 と load_process_max_memory_limit_bytes の小さい方の値を取ります。インポートメモリが限界に達すると、フラッシュとバックプレッシャーのロジックがトリガーされます。|
| load_process_max_memory_limit_percent | 30 | 単一ノード上のすべてのインポートスレッドが占有するメモリの上限比率。mem_limit * load_process_max_memory_limit_percent / 100 と load_process_max_memory_limit_bytes の小さい方の値を取ります。インポートメモリが限界に達すると、フラッシュとバックプレッシャーのロジックがトリガーされます。|
| compaction_max_memory_limit | -1 | Compaction のメモリ上限。mem_limit * compaction_max_memory_limit_percent / 100 と compaction_max_memory_limit の小さい方の値を取ります。-1 は制限なしを意味します。現在はデフォルト設定の変更を推奨しません。Compaction メモリが限界に達すると、Compaction タスクが失敗します。|
| compaction_max_memory_limit_percent | 100 | Compaction のメモリ上限比率。mem_limit * compaction_max_memory_limit_percent / 100 と compaction_max_memory_limit の小さい方の値を取ります。-1 は制限なしを意味します。現在はデフォルト設定の変更を推奨しません。Compaction メモリが限界に達すると、Compaction タスクが失敗します。|
| disable_storage_page_cache | false | PageCache を有効にするかどうか。PageCache を有効にすると、StarRocks は最近スキャンしたデータをキャッシュし、クエリの重複性が高いシナリオではクエリの効率が大幅に向上します。`true` は無効を意味します。この設定は storage_page_cache_limit と連携して使用され、メモリリソースが豊富で大量のデータスキャンがあるシナリオでは、クエリのパフォーマンスを向上させるために有効にすることができます。バージョン 2.4 から、このパラメータのデフォルト値は `TRUE` から `FALSE` に変更されました。バージョン 3.1 からは、このパラメータは静的から動的に変更されました。|
| storage_page_cache_limit | 20% | BE ストレージ層の page キャッシュが使用できるメモリの上限。|
| chunk_reserved_bytes_limit | 2147483648 | 小さなメモリブロックの割り当てを加速するための Cache。デフォルトの上限は 2GB です。メモリリソースが十分にある場合は有効にすることができます。|
| consistency_max_memory_limit_percent | 20 | 一貫性チェックタスクが使用するメモリの上限。mem_limit * consistency_max_memory_limit_percent / 100 と consistency_max_memory_limit の小さい方の値を取ります。メモリ使用が上限を超えると、一貫性チェックタスクが失敗します。 |
| consistency_max_memory_limit | 10G | 一貫性チェックタスクが使用するメモリの上限。mem_limit * consistency_max_memory_limit_percent / 100 と consistency_max_memory_limit の小さい方の値を取ります。メモリ使用が上限を超えると、一貫性チェックタスクが失敗します。 |
| memory_limitation_per_thread_for_schema_change | 2 | 単一の Schema Change タスクが使用するメモリの上限。メモリ使用が上限を超えると、Schema Change タスクが失敗します。|
| max_compaction_concurrency | -1 | Compaction スレッドの最大同時実行数（BaseCompaction と CumulativeCompaction の最大合計）。このパラメータは Compaction が過剰なメモリを使用するのを防ぎます。-1 は制限なしを意味します。0 は compaction を許可しないことを意味します。|

### Session 変数

| 名称| デフォルト値| 説明|
|  --- |  --- | --- |
| query_mem_limit | 0 | 各 BE ノード上の単一クエリのメモリ制限。単位は Byte です。17179869184（16GB）以上に設定することを推奨します。 |
| load_mem_limit| 0| 各 BE ノード上の単一インポートタスクのメモリ制限。単位は Byte です。`0` に設定されている場合、StarRocks は `exec_mem_limit` をメモリ制限として使用します。|
