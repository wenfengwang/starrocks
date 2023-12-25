---
displayed_sidebar: English
---

# 環境設定の確認

このトピックでは、StarRocksをデプロイする前に確認および設定する必要があるすべての環境およびシステム設定項目をリストしています。これらの設定項目を適切に設定することで、StarRocksクラスターは高可用性と高性能で動作します。

## ポート

StarRocksは、異なるサービスに対して特定のポートを使用します。これらのインスタンスで他のサービスをデプロイしている場合は、これらのポートが各インスタンスで占有されていないかを確認してください。

### FEポート

FEデプロイ用のインスタンスでは、以下のポートを確認する必要があります：

- `8030`：FE HTTPサーバーポート（`http_port`）
- `9020`：FE Thriftサーバーポート（`rpc_port`）
- `9030`：FE MySQLサーバーポート（`query_port`）
- `9010`：FE内部通信ポート（`edit_log_port`）

以下のコマンドをFEインスタンスで実行し、これらのポートが占有されているかどうかを確認します：

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

上記のポートが占有されている場合は、代替ポートを見つけて、FEノードをデプロイする際に後で指定する必要があります。詳細な手順については、[StarRocksのデプロイ - リーダーFEノードの起動](../deployment/deploy_manually.md#step-1-start-the-leader-fe-node)を参照してください。

### BEポート

BEデプロイ用のインスタンスでは、以下のポートを確認する必要があります：

- `9060`：BE Thriftサーバーポート（`be_port`）
- `8040`：BE HTTPサーバーポート（`be_http_port`）
- `9050`：BEハートビートサービスポート（`heartbeat_service_port`）
- `8060`：BE bRPCポート（`brpc_port`）

以下のコマンドをBEインスタンスで実行し、これらのポートが占有されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のポートが占有されている場合は、代替ポートを見つけて、BEノードをデプロイする際に後で指定する必要があります。詳細な手順については、[StarRocksのデプロイ - BEサービスの開始](../deployment/deploy_manually.md#step-2-start-the-be-service)を参照してください。

### CNポート

CNデプロイ用のインスタンスでは、以下のポートを確認する必要があります：

- `9060`：CN Thriftサーバーポート（`be_port`）
- `8040`：CN HTTPサーバーポート（`be_http_port`）
- `9050`：CNハートビートサービスポート（`heartbeat_service_port`）
- `8060`：CN bRPCポート（`brpc_port`）
- `9070`：共有データクラスター内のCN（v3.0のBE）用の追加エージェントサービスポート（`starlet_port`）

以下のコマンドをCNインスタンスで実行し、これらのポートが占有されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
netstat -tunlp | grep 9070
```

上記のポートが占有されている場合は、代替ポートを見つけて、CNノードをデプロイする際に後で指定する必要があります。詳細な手順については、[StarRocksのデプロイ - CNサービスの開始](../deployment/deploy_manually.md#step-3-optional-start-the-cn-service)を参照してください。

## ホスト名

StarRocksクラスターでFQDNアクセスを[有効にする](../administration/enable_fqdn.md)場合、各インスタンスにホスト名を割り当てる必要があります。

各インスタンスの**/etc/hosts**ファイルには、クラスター内の他のすべてのインスタンスのIPアドレスと対応するホスト名を指定する必要があります。

> **注意**
>
> **/etc/hosts**ファイル内のすべてのIPアドレスは一意でなければなりません。

## JDK設定

StarRocksは、インスタンス上のJava依存関係を特定するために環境変数`JAVA_HOME`を使用します。

環境変数`JAVA_HOME`を確認するには、次のコマンドを実行します：

```Bash
echo $JAVA_HOME
```

`JAVA_HOME`を設定するには、以下の手順を実行します：

1. **/etc/profile**ファイルで`JAVA_HOME`を設定します：

   ```Bash
   sudo vi /etc/profile
   # JDKがインストールされているパスに<path_to_JDK>を置き換えてください。
   export JAVA_HOME=<path_to_JDK>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 変更を反映させます：

   ```Bash
   source /etc/profile
   ```

変更が反映されたかを確認するには、以下のコマンドを実行します：

```Bash
java -version
```

## CPUスケーリングガバナー

この設定項目は**オプショナル**です。CPUがスケーリングガバナーをサポートしていない場合は、この項目をスキップしても構いません。

CPUスケーリングガバナーはCPUの電源モードを制御します。CPUがこれをサポートしている場合は、より良いCPU性能のために`performance`モードに設定することを推奨します：

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## メモリ設定

### メモリオーバーコミット

メモリオーバーコミットを有効にすると、OSはプロセスに対してメモリリソースをオーバーコミットすることができます。メモリオーバーコミットを有効にすることを推奨します。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### トランスペアレントヒュージページ


透過的なヒュージページはデフォルトで有効になっています。この機能はメモリアロケータに干渉し、パフォーマンスの低下につながる可能性があるため、無効にすることをお勧めします。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### スワップスペース

スワップスペースを無効にすることをお勧めします。

次の手順に従って、スワップスペースを確認および無効にします。

1. スワップスペースを無効にします。

   ```Bash
   swapoff /<path_to_swap_space>
   ```

2. 構成ファイル **/etc/fstab** からスワップスペース情報を削除します。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. スワップスペースが無効になっていることを確認します。

   ```Bash
   free -m
   ```

### Swappiness

パフォーマンスへの影響を排除するために、swappinessを無効にすることをお勧めします。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## ストレージ設定

使用するストレージ媒体に応じて、適切なスケジューラアルゴリズムを選択することをお勧めします。

次のコマンドを実行して、使用しているスケジューラアルゴリズムを確認できます。

```Bash
cat /sys/block/${disk}/queue/scheduler
# 例えば、cat /sys/block/vdb/queue/scheduler を実行します
```

SATAディスクにはmq-deadlineスケジューラを、SSDおよびNVMeディスクにはkyberスケジューラアルゴリズムを使用することをお勧めします。

### SATA

mq-deadlineスケジューラアルゴリズムはSATAディスクに適しています。

この項目を一時的に変更します。

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

変更を永続的にするには、この項目を変更した後に次のコマンドを実行します。

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSDとNVMe

kyberスケジューラアルゴリズムはNVMeまたはSSDディスクに適しています。

この項目を一時的に変更します。

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

システムがSSDとNVMeのkyberスケジューラをサポートしていない場合は、none（またはnoop）スケジューラを使用することをお勧めします。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

変更を永続的にするには、この項目を変更した後に次のコマンドを実行します。

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

ファイアウォールが有効な場合は、FEノード、BEノード、およびBrokerの内部ポートを開放します。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG変数

次のコマンドを実行して、LANG変数を手動で確認および設定します。

```Bash
echo "export LANG=en_US.UTF-8" >> /etc/profile
source /etc/profile
```

## タイムゾーン

実際のタイムゾーンに合わせてこの項目を設定してください。

次の例では、タイムゾーンを `/Asia/Shanghai` に設定します。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock -w
```

## ulimit設定

StarRocksでは、**最大ファイルディスクリプタ数**と**最大ユーザプロセス数**の値が異常に小さい場合に問題が発生することがあります。

### 最大ファイルディスクリプタ数

次のコマンドを実行して、ファイルディスクリプタの最大数を設定します。

```Bash
ulimit -n 655350
```

### 最大ユーザプロセス数

次のコマンドを実行して、ユーザプロセスの最大数を設定します。

```Bash
ulimit -u 40960
```

## ファイルシステム設定

ext4またはxfsジャーナリングファイルシステムの使用をお勧めします。次のコマンドを実行して、マウントタイプを確認できます。

```Bash
df -Th
```

## ネットワーク設定

### tcp_abort_on_overflow

デーモンが処理できない新しい接続試行でシステムが現在オーバーフローしている場合、システムが新しい接続をリセットできるようにします。

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

リッスンしているソケットにキューに入れられる接続要求の最大数を`1024`に指定します。

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTP設定

StarRocksクラスタ内のノード間で時刻同期を構成し、トランザクションの線形一貫性を確保する必要があります。pool.ntp.orgが提供するインターネットタイムサービスを使用するか、オフライン環境で構築したNTPサービスを使用できます。例えば、クラウドサービスプロバイダーが提供するNTPサービスを使用することができます。

1. NTPタイムサーバーが存在するかどうかを確認します。

   ```Bash
   rpm -qa | grep ntp
   ```

2. NTPサービスがない場合はインストールします。

   ```Bash
   sudo yum install ntp ntpdate -y && \
   sudo systemctl start ntpd.service && \
   sudo systemctl enable ntpd.service
   ```

3. NTPサービスを確認します。

   ```Bash
   systemctl list-unit-files | grep ntp
   ```

4. NTPサービスの接続性と監視状態を確認します。

   ```Bash
   netstat -tlunp | grep ntp
   ```

5. アプリケーションがNTPサーバーと同期しているかどうかを確認します。

   ```Bash
   ntpstat
   ```

6. ネットワーク内の全ての設定されたNTPサーバーの状態をチェックします。

   ```Bash
   ntpq -p
   ```

## 高並行性の設定

StarRocksクラスターに高い負荷の並行性がある場合、以下の設定を行うことを推奨します：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```

