---
displayed_sidebar: "Japanese"
---

# ブローカーロード

## 1. ブローカーロードは、正常に実行されてFINISHED状態にあるロードジョブを再実行することをサポートしていますか？

ブローカーロードは、正常に実行されてFINISHED状態にあるロードジョブを再実行することをサポートしていません。また、データの損失や重複を防ぐため、ブローカーロードでは正常に実行されたロードジョブのラベルを再利用することができません。 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を使用して、ロードジョブの履歴を表示し、再実行したいロードジョブを見つけることができます。その後、そのロードジョブの情報をコピーし、ラベル以外のジョブ情報を使用して別のロードジョブを作成することができます。

## 2. ブローカーロードを使用してHDFSからデータをロードする際、宛先のStarRocksテーブルにロードされる日付と時間値が、ソースデータファイルの日付と時間値よりも8時間遅れている場合はどうすればよいですか？

宛先のStarRocksテーブルとブローカーロードジョブは、中国標準時（`timezone` パラメータを使用して指定）を使用して作成されます。しかし、サーバーは協定世界時（UTC）タイムゾーンで実行されています。その結果、データのロード中に元のデータファイルからの日付と時間値に8時間が追加されます。この問題を防ぐためには、宛先のStarRocksテーブルを作成する際に `timezone` パラメータを指定しないでください。

## 3. ブローカーロードを使用してORC形式のデータをロードする際、`ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>` エラーが発生した場合はどうすればよいですか？

ソースデータファイルの列名が宛先のStarRocksテーブルの列名と異なる場合、ロードステートメントで `SET` 句を使用してファイルとテーブルの列マッピングを指定する必要があります。 `SET` 句を実行する際、StarRocksは型推論を実行する必要がありますが、ソースデータを宛先のデータ型に変換するために [cast](../../sql-reference/sql-functions/cast.md) 関数を呼び出すことが失敗します。この問題を解決するには、ソースデータファイルが宛先のStarRocksテーブルと同じ列名を持つことを確認してください。そのような場合、`SET` 句は不要ですし、それによってStarRocksはデータ型の変換を実行するためにキャスト関数を呼び出す必要もありません。その後、ブローカーロードジョブを正常に実行できます。

## 4. ブローカーロードジョブはエラーを報告しないのに、ロードされたデータをクエリできないのはなぜですか？

ブローカーロードは非同期のローディング方式です。ロードステートメントがエラーを返さなくても、ロードジョブが失敗することがあります。ブローカーロードジョブを実行した後、[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を使用してロードジョブの結果と `errmsg` を表示し、ジョブ構成を変更して再試行することができます。

## 5. "failed to send batch" や "TabletWriter add batch with unknown id" エラーが発生した場合はどうすればよいですか？

データの書き込みにかかる時間が上限を超えたため、タイムアウトエラーが発生しています。この問題を解決するには、[session variable](../../reference/System_variable.md) `query_timeout` および [BE configuration item](../../administration/Configuration.md#configure-be-static-parameters) `streaming_load_rpc_max_alive_time_sec` の設定を、ビジネス要件に基づいて変更してください。

## 6. "LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found" エラーが発生した場合はどうすればよいですか？

Parquet形式またはORC形式のデータをロードしている場合は、ソースデータファイルの最初の行にある列名が、宛先のStarRocksテーブルの列名と同じかどうかを確認してください。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上記の例では、ソースデータファイルの `tmp_c1` と `tmp_c2` の列をそれぞれ宛先のStarRocksテーブルの `name` と `id` の列にマッピングしています。`SET` 句を指定しない場合は、 `column_list` パラメータで指定された列名が列マッピングの宣言に使用されます。詳細は、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

> **注意**
>
> ソースデータファイルがApache Hive™によって生成されたORC形式のファイルで、ファイルの最初の行に `(_col0, _col1, _col2, ...)` が含まれている場合、「Invalid Column Name」 エラーが発生する可能性があります。このエラーが発生した場合は、列マッピングを指定するために `SET` 句を使用する必要があります。

## 7. ブローカーロードジョブが長時間実行される原因となるエラーの処理方法はどのようにすればよいですか？

FEログファイル **fe.log** を表示し、ジョブラベルに基づいてロードジョブのIDを検索します。その後、BEログファイル **be.INFO** を表示し、ジョブIDに基づいてロードジョブのログレコードを取得してエラーの原因を特定します。

## 8. HAモードで実行されるApache HDFSクラスターを構成する方法は？

HDFSクラスターが高可用性（HA）モードで実行される場合、次のように構成します：

- `dfs.nameservices`：HDFSクラスターの名前。例：`"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`：HDFSクラスターのNameNodeの名前。複数のNameNode名を指定する場合は、カンマ（`,`）で区切ります。`xxx` は `dfs.nameservices` で指定したHDFSクラスター名です。例：`"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：HDFSクラスターのNameNodeのRPCアドレス。 `nn` は `dfs.ha.namenodes.xxx` で指定したNameNode名です。例：`"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`：クライアントが接続するNameNodeのプロバイダ。デフォルト値：`org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。

例：

```SQL
(
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my-namenode1, my-namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

HAモードは、簡単な認証またはKerberos認証と組み合わせて使用できます。たとえば、HAモードで実行されるHDFSクラスターに簡単な認証を使用する場合、次の構成を指定する必要があります：

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

HDFSクラスターの構成情報を **hdfs-site.xml** ファイルに追加できます。これにより、ブローカーを使用してHDFSクラスターからデータをロードする際に、ファイルパスと認証情報のみを指定すれば済むようになります。

## 9. Hadoop ViewFS Federationをどのように構成すればよいですか？

ViewFs関連の構成ファイル `core-site.xml` および `hdfs-site.xml` を **broker/conf** ディレクトリにコピーします。

カスタムファイルシステムがある場合は、ファイルシステム関連の **.jar** ファイルも **broker/lib** ディレクトリにコピーする必要があります。

## 10. Kerberos認証が必要なHDFSクラスターにアクセスする際、"Can't get Kerberos realm" エラーが発生した場合はどうすればよいですか？

`/etc/krb5.conf` ファイルがブローカーが展開されているすべてのホストで構成されていることを確認してください。

エラーが解消されない場合は、ブローカーの起動スクリプトの `JAVA_OPTS` 変数の末尾に `-Djava.security.krb5.conf:/etc/krb5.conf` を追加してください。