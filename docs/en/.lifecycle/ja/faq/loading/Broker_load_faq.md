---
displayed_sidebar: "Japanese"
---

# Broker Load（ブローカーロード）

## 1. FINISHED状態で正常に実行されたロードジョブを再実行することはできますか？

Broker Loadでは、FINISHED状態で正常に実行されたロードジョブを再実行することはサポートされていません。また、データの損失や重複を防ぐために、正常に実行されたロードジョブのラベルを再利用することもできません。[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してロードジョブの履歴を表示し、再実行したいロードジョブを見つけることができます。その後、そのロードジョブの情報をコピーし、ラベル以外のジョブ情報を使用して別のロードジョブを作成することができます。

## 2. Broker Loadを使用してHDFSからデータをロードする際、宛先のStarRocksテーブルにロードされる日時の値がソースデータファイルの日時の値よりも8時間遅れている場合、どうすればよいですか？

宛先のStarRocksテーブルとBroker Loadジョブは、作成時に中国標準時(CST)のタイムゾーン（`timezone`パラメータを使用して指定）を使用してコンパイルされます。しかし、サーバーは協定世界時(UTC)のタイムゾーンで実行されるように設定されています。その結果、データのロード中にソースデータファイルの日時の値に8時間が追加されます。この問題を解決するには、宛先のStarRocksテーブルを作成する際に`timezone`パラメータを指定しないようにしてください。

## 3. Broker Loadを使用してORC形式のデータをロードする際、`ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>`エラーが発生した場合、どうすればよいですか？

ソースデータファイルの列名が宛先のStarRocksテーブルと異なる場合、ロードステートメントで`SET`句を使用してファイルとテーブルの列のマッピングを指定する必要があります。`SET`句を実行する際、StarRocksは型推論を行う必要がありますが、ソースデータを宛先のデータ型に変換するために[cast](../../sql-reference/sql-functions/cast.md)関数を呼び出す際に失敗します。この問題を解決するには、ソースデータファイルが宛先のStarRocksテーブルと同じ列名を持つことを確認してください。そのような場合、`SET`句は必要なくなり、StarRocksはデータ型変換のためにキャスト関数を呼び出す必要もありません。その後、Broker Loadジョブを正常に実行することができます。

## 4. Broker Loadジョブはエラーを報告していませんが、ロードされたデータをクエリできません。どうすればよいですか？

Broker Loadは非同期のロード方法です。ロードステートメントがエラーを返さなくても、ロードジョブが失敗することがあります。Broker Loadジョブを実行した後、[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してロードジョブの結果と`errmsg`を表示することができます。その後、ジョブの設定を変更して再試行することができます。

## 5. "failed to send batch"または"TabletWriter add batch with unknown id"エラーが発生した場合、どうすればよいですか？

データの書き込みにかかる時間が上限を超えるため、タイムアウトエラーが発生しています。この問題を解決するには、ビジネス要件に基づいて[session variable](../../reference/System_variable.md)`query_timeout`と[BE configuration item](../../administration/Configuration.md#configure-be-static-parameters)`streaming_load_rpc_max_alive_time_sec`の設定を変更してください。

## 6. "LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found"エラーが発生した場合、どうすればよいですか？

Parquet形式またはORC形式のデータをロードしている場合、ソースデータファイルの最初の行にある列名が宛先のStarRocksテーブルの列名と同じかどうかを確認してください。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上記の例では、ソースデータファイルの`tmp_c1`と`tmp_c2`の列をそれぞれ宛先のStarRocksテーブルの`name`と`id`の列にマッピングしています。`SET`句を指定しない場合、`column_list`パラメータで指定された列名が列マッピングの宣言に使用されます。詳細については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

> **注意**
>
> ソースデータファイルがApache Hive™によって生成されたORC形式のファイルであり、ファイルの最初の行に`(_col0, _col1, _col2, ...)`が含まれている場合、"Invalid Column Name"エラーが発生する可能性があります。このエラーが発生した場合、`SET`句を使用して列のマッピングを指定する必要があります。

## 7. ブローカーロードジョブが長時間実行されるエラーなどのエラーを処理するにはどうすればよいですか？

FEログファイル**fe.log**を表示し、ジョブラベルに基づいてロードジョブのIDを検索します。次に、BEログファイル**be.INFO**を表示し、ジョブIDに基づいてロードジョブのログレコードを取得してエラーの原因を特定します。

## 8. Apache HDFSクラスタをHAモードで実行するにはどうすればよいですか？

HDFSクラスタが高可用性（HA）モードで実行される場合、次のように設定します。

- `dfs.nameservices`：HDFSクラスタの名前。例：`"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`：HDFSクラスタのNameNodeの名前。複数のNameNode名を指定する場合は、カンマ（`,`）で区切ります。`xxx`は、`dfs.nameservices`で指定したHDFSクラスタ名です。例：`"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：HDFSクラスタのNameNodeのRPCアドレス。`nn`は、`dfs.ha.namenodes.xxx`で指定したNameNode名です。例：`"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

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

HAモードは、単純な認証またはKerberos認証と共に使用することができます。たとえば、HAモードで実行されるHDFSクラスタにアクセスするために単純な認証を使用する場合、次の設定を指定する必要があります。

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

HDFSクラスタの設定を**hdfs-site.xml**ファイルに追加することもできます。これにより、ブローカーを使用してHDFSクラスタからデータをロードする際に、ファイルパスと認証情報の指定のみが必要になります。

## 9. Hadoop ViewFS Federationをどのように設定しますか？

ViewFs関連の設定ファイル`core-site.xml`と`hdfs-site.xml`を**broker/conf**ディレクトリにコピーします。

カスタムファイルシステムを使用している場合は、ファイルシステム関連の**.jar**ファイルも**broker/lib**ディレクトリにコピーする必要があります。

## 10. Kerberos認証が必要なHDFSクラスタにアクセスする際に、"Can't get Kerberos realm"エラーが発生した場合、どうすればよいですか？

すべてのブローカーが展開されているホストで**/etc/krb5.conf**ファイルが設定されていることを確認してください。

エラーが解消されない場合は、ブローカーの起動スクリプトの`JAVA_OPTS`変数の末尾に`-Djava.security.krb5.conf:/etc/krb5.conf`を追加してください。
