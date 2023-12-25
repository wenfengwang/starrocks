---
unlisted: true
---

# Broker ノードのデプロイと管理

このトピックでは、Broker ノードをデプロイする方法について説明します。Broker ノードを使用すると、StarRocks は HDFS や S3 などのソースからデータを読み取り、独自のコンピューティングリソースを使用してデータの前処理、ロード、バックアップを行うことができます。

BE ノードをホストする各インスタンスに 1 つの Broker ノードをデプロイし、同じ `broker_name` を使用してすべての Broker ノードを追加することをお勧めします。Broker ノードは、タスク処理時にデータ転送の負荷を自動的にバランスします。

Broker ノードはネットワーク接続を使用して BE ノードにデータを送信します。Broker ノードと BE ノードが同じマシンにデプロイされている場合、Broker ノードはローカルの BE ノードにデータを送信します。

## 始める前に

[デプロイの前提条件](../deployment/deployment_prerequisites.md)、[環境設定の確認](../deployment/environment_configurations.md)、および[デプロイファイルの準備](../deployment/prepare_deployment_files.md)に記載されている手順に従って、必要な設定が完了していることを確認してください。

## Broker サービスの開始

以下の手順は BE インスタンスで実行されます。

1. 事前に準備した [StarRocks Broker デプロイメントファイル](../deployment/prepare_deployment_files.md)が格納されているディレクトリに移動し、Broker 設定ファイル **apache_hdfs_broker/conf/apache_hdfs_broker.conf** を変更します。

   インスタンスの HDFS Thrift RPC ポート (`broker_ipc_port`、デフォルト：`8000`) が使用中の場合は、Broker 設定ファイルで有効な代替ポートを指定する必要があります。

   ```YAML
   broker_ipc_port = aaaa        # Default: 8000
   ```

   次の表に、Broker がサポートする設定項目を示します。

   | 設定項目 | デフォルト | 単位 | 説明 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | HDFS からデータを読み取る際に使用されるバッファのサイズ。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | HDFS にデータを書き込む際に使用されるバッファのサイズ。 |
   | client_expire_seconds | 300 | 秒 | 指定された時間内に何も ping を受信しないクライアントセッションは削除されます。 |
   | broker_ipc_port | 8000 | N/A | HDFS Thrift RPC ポート。 |
   | disable_broker_client_expiration_checking | false | N/A | 期限切れの OSS ファイルディスクリプタのチェックとクリアを無効にするかどうか。場合によっては、OSS がクローズされたときに Broker がスタックする原因となることがあります。この状況を回避するために、このパラメータを `true` に設定してチェックを無効にすることができます。 |
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | システムログ（INFO、WARNING、ERROR、FATAL を含む）を保存するディレクトリ。 |
   | sys_log_level | INFO | N/A | ログレベル。有効な値は INFO、WARNING、ERROR、FATAL です。 |
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | システムログがログロールに分割される方法。有効な値は TIME-DAY、TIME-HOUR、SIZE-MB-nnn です。デフォルト値は、ログが 1 GB ごとにロールに分割されることを示します。 |
   | sys_log_roll_num | 30 | N/A | 保持するログロールの数。 |
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | 監査ログファイルを保存するディレクトリ。 |
   | audit_log_modules | 空文字列 | N/A | StarRocks が監査ログエントリを生成するモジュール。デフォルトでは、slow_query モジュールと query モジュールの監査ログが生成されます。複数のモジュールを指定することができ、その名前はコンマ (,) とスペースで区切る必要があります。 |
   | audit_log_roll_mode | TIME-DAY | N/A | 有効な値は `TIME-DAY`、`TIME-HOUR`、`SIZE-MB-<size>` です。 |
   | audit_log_roll_num | 10 | N/A | audit_log_roll_mode が `SIZE-MB-<size>` に設定されている場合、この設定は機能しません。 |
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocks がシステムログを生成するモジュール。有効な値は BE の名前空間を含む `starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup` です。 |

2. Broker ノードを起動します。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. Broker ログを確認して、Broker ノードが正常に起動されたかどうかを検証します。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 他のインスタンスで上記の手順を繰り返すことで、新しい Broker ノードを起動できます。

## クラスターへの Broker ノードの追加

以下の手順は MySQL クライアントで実行されます。MySQL クライアント 5.5.0 以降がインストールされている必要があります。

1. MySQL クライアントを介して StarRocks に接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> を接続する FE ノードの IP アドレス（priority_networks）または FQDN に置き換え、
   # <query_port>（デフォルト：9030）を fe.conf で指定した query_port に置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のコマンドを実行して、Broker ノードをクラスターに追加します。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

   > **注記**
   >
   > - 上記のコマンドを使用して、一度に複数の Broker ノードを追加できます。各 `<broker_address>:<broker_ipc_port>` ペアは 1 つの Broker ノードを表します。
   > - 同じ `broker_name` を使用して複数の Broker ノードを追加できます。

3. MySQL クライアントを介して Broker ノードが適切にクラスターに追加されたかどうかを検証します。

```sql
SHOW PROC "/brokers"\G
```

例：

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

`Alive` フィールドが `true` の場合、この Broker は適切に起動され、クラスターに追加されています。

## Broker ノードの停止

Broker ノードを停止するには、次のコマンドを実行します。

```bash
./bin/stop_broker.sh --daemon
```

## Broker ノードのアップグレード

1. Broker ノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir> を Broker ノードのデプロイメントディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin** と **lib** の下にある元のデプロイメントファイルを新しいバージョンのものに置き換えます。

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

4. Broker ノードが正常に起動されたかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返して他の Broker ノードをアップグレードします。

## Broker ノードのダウングレード

1. Broker ノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir> を Broker ノードのデプロイメントディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin** と **lib** の下にある元のデプロイメントファイルを以前のバージョンのものに置き換えます。

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

4. Broker ノードが正常に起動されたかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返して他の Broker ノードをダウングレードします。
