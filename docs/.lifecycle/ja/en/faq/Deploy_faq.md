---
displayed_sidebar: English
---

# デプロイメント

このトピックでは、デプロイメントに関するよくある質問への回答を提供します。

## `fe.conf` ファイルの `priority_networks` パラメータを使用して固定IPアドレスをバインドするにはどうすればよいですか？

### 問題の説明

例えば、2つのIPアドレスがあるとします：192.168.108.23 と 192.168.108.43。IPアドレスを以下のように指定するかもしれません：

- アドレスを192.168.108.23/24として指定した場合、StarRocksはそれを192.168.108.43と認識します。
- アドレスを192.168.108.23/32として指定した場合、StarRocksはそれを127.0.0.1と認識します。

### 解決策

この問題を解決するには以下の2つの方法があります：

- IPアドレスの末尾に「32」を追加しないか、「32」を「28」に変更してください。
- StarRocks 2.1以降にアップグレードすることもできます。

## インストール後にバックエンド（BE）を起動すると、「StarRocks BE http service did not start correctly, exiting」というエラーが発生するのはなぜですか？

BEをインストールする際、システムは起動エラーを報告します：StarRocks BE httpサービスが正しく起動しませんでした、終了します。

このエラーは、BEのWebサービスポートが使用中であるために発生します。`be.conf` ファイル内のポートを変更して、BEを再起動してみてください。

## エラーが発生した場合はどうすればよいですか：ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy？

このエラーは、Java Runtime Environment（JRE）でプログラムを実行する際に発生します。この問題を解決するには、JREをJava Development Kit（JDK）に置き換えてください。OracleのJDK 1.8以降の使用を推奨します。

## Enterprise EditionのStarRocksをデプロイしてノードを設定すると、「Failed to Distribute files to node」というエラーが発生するのはなぜですか？

このエラーは、複数のフロントエンド（FE）にインストールされているSetuptoolsのバージョンが一致していない場合に発生します。この問題を解決するには、rootユーザーとして以下のコマンドを実行できます。

```plaintext
yum remove python-setuptools

rm -rf /usr/lib/python2.7/site-packages/setuptool*

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocksのFEおよびBEの設定を変更しても、クラスターを再起動せずに有効にすることはできますか？

はい。FEおよびBEの変更を完了するには、以下の手順を実行します：

- FE：FEの変更は、以下のいずれかの方法で完了できます：
  - SQL

```plaintext
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

例：

```plaintext
ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
```

- Shell

```plaintext
curl --location-trusted -u username:password \
http://<ip>:<fe_http_port>/api/_set_config?key=value
```

例：

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE：BEの変更は、以下の方法で完了できます：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：ユーザーがリモートログインの権限を持っていることを確認してください。そうでない場合は、以下の方法でユーザーに権限を付与できます：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON *.* TO 'test'@'%';
```

## BEのディスク容量を拡張した後、「Failed to get scan range, no queryable replica found in tablet:xxxxx」というエラーが発生した場合はどうすればよいですか？

### 問題の説明

このエラーは、プライマリキーテーブルへのデータロード中に発生することがあります。データロード中、宛先のBEにはロードされたデータ用の十分なディスクスペースがなく、BEがクラッシュします。その後、新しいディスクが追加されてディスクスペースが拡張されますが、プライマリキーテーブルはディスクスペースの再バランスをサポートしておらず、データを他のディスクにオフロードすることはできません。

### 解決策

このバグ（プライマリキーテーブルはBEディスクスペースの再バランスをサポートしていません）に対するパッチは現在も積極的に開発中です。現在、以下の2つの方法で修正できます：

- ディスク間でデータを手動で分散させます。例えば、使用率の高いディスクからより大きなスペースのディスクにディレクトリをコピーします。
- これらのディスク上のデータが重要でない場合、ディスクを削除しディスクパスを変更することをお勧めします。このエラーが続く場合は、[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md)を使用してテーブル内のデータをクリアし、スペースを解放してください。

## クラスタの再起動中にFEを起動すると、「Fe type:unknown, is ready:false.」というエラーが発生するのはなぜですか？

リーダーFEが稼働しているかどうかを確認してください。そうでない場合は、クラスタ内のFEノードを順番に再起動してください。

## クラスターをデプロイする際に「failed to get service info err.」というエラーが発生するのはなぜですか？

OpenSSHデーモン（sshd）が有効になっているかどうかを確認してください。そうでない場合は、`/etc/init.d/sshd status` コマンドを実行して有効にしてください。

## BEを起動すると「Fail to get master client from cache. host= port=0 code=THRIFT_RPC_ERROR」というエラーが表示されるのはなぜですか？

`netstat -anp | grep port` コマンドを実行して、`be.conf` ファイル内のポートが使用中かどうかを確認してください。使用中の場合は、そのポートを空いているポートに変更してから、BEを再起動してください。

## Enterprise Editionのクラスタをアップグレードする際に「Failed to transport upgrade files to agent host. src:…」というエラーが発生するのはなぜですか？

このエラーは、デプロイメントディレクトリに指定されたディスクスペースが不足している場合に発生します。クラスタのアップグレード中に、StarRocks Managerは新バージョンのバイナリファイルを各ノードに配布します。デプロイメントディレクトリに指定されたディスクスペースが不足している場合、ファイルを各ノードに配布できません。この問題を解決するには、データディスクを追加してください。

## StarRocks Managerの診断ページのFEノードログに、「Search log failed.」と表示されるのは、正常に動作している新しくデプロイされたFEノードに対してなぜですか？

デフォルトでは、StarRocks Managerは新しくデプロイされたFEのパス設定を30秒以内に取得します。このエラーは、FEの起動が遅いか、他の理由で30秒以内に応答しない場合に発生します。Manager Webのログを以下のパスで確認してください：

`/starrocks-manager-xxx/center/log/webcenter/log/web/drms.INFO`（パスはカスタマイズ可能です）。その後、ログに「Failed to update FE configurations」というメッセージが表示されているかどうかを確認してください。表示されている場合は、対応するFEを再起動して新しいパス設定を取得してください。

## FEを起動すると「exceeds max permissible delta:5000ms.」というエラーが発生するのはなぜですか？

このエラーは、2台のマシン間の時刻差が5秒以上ある場合に発生します。この問題を解決するには、これら2台のマシンの時刻を同期させてください。

## データストレージ用のBEに複数のディスクがある場合、`storage_root_path`パラメータをどのように設定すればよいですか？

`be.conf` ファイルの `storage_root_path` パラメータを設定し、このパラメータの値を `;` で区切ってください。例えば：`storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## FEがクラスタに追加された後に「invalid cluster id: 209721925.」というエラーが発生するのはなぜですか？

クラスタを初めて起動する際にこのFEに `--helper` オプションを追加しなかった場合、2台のマシン間でメタデータが一致しないため、このエラーが発生します。この問題を解決するには、メタディレクトリ下のすべてのメタデータをクリアし、`--helper` オプションを使用してFEを追加する必要があります。

## FEが動作しているときにAliveが`false`であり、ログに`transfer: follower`と出力されるのはなぜですか？

この問題は、Java Virtual Machine（JVM）のメモリの半分以上が使用され、チェックポイントがマークされていない場合に発生します。通常、システムが50,000件のログを蓄積した後にチェックポイントがマークされます。各FEのJVMパラメータを変更し、負荷が低い時にこれらのFEを再起動することをお勧めします。
