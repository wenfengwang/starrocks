---
displayed_sidebar: English
---

# メモリ管理

このセクションでは、メモリの分類とStarRocksのメモリ管理方法について簡単に紹介します。

## メモリの分類

説明：

|   メトリック  | 名前 | 説明 |
| --- | --- | --- |
|  process   |  BEの全メモリ使用量  | |
|  query\_pool   |   データクエリに使用されるメモリ  | 実行層とストレージ層で使用されるメモリの2部分から構成されます。|
|  load   |  データロードに使用されるメモリ    | 一般的にはMemTableです。|
|  table_meta   |   メタデータメモリ | スキーマ、タブレットメタデータ、RowSetメタデータ、カラムメタデータ、ColumnReader、IndexReaderなど |
|  compaction   |   マルチバージョンメモリ圧縮  |  データインポート完了後に行われる圧縮 |
|  snapshot  |   スナップショットメモリ  | 通常はクローン用で、メモリ使用量は少ない |
|  column_pool   |    カラムプールメモリ   | 加速されたカラムのカラムキャッシュを解放するためのリクエスト |
|  page_cache   |   BEのPageCache   | デフォルトはオフで、ユーザーはBEの設定を変更してオンにすることができます |

## メモリ関連の設定

* **BE設定**

| 名前 | デフォルト| 説明|  
| --- | --- | --- |
| vector_chunk_size | 4096 | チャンクの行数 |
| mem_limit | 80% | BEが使用できる全メモリの割合。BEが単独でデプロイされている場合は設定不要です。他のメモリを多く消費するサービスと共にデプロイされる場合は、個別に設定する必要があります。 |
| disable_storage_page_cache | false | PageCacheを無効にするかどうかを制御するブール値。PageCacheが有効の場合、StarRocksは最近スキャンしたデータをキャッシュします。PageCacheは、類似のクエリが頻繁に繰り返される場合にクエリパフォーマンスを大幅に向上させることができます。`true`はPageCacheを無効にすることを示します。この項目を`storage_page_cache_limit`と一緒に使用すると、十分なメモリリソースと多くのデータスキャンがあるシナリオでクエリパフォーマンスを向上させることができます。この項目のデフォルト値はStarRocks v2.4以降`true`から`false`に変更されました。|
| write_buffer_size | 104857600 |  単一MemTableの容量制限で、これを超えるとディスクスワップが実行されます。 |
| load_process_max_memory_limit_bytes | 107374182400 | BEノード上のすべてのロードプロセスが使用できるメモリリソースの上限。その値は`mem_limit * load_process_max_memory_limit_percent / 100`と`load_process_max_memory_limit_bytes`のうち小さい方です。この閾値を超えると、フラッシュとバックプレッシャーが発生します。  |
| load_process_max_memory_limit_percent | 30 | BEノード上のすべてのロードプロセスが使用できるメモリリソースの最大割合。その値は`mem_limit * load_process_max_memory_limit_percent / 100`と`load_process_max_memory_limit_bytes`のうち小さい方です。この閾値を超えると、フラッシュとバックプレッシャーが発生します。 |
| default_load_mem_limit | 2147483648 | 単一インポートインスタンスの受信側メモリ制限に達すると、ディスクスワップがトリガされます。これを変更するには、セッション変数`load_mem_limit`とともに設定する必要があります。 |
| max_compaction_concurrency | -1 | 圧縮（ベース圧縮と累積圧縮の両方）の最大同時実行数。-1の値は、同時実行数に制限がないことを示します。 |
| cumulative_compaction_check_interval_seconds | 1 | 圧縮チェックの間隔 |

* **セッション変数**

| 名前| デフォルト| 説明|
| --- | --- | --- |
| query_mem_limit| 0| 各バックエンドノードでのクエリのメモリ制限 |
| load_mem_limit | 0| 単一インポートタスクのメモリ制限。値が0の場合は`exec_mem_limit`が使用されます。|

## メモリ使用状況の確認

* **mem\_tracker**

~~~ bash
// 全体的なメモリ統計を表示
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
MALLOC:      777276768 (  741.3 MiB) アプリケーションが使用中のバイト数
MALLOC: +   8851890176 ( 8441.8 MiB) ページヒープフリーリストのバイト数
MALLOC: +    143722232 (  137.1 MiB) セントラルキャッシュフリーリストのバイト数
MALLOC: +     21869824 (   20.9 MiB) トランスファーキャッシュフリーリストのバイト数
MALLOC: +    832509608 (  793.9 MiB) スレッドキャッシュフリーリストのバイト数
MALLOC: +     58195968 (   55.5 MiB) mallocメタデータのバイト数
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 実際に使用されているメモリ（物理メモリ + スワップ）
MALLOC: +  25231564800 (24062.7 MiB) OSに解放されたバイト数（別名アンマップ）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用されている仮想アドレス空間
MALLOC:
MALLOC:         112388              使用中のスパン
MALLOC:            335              使用中のスレッドヒープ
MALLOC:           8192              Tcmallocページサイズ
------------------------------------------------
ReleaseFreeMemory()を呼び出して、フリーリストメモリをOSに解放します（madvise()を介して）。
OSに解放されたバイトは仮想アドレス空間を占めますが、物理メモリは使用しません。
~~~

この方法でクエリされたメモリは正確です。しかし、StarRocksには予約されているが使用されていないメモリがあります。TcMallocは、使用されているメモリではなく、予約されているメモリをカウントします。

ここでの`アプリケーションが使用中のバイト数`は、現在使用中のメモリを指します。

* **メトリクス**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

メトリクスの値は10秒ごとに更新されます。古いバージョンでは、メモリ統計の一部をモニタリングすることが可能です。
