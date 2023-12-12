---
unlisted: true
---

# ブローカーノードを展開および管理する

このトピックでは、ブローカーノードを展開する方法について説明します。ブローカーノードを使用すると、StarRocksはHDFSやS3などのソースからデータを読み取り、独自の計算リソースを使用してデータを前処理、ロード、およびバックアップできます。

BEノードをホストするインスタンスごとに1つのブローカーノードを展開し、同じ`broker_name`を使用してすべてのブローカーノードを追加することをお勧めします。ブローカーノードは、タスクを処理する際にデータ伝送負荷を自動的にバランス調整します。

ブローカーノードは、データをBEノードに伝送するためにネットワーク接続を使用します。ブローカーノードとBEノードが同じマシンに展開されている場合、ブローカーノードはデータをローカルBEノードに伝送します。

## 開始する前に

[Deployment prerequisites](../deployment/deployment_prerequisites.md)、[Check environment configurations](../deployment/environment_configurations.md)、および[Prepare deployment files](../deployment/prepare_deployment_files.md)の手順に従って必要な構成を完了していることを確認してください。

## ブローカーサービスを開始する

以下の手順はBEインスタンスで実行されます。

1. 以前に準備した[StarRocksブローカー展開ファイル](../deployment/prepare_deployment_files.md)を保存しているディレクトリに移動し、ブローカー設定ファイル**apache_hdfs_broker/conf/apache_hdfs_broker.conf**を変更します。

   インスタンスのHDFSスリフトRPCポート（'broker_ipc_port'、デフォルト値：`8000`）が使用中の場合、ブローカー設定ファイルで有効な代替ポートを指定する必要があります。

   ```YAML
   broker_ipc_port = aaaa        # デフォルト：8000
   ```

   以下の表には、ブローカーでサポートされている構成項目が示されています。

   | 構成項目 | デフォルト | 単位 | 説明 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | HDFSからデータを読み取るために使用されるバッファのサイズです。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | HDFSにデータを書き込むために使用されるバッファのサイズです。 |
   | client_expire_seconds | 300 | 秒 | 指定された時間経過後にpingを受信しない場合、クライアントセッションは削除されます。 |
   | broker_ipc_port | 8000 | N/A | HDFSスリフトRPCポートです。 |
   | disable_broker_client_expiration_checking | false | N/A | 期限切れのOSSファイル記述子のチェックおよびクリアを無効にするかどうかを示します。これにより、一部の場合、ブローカーがOSSがクローズされるとスタックする可能性があります。この状況を避けるために、このパラメータを`true`に設定してチェックを無効にできます。 |
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | システムログ（INFO、WARNING、ERROR、FATALを含む）を保存するディレクトリです。 |
   | sys_log_level | INFO | N/A | ログレベル。INFO、WARNING、ERROR、FATALなどの有効な値があります。 |
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | システムログがログロールにセグメント化される方法のモードです。TIME-DAY、TIME-HOUR、SIZE-MB-nnnなどが有効な値です。デフォルト値は、ログがそれぞれ1 GBのロールにセグメント化されることを示します。 |
   | sys_log_roll_num | 30 | N/A | 予約するログロールの数です。 |
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | オーディットログファイルを保存するディレクトリです。 |
   | audit_log_modules | 空の文字列 | N/A | StarRocksがオーディットログエントリを生成するモジュールです。デフォルトでは、StarRocksはslow_queryモジュールとqueryモジュールのためにオーディットログを生成します。複数のモジュールを指定することができます。モジュール名はコンマ（,）とスペースで区切られなければなりません。 |
   | audit_log_roll_mode | TIME-DAY | N/A | `TIME-DAY`、`TIME-HOUR`、`SIZE-MB-<サイズ>`などが有効な値です。 |
   | audit_log_roll_num | 10 | N/A | この構成は、`audit_log_roll_mode`が`SIZE-MB-<サイズ>`に設定されている場合は機能しません。 |
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocksがシステムログを生成するモジュールです。有効な値はBEの名前空間であり、`starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup`があります。 |

2. ブローカーノードを開始します。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. ブローカーログをチェックして、ブローカーノードが正常に開始されたかどうかを確認します。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 他のインスタンス上で上記の手順を繰り返すことで新しいブローカーノードを開始できます。

## クラスターにブローカーノードを追加する

以下の手順はMySQLクライアントで実行されます。MySQLクライアント5.5.0以降がインストールされている必要があります。

1. MySQLクライアントに接続します。初期アカウント`root`でログインする必要があります。デフォルトではパスワードは空です。

   ```Bash
   # <fe_address>を接続するFEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>（デフォルト：9030）をfe.confで指定したquery_portに置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のコマンドを実行して、ブローカーノードをクラスターに追加します。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

   > **注意**
   >
   > - 上記のコマンドを使用して一度に複数のブローカーノードを追加できます。`<broker_address>:<broker_ipc_port>`のペアごとに1つのブローカーノードを表します。
   > - 同じ`broker_name`で複数のブローカーノードを追加できます。

3. MySQLクライアントを使用して、ブローカーノードが適切にクラスターに追加されたかを確認します。

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

   `Alive`フィールドが`true`である場合、このブローカーは正常に起動し、クラスターに追加されています。

## ブローカーノードを停止する

以下のコマンドを実行して、ブローカーノードを停止します。

```bash
./bin/stop_broker.sh --daemon
```

## ブローカーノードをアップグレードする

1. ブローカーノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir>をブローカーノードの展開ディレクトリで置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin**および**lib**の元の展開ファイルを新しいバージョンのファイルで置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. ブローカーノードを開始します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ブローカーノードが正常に開始されたかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を他のブローカーノードをアップグレードするために繰り返します。

## ブローカーノードをダウングレードする

1. ブローカーノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir>をブローカーノードの展開ディレクトリで置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin**および**lib**の元の展開ファイルを以前のバージョンのファイルで置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. ブローカーノードを開始します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ブローカーノードが正常に開始されたかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を他のブローカーノードをダウングレードするために繰り返します。