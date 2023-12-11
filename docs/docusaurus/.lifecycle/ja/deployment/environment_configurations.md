---
displayed_sidebar: "Japanese"
---

# 環境構成を確認する

このトピックでは、StarRocksを展開する前に確認および設定する必要があるすべての環境およびシステム構成項目のリストが示されています。これらの構成項目を適切に設定すると、StarRocksクラスタは高い可用性とパフォーマンスで動作します。

## ポート

StarRocksは異なるサービスに特定のポートを使用します。これらのポートが他のサービスで使用されていないかどうかを確認します。

### FEポート

FE展開に使用されるインスタンスで、以下のポートを確認する必要があります。

- `8030`: FE HTTPサーバーポート (`http_port`)
- `9020`: FE Thriftサーバーポート (`rpc_port`)
- `9030`: FE MySQLサーバーポート (`query_port`)
- `9010`: FE内部通信ポート (`edit_log_port`)

以下のコマンドをFEインスタンスで実行して、これらのポートが使用されているかどうかを確認してください。

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

上記のポートのいずれかが使用されている場合、後でFEノードを展開する際に代替ポートを見つけて指定する必要があります。詳細な手順については、[StarRocksを展開する - リーダーFEノードの起動](../deployment/deploy_manually.md#step-1-start-the-leader-fe-node)を参照してください。

### BEポート

BE展開に使用されるインスタンスで、以下のポートを確認する必要があります。

- `9060`: BE Thriftサーバーポート (`be_port`)
- `8040`: BE HTTPサーバーポート (`be_http_port`)
- `9050`: BEハートビートサービスポート (`heartbeat_service_port`)
- `8060`: BE bRPCポート (`brpc_port`)

以下のコマンドをBEインスタンスで実行して、これらのポートが使用されているかどうかを確認してください。

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のポートのいずれかが使用されている場合、後でBEノードを展開する際に代替ポートを見つけて指定する必要があります。詳細な手順については、[StarRocksを展開する - BEサービスの起動](../deployment/deploy_manually.md#step-2-start-the-be-service)を参照してください。

### CNポート

CN展開に使用されるインスタンスで、以下のポートを確認する必要があります。

- `9060`: CN Thriftサーバーポート (`be_port`)
- `8040`: CN HTTPサーバーポート (`be_http_port`)
- `9050`: CNハートビートサービスポート (`heartbeat_service_port`)
- `8060`: CN bRPCポート (`brpc_port`)
- `9070`: 共有データクラスタ（v3.0ではBE）のCN用の追加エージェントサービスポート (`starlet_port`)

以下のコマンドをCNインスタンスで実行して、これらのポートが使用されているかどうかを確認してください。

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
netstat -tunlp | grep 9070
```

上記のポートのいずれかが使用されている場合、後でCNノードを展開する際に代替ポートを見つけて指定する必要があります。詳細な手順については、[StarRocksを展開する - CNサービスの起動](../deployment/deploy_manually.md#step-3-optional-start-the-cn-service)を参照してください。

## ホスト名

StarRocksクラスタで[FQDNアクセスを有効にする](../administration/enable_fqdn.md)場合、各インスタンスにホスト名を割り当てる必要があります。

各インスタンスのファイル**/etc/hosts**に、クラスタ内の他のすべてのインスタンスのIPアドレスと対応するホスト名を指定する必要があります。

> **注意**
>
> ファイル**/etc/hosts**のすべてのIPアドレスは一意である必要があります。

## JDKの構成

StarRocksはインスタンス上のJava依存性を特定するために環境変数`JAVA_HOME`に依存しています。

以下のコマンドを実行して環境変数`JAVA_HOME`を確認してください。

```Bash
echo $JAVA_HOME
```

`JAVA_HOME`を設定するには以下の手順に従ってください。

1. ファイル**/etc/profile**で`JAVA_HOME`を設定します:

   ```Bash
   sudo vi /etc/profile
   # JDKがインストールされているパスに<Java_Home>を置き換えてください。
   export JAVA_HOME=<Java_Home>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 変更を有効にします:

   ```Bash
   source /etc/profile
   ```

変更を確認するには以下のコマンドを実行してください:

```Bash
java -version
```

## CPUのスケーリングガバナー

この構成項目は**オプション**です。CPUがスケーリングガバナーをサポートしていない場合はスキップできます。

CPUスケーリングガバナーはCPUの電力モードを制御します。CPUがサポートしている場合、CPUの性能を向上させるために`performance`に設定することをお勧めします。

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## メモリ構成

### メモリオーバーコミット

メモリオーバーコミットを有効にすると、オペレーティングシステムはプロセスへのメモリリソースを過剰に割り当てることができます。メモリオーバーコミットを有効にすることをお勧めします。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### 透過的な巨大ページ

透過的な巨大ページはデフォルトで有効になっています。メモリ割り当てプログラムに干渉してパフォーマンスの低下につながる恐れがあるため、この機能を無効にすることをお勧めします。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### スワップ領域

スワップ領域を無効にすることをお勧めします。

スワップ領域を確認および無効にする手順は以下の通りです:

1. スワップ領域を無効にします。

   ```SQL
   swapoff /<path_to_swap_space>
   ```

2. 構成ファイル**/etc/fstab**からスワップ領域の情報を削除します。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. スワップ領域が無効になっていることを確認してください。

   ```Bash
   free -m
   ```

### スワップネス

スワップネスを無効にしてパフォーマンスに影響を及ぼす可能性を排除することをお勧めします。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## ストレージ構成

使用するストレージメディアに応じて適切なスケジューラーアルゴリズムを選択することをお勧めします。

使用しているスケジューラーアルゴリズムを確認するには以下のコマンドを実行できます:

```Bash
cat /sys/block/${disk}/queue/scheduler
# たとえば、cat /sys/block/vdb/queue/schedulerを実行します
```

SATAディスクにはmq-deadlineスケジューラーアルゴリズムが適しています。SSDおよびNVMeディスクにはkyberスケジューラーアルゴリズムが適しています。

### SATA

mq-deadlineスケジューラーアルゴリズムはSATAディスクに適しています。

この項目を一時的に変更するには以下のコマンドを実行します:

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

この項目を永続的に変更するには、この項目を変更した後に以下のコマンドを実行してください:

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSDおよびNVMe

kyberスケジューラーアルゴリズムはNVMeまたはSSDディスクに適しています。

この項目を一時的に変更するには以下のコマンドを実行します:

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

システムがSSDおよびNVMe用のkyberスケジューラをサポートしていない場合は、none (またはnoop)スケジューラを使用することをお勧めします。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

この項目を変更した後に以下のコマンドを実行してください:

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

ファイアウォールが有効になっている場合、FEノード、BEノード、およびブローカーの内部ポートを開いてください。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG変数

LANG変数を手動で確認および構成するには以下のコマンドを実行してください:

```Bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

## タイムゾーン

実際のタイムゾーンに応じてこの項目を設定してください。

次の例は、タイムゾーンを`/Asia/Shanghai`に設定します。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

## ulimit構成

StarRocksで**最大ファイルディスクリプタ**および**最大ユーザープロセス**の値が異常に小さい場合、問題が発生する可能性があります。

### 最大ファイルディスクリプタ

以下のコマンドを実行して最大ファイルディスクリプタの数を設定できます:

```Bash
ulimit -n 655350
```

### 最大ユーザープロセス

以下のコマンドを実行して最大ユーザープロセスの数を設定できます:
```
ulimit -u 40960
```

## File system configuration

ext4またはxfsのジャーナリングファイルシステムを使用することをお勧めします。マウントタイプを確認するには、次のコマンドを実行できます：

```Bash
df -Th
```

## Network configuration

### tcp_abort_on_overflow

システムが新しい接続要求を処理できないほど過負荷になっている場合、新しい接続をリセットするようにシステムを許可します：

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

任意のリスニングソケットに対してキューイングされる接続要求数の最大を`1024`に指定します：

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTP configuration

StarRocksクラスタ内のノード間で時間同期を構成する必要があり、トランザクションの線形の整合性を確保するためです。pool.ntp.orgが提供するインターネット時刻サービスを使用するか、オフライン環境に組み込まれたNTPサービスを使用することができます。たとえば、クラウドサービスプロバイダーが提供するNTPサービスを使用できます。

1. NTP時刻サーバーが存在するかどうかを確認します。

   ```Bash
   rpm -qa | grep ntp
   ```

2. サービスが存在しない場合は、NTPサービスをインストールします。

   ```Bash
   sudo yum install ntp ntpdate && \
   sudo systemctl start ntpd.service && \
   sudo systemctl enable ntpd.service
   ```

3. NTPサービスを確認します。

   ```Bash
   systemctl list-unit-files | grep ntp
   ```

4. NTPサービスの接続性とモニタリング状況を確認します。

   ```Bash
   netstat -tlunp | grep ntp
   ```

5. アプリケーションがNTPサーバーと同期しているかどうかを確認します。

   ```Bash
   ntpstat
   ```

6. ネットワーク内のすべての構成されたNTPサーバーの状態を確認します。

   ```Bash
   ntpq -p
   ```

## High concurrency configurations

StarRocksクラスタが高い負荷並列処理を持つ場合、以下の構成を設定することをお勧めします：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```