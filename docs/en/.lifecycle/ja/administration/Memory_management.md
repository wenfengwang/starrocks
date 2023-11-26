---
displayed_sidebar: "Japanese"
---

# メモリ管理

このセクションでは、メモリの分類とStarRocksのメモリ管理方法について簡単に紹介します。

## メモリの分類

説明：

|   メトリック  | 名前 | 説明 |
| --- | --- | --- |
|  process   |  BEの使用済み総メモリ  | |
|  query\_pool   |   データクエリの使用メモリ  | 実行レイヤーとストレージレイヤーのメモリで構成されます。|
|  load   |  データロードの使用メモリ    | 通常はMemTableです。|
|  table_meta   |   メタデータメモリ | Sスキーマ、タブレットメタデータ、RowSetメタデータ、カラムメタデータ、カラムリーダー、インデックスリーダー |
|  compaction   |   マルチバージョンメモリコンパクション  |  データのインポートが完了した後に発生するコンパクション |
|  snapshot  |   スナップショットメモリ  | 通常はクローンに使用され、メモリ使用量は少ないです。 |
|  column_pool   |    カラムプールメモリ   | 高速化されたカラムのためのカラムキャッシュの解放要求 |
|  page_cache   |   BEの独自のPageCache   | デフォルトはオフで、ユーザーはBEファイルを変更することでオンにすることができます。 |

## メモリ関連の設定

* **BEの設定**

| 名前 | デフォルト| 説明|  
| --- | --- | --- |
| vector_chunk_size | 4096 | チャンクの行数 |
| mem_limit | 80% | BEが使用できる総メモリの割合。BEがスタンドアロンで展開されている場合は、設定する必要はありません。他のメモリを消費するサービスと一緒に展開されている場合は、個別に設定する必要があります。 |
| disable_storage_page_cache | false | PageCacheを無効にするかどうかを制御するブール値。PageCacheが有効になっている場合、StarRocksは最近スキャンされたデータをキャッシュします。PageCacheは、類似のクエリが頻繁に繰り返される場合にクエリのパフォーマンスを大幅に向上させることができます。`true`はPageCacheを無効にすることを示します。この項目を`storage_page_cache_limit`と一緒に使用すると、十分なメモリリソースと多くのデータスキャンがあるシナリオでクエリのパフォーマンスを高速化することができます。StarRocks v2.4以降、この項目のデフォルト値は`true`から`false`に変更されました。 |
| write_buffer_size | 104857600 |  単一のMemTableの容量制限。これを超えるとディスクスワイプが実行されます。 |
| load_process_max_memory_limit_bytes | 107374182400 | BEノード上のすべてのロードプロセスが使用できるメモリリソースの上限。その値は、`mem_limit * load_process_max_memory_limit_percent / 100`と`load_process_max_memory_limit_bytes`のうち小さい方です。この閾値を超えると、フラッシュとバックプレッシャーがトリガーされます。  |
| load_process_max_memory_limit_percent | 30 | BEノード上のすべてのロードプロセスが使用できるメモリリソースの最大パーセンテージ。その値は、`mem_limit * load_process_max_memory_limit_percent / 100`と`load_process_max_memory_limit_bytes`のうち小さい方です。この閾値を超えると、フラッシュとバックプレッシャーがトリガーされます。 |
| default_load_mem_limit | 2147483648 | 単一のインポートインスタンスの受信側のメモリ制限に達した場合、ディスクスワイプがトリガーされます。これはセッション変数`load_mem_limit`と一緒に変更する必要があります。 |
| max_compaction_concurrency | -1 | コンパクションの最大同時実行数（ベースコンパクションと累積コンパクションの両方）。値-1は同時実行数に制限がないことを示します。 |
| cumulative_compaction_check_interval_seconds | 1 | コンパクションチェックの間隔|

* **セッション変数**

| 名前| デフォルト| 説明|
| --- | --- | --- |
| query_mem_limit| 0| 各バックエンドノードのクエリのメモリ制限 |
| load_mem_limit | 0| 単一のインポートタスクのメモリ制限。値が0の場合、`exec_mem_limit`が使用されます。 |

## メモリ使用状況の表示

* **mem\_tracker**

~~~ bash
// 全体のメモリ統計を表示
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
MALLOC:      777276768 (  741.3 MiB) アプリケーションによって使用されているバイト数
MALLOC: +   8851890176 ( 8441.8 MiB) ページヒープフリーリストによって解放されたバイト数
MALLOC: +    143722232 (  137.1 MiB) セントラルキャッシュフリーリストによって解放されたバイト数
MALLOC: +     21869824 (   20.9 MiB) トランスファーキャッシュフリーリストによって解放されたバイト数
MALLOC: +    832509608 (  793.9 MiB) スレッドキャッシュフリーリストによって解放されたバイト数
MALLOC: +     58195968 (   55.5 MiB) mallocメタデータによって使用されているバイト数
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 実際に使用されているメモリ（物理メモリ+スワップ）
MALLOC: +  25231564800 (24062.7 MiB) OSに解放されたバイト数（アンマップされたもの）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用されている仮想アドレススペース
MALLOC:
MALLOC:         112388              使用中のスパン
MALLOC:            335              使用中のスレッドヒープ
MALLOC:           8192              Tcmallocのページサイズ
------------------------------------------------
ReleaseFreeMemory()を呼び出してフリーリストメモリをOSに解放します（madvise()を介して）。
OSに解放されたバイトは仮想アドレススペースを占有しますが、物理メモリは占有しません。
~~~

この方法でクエリされるメモリは正確です。ただし、StarRocksの一部のメモリは予約されていますが使用されていません。TcMallocは予約されたメモリをカウントし、使用されているメモリをカウントしません。

ここで、「アプリケーションによって使用されているバイト数」とは、現在使用されているメモリを指します。

* **メトリクス**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

メトリクスの値は10秒ごとに更新されます。古いバージョンでも一部のメモリ統計を監視することができます。
