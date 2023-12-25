---
displayed_sidebar: Chinese
---

# Broker Load 常見の問題点

## 1. Broker Load は、既に成功して FINISHED 状態のインポートジョブを再実行できますか？

Broker Load は、既に成功し FINISHED 状態のインポートジョブを再実行することはできません。また、データの損失や重複を防ぐため、成功したインポートジョブのラベル（Label）は再利用できません。[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) ステートメントを使用して、過去のインポート履歴を確認し、再実行したいインポートジョブを見つけ、ジョブ情報をコピーしてラベルを変更した後、新しいインポートジョブを作成して実行します。

## 2. Broker Load を使用して HDFS データをインポートする際、なぜデータのインポート日付フィールドに異常が発生し、正しい日時よりも 8 時間多く加算されるのですか？この状況をどのように処理すればよいですか？

StarRocks のテーブルを作成する際に設定された `timezone` は中国標準時であり、Broker Load インポートジョブを作成する際に設定された `timezone` も中国標準時ですが、サーバーは UTC 時間帯に設定されています。したがって、インポート時に日付フィールドに正しい日時よりも 8 時間が加算されます。この問題を避けるためには、テーブルを作成する際に `timezone` パラメータを削除する必要があります。

## 3. Broker Load を使用して ORC 形式のデータをインポートする際に `ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>` エラーが発生した場合、どのように対処すればよいですか？

ソースデータファイルと StarRocks テーブルの両方の列名が一致していないため、`SET` 句を実行する際にシステム内部で型推論が行われますが、[cast](../../sql-reference/sql-functions/cast.md) 関数を呼び出してデータ型の変換を実行する際に失敗しました。解決策は、両方の列名を一致させることです。そうすれば `SET` 句は不要になり、cast 関数を呼び出してデータ型の変換を行うこともなく、インポートが成功します。

## 4. Broker Load インポートジョブにエラーがないのに、なぜデータをクエリできないのですか？

Broker Load は非同期のインポート方法です。インポートジョブを作成するステートメントにエラーがないということは、インポートジョブが成功したとは限りません。[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) ステートメントを使用してインポートジョブの結果ステータスと `errmsg` 情報を確認し、インポートジョブのパラメータ設定を修正した後、インポートジョブを再試行します。

## 5. インポート時に "failed to send batch" または "TabletWriter add batch with unknown id" エラーが報告された場合、どのように対処すればよいですか？

このエラーはデータ書き込みのタイムアウトが原因です。[システム変数](../../reference/System_variable.md) `query_timeout` と [BE 設定項目](../../administration/BE_configuration.md) `streaming_load_rpc_max_alive_time_sec` の設定を変更する必要があります。

## 6. インポート時に "LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found" エラーが報告された場合、どのように対処すればよいですか？

Parquet または ORC 形式のデータをインポートする場合、ファイルヘッダーの列名が StarRocks テーブルの列名と一致しているかを確認してください。例えば：

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上記の例では、Parquet または ORC ファイル内の `tmp_c1` と `tmp_c2` という列名の列を、それぞれ StarRocks テーブル内の `name` と `id` 列にマッピングしています。`SET` 句を使用しない場合は、`column_list` パラメータで指定された列をマッピングとして使用します。詳細は [BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

> **注意**
>
> Apache Hive™ が直接生成した ORC ファイルをインポートする場合、ORC ファイルのヘッダーが `(_col0, _col1, _col2, ...)` であると "Invalid Column Name" エラーが発生する可能性があります。この場合は `SET` 句を使用して列の変換ルールを設定する必要があります。

## 7. その他、例えばインポートジョブが長時間終了しないなどの問題が発生した場合、どのように対処すればよいですか？

FE のログファイル **fe.log** で、インポートジョブのラベルに基づいてインポートジョブの ID を検索します。次に、BE のログファイル **be.INFO** で、インポートジョブの ID に基づいてコンテキストログを検索し、具体的な原因を確認します。

## 8. 高可用性（HA）モードの Apache HDFS クラスターにアクセスするにはどのように設定しますか？

以下のように設定します：

- `dfs.nameservices`：HDFS クラスターの名前をカスタマイズします。例：`"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`：NameNode の名前をカスタマイズします。複数の名前はコンマ（,）で区切ります。ここでの `xxx` は `dfs.nameservices` で設定された HDFS クラスターの名前です。例：`"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：NameNode の RPC アドレス情報を指定します。ここでの `nn` は `dfs.ha.namenodes.xxx` でカスタマイズされた NameNode の名前を表します。例：`"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`：クライアントが NameNode に接続するためのプロバイダーを指定します。デフォルトは `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider` です。

例は以下の通りです：

```SQL
(
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my-namenode1, my-namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

高可用性モードは、シンプル認証および Kerberos 認証の2つの認証方式と組み合わせて HDFS クラスターにアクセスできます。例えば、シンプル認証を使用して高可用性モードでデプロイされた HDFS クラスターにアクセスするには、以下の設定を指定する必要があります：

```SQL
(
    "username"="user",
    "password"="passwd",
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my_namenode1, my_namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

HDFS クラスターの設定は **hdfs-site.xml** ファイルに記述できます。これにより、Broker プログラムを使用して HDFS クラスターのデータを読み取る際には、クラスターのファイルパス名と認証情報のみを入力する必要があります。

## 9. Hadoop ViewFS Federation をどのように設定しますか？

ViewFs 関連の設定ファイル `core-site.xml` と `hdfs-site.xml` を **broker/conf** ディレクトリにコピーする必要があります。

カスタムファイルシステムがある場合は、関連する **.jar** ファイルを **broker/lib** ディレクトリにコピーする必要があります。

## 10. Kerberos 認証を使用して HDFS クラスターにアクセスする際に "Can't get Kerberos realm" エラーが報告された場合、どのように対処すればよいですか？

まず、すべての Broker が配置されているマシンに **/etc/krb5.conf** ファイルが設定されているかを確認してください。

設定されていてもエラーが発生する場合は、Broker の起動スクリプトで `JAVA_OPTS` 変数の最後に `-Djava.security.krb5.conf=/etc/krb5.conf` を追加する必要があります。
