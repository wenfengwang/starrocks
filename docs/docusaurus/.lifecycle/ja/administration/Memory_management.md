---
displayed_sidebar: "Japanese"
---

# メモリ管理

このセクションでは、メモリの分類とStarRocksのメモリ管理方法について簡単に紹介します。

## メモリの分類

説明：

| メトリック | 名前 | 説明 |
| --- | --- | --- |
| プロセス | BEの使用済み総メモリ | |
| クエリプール | データクエリの使用メモリ | 実行レイヤーとストレージレイヤーのメモリから構成されます。|
| ロード | データロードに使用されるメモリ | 通常はMemTable|
| table_meta | メタデータメモリ | Sスキーマ、Tabletメタデータ、RowSetメタデータ、Columnメタデータ、ColumnReader、IndexReader |
| compaction | マルチバージョンメモリコンパクション | データのインポート完了後に発生するコンパクション |
| snapshot | スナップショットメモリ | 通常、クローンに使用され、ほとんどメモリを使用しません |
| column_pool | カラムプールメモリ | カラムを加速化するためのカラムキャッシュの解放要求 |
| page_cache | BE独自のPageCache | デフォルトはオフであり、ユーザーはBEファイルを修正することでオンにできます。 |

## メモリ関連の設定

* **BE設定**

| 名前 | デフォルト | 説明 |  
| --- | --- | --- |
| vector_chunk_size | 4096 | チャンクの行数 |
| mem_limit | 80% | BEが使用できる総メモリの割合。BEが単独で展開される場合は設定する必要はありません。より多くのメモリを消費する他のサービスと一緒に展開される場合は、それぞれに設定する必要があります。 |
| disable_storage_page_cache | false | PageCacheを無効にするかどうかを制御するブール値。PageCacheが有効になっていると、StarRocksは最近スキャンされたデータをキャッシュします。 PageCacheは類似のクエリが頻繁に繰り返される場合にクエリのパフォーマンスを大幅に向上させることができます。 `true` はPageCacheを無効にすることを示します。 `storage_page_cache_limit`と一緒に使用して、十分なメモリリソースと多くのデータスキャンを伴うシナリオでクエリのパフォーマンスを高速化できます。この項目のデフォルト値は、StarRocks v2.4以降で `true` から `false` に変更されました。 |
| write_buffer_size | 104857600 | 単一のMemTableの容量制限。これを超えるとディスクスワイプが実行されます。 |
| load_process_max_memory_limit_bytes | 107374182400 | BEノード上のすべてのロードプロセスが取り込むことができるメモリリソースの上限値。その値は `mem_limit * load_process_max_memory_limit_percent / 100` と `load_process_max_memory_limit_bytes` のうち小さい方です。このしきい値を超えると、フラッシュとバックプレッシャがトリガーされます。 |
| load_process_max_memory_limit_percent | 30 | BEノード上のすべてのロードプロセスが取り込むことができるメモリリソースの最大パーセンテージ。その値は `mem_limit * load_process_max_memory_limit_percent / 100` と `load_process_max_memory_limit_bytes` のうち小さい方です。このしきい値を超えると、フラッシュとバックプレッシャがトリガーされます。 |
| default_load_mem_limit | 2147483648 | 単一のインポートインスタンスの受信側でメモリ制限に達した場合、ディスクスワイプが実行されます。これを有効にするには、 `load_mem_limit` のセッション変数を修正する必要があります。 |
| max_compaction_concurrency | -1 | コンパクションの最大並行性（ベースコンパクションと累積コンパクションの両方） 。値が -1 の場合、並行性に制限はありません。 |
| cumulative_compaction_check_interval_seconds | 1 | コンパクションチェックのインターバル|

* **セッション変数**

| 名前| デフォルト| 説明|
| --- | --- | --- |
| query_mem_limit| 0| 各バックエンドノードのクエリのメモリ制限 |
| load_mem_limit | 0| 単一のインポートタスクのメモリ制限。値が0の場合、 `exec_mem_limit` が適用されます|

## メモリ使用量の表示

* **mem_tracker**

~~~ bash
//全体のメモリ統計を表示
<http://be_ip:be_http_port/mem_tracker>

// 細かいメモリ統計を表示
<http://be_ip:be_http_port/mem_tracker?type=query_pool&upper_level=3>
~~~

* **tcmalloc**

~~~ bash
<http://be_ip:be_http_port/memz>
~~~

~~~plain text
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) アプリケーションによって使用されるバイト数
MALLOC: +   8851890176 ( 8441.8 MiB) ページヒープフリーリストにあるバイト数
MALLOC: +    143722232 (  137.1 MiB) セントラルキャッシュフリーリストにあるバイト数
MALLOC: +     21869824 (   20.9 MiB) 転送キャッシュフリーリストにあるバイト数
MALLOC: +    832509608 (  793.9 MiB) スレッドキャッシュフリーリストにあるバイト数
MALLOC: +     58195968 (   55.5 MiB) Mallocメタデータにあるバイト数
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 実際に使用されているメモリ（物理+スワップ）
MALLOC: +  25231564800 (24062.7 MiB) OSにリリースされたバイト数（アンマップとも呼ばれる）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用されている仮想アドレス空間
MALLOC:
MALLOC:         112388              使用中のスパン
MALLOC:            335              使用中のスレッドヒープ
MALLOC:           8192              Tcmallocページサイズ
------------------------------------------------
ReleaseFreeMemory（）を呼び出して、フリーリストのメモリをOSにリリースできます（madvise（）を介して）。 
OSに解放されたバイトは仮想アドレス空間を占有しますが、物理メモリを占有しません。
~~~

この方法でクエリされるメモリは正確です。ただし、StarRocksの一部のメモリは予約されていますが使用されていません。TcMallocは予約されたメモリを数えており、使用されていないメモリを数えていません。

ここでの `アプリケーションによって使用されるバイト数` は、現在使用されているメモリを指します。

* **メトリクス**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

メトリクスの値は10秒ごとに更新されます。以前のバージョンでは一部のメモリ統計を監視することができる可能性があります。