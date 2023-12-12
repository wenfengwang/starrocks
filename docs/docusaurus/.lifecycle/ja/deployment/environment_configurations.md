---
displayed_sidebar: "Japanese"
---

# 環境設定を確認する

このトピックでは、StarRocksを展開する前に確認および設定する必要がある環境およびシステム設定項目をすべて列挙します。これらの設定項目を適切に設定することで、StarRocksクラスタは高可用性とパフォーマンスで動作します。

## ポート

StarRocksは異なるサービスのために特定のポートを使用します。これらのポートが他のサービスで使用されていないかどうかを確認してください。

### FEポート

FEの展開に使用されるインスタンスでは、次のポートを確認する必要があります：

- `8030`： FE HTTPサーバーポート（`http_port`）
- `9020`： FE Thriftサーバーポート（`rpc_port`）
- `9030`： FE MySQLサーバーポート（`query_port`）
- `9010`： FEインターナル通信ポート（`edit_log_port`）

次のコマンドをFEインスタンスで実行して、これらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

上記のポートのいずれかが使用されている場合、代替案を見つけ、FEノードを展開する際に後で指定する必要があります。詳しい手順については、[StarRocksの展開 - リーダーFEノードの起動](../deployment/deploy_manually.md#step-1-start-the-leader-fe-node)を参照してください。

### BEポート

BEの展開に使用されるインスタンスでは、次のポートを確認する必要があります：

- `9060`： BE Thriftサーバーポート（`be_port`）
- `8040`： BE HTTPサーバーポート（`be_http_port`）
- `9050`： BEハートビートサービスポート（`heartbeat_service_port`）
- `8060`： BE bRPCポート（`brpc_port`）

次のコマンドをBEインスタンスで実行して、これらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のポートのいずれかが使用されている場合、代替案を見つけ、BEノードを展開する際に後で指定する必要があります。詳しい手順については、[StarRocksの展開 - BEサービスの起動](../deployment/deploy_manually.md#step-2-start-the-be-service)を参照してください。

### CNポート

CNの展開に使用されるインスタンスでは、次のポートを確認する必要があります：

- `9060`： CN Thriftサーバーポート（`be_port`）
- `8040`： CN HTTPサーバーポート（`be_http_port`）
- `9050`： CNハートビートサービスポート（`heartbeat_service_port`）
- `8060`： CN bRPCポート（`brpc_port`）
- `9070`：CNの追加エージェントサービスポート（v3.0でのBE）（`starlet_port`） - 共有データクラスタ内での使用

次のコマンドをCNインスタンスで実行して、これらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
netstat -tunlp | grep 9070
```

上記のポートのいずれかが使用されている場合、代替案を見つけ、CNノードを展開する際に後で指定する必要があります。詳しい手順については、[StarRocksの展開 - CNサービスの起動（オプション）](../deployment/deploy_manually.md#step-3-optional-start-the-cn-service)を参照してください。

## ホスト名

StarRocksクラスタの[FQDNアクセスを有効にする](../administration/enable_fqdn.md)場合、各インスタンスにホスト名を割り当てる必要があります。

各インスタンスのファイル**/etc/hosts**に、クラスタ内の他のすべてのインスタンスのIPアドレスと対応するホスト名を指定する必要があります。

> **注意**
>
> ファイル**/etc/hosts**内のすべてのIPアドレスは一意である必要があります。

## JDKの設定

StarRocksは、インスタンス上のJava依存関係を見つけるために、環境変数`JAVA_HOME`に依存しています。

次のコマンドを実行して環境変数`JAVA_HOME`を確認してください：

```Bash
echo $JAVA_HOME
```

次の手順に従って`JAVA_HOME`を設定してください：

1. ファイル**/etc/profile**で`JAVA_HOME`を設定します：

   ```Bash
   sudo vi /etc/profile
   # JDKがインストールされているパスに置き換えてください。
   export JAVA_HOME=<path_to_JDK>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 変更を有効にします：

   ```Bash
   source /etc/profile
   ```

変更を確認するには次のコマンドを実行してください：

```Bash
java -version
```

## CPUのスケーリングガバナー

この設定項目は**オプション**です。CPUがスケーリングガバナーをサポートしていない場合は無視できます。

CPUのスケーリングガバナーは、CPUのパワーモードを制御します。サポートしている場合は、より良いCPUパフォーマンスのために`performance`に設定することをお勧めします：

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## メモリー設定

### メモリーのオーバーコミット

メモリーオーバーコミットは、オペレーティングシステムにメモリーリソースをプロセスにオーバーコミットすることを許可します。メモリーオーバーコミットを有効にすることをお勧めします。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### 透過的な巨大ページ

透過的な巨大ページはデフォルトで有効になっています。メモリーアロケーターに干渉し、パフォーマンスの低下につながる可能性があるため、この機能を無効にすることをお勧めします。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### スワップ領域

スワップ領域を無効にすることをお勧めします。

次の手順に従ってスワップ領域を確認および無効にしてください：

1. スワップ領域を無効にします。

   ```SQL
   swapoff /<path_to_swap_space>
   ```

2. 設定ファイル**/etc/fstab**からスワップ領域の情報を削除します。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. スワップ領域が無効になっていることを確認します。

   ```Bash
   free -m
   ```

### スワップネス

パフォーマンスに与える影響を排除するために、スワップネスを無効にすることをお勧めします。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## ストレージ設定

ストレージメディアに応じて、適切なスケジューラーアルゴリズムを選択することをお勧めします。

次のコマンドを実行して使用しているスケジューラーアルゴリズムを確認できます：

```Bash
cat /sys/block/${disk}/queue/scheduler
# 例：cat /sys/block/vdb/queue/scheduler を実行してください
```

SATAディスクにはmq-deadlineスケジューラーアルゴリズムが適しています。SSDおよびNVMeディスクにはkyberスケジューラーアルゴリズムが適しています。

### SATA

mq-deadlineスケジューラーアルゴリズムはSATAディスクに適しています。

この項目を一時的に変更します：

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

変更を永続的にするには、この項目を変更した後に次のコマンドを実行してください：

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSDおよびNVMe

kyberスケジューラーアルゴリズムはNVMeまたはSSDディスクに適しています。

この項目を一時的に変更します：

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

システムがSSDおよびNVMe用にkyberスケジューラをサポートしていない場合は、none（またはnoop）スケジューラを使用することをお勧めします。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

これらの項目を変更した後に、永続的な変更を行うには次のコマンドを実行してください：

```Bash
chmod +x /etc/rc.d/rc.local
```

## SELinux

SELinuxを無効にすることをお勧めします。

```Bash
sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sed -i 's/SELINUXTYPE/#SELINUXTYPE/' /etc/selinux/config
setenforce 0 
```

## ファイアウォール

ファイアウォールが有効になっている場合は、FEノード、BEノード、およびBrokerの内部ポートを開いてください。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG変数

次のコマンドを実行してLANG変数を確認および手動で構成してください：

```Bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

## タイムゾーン

実際のタイムゾーンに応じてこの項目を設定してください。

次の例では、タイムゾーンを`/Asia/Shanghai`に設定しています。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

## ulimitの設定

StarRocksでの問題が発生する可能性があります。それは**最大ファイルディスクリプタ**の値と**最大ユーザープロセス**の値が異常に小さい場合です。

### 最大ファイルディスクリプタ

次のコマンドを実行してファイルディスクリプタの最大数を設定できます：

```Bash
ulimit -n 655350
```

### 最大ユーザープロセス

次のコマンドを実行してユーザープロセスの最大数を設定できます：

```Bash
```Bash
ulimit -u 40960
```

## ファイルシステムの構成

ext4またはxfsジャーナリングファイルシステムを使用することをお勧めします。次のコマンドを実行してマウントタイプを確認できます：

```Bash
df -Th
```

## ネットワーク構成

### tcp_abort_on_overflow

システムが現在新しい接続試行で過負荷になっており、デーモンが処理できない場合、新しい接続をリセットするように許可します：

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

任意のリスニングソケットにキューイングされた最大接続要求の数を `1024` に指定します：

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTP構成

StarRocksクラスタ内のノード間での時間同期を構成する必要があります。これによりトランザクションの線形一貫性が確保されます。 pool.ntp.org が提供するインターネット時刻サービスを使用するか、オフライン環境に組み込まれたNTPサービスを使用できます。例えば、クラウドサービスプロバイダが提供するNTPサービスを使用できます。

1. NTP時刻サーバーが存在するかどうかを確認します。

   ```Bash
   rpm -qa | grep ntp
   ```

2. そのようなものがない場合は、NTPサービスをインストールします。

   ```Bash
   sudo yum install ntp ntpdate && \
   sudo systemctl start ntpd.service && \
   sudo systemctl enable ntpd.service
   ```

3. NTPサービスを確認します。

   ```Bash
   systemctl list-unit-files | grep ntp
   ```

4. NTPサービスの接続とモニタリング状態を確認します。

   ```Bash
   netstat -tlunp | grep ntp
   ```

5. アプリケーションがNTPサーバーと同期しているかを確認します。

   ```Bash
   ntpstat
   ```

6. ネットワーク内のすべての構成済みNTPサーバーの状態を確認します。

   ```Bash
   ntpq -p
   ```

## 高同時実行構成

StarRocksクラスタが高負荷の同時実行を持つ場合は、次の構成を設定することをお勧めします：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```