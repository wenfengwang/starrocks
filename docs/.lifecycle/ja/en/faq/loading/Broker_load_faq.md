---
displayed_sidebar: English
---

# Broker Load

## 1. Broker Loadは、正常に実行されFINISHED状態にあるロードジョブを再実行することはできますか？

Broker Loadは、正常に実行されFINISHED状態にあるロードジョブの再実行をサポートしていません。また、データの損失や重複を防ぐため、Broker Loadでは成功したロードジョブのラベルを再利用することは許可されていません。[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してロードジョブの履歴を表示し、再実行したいロードジョブを探します。その後、そのロードジョブの情報をコピーし、ラベルを除くジョブ情報を使用して新しいロードジョブを作成できます。

## 2. Broker Loadを使用してHDFSからデータをロードする際、宛先のStarRocksテーブルにロードされた日付と時刻の値が、ソースデータファイルの日付と時刻の値より8時間遅れるのはなぜですか？

宛先のStarRocksテーブルとBroker Loadジョブは、作成時に中国標準時（CST）タイムゾーン（`timezone`パラメーターを使用して指定）を使用するようにコンパイルされます。しかし、サーバーは協定世界時（UTC）タイムゾーンに基づいて設定されています。その結果、データロード中にソースデータファイルの日付と時刻の値に8時間が追加されます。この問題を防ぐためには、宛先のStarRocksテーブルを作成する際に`timezone`パラメーターを指定しないでください。

## 3. Broker Loadを使用してORC形式のデータをロードする際、`ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>`というエラーが発生した場合、どうすればよいですか？

ソースデータファイルの列名が宛先のStarRocksテーブルの列名と異なります。この場合、ロードステートメント内で`SET`句を使用してファイルとテーブル間の列マッピングを指定する必要があります。`SET`句を実行する際、StarRocksは型推論を行う必要がありますが、ソースデータを宛先データ型に変換するための[cast](../../sql-reference/sql-functions/cast.md)関数の呼び出しに失敗します。この問題を解決するには、ソースデータファイルの列名が宛先のStarRocksテーブルと同じであることを確認してください。そうすれば、`SET`句は不要となり、StarRocksはデータ型変換を行うためにcast関数を呼び出す必要がなくなります。その後、Broker Loadジョブを正常に実行できます。

## 4. Broker Loadジョブはエラーを報告しませんが、ロードされたデータをクエリできないのはなぜですか？

Broker Loadは非同期ローディング方式です。ロードステートメントがエラーを返さなくても、ロードジョブが失敗する可能性があります。Broker Loadジョブを実行した後、[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してロードジョブの結果と`errmsg`を確認できます。その後、ジョブ設定を変更してリトライしてください。

## 5. 「failed to send batch」または「TabletWriter add batch with unknown id」エラーが発生した場合、どうすればよいですか？

データの書き込みにかかる時間が上限を超え、タイムアウトエラーが発生しています。この問題を解決するには、ビジネス要件に基づいてセッション変数`query_timeout`と[BE設定項目](../../administration/BE_configuration.md#configure-be-static-parameters)`streaming_load_rpc_max_alive_time_sec`の設定を変更してください。

## 6. 「LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found」というエラーが発生した場合、どうすればよいですか？

Parquet形式またはORC形式のデータをロードする場合、ソースデータファイルの最初の行に含まれる列名が宛先のStarRocksテーブルの列名と同じであるかを確認してください。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上記の例では、ソースデータファイルの`tmp_c1`と`tmp_c2`列をそれぞれ宛先のStarRocksテーブルの`name`と`id`列にマッピングしています。`SET`句を指定しない場合、`column_list`パラメーターで指定された列名が列マッピングの宣言に使用されます。詳細については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

> **注意**
>
> ソースデータファイルがApache Hive™によって生成されたORC形式のファイルであり、ファイルの最初の行が`(_col0, _col1, _col2, ...)`を保持している場合、「Invalid Column Name」エラーが発生する可能性があります。このエラーが発生した場合は、`SET`句を使用して列マッピングを指定する必要があります。

## 7. ブローカー・ロード・ジョブが過度に長い時間実行されるなどのエラーを処理するにはどうすればよいですか？

FEログファイル**fe.log**を確認し、ジョブラベルに基づいてロードジョブのIDを検索します。次に、BEログファイル**be.INFO**を確認し、ジョブIDに基づいてロードジョブのログレコードを取得し、エラーの根本原因を特定します。

## 8. HAモードで動作するApache HDFSクラスタを設定する方法は？

HDFSクラスタが高可用性（HA）モードで実行されている場合、以下のように設定します。

- `dfs.nameservices`: HDFSクラスタの名前です。例：`"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`: HDFSクラスタ内のNameNodeの名前です。複数のNameNode名を指定する場合は、コンマ（`,`）で区切ります。`xxx`は`dfs.nameservices`で指定したHDFSクラスタ名です。例：`"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`: HDFSクラスタ内のNameNodeのRPCアドレスです。`nn`は`dfs.ha.namenodes.xxx`で指定したNameNode名です。例：`"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`: クライアントが接続するNameNodeのプロバイダです。デフォルト値は`org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`です。

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

HAモードはシンプル認証またはKerberos認証で使用できます。例えば、HAモードで実行されるHDFSクラスタにシンプル認証でアクセスするには、以下の設定を指定する必要があります。

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

HDFSクラスタの設定を**hdfs-site.xml**ファイルに追加することができます。この方法では、ブローカーを使用してHDFSクラスタからデータをロードする際に、ファイルパスと認証情報のみを指定すれば済みます。

## 9. Hadoop ViewFSフェデレーションを設定する方法は？

ViewFS関連の設定ファイル`core-site.xml`と`hdfs-site.xml`を**broker/conf**ディレクトリにコピーします。

カスタムファイルシステムを使用している場合は、ファイルシステム関連の**.jar**ファイルも**broker/lib**ディレクトリにコピーする必要があります。

## 10. Kerberos認証が必要なHDFSクラスタにアクセスする際、「Kerberosレルムを取得できません」というエラーが発生した場合、どうすればよいですか？

ブローカーがデプロイされているすべてのホストで**/etc/krb5.conf**ファイルが設定されていることを確認してください。

エラーが続く場合は、ブローカー起動スクリプトの`JAVA_OPTS`変数の末尾に`-Djava.security.krb5.conf=/etc/krb5.conf`を追加してください。
