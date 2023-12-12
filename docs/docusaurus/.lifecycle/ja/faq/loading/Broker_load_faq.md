---
displayed_sidebar: "Japanese"
---

# Broker Load（ブローカー ロード）

## 1. ブローカー ロードは、正常に実行され、FINISHED ステートにあるロード ジョブを再実行できますか？

ブローカー ロードは、正常に実行され、FINISHED ステートにあるロード ジョブを再実行することはできません。また、データの損失と重複を防ぐため、ブローカー ロードでは正常に実行されたロード ジョブのラベルを再利用することはできません。[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を使用して、ロード ジョブの履歴を表示し、再実行したいロード ジョブを見つけることができます。その後、そのロード ジョブの情報をコピーし、ラベル以外の情報を使用して、別のロード ジョブを作成することができます。

## 2. Broker Load を使用して HDFS からデータをロードする際、宛先の StarRocks テーブルにロードされる日時値が、ソース データ ファイルの日時値よりも 8 時間遅れている場合はどうすればよいですか？

宛先の StarRocks テーブルとブローカー ロード ジョブは、`timezone` パラメータを使用して作成され、中国標準時（CST）タイムゾーンを使用します。しかし、サーバーは協定世界時（UTC）タイムゾーンで実行されています。その結果、データのロード中に、ソース データ ファイルの日時値に 8 時間が追加されます。この問題を防ぐには、宛先の StarRocks テーブルを作成する際に `timezone` パラメータを指定しないでください。

## 3. Broker Load を使用して ORC 形式のデータをロードする際、`ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>` エラーが発生した場合はどうすればよいですか？

ソース データ ファイルの列名が宛先の StarRocks テーブルと異なる場合、`SET` 句を使用してファイルとテーブルの列のマッピングを指定する必要があります。`SET` 句を実行する際、StarRocks は型の推論を行う必要がありますが、それに失敗しています。これは、[cast](../../sql-reference/sql-functions/cast.md) 関数を呼び出してソース データを宛先のデータ型に変換しようとした際に失敗したことです。この問題を解決するには、ソース データ ファイルに宛先の StarRocks テーブルと同じ列名があることを確認してください。そのため、`SET` 句は不要であり、したがって StarRocks はデータ型変換を行うために cast 関数を呼び出す必要がありません。その後に、ブローカー ロード ジョブを正常に実行できます。

## 4. Broker Load ジョブはエラーを報告しませんが、ロードされたデータをクエリできない理由は何ですか？

ブローカー ロードは非同期ロードの方法です。ロード ジョブがエラーを返さなくても、実際には失敗することがあります。Broker Load ジョブを実行した後、[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を使用して、ロード ジョブの結果や `errmsg` を表示することができます。その後、ジョブの構成を変更して再実行することができます。

## 5. "failed to send batch" または "TabletWriter add batch with unknown id" エラーが発生した場合はどうすればよいですか？

データの書き込みにかかる時間が上限を超えてしまい、タイムアウト エラーが発生しています。この問題を解決するには、ビジネス要件に基づいて、[session variable](../../reference/System_variable.md) `query_timeout` および [BE configuration item](../../administration/Configuration.md#configure-be-static-parameters) `streaming_load_rpc_max_alive_time_sec` の設定を変更してください。

## 6. "LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found" エラーが発生した場合はどうすればよいですか？

Parquet 形式または ORC 形式のデータをロードしている場合は、ソース データ ファイルの最初の行で保持されている列名が宛先の StarRocks テーブルの列名と同じかどうかを確認してください。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上記の例では、ソース データ ファイルの `tmp_c1` および `tmp_c2` 列を、宛先の StarRocks テーブルの `name` 列および `id` 列にそれぞれマッピングしています。`SET` 句を指定しない場合、`column_list` パラメータで指定された列名が、列のマッピングを宣言するために使用されます。詳細については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

> **NOTICE**
>
> ソース データ ファイルが Apache Hive™ によって生成された ORC 形式のファイルであり、ファイルの最初の行に `(_col0, _col1, _col2, ...)` が保持されている場合、"Invalid Column Name" エラーが発生することがあります。このエラーが発生した場合は、`SET` 句を使用して列のマッピングを指定する必要があります。

## 7. ブローカー ロード ジョブが長時間実行される原因となるエラーを処理するにはどうすればよいですか？

FE ログ ファイル **fe.log** を表示し、ジョブ ラベルに基づいてロード ジョブの ID を検索します。その後、BE ログ ファイル **be.INFO** を表示し、ジョブ ID に基づいてロード ジョブのログ レコードを取得して、エラーの原因を特定します。

## 8. 高可用性（HA）モードで実行される Apache HDFS クラスタを構成する方法は？

HDFS クラスタが高可用性（HA）モードで実行される場合、以下のように構成します。

- `dfs.nameservices` : HDFS クラスタの名前です。たとえば、 `"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx` : HDFS クラスタ内の NameNode の名前です。複数の NameNode 名を指定する場合は、カンマ（`,`）で区切ります。`xxx` は、`dfs.nameservices` で指定した HDFS クラスタ名です。たとえば、 `"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn` : HDFS クラスタ内の NameNode の RPC アドレスです。`nn` は、`dfs.ha.namenodes.xxx` で指定した NameNode 名です。たとえば、 `"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider` : クライアントが接続する NameNode のプロバイダーです。デフォルト値: `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。

例:

```SQL
(
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my-namenode1, my-namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

HA モードは、簡単な認証または Kerberos 認証と併用できます。たとえば、HA モードで実行される HDFS クラスタに簡単な認証を使用するには、次の構成を指定する必要があります。

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

HDFS クラスタの構成を **hdfs-site.xml** ファイルに追加することができます。これにより、ブローカーを使用して HDFS クラスタからデータをロードする際には、ファイル パスと認証情報のみを指定すればよくなります。

## 9. Hadoop ViewFS Federation を構成する方法は？

ViewFs 関連の構成ファイル `core-site.xml` と `hdfs-site.xml` を **broker/conf** ディレクトリにコピーします。

カスタム ファイル システムがある場合は、ファイル システム関連の **.jar** ファイルも **broker/lib** ディレクトリにコピーする必要があります。

## 10. Kerberos 認証が必要な HDFS クラスタにアクセスする際、"Can't get Kerberos realm" エラーが発生した場合はどうすればよいですか？

すべてのブローカーが展開されているホスト上で、**/etc/krb5.conf** ファイルが構成されていることを確認してください。

エラーが解消しない場合は、ブローカーの起動スクリプトの `JAVA_OPTS` 変数の末尾に、`-Djava.security.krb5.conf:/etc/krb5.conf` を追加してください。