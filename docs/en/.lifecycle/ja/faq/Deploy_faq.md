---
displayed_sidebar: "Japanese"
---

# デプロイメント

このトピックでは、デプロイメントに関するよくある質問に対する回答を提供します。

## `fe.conf` ファイルの `priority_networks` パラメータで固定 IP アドレスをバインドする方法は？

### 問題の説明

例えば、以下の2つのIPアドレスがある場合、IPアドレスは次のように指定することができます。

- アドレスを `192.168.108.23/24` と指定すると、StarRocks はそれを `192.168.108.43` と認識します。
- アドレスを `192.168.108.23/32` と指定すると、StarRocks はそれを `127.0.0.1` と認識します。

### 解決策

この問題を解決するためには、次の2つの方法があります。

- IPアドレスの末尾に "32" を追加しないか、"32" を "28" に変更します。
- StarRocks 2.1 以降にアップグレードすることもできます。

## インストール後にバックエンド（BE）を起動するとエラー "StarRocks BE http service did not start correctly, exiting" が発生する理由は？

BE をインストールする際に、システムは起動エラー "StarRocks Be http service did not start correctly, exiting" を報告します。

このエラーは、BE のウェブサービスポートが占有されているため発生します。`be.conf` ファイルのポートを変更し、BE を再起動してみてください。

## "ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy" エラーが発生した場合はどうすればよいですか？

このエラーは、Java Runtime Environment (JRE) でプログラムを実行した場合に発生します。この問題を解決するには、JRE を Java Development Kit (JDK) に置き換えてください。Oracle の JDK 1.8 以降を使用することをおすすめします。

## Enterprise Edition の StarRocks をデプロイし、ノードを構成する際に "Failed to Distribute files to node" エラーが発生する理由は？

このエラーは、複数のフロントエンド (FE) にインストールされている Setuptools のバージョンが一致していない場合に発生します。この問題を解決するには、次のコマンドを root ユーザーとして実行します。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks の FE と BE の設定を変更しても、クラスタを再起動せずに変更を反映するにはどうすればよいですか？

はい。FE と BE の変更を反映するには、次の手順を実行します。

- FE: FE の変更を反映するには、次のいずれかの方法で FE の変更を完了します。
  - SQL

```plaintext
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

例:

```plaintext
ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
```

- シェル

```plaintext
curl --location-trusted -u username:password \
http://<ip>:<fe_http_port/api/_set_config?key=value>
```

例:

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE: BE の変更を反映するには、次の方法を使用します。

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意: ユーザーがリモートログインする権限を持っていることを確認してください。持っていない場合は、次の方法でユーザーに権限を付与できます。

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## BE ディスクスペースを拡張した後に "Failed to get scan range, no queryable replica found in tablet:xxxxx" エラーが発生した場合はどうすればよいですか？

### 問題の説明

このエラーは、プライマリキーのテーブルにデータをロードする際に発生する場合があります。データのロード中に、宛先の BE に十分なディスクスペースがなく、BE がクラッシュします。その後、ディスクを追加してディスクスペースを拡張します。しかし、プライマリキーのテーブルはディスクスペースの再バランスをサポートしておらず、データを他のディスクにオフロードすることはできません。

### 解決策

このバグ（プライマリキーのテーブルは BE ディスクスペースの再バランスをサポートしていない）へのパッチは現在も開発中です。現在、次の2つの方法で修正することができます。

- ディスク間でデータを手動で分散します。例えば、ディスク使用量が高いディスクからディスク使用量が大きいディスクにディレクトリをコピーします。
- これらのディスクのデータが重要でない場合は、ディスクを削除し、ディスクパスを変更します。このエラーが解消しない場合は、[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md) を使用してテーブルのデータをクリアし、一部のスペースを解放します。

## クラスタの再起動中に FE を起動するとエラー "Fe type:unknown ,is ready :false." が発生する理由は？

リーダー FE が実行されているかどうかを確認してください。実行されていない場合は、クラスタ内の FE ノードを1つずつ再起動してください。

## クラスタをデプロイする際にエラー "failed to get service info err." が発生する理由は？

OpenSSH デーモン (sshd) が有効になっているかどうかを確認してください。有効になっていない場合は、`/etc/init.d/sshd`` status` コマンドを実行して有効にしてください。

## BE を起動する際にエラー "Fail to get master client from `cache. ``host= port=0 code=THRIFT_RPC_ERROR`" が発生する理由は？

`netstat -anp |grep port` コマンドを実行して、`be.conf` ファイルのポートが占有されているかどうかを確認してください。占有されている場合は、占有されているポートを空きポートに置き換えてから BE を再起動してください。

## Enterprise Edition のクラスタをアップグレードする際にエラー "Failed to transport upgrade files to agent host. src:…" が発生する理由は？

このエラーは、デプロイディレクトリで指定されたディスクスペースが不足している場合に発生します。クラスタのアップグレード中、StarRocks Manager は新しいバージョンのバイナリファイルを各ノードに配布します。デプロイディレクトリで指定されたディスクスペースが不足している場合、ファイルは各ノードに配布することができません。この問題を解決するには、データディスクを追加してください。

## 正常に実行されている新しくデプロイされた FE ノードの StarRocks Manager の診断ページで "Search log failed." と表示される理由は？

デフォルトでは、StarRocks Manager は新しくデプロイされた FE のパス設定を30秒以内に取得します。このエラーは、FE の起動が遅いか、他の理由で30秒以内に応答しない場合に発生します。Manager Web のログを次のパスから確認してください。

`/starrocks-manager-xxx/center/log/webcenter/log/web/``drms.INFO`(パスはカスタマイズ可能です)。その後、ログ内に "Failed to update FE configurations" というメッセージが表示されているかどうかを確認してください。表示されている場合は、対応する FE を再起動して新しいパス設定を取得してください。

## FE を起動する際にエラー "exceeds max permissable delta:5000ms." が発生する理由は？

このエラーは、2つのマシン間の時差が5秒以上ある場合に発生します。この問題を解決するには、これら2つのマシンの時刻を合わせてください。

## BE のデータストレージに複数のディスクがある場合、`storage_root_path` パラメータをどのように設定すればよいですか？

`be.conf` ファイルで `storage_root_path` パラメータを設定し、このパラメータの値を `;` で区切ってください。例: `storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## FE をクラスタに追加した後にエラー "invalid cluster id: 209721925." が発生する理由は？

クラスタを初めて起動する際にこの FE に `--helper` オプションを追加しなかった場合、2つのマシン間のメタデータが一貫していないため、このエラーが発生します。この問題を解決するには、メタディレクトリの下のすべてのメタデータをクリアし、`--helper` オプションを使用して FE を追加する必要があります。

## FE が実行中であり、ログに "transfer: follower" と表示される場合、Alive が `false` になる理由は？

この問題は、Java Virtual Machine (JVM) のメモリの半分以上が使用され、チェックポイントがマークされていない場合に発生します。一般的に、システムが5万件のログを蓄積した後にチェックポイントがマークされます。各 FE の JVM のパラメータを変更し、負荷がかかっていないときにこれらの FE を再起動することをおすすめします。
