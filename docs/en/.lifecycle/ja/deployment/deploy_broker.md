---
displayed_sidebar: "Japanese"
---

# ブローカーノードの展開と管理

このトピックでは、ブローカーノードの展開方法について説明します。ブローカーノードを使用すると、StarRocksはHDFSやS3などのソースからデータを読み取り、独自の計算リソースを使用してデータを前処理、ロード、バックアップすることができます。

BEノードをホストする各インスタンスには、1つのブローカーノードをデプロイし、すべてのブローカーノードを同じ`broker_name`で追加することをおすすめします。ブローカーノードは、タスクを処理する際にデータ転送負荷を自動的にバランスします。

ブローカーノードは、BEノードにデータを転送するためにネットワーク接続を使用します。ブローカーノードとBEノードが同じマシンに展開されている場合、ブローカーノードはデータをローカルのBEノードに転送します。

## 開始ブローカーサービス

以下の手順はBEインスタンスで実行されます。

1. 以前に準備した[StarRocksブローカーデプロイメントファイル](../deployment/prepare_deployment_files.md)を格納しているディレクトリに移動し、ブローカー設定ファイル **apache_hdfs_broker/conf/apache_hdfs_broker.conf** を編集します。

   インスタンス上のHDFS Thrift RPCポート（`broker_ipc_port`、デフォルト：`8000`）が使用中の場合、ブローカー設定ファイルで有効な代替ポートを割り当てる必要があります。

   ```YAML
   broker_ipc_port = aaaa        # デフォルト：8000
   ```

   以下の表には、ブローカーがサポートする設定項目がリストされています。

   | 設定項目 | デフォルト | 単位 | 説明 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | HDFSからデータを読み取るために使用されるバッファのサイズです。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | HDFSにデータを書き込むために使用されるバッファのサイズです。 |
   | client_expire_seconds | 300 | 秒 | 指定された時間後にpingを受信しない場合、クライアントセッションは削除されます。 |
   | broker_ipc_port | 8000 | N/A | HDFSスリフトRPCポートです。 |
   | disable_broker_client_expiration_checking | false | N/A | 期限切れのOSSファイルディスクリプタのチェックとクリアを無効にするかどうかを指定します。これにより、一部の場合においてOSSが閉じられるとブローカーがスタックすることがあります。この状況を回避するために、このパラメータを`true`に設定してチェックを無効にすることができます。 |
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | システムログ（INFO、WARNING、ERROR、FATALを含む）を保存するディレクトリです。 |
   | sys_log_level | INFO | N/A | ログレベルです。INFO、WARNING、ERROR、FATALなどの有効な値があります。 |
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | システムログがログロールにセグメント化される方法です。TIME-DAY、TIME-HOUR、SIZE-MB-nnnなどの有効な値があります。デフォルト値は、ログが1GBごとにセグメント化されることを示します。 |
   | sys_log_roll_num | 30 | N/A | 予約するログロールの数です。 |
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | オーディットログファイルを保存するディレクトリです。 |
   | audit_log_modules | 空の文字列 | N/A | StarRocksがオーディットログエントリを生成するモジュールです。デフォルトでは、StarRocksはslow_queryモジュールとqueryモジュールのためにオーディットログを生成します。複数のモジュールを指定する場合は、モジュール名をカンマ（,）とスペースで区切る必要があります。 |
   | audit_log_roll_mode | TIME-DAY | N/A | `TIME-DAY`、`TIME-HOUR`、`SIZE-MB-<size>`などの有効な値があります。 |
   | audit_log_roll_num | 10 | N/A | この設定は、audit_log_roll_modeが`SIZE-MB-<size>`に設定されている場合は機能しません。 |
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocksがシステムログを生成するモジュールです。有効な値は、`starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup`などのBEの名前空間です。 |

2. ブローカーノードを起動します。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. ブローカーログを確認して、ブローカーノードが正常に起動しているかどうかを確認します。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 上記の手順を他のインスタンスで繰り返すことで、新しいブローカーノードを起動することができます。

## クラスターにブローカーノードを追加する

以下の手順はMySQLクライアントで実行されます。MySQLクライアント5.5.0以降がインストールされている必要があります。

1. MySQLクライアントを使用してStarRocksに接続します。初期アカウント`root`でログインし、パスワードはデフォルトで空です。

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
   > - 上記のコマンドを使用して複数のブローカーノードを一度に追加することができます。各`<broker_address>:<broker_ipc_port>`のペアは1つのブローカーノードを表します。
   > - 同じ`broker_name`で複数のブローカーノードを追加することができます。

3. MySQLクライアントを使用して、ブローカーノードが正しく追加されたかどうかを確認します。

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

`Alive`フィールドが`true`の場合、このブローカーは正常に起動され、クラスターに追加されています。

## ブローカーノードの停止

以下のコマンドを実行して、ブローカーノードを停止します。

```bash
./bin/stop_broker.sh --daemon
```

## ブローカーノードのアップグレード

1. ブローカーノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir>をブローカーノードのデプロイディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin**と**lib**の元のデプロイメントファイルを新しいバージョンのファイルで置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. ブローカーノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ブローカーノードが正常に起動しているかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を他のブローカーノードでアップグレードするために繰り返します。

## ブローカーノードのダウングレード

1. ブローカーノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir>をブローカーノードのデプロイディレクトリに置き換えます。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. **bin**と**lib**の元のデプロイメントファイルを以前のバージョンのファイルで置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. ブローカーノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. ブローカーノードが正常に起動しているかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を他のブローカーノードでダウングレードするために繰り返します。
