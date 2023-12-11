---
unlisted: true
---

# Brokerノードのデプロイと管理

このトピックでは、Brokerノードのデプロイ方法について説明します。Brokerノードを使用すると、StarRocksはHDFSやS3などのソースからデータを読み取り、独自の計算リソースを使用してデータの前処理、ロード、およびバックアップを行うことができます。

BEノードをホストする各インスタンスには、1つのBrokerノードをデプロイし、すべてのBrokerノードを同じ `broker_name` を使用して追加することをお勧めします。Brokerノードは、タスク処理時にデータ転送の負荷を自動的にバランスさせます。

Brokerノードは、データをBEノードに転送するためにネットワーク接続を使用します。BrokerノードとBEノードが同じマシンにデプロイされている場合、BrokerノードはデータをローカルのBEノードに転送します。

## 開始する前に

[デプロイの前提条件](../deployment/deployment_prerequisites.md)、[環境構成の確認](../deployment/environment_configurations.md)、および[デプロイファイルの準備](../deployment/prepare_deployment_files.md) で提供されている手順に従い、必要な構成を完了していることを確認してください。

## Brokerサービスの開始

以下の手順は、BEインスタンス上で実行されます。

1. 以前に準備した[StarRocks Brokerのデプロイファイル](../deployment/prepare_deployment_files.md)が格納されているディレクトリに移動し、Broker構成ファイル **apache_hdfs_broker/conf/apache_hdfs_broker.conf** を変更します。

   インスタンス上のHDFS Thrift RPCポート (`broker_ipc_port`, デフォルト: `8000`) が使用中の場合、Broker構成ファイルで有効な代替ポートを割り当てる必要があります。

   ```YAML
   broker_ipc_port = aaaa        # Default: 8000
   ```

   次の表は、Brokerがサポートする構成項目を示しています。

   | 構成項目 | デフォルト | 単位 | 説明 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | HDFSからデータを読み取るために使用されるバッファのサイズ。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | HDFSにデータを書き込むために使用されるバッファのサイズ。 |
   | client_expire_seconds | 300 | 秒 | 指定した時間内にpingを受信しないクライアントセッションは削除されます。 |
   | broker_ipc_port | 8000 | N/A | HDFSサービスのThrift RPCポート。 |
   | disable_broker_client_expiration_checking | false | N/A | 一部の場合、ブローカがOSSがクローズされるとブローカがスタックすることがあります。この状況を回避するためには、このパラメータを `true` に設定してチェックを無効にすることができます。 |
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | システムログ（INFO、WARNING、ERROR、FATALを含む）を保存するディレクトリ。 |
   | sys_log_level | INFO | N/A | ログレベル。`INFO`、`WARNING`、`ERROR`、`FATAL` の有効な値があります。 |
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | システムログがログロールされる方法のモード。`TIME-DAY`、`TIME-HOUR`、`SIZE-MB-nnn` の有効な値があります。デフォルト値は、1GBずつログがロールされることを示します。 |
   | sys_log_roll_num | 30 | N/A | 予約するログロールの数。 |
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | オーディットログファイルを保存するディレクトリ。 |
   | audit_log_modules | 空の文字列 | N/A | StarRocksがオーディットログエントリを生成するモジュール。デフォルトでは、StarRocksはslow_queryモジュールとqueryモジュールのためにオーディットログを生成します。複数のモジュールを指定することができます。 |
   | audit_log_roll_mode | TIME-DAY | N/A | `TIME-DAY`、`TIME-HOUR`、`SIZE-MB-<size>` の有効な値があります。 |
   | audit_log_roll_num | 10 | N/A | この構成は、audit_log_roll_modeが `SIZE-MB-<size>` に設定されている場合には機能しません。 |
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocksがシステムログを生成するモジュール。有効な値はBE内のネームスペースであり、`starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup` があります。 |

2. Brokerノードを起動します。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. Brokerのログをチェックし、Brokerノードが正常に起動したかどうかを確認します。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 他のインスタンスでも上記の手順を繰り返すことで、新しいBrokerノードを起動することができます。

## Brokerノードをクラスタに追加

以下の手順は、MySQLクライアント上で実行されます。MySQLクライアント5.5.0以降がインストールされている必要があります。

1. MySQLクライアントに接続します。初期アカウント `root` でログインする必要があります。デフォルトでパスワードは空です。

   ```Bash
   # <fe_address> を接続するFEノードのIPアドレス（priority_networks）またはFQDNに、
   # <query_port>（デフォルト: 9030）をfe.confで指定したquery_portに置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のコマンドを実行して、Brokerノードをクラスタに追加します。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

   > **注意**
   >
   > - 上記のコマンドを使用して一度に複数のBrokerノードを追加することができます。`<broker_address>:<broker_ipc_port>` のペアごとに1つのBrokerノードを表します。
   > - 同じ `broker_name` で複数のBrokerノードを追加することができます。

3. MySQLクライアントを使用して、Brokerノードが適切にクラスタに追加されたかどうかを確認します。

```sql
SHOW PROC "/brokers"\G
```

例：

```plain text
MySQL [(none)]> SHOW PROC "/brokers"\G
*************************** 1.行 ***************************
          名前: broker1
            IP: x.x.x.x
          ポート: 8000
         Alive: true
 LastStartTime: 2022-05-19 11:21:36
LastUpdateTime: 2022-05-19 11:28:31
        ErrMsg:
1 行が選択されました (0.00秒)
```

`Alive` のフィールドが `true` の場合、このBrokerは正常に起動され、クラスタに追加されています。

## Brokerノードの停止

以下のコマンドを実行して、Brokerノードを停止します。

```bash
./bin/stop_broker.sh --daemon
```

## Brokerノードのアップグレード

1. Brokerノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir> をBrokerノードのデプロイディレクトリに置き換えてください。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 新しいバージョンのファイルで **bin** と **lib** の元のデプロイファイルを置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. Brokerノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. Brokerノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返し、他のBrokerノードをアップグレードします。

## Brokerノードのダウングレード

1. Brokerノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <broker_dir> をBrokerノードのデプロイディレクトリに置き換えてください。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 元のバージョンのファイルで **bin** と **lib** の元のデプロイファイルを置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. Brokerノードを起動します。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. Brokerノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep broker
   ```

5. 上記の手順を繰り返し、他のBrokerノードをダウングレードします。