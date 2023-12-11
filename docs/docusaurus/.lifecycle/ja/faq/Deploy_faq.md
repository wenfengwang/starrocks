---
displayed_sidebar: "Japanese"
---

# デプロイ

このトピックでは、デプロイに関するよくある質問に対する回答を提供します。

## `fe.conf` ファイルの `priority_networks` パラメータで固定IPアドレスをバインドするにはどうすればよいですか？

### 問題の説明

たとえば、IPアドレスが 192.168.108.23 および 192.168.108.43 の2つある場合、次のようにIPアドレスを指定できます。

- アドレスを 192.168.108.23/24 と指定すると、StarRocks はそれを 192.168.108.43 と認識します。
- アドレスを 192.168.108.23/32 と指定すると、StarRocks はそれを 127.0.0.1 と認識します。

### 解決方法

この問題を解決する方法は次の2つあります。

- IPアドレスの末尾に "32" を追加しないか、"32" を "28" に変更します。
- StarRocks を 2.1 または以降にアップグレードすることもできます。

## インストール後にバックエンド（BE）を起動すると、「StarRocks BE http service did not start correctly, exiting」というエラーが発生するのはなぜですか？

BE をインストールする際に、システムが起動エラーを報告します。「StarRocks Be http service did not start correctly, exiting」というエラーが発生します。

このエラーは、BEのWebサービスポートが占有されているために発生します。`be.conf` ファイルのポートを変更し、BEを再起動してみてください。

## 「ERROR 1064（HY000）：Could not initialize class com.starrocks.rpc.BackendServiceProxy」エラーが発生した場合、どうすればよいですか？

このエラーは、Java Runtime Environment（JRE）でプログラムを実行したときに発生します。この問題を解決するには、JRE を Java Development Kit（JDK）に置き換えてください。Oracle の JDK 1.8 以降を使用することをお勧めします。

## Enterprise EditionのStarRocksを展開し、ノードを構成する際に「Failed to Distribute files to node」というエラーが発生するのはなぜですか？

このエラーは、複数のフロントエンド（FE）にインストールされた Setuptools のバージョンが一貫していないときに発生します。この問題を解決するには、rootユーザーとして次のコマンドを実行できます。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocksのFEおよびBEの構成を変更し、クラスターを再起動せずに有効にすることはできますか？

はい、FEおよびBEの変更を有効にするには、次の手順を実行します。

- FE：FEの変更を以下のいずれかの方法で完了できます:
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
curl --location-trusted -u ユーザー名:パスワード \
http://<ip>:<fe_http_port/api/_set_config?key=value>
```

例:

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE：BEの変更は以下の方法で完了できます：

```plaintext
curl -XPOST -u ユーザー名:パスワード \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：ユーザーがリモートでログインする権限があることを確認してください。権限がない場合は、次の方法でユーザーに権限を付与できます。

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## BEディスクスペースを拡張した後、「Failed to get scan range, no queryable replica found in tablet:xxxxx」というエラーが発生する場合、どうすればよいですか？

### 問題の説明

このエラーは、プライマリキーのテーブルにデータをロードする際に発生する場合があります。データをロードする際、宛先のBEにはロードされるデータに十分なディスク容量がなく、BEがクラッシュします。その後、新しいディスクが追加されてディスク容量が拡張されます。しかし、プライマリキーテーブルはディスク容量の再バランスをサポートせず、データを他のディスクにオフロードすることができません。

### 解決方法

このバグ（プライマリキーテーブルがBEディスク容量の再バランスをサポートしない）への修正術はまだ積極的に開発中です。現在は、次の2つの方法で修正できます。

- ディスク間でデータを手動で分散させる。たとえば、ディスク使用量が高いディレクトリを容量の大きいディスクにコピーします。
- これらのディスクのデータが重要でない場合、ディスクを削除し、ディスクパスを変更します。これらのエラーが解消しない場合は、[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md) を使用してテーブルのデータをクリアし、余分なスペースを解放してください。

## クラスター再起動中にFEを開始すると、「Fe type:unknown ,is ready :false.」というエラーがなぜ発生しますか？

リーダーFEが実行中かどうかを確認してください。実行されていない場合は、クラスターのFEノードを1つずつ再起動してください。

## クラスターを展開する際、「failed to get service info err.」というエラーがなぜ発生しますか？

OpenSSHデーモン（sshd）が有効になっているかどうかを確認してください。していない場合は、`/etc/init.d/sshd` status` コマンドを実行して有効にしてください。

## BEを開始する際、「Fail to get master client from `cache.`host= port=0 code=THRIFT_RPC_ERROR`」というエラーがなぜ発生しますか？

`netstat -anp |grep port` コマンドを実行して、`be.conf` ファイルのポートが占有されているかどうかを確認してください。占有されている場合は、占有されているポートを空きポートに置き換え、その後BEを再起動してください。

## Enterprse Editionのクラスターをアップグレードする際、「Failed to transport upgrade files to agent host. src:…」というエラーがなぜ発生しますか？

このエラーは、展開ディレクトリに指定されたディスク容量が不足している場合に発生します。クラスターのアップグレード中、StarRocks Manager は新しいバージョンのバイナリファイルを各ノードに配布します。展開ディレクトリに指定されたディスク容量が不足している場合、ファイルを各ノードに配布することができません。この問題を解決するには、データディスクを追加してください。

## 正常に実行される新しく展開されたFEノードのStarRocks Managerの診断ページで、「Search log failed.」が表示されるのはなぜですか？

デフォルトでは、StarRocks Manager は新しく展開されたFEのパス構成を30秒以内に取得します。このエラーは、FEが遅く起動するか、他の理由で30秒以内に応答しない場合に発生します。StarRocks Manager Webのログを次のパスで確認すると（パスをカスタマイズできます）:

`/starrocks-manager-xxx/center/log/webcenter/log/web/``drms.INFO`、ログに "Failed to update FE configurations" というメッセージが表示されているかを確認してください。もしそうであれば、対応するFEを再起動して、新しいパス構成を取得してください。

## FEを起動する際に「exceeds max permissable delta:5000ms.」というエラーが発生するのはなぜですか？

このエラーは、2つのマシン間の時間の差が5秒を超えた場合に発生します。この問題を解決するには、これら2つのマシンの時間を合わせてください。

## BEに複数のディスクがある場合、`storage_root_path`パラメータをどのように設定すればよいですか？

`be.conf` ファイルで `storage_root_path` パラメータを設定し、このパラメータの値を `;` で区切ってください。例: `storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## FEがクラスターに追加された後に「invalid cluster id: 209721925.」というエラーが発生するのはなぜですか？

最初のクラスターの起動時にこのFEに `--helper` オプションを追加しなかった場合、2つのマシンの間でメタデータが一貫しないために このエラーが発生します。この問題を解決するには、metaディレクトリの下のすべてのメタデータをクリアし、`--helper` オプションを使用して FE を追加してください。

## FEが実行されており、ログが「transfer: follower」と表示された場合、なぜ Alive が `false` ですか？

この問題は、Java Virtual Machine（JVM）のメモリの半分以上が使用され、チェックポイントがマークされていない場合に発生します。一般的には、システムが5万件のログを蓄積した後にチェックポイントがマークされます。そのため、これらのFEのJVMのパラメータを変更し、負荷がかかっていないときにこれらのFEを再起動することをお勧めします。