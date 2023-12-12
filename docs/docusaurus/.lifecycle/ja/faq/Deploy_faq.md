---
displayed_sidebar: "Japanese"
---

# デプロイ

このトピックでは、デプロイに関するよくある質問に対する回答を提供します。

## `fe.conf` ファイル内の `priority_networks` パラメーターで固定IPアドレスをバインドする方法は？

### 問題の説明

たとえば、192.168.108.23 と 192.168.108.43 の2つのIPアドレスがある場合、以下のようにIPアドレスを指定できます。

- アドレスを 192.168.108.23/24 と指定した場合、StarRocks はそれを 192.168.108.43 と認識します。
- アドレスを 192.168.108.23/32 と指定した場合、StarRocks はそれを 127.0.0.1 と認識します。

### 解決策

この問題を解決するためには、次の2つの方法があります：

- IPアドレスの末尾に「32」を追加せず、「32」を「28」に変更する。
- StarRocks を 2.1 またはそれ以降にアップグレードする。

## インストール後にバックエンド（BE）を起動すると、エラー「StarRocks BE http service did not start correctly, exiting」が発生する理由は？

BE をインストールすると、システムが起動エラーを報告します：StarRocks Be http service did not start correctly, exiting.

このエラーは、BE のウェブサービス ポートが使用されているために発生します。`be.conf` ファイル内のポートを変更して BE を再起動してみてください。

## プログラムを実行するとエラー「ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy」が発生する場合の対処方法は？

このエラーは、Java Runtime Environment (JRE) でプログラムを実行すると発生します。この問題を解決するためには、JRE を Java Development Kit (JDK) と置き換えます。Oracle の JDK 1.8 以降を使用することをお勧めします。

## エンタープライズエディションの StarRocks をデプロイし、ノードを設定する際にエラー「Failed to Distribute files to node」が発生する理由は？

このエラーは、複数のフロントエンド (FE) にインストールされている Setuptools のバージョンが一貫していない場合に発生します。この問題を解決するためには、rootユーザーとして次のコマンドを実行します。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks の FE と BE の構成を変更し、クラスタを再起動せずに適用する方法は？

はい。次の手順を実行して、FE と BE の変更を適用します：

- FE: FE の変更は次のいずれかの方法で行えます：
  - SQL

```plaintext
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

例：

```plaintext
ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
```

- シェル

```plaintext
curl --location-trusted -u username:password \
http://<ip>:<fe_http_port/api/_set_config?key=value>
```

例：

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE: 次の方法で BE の変更を完了します：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注：ユーザーがリモートログインする権限を持っていることを確認してください。権限がない場合は、次の方法でユーザーに権限を付与できます：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## BE のディスク容量を拡張した後に「Failed to get scan range, no queryable replica found in tablet:xxxxx」というエラーが発生した場合の対処方法は？

### 問題の説明

このエラーは、プライマリキー テーブルにデータをロードする際に発生する場合があります。データをロードする際、宛先 BE にはロードされたデータのための十分なディスク容量がありません。そのため BE がクラッシュします。次に新しいディスクが追加されてディスク容量が拡張されます。しかし、プライマリキー テーブルではディスク容量の再バランスがサポートされておらず、データを他のディスクにオフロードすることはできません。

### 解決策

このバグ（プライマリキー テーブルは BE ディスク容量の再バランスをサポートしていません）のパッチは現在積極的に開発中です。現時点では、次のいずれかの方法で修正できます：

- ディスク間でデータを手動で分散する。たとえば、ディスクの使用量が高いディレクトリから使用量の大きいディスクにディレクトリをコピーします。
- これらのディスク上のデータが重要でない場合は、ディスクを削除してディスクのパスを変更することをお勧めします。このエラーが続く場合は、[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md) を使用して表内のデータを消去して容量を解放します。

## クラスタの再起動時に FE が起動する際にエラー「Fe type:unknown ,is ready :false.」が発生する理由は？

リーダー FE が実行されているかどうかを確認してください。実行されていない場合は、クラスタ内の FE ノードを1つずつ再起動してください。

## クラスタをデプロイする際にエラー「failed to get service info err.」が発生する理由は？

OpenSSHデーモン（sshd）が有効になっているかどうかを確認してください。有効でない場合は、`/etc/init.d/sshd status` コマンドを実行して有効にしてください。

## BE を起動する際にエラー「Fail to get master client from `cache. ``host= port=0 code=THRIFT_RPC_ERROR`」が発生する理由は？

`netstat -anp |grep port` コマンドを実行して、`be.conf` ファイル内のポートが使用されているかどうかを確認してください。使用されている場合は、使用されているポートを空きポートに置き換えてから BE を再起動してください。

## エンタープライズエディションのクラスタをアップグレードする際にエラー「Failed to transport upgrade files to agent host. src:…」が発生する理由は？

このエラーは、デプロイディレクトリに指定されたディスク容量が不足している場合に発生します。クラスタのアップグレード中、StarRocks Manager は新しいバージョンのバイナリファイルを各ノードに配布します。デプロイディレクトリに指定されたディスク容量が不足していると、ファイルを各ノードに配布できません。この問題を解決するには、データディスクを追加してください。

## 正しく実行されている新しくデプロイされた FE ノードの診断ページ上の StarRocks Manager ログに「Search log failed.」と表示される理由は？

デフォルトでは、StarRocks Manager は新しくデプロイされた FE のパス構成を 30 秒以内に取得します。このエラーは、FE が遅い速度で開始されるか、または他の理由で 30 秒以内に応答しない場合に発生します。Manager Web のログを次のパスから確認できます：

`/starrocks-manager-xxx/center/log/webcenter/log/web/``drms.INFO`（パスをカスタマイズできます）。次にログ内でメッセージ「Failed to update FE configurations」が表示されるかどうかを確認します。表示される場合は、対応する FE を再起動して新しいパス構成を取得してください。

## FE を起動する際にエラー「exceeds max permissable delta:5000ms.」が発生する理由は？

このエラーは、2つのマシン間の時間差が5秒を超える場合に発生します。この問題を解決するには、これら2つのマシンの時間を整えてください。

## BE のデータを複数のディスクに格納する場合の `storage_root_path` パラメーターの設定方法は？

`be.conf` ファイルで `storage_root_path` パラメーターを構成し、このパラメーターの値を `;` で区切って設定します。たとえば：`storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## FE がクラスタに追加された後にエラー「invalid cluster id: 209721925.」が発生する理由は？

最初にクラスタを開始する際にこの FE に `--helper` オプションを追加しなかった場合、2つのマシン間のメタデータが不整合になり、このエラーが発生します。この問題を解決するには、メタデータディレクトリ内のすべてのメタデータをクリアしてから `--helper` オプションを使用して FE を追加してください。

## FE が実行されておりログに `transfer: follower` を出力した際に Alive が `false` になる原因は？

この問題は、Java Virtual Machine (JVM) のメモリの半分以上が使用され、チェックポイントがマークされていない場合に発生します。一般的には、システムが 50,000個のログを蓄積した後にチェックポイントがマークされます。これらの FE の JVM パラメータを変更し、それらの FE を負荷がかからない状態で再起動することをお勧めします。