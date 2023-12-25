---
unlisted: true
---

# Broker ノードのデプロイ

この文書では、Broker ノードのデプロイと管理について説明します。Broker を通じて、StarRocks は対応するデータソース（例：HDFS、S3）上のデータを読み取り、自身の計算リソースを利用してデータの前処理とインポートを行います。また、Broker はデータのエクスポート、バックアップとリカバリなどの機能にも使用されます。

各 BE ノードがデプロイされているマシン上に Broker ノードをデプロイし、すべての Broker ノードを同じ `broker_name` に追加することをお勧めします。Broker ノードは、タスク処理時にデータ転送のプレッシャーを自動的にスケジュールします。

Broker ノードは BE ノードとネットワークを介してデータを転送します。Broker ノードと BE ノードが同じマシンにデプロイされている場合、ローカルの BE ノードを優先してデータ転送を行います。

## 準備作業

[デプロイ前提条件](../deployment/deployment_prerequisites.md)、[環境設定の確認](../deployment/environment_configurations.md)、[デプロイファイルの準備](../deployment/prepare_deployment_files.md)を参照して、準備作業を完了してください。

## Broker サービスの起動

以下の操作は BE インスタンス上で実行します。

1. 事前に準備した [StarRocks Broker デプロイファイル](../deployment/prepare_deployment_files.md)のパスに移動し、Broker 設定ファイル **apache_hdfs_broker/conf/apache_hdfs_broker.conf** を変更します。

   Broker ノードの HDFS Thrift RPC ポート（`broker_ipc_port`、デフォルト値：`8000`）が使用中の場合、Broker 設定ファイルで他の利用可能なポートを割り当てる必要があります。

   ```YAML
   broker_ipc_port = aaaa        # デフォルト値：8000
   ```

   下記の表は、Broker がサポートする設定項目を示しています。

   | 設定項目 | デフォルト値 | 単位 | 説明 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | HDFS からデータを読み取るためのメモリサイズ。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | HDFS にデータを書き込むためのメモリサイズ。 |
   | client_expire_seconds | 300 | 秒 | クライアントの有効期限。指定された時間後に ping が受信されない場合、クライアントセッションは削除されます。 |
   | broker_ipc_port | 8000 | 該当なし | HDFS Thrift RPC ポート。 |
   | disable_broker_client_expiration_checking | false | 該当なし | 期限切れ OSS ファイルハンドルのチェックとクリアを無効にするかどうか。クリアすると、OSS シャットダウン時に Broker がハングすることがあります。このような状況を避けるために、このパラメータを `true` に設定してチェックを無効にすることができます。 |
   | sys_log_dir | `${BROKER_HOME}/log` | 該当なし | システムログ（INFO、WARNING、ERROR、FATAL を含む）を保存するディレクトリ。 |
   | sys_log_level | INFO | 該当なし | ログレベル。有効な値には INFO、WARNING、ERROR、FATAL があります。 |
   | sys_log_roll_mode | SIZE-MB-1024 | 該当なし | システムログのロールモード。有効な値には TIME-DAY、TIME-HOUR、SIZE-MB-nnn があります。デフォルト値は、ログを 1 GB ごとに分割することを意味します。 |
   | sys_log_roll_num | 30 | 該当なし | 保持するシステムログの巻数。 |
   | audit_log_dir | `${BROKER_HOME}/log` | 該当なし | 監査ログファイルを保存するディレクトリ。 |
   | audit_log_modules | 空文字列 | 該当なし | StarRocks が監査ログエントリを生成するモジュール。デフォルトでは、StarRocks は slow_query モジュールと query モジュールの監査ログを生成します。カンマ（,）とスペースを使用して複数のモジュールを指定することができます。|
   | audit_log_roll_mode | TIME-DAY | 該当なし | 監査ログのロールモード。有効な値には TIME-DAY、TIME-HOUR、SIZE-MB-nnn があります。 |
   | audit_log_roll_num | 10 | 該当なし | 保持する監査ログの巻数。`audit_log_roll_mode` が `SIZE-MB-nnn` に設定されている場合、この設定は無効です。 |
   | sys_log_verbose_modules | com.starrocks | 該当なし | StarRocks がシステムログを生成するモジュール。有効な値は BE の namespace で、`starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup` を含みます。 |

2. Broker ノードを起動します。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. Broker ログを確認し、Broker ノードが正常に起動したかをチェックします。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 他の BE インスタンスで上記の手順を繰り返し、新しい Broker ノードを起動します。

## クラスターへの Broker ノードの追加

以下の手順は MySQL クライアントインスタンスで実行します。MySQL クライアント（バージョン 5.5.0 以上）をインストールする必要があります。

1. MySQL クライアントを使用して StarRocks に接続します。初期ユーザー `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> を接続する FE ノードの IP アドレス（priority_networks）または FQDN に置き換え、
   # <query_port>（デフォルト：9030）を fe.conf で指定した query_port に置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下の SQL を実行して、Broker ノードをクラスターに追加します。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_ip>:<broker_ipc_port>";
   ```

   > **注記**
   >
   > - 一つの SQL で複数の Broker ノードを追加することができます。`<broker_ip>:<broker_ipc_port>` の各ペアは一つの Broker ノードを表します。
   > - 同じ `broker_name` を持つ複数の Brokers ノードを追加することができます。

3. 以下の SQL を実行して、Broker ノードの状態を確認します。

```sql
SHOW PROC "/brokers"\G
```

例

```plain text
MySQL [(none)]> SHOW PROC "/brokers"\G
*************************** 1. row ***************************
          Name: broker1
            IP: x.x.x.x
          Port: 8000
         Alive: true
 LastStartTime: 2022-05-19 11:21:36
LastUpdateTime: 2022-05-19 11:28:31
        ErrMsg:
1 row in set (0.00 sec)
```

`Alive` フィールドが `true` であれば、その Broker ノードは正常に起動し、クラスターに参加していることを意味します。

## Broker ノードの停止

以下のコマンドを実行して Broker ノードを停止します。

```bash
./bin/stop_broker.sh --daemon
```

## Broker ノードのアップグレード

1. Broker ノードの作業ディレクトリに入り、ノードを停止します。

   ```Bash
   # <broker_dir> を Broker ノードのデプロイディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. デプロイファイルの既存の **bin** と **lib** パスを新しいバージョンのデプロイファイルに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. Broker ノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ノードが正常に起動したかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返して、他の Broker ノードをアップグレードします。

## Broker ノードのダウングレード

1. Broker ノードの作業ディレクトリに入り、ノードを停止します。

   ```Bash
   # <broker_dir> を Broker ノードのデプロイディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. デプロイファイルの既存の **bin** と **lib** パスを新しいバージョンのデプロイファイルに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. Broker ノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ノードが正常に起動したかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返して、他の Broker ノードをダウングレードします。
