---
displayed_sidebar: Chinese
---

# StarRocks Manager を使用して StarRocks クラスターを自動デプロイする

この文書では、StarRocks Manager を使用して StarRocks クラスターを自動デプロイする方法について説明します。

> **説明**
>
> StarRocks Manager はエンタープライズ版の機能です。試用を希望される場合は、[ダウンロードページ](https://www.mirrorship.cn/zh-CN/download/community)の下部にある「今すぐ相談」をクリックして入手してください。

## 前提条件

StarRocks をデプロイする予定のすべてのノードに、以下の依存関係をインストールする必要があります：

|依存関係|説明|
|----|----|
|Python（2.7 以上）|Linux オペレーティングシステムに付属の Python 2 は、Manager のデプロイ要件を満たしています。|
|python-setuptools|`easy_install --version` コマンドを実行してチェックできます。戻り値があれば問題ありません。バージョンに特別な要件はありません。戻り値がない場合は、`yum install setuptools` または `apt-get install setuptools` を使用してインストールできます。|
|MySQL（5.5 以上）|StarRocks Manager プラットフォームのデータを保存するために MySQL を使用する必要があります。|

## StarRocks Manager のインストール

StarRocks Manager インストールパッケージをダウンロードして解凍します。

解凍が完了したら、StarRocks Manager をインストールします。

```shell
bin/install.sh -h \
-d /home/disk1/starrocks/starrocks-manager-20200101 \
-y /usr/bin/python -p 19321 -s 19320
```

* `-d`：StarRocks Manager のインストールパス。
* `-y`：Python のパス。
* `-p`：`admin_console_port`、デフォルトは `19321`。
* `-s`：`supervisor_http_port`、デフォルトは `19320`。

## StarRocks のインストールとデプロイ

StarRocks Manager のインストールが完了したら、Web ページで StarRocks クラスターをインストールしてデプロイできます。

### MySQL データベースの設定

まず、管理、クエリ、アラート情報などを保存するために、インストール済みの MySQL データベースを設定する必要があります。

![MySQL の設定](../assets/8.1.1.3-1.png)

### ノード情報の設定

デプロイするノードを追加し、Agent と Supervisor のインストールディレクトリとポート、Python のパス、その他の情報を設定します。

> 説明
> Agent はマシンの統計情報を収集する責任があり、Supervisor はプロセスの起動と停止を管理します。両方ともユーザー環境にインストールされ、システム環境には影響しません。

![ノードの設定](../assets/8.1.1.3-2.png)

### FE ノードのインストール

FE ノードに関する情報を設定します。ポートの意味については、以下の[ポートリスト](#ポートリスト)を参照してください。

3 つの Follower FE を設定することをお勧めします。リクエストの負荷が高い場合は、Observer FE の数を適宜増やすことを検討してください。

![FE インスタンスの設定](../assets/8.1.1.3-3.png)

`Meta Dir`：StarRocks のメタデータディレクトリ。**meta** と FE ノードのログディレクトリを別々に設定することをお勧めします。

### BE ノードのインストール

BE ノードに関する情報を設定します。ポートの意味については、以下の[ポートリスト](#ポートリスト)を参照してください。

![BE インスタンスの設定](../assets/8.1.1.3-4.png)

### Broker のインストール

すべての BE ノードに Broker をインストールすることをお勧めします。ポートの意味については、以下の[ポートリスト](#ポートリスト)を参照してください。

![HDFS Broker](../assets/8.1.1.3-5.png)

### 中央サービスのインストール

中央サービスは、Agent から情報を収集して MySQL に保存し、監視とアラートサービスを提供します。ここでのメールサービスは、アラート通知をメールで受け取ることを指します。メールサービスは後で設定できます。

中央サービスとメールサービスの関連情報を設定します。

![中央サービスの設定](../assets/8.1.1.3-6.png)

## ポートリスト

|インスタンス名|ポート名|デフォルトポート|通信方向|説明|
|---|---|---|---|---|
|BE|be_port|9060|FE --> BE|BE 上の thrift server のポート。<br/>FE からのリクエストを受け取ります。|
|BE|be_http_port|8040|BE <--> BE|BE 上の http server のポート。|
|BE|heartbeat_service_port|9050|FE --> BE|BE 上のハートビートサービスポート（thrift）。<br/>FE からのハートビートを受け取ります。|
|BE|brpc_port|8060|BE <--> BE|BE 間の通信に使用される bRPC ポート。|
|FE|**http_port**|**8030**|FE <--> ユーザー|FE 上の http server のポート。|
|FE|rpc_port|9020|BE --> FE<br/> FE <--> FE|FE 上の thrift server のポート。|
|FE|**query_port**|**9030**| FE <--> ユーザー|FE 上の mysql server のポート。|
|FE|edit_log_port|9010|FE <--> FE|FE 間の BDBJE 通信ポート。|
|Broker|broker_ipc_port|8000|FE --> Broker <br/>BE --> Broker|Broker 上の thrift server。<br/>リクエストを受け取ります。|
|Drms|admin_console_port|19321|Drms 対外|外部 Web ポート、nginx によるポート転送が行われています。|
|Supervisor|supervisor_http_port|19320/19319|supervisor 内部|supervisor 管理プロセス。|
|Agent|agent_port|19323|Agent --> Center|Agent と center service の通信、モニタリング情報の報告。|

`http_port` と `query_port` はよく使われるポートで、前者は FE への Web アクセスに、後者は MySQL クライアントによる FE アクセスに使用されます。

## FAQ

**Q**：`ulimit` をどのように設定しますか？

**A**：**すべてのマシン**で `ulimit -n 65536` コマンドを実行することにより設定できます。システムが「権限がない」と表示された場合は、以下の手順を試してください：

まず、**/etc/security/limits.conf** に以下の設定を追加してください：

```Plain Text
# 4つの要素があります。詳細は limits.conf の説明を参照してください。* はすべてのユーザーを意味します。
* soft nofile 65535
* hard nofile 65535
```

次に、**/etc/pam.d/login** と **/etc/pam.d/sshd** に以下の設定を追加してください：

```Plain Text
session required pam_limits.so
```

最後に、**/etc/ssh/sshd_config** に **UsePAM yes** が存在することを確認してください。存在しない場合は、このパラメータを追加し、`restart sshd` を実行してください。

**Q**：Python のインストール時に `__init__() takes 2 arguments (4 given)` の問題が発生した場合、どのように対処しますか？

**A**：Python のインストール時に `__init__() takes 2 arguments (4 given)` の問題が発生した場合は、以下の手順を実行してください：

まず、`which python` コマンドを実行して、Python のインストールパスが **/usr/bin/python** であることを確認してください。

次に、python-setuptools パッケージを削除してください：

```shell
yum remove python-setuptools
```

その後、setuptools 関連のファイルを削除してください。

```shell
rm /usr/lib/python2.7/site-packages/setuptool* -rf
```

最後に、**ez_setup.py** ファイルを取得する必要があります。

```shell
wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```
