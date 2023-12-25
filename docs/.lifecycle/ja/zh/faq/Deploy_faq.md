---
displayed_sidebar: Chinese
---

# 部署常見問題

本ページでは、StarRocksをデプロイする際に発生する可能性のある一般的な問題とその潜在的な解決策を列挙しています。

## 設定ファイル **fe.conf** で `priority_networks` パラメータに固定IPを設定する方法は？

**問題の説明**

現在のノードには2つのIPアドレスがあるとします：`192.168.108.23` と `192.168.108.43`。

* `priority_networks` を `192.168.108.23/24` に設定すると、StarRocksはそのアドレスを `192.168.108.43` として認識します。
* `priority_networks` を `192.168.108.23/32` に設定すると、起動後にStarRocksがエラーを出し、そのアドレスを `127.0.0.1` として認識します。

**解決策**

上記の問題には以下の2つの解決策があります：

* CIDR接尾辞 `32` を削除するか、`28` に変更します。
* StarRocksを2.1またはそれ以上のバージョンにアップグレードします。

## BEノードのインストール後に起動に失敗し、エラー "StarRocks BE http service did not start correctly, exiting" が返されました。どうすればいいですか？

BEをインストールした後に `StarRocks Be http service did not start correctly,exiting` というエラーが発生した場合、これはBEノードの `be_http_port` ポートが使用中であるためです。BEの設定ファイル **be.conf** の `be_http_port` 設定項目を変更し、BEサービスを再起動して設定を有効にする必要があります。使用中でないポートに何度か変更してもシステムが同じエラーを繰り返す場合は、Yarnなどのプログラムがノードにインストールされているかどうかを確認し、リスニングポートの選択を変更するか、BEのポート選択範囲を回避する必要があります。

## 企業版StarRocksのデプロイ中にノードを設定する際に "Failed to Distribute files to node" というエラーが発生しました。どうすればいいですか？

上記のエラーは、FEノード間の setuptools バージョンが一致していないために発生します。以下のコマンドを実行して、クラスタ内のすべてのマシンで root 権限を使用して setuptools を削除する必要があります：

```shell
yum remove python-setuptools
rm /usr/lib/python2.7/site-packages/setuptool* -rf
wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocksはFE、BEの設定項目を動的に変更できますか？

FEノードとBEノードの一部の設定項目は動的に変更できます。具体的な操作は [設定パラメータ](../administration/FE_configuration.md) を参照してください。

* FEノードの設定項目を動的に変更するには：
  * SQLを使用して動的に変更する：

    ```sql
    ADMIN SET FRONTEND CONFIG ("key" = "value");
    ```

    例：

    ```sql
    --例：
    ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
    ```

  * コマンドラインを使用して動的に変更する：

    ```shell
    curl --location-trusted -u username:password \
    'http://<ip>:<fe_http_port>/api/_set_config?key=value'
    ```

    例：

    ```plain text
    curl --location-trusted -u <username>:<password> \
    'http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true'
    ```

* BEノードの設定項目を動的に変更するには：
  * コマンドラインを使用して動的に変更する：

    ```plain text
    curl -XPOST -u username:password \
    'http://<ip>:<be_http_port>/api/update_config?key=value'

> 注意
>
> 上記の方法でパラメータを変更する際は、現在のユーザーにリモートログイン権限があることを確認してください。

以下の例では、ユーザー test を作成し、適切な権限を付与しています。

```sql
CREATE USER 'test'@'%' IDENTIFIED BY '123456';
GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## BEノードにディスクスペースを追加した後、データストレージが負荷を均等に分散できず、"Failed to get scan range, no queryable replica found in tablet: xxxxx" というエラーが発生しました。どうすればいいですか？

**問題の説明**

このエラーは、主キーモデルテーブル（Primary Key）にデータをインポートする際にBEノードのディスクスペースが不足してBEがクラッシュすることが原因で発生する可能性があります。ディスクを拡張した後、PKテーブルは現在BE内のディスク間でのバランスをサポートしていないため、データストレージは負荷を均等に分散できません。

**解決策：**

この問題（PKテーブルはBE内のディスク間のバランスをサポートしていない）は現在修正中ですが、以下の2つの方法で解決できます：

* 手動でデータストレージの負荷を均等に分散する。たとえば、使用率が高いディスクのデータディレクトリをより大きなディスクスペースのディレクトリにコピーします。
* 現在のディスクに重要でないデータがある場合は、ディスクを直接削除し、対応するディスクパスを変更することをお勧めします。システムが引き続きエラーを報告する場合は、`TRUNCATE TABLE` コマンドを使用して現在のテーブルのデータをクリアする必要があります。

## クラスタを再起動する際にFEが起動に失敗し、"Fe type:unknown, is ready :false" というエラーが発生しました。どうすればいいですか？

Leader FEノードが起動しているかどうかを確認してください。起動していない場合は、クラスタ内のFEノードを順番に再起動してみてください。

## クラスタのインストール中に "failed to get service info err" というエラーが発生しました。どうすればいいですか？

現在のマシンでOpenSSH Daemon（SSHD）が起動しているかどうかを確認してください。

以下のコマンドでSSHDの状態を確認できます。

```shell
/etc/init.d/sshd status
```

## BEの起動に失敗し、"Fail to get master client from cache. host= port=0 code=THRIFT_RPC_ERROR" というエラーが発生しました。どうすればいいですか？

以下のコマンドで **be.conf** に指定されたポートが使用中かどうかを確認してください。

```shell
netstat  -anp  | grep  port
```

使用中の場合は、他の空いているポートに変更して再起動してください。

## StarRocks Managerを使用してクラスタをアップグレードする際に "Failed to transport upgrade files to agent host. src:..." というエラーが発生しました。どうすればいいですか？

上記のエラーは、デプロイパスのディスクスペースが不足しているために発生します。クラスタをアップグレードする際に、StarRocks Managerは新しいバージョンのバイナリファイルを各ノードに配布しますが、デプロイディレクトリのディスクスペースが不足している場合、ファイルは配布されず、上記のエラーが発生します。

対応するディスクのストレージスペースを確認してください。ストレージスペースが不足している場合は、対応するディスクスペースを拡張してください。

## 新しく拡張されたノードのFEの状態は正常ですが、StarRocks Managerの **診断** ページでそのFEノードのログに "Failed to search log" というエラーが表示されます。どうすればいいですか？

デフォルトの設定では、StarRocks Managerは30秒以内に新しくデプロイされたFEノードのパス設定を取得します。現在のFEノードが遅く起動するか、他の理由で30秒以内に応答しない場合、上記の問題が発生する可能性があります。**/starrocks-manager-xxx/center/log/webcenter/log/web/drms.INFO** パスでStarRocks Manager Webログを確認し、ログに "Failed to update fe configurations" というエラーが含まれているかどうかを検索してください。含まれている場合は、対応するFEサービスを再起動してください。StarRocks Managerはパス設定を再取得します。

## FEの起動に失敗し、"exceeds max permissable delta:5000ms" というエラーが発生しました。どうすればいいですか？

上記のエラーは、クラスタ内の異なるマシン間の時刻差が5秒を超えているために発生します。マシンの時刻を同期させることでこの問題を解決する必要があります。

## BEノードに複数のディスクがストレージとしてある場合、`storage_root_path` 設定項目をどのように設定しますか？

この設定項目はBEの設定ファイル **be.conf** にあります。`;` を使用して複数のディスクパスを区切ることができます。

## 新しいFEノードをクラスタに追加した後に "invalid cluster id: xxxxxxxx" というエラーが発生しました。どうすればいいですか？

上記のエラーは、クラスタを初めて起動する際に `--helper` オプションを使用せずにそのFEノードを追加したため、異なるノード間でメタデータが一致していないことが原因です。対応するディレクトリのすべてのメタデータをクリアし、`--helper` オプションを使用してそのFEノードをクラスタに再追加する必要があります。

## 現在のFEノードは起動しており、状態は `transfer：follower` ですが、`show frontends` コマンドを実行すると `isAlive` の状態が `false` と返されます。どうすればいいですか？

上記の問題は、Java Virtual Machine（JVM）のメモリの半分以上が使用されており、checkpointがマークされていないために発生します。通常、50,000件のログが蓄積されるごとにシステムはcheckpointをマークします。業務のオフピーク時に各FEノードのJVMパラメータを変更し、変更を有効にするためにFEを再起動することをお勧めします。

## クエリでエラー "could not initialize class com.starrocks.rpc.BackendServiceProxy" が発生しました。どうすればいいですか？

* 環境変数 `$JAVA_HOME` が正しいJDKのパスであることを確認してください。
* すべてのノードのJDKが同じバージョンであることを確認してください。すべてのノードは同じバージョンのJDKを使用する必要があります。
