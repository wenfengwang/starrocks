---
displayed_sidebar: "Japanese"
---

# メモリ管理

このセクションでは、メモリの分類とStarRocksのメモリ管理方法について簡単に紹介します。

## メモリの分類

説明：

| メトリック | 名前 | 説明 |
| --- | --- | --- |
| process   |  BEが使用する合計メモリ  | |
| query\_pool   |   データクエリの使用メモリ  | 実行レイヤーのメモリとストレージレイヤーのメモリで構成されます。|
| load   |  データのロードに使用されるメモリ    | 一般的にはMemTable|
| table_meta   |   メタデータのメモリ  | スキーマ、テーブルメタデータ、RowSetメタデータ、列メタデータ、列リーダー、インデックスリーダー |
| compaction   |   マルチバージョンメモリコンパクション  |  データのインポート完了後に発生するコンパクション |
| snapshot  |   スナップショットメモリ  |  一般的にはクローンに使用され、メモリ使用量が少ない |
| column_pool   |    カラムプールメモリ   | 加速カラムのためのカラムキャッシュの解放要求 |
| page_cache   |   BE独自のページキャッシュ   | デフォルトはオフで、ユーザーはBEファイルを変更してオンにすることができます。|

## メモリ関連の設定

* **BE構成**

| 名前 | デフォルト| 説明|  
| --- | --- | --- |
| vector_chunk_size | 4096 | チャンク行数 |
| mem_limit | 80% | BEが使用できる総メモリの割合。BEが独立して展開されている場合は設定する必要はありません。他のメモリを多く消費するサービスと共に展開されている場合は、それぞれに設定する必要があります。 |
| disable_storage_page_cache | false | PageCacheを無効にするかどうかを制御するブール値。PageCacheが有効の場合、StarRocksは最近スキャンされたデータをキャッシュします。PageCacheは、類似のクエリが頻繁に繰り返される場合にクエリパフォーマンスを大幅に改善できます。 `true` はPageCacheを無効にします。このアイテムは `storage_page_cache_limit` と一緒に使用します。これにより、十分なメモリリソースと多くのデータスキャンのシナリオでクエリパフォーマンスを加速できます。このアイテムのデフォルト値は、StarRocks v2.4 以降で `true` から `false` に変更されています。 |
| write_buffer_size | 104857600 |  単一MemTableの容量制限。これを超えるとディスクスワイプが実行されます。 |
| load_process_max_memory_limit_bytes | 107374182400 | BEノード上のすべてのロードプロセスが利用できるメモリリソースの上限値。その値は、`mem_limit * load_process_max_memory_limit_percent / 100` と `load_process_max_memory_limit_bytes` の小さい方です。この閾値を超えると、フラッシュとバックプレッシャがトリガーされます。 |
| load_process_max_memory_limit_percent | 30 | BEノード上のすべてのロードプロセスによって利用できるメモリリソースの最大パーセンテージ。その値は、`mem_limit * load_process_max_memory_limit_percent / 100` と `load_process_max_memory_limit_bytes` の小さい方です。この閾値を超えると、フラッシュとバックプレッシャがトリガーされます。 |
| default_load_mem_limit | 2147483648 | シングルインポートタスクの受信側のメモリ制限に達するとディスクスワイプが実行されます。これは `load_mem_limit` セッション変数を使用して変更する必要があります。 |
| max_compaction_concurrency | -1 | コンパクションの最大同時処理数（ベースコンパクションと累積コンパクションの両方）。値 -1 は同時処理数に制限がないことを示します。 |
| cumulative_compaction_check_interval_seconds | 1 | コンパクションチェックの間隔|

* **セッション変数**

| 名前| デフォルト| 説明|
| --- | --- | --- |
| query_mem_limit| 0| 各バックエンドノードのクエリのメモリ制限 |
| load_mem_limit | 0| シングルインポートタスクのメモリ制限。値が 0 の場合、 `exec_mem_limit` が適用されます|

## メモリ使用状況の表示

* **mem\_tracker**

~~~ bash
//全体のメモリ統計を表示
<http://be_ip:be_http_port/mem_tracker>

//詳細なメモリ統計を表示
<http://be_ip:be_http_port/mem_tracker?type=query_pool&upper_level=3>
~~~

* **tcmalloc**

~~~ bash
<http://be_ip:be_http_port/memz>
~~~

~~~plain text
------------------------------------------------
MALLOC:      777276768 (  741.3 MiB) アプリケーションが使用するバイト
MALLOC: +   8851890176 ( 8441.8 MiB) ページヒープフリーリストにあるバイト
MALLOC: +    143722232 (  137.1 MiB) 中央キャッシュフリーリストにあるバイト
MALLOC: +     21869824 (   20.9 MiB) 転送キャッシュフリーリストにあるバイト
MALLOC: +    832509608 (  793.9 MiB) スレッドキャッシュフリーリストにあるバイト
MALLOC: +     58195968 (   55.5 MiB) mallocメタデータにあるバイト
MALLOC:   ------------
MALLOC: =  10685464576 (10190.5 MiB) 実際に使用されているメモリ（物理的+スワップ）
MALLOC: +  25231564800 (24062.7 MiB) OSにリリースされたバイト（アンマップされたものとも呼ばれます）
MALLOC:   ------------
MALLOC: =  35917029376 (34253.1 MiB) 使用されている仮想アドレス空間
MALLOC:
MALLOC:         112388              使用中のスパン
MALLOC:            335              使用中のスレッドヒープ
MALLOC:           8192              Tcmallocページサイズ
------------------------------------------------
ReleaseFreeMemory() を呼び出して、フリーリストメモリをOSにリリースします（madvise() を介して）。  
OSにリリースされたバイトは仮想アドレス空間を占有しますが、物理メモリは消費しません。

`Bytes in use by application` は現在使用中のメモリを指します。

* **metrics**

~~~bash
curl -XGET http://be_ip:be_http_port/metrics | grep 'mem'
curl -XGET http://be_ip:be_http_port/metrics | grep 'column_pool'
~~~

メトリクスの値は10秒ごとに更新されます。古いバージョンでも一部のメモリ統計を監視することができます。