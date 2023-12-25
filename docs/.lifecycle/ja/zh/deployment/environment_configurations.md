---
displayed_sidebar: Chinese
---
# 環境設定の確認

このドキュメントでは、StarRocksをデプロイする前に確認して設定する必要があるすべての環境とシステム設定項目をリストアップしています。これらの設定項目を正しく設定することで、クラスタの高可用性が確保され、パフォーマンスが向上します。

## ポート

StarRocksは異なるサービスに特定のポートを使用します。これらのインスタンスに他のサービスをデプロイしている場合は、これらのポートが使用されていないかどうかを確認してください。

### FEポート

FEデプロイ用のインスタンスでは、次のポートを確認する必要があります：

- `8030`：FE HTTPサーバーポート（`http_port`）
- `9020`：FE Thriftサーバーポート（`rpc_port`）
- `9030`：FE MySQLサーバーポート（`query_port`）
- `9010`：FE内部通信ポート（`edit_log_port`）

FEインスタンスで次のコマンドを実行してこれらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

上記のいずれかのポートが使用されている場合、FEノードのデプロイ時に置き換えに使用できるポートを指定する必要があります。詳細な説明については、[手動でStarRocksをデプロイする-リーダーFEノードを起動する](../deployment/deploy_manually.md#第一步启动-leader-fe-节点)を参照してください。

### BEポート

BEデプロイ用のインスタンスでは、次のポートを確認する必要があります：

- `9060`：BE Thriftサーバーポート（`be_port`）
- `8040`：BE HTTPサーバーポート（`be_http_port`）
- `9050`：BEハートビートサービスポート（`heartbeat_service_port`）
- `8060`：BE bRPCポート（`brpc_port`）

BEインスタンスで次のコマンドを実行してこれらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のいずれかのポートが使用されている場合、BEノードのデプロイ時に置き換えに使用できるポートを指定する必要があります。詳細な説明については、[StarRocksをデプロイする- BEサービスを起動する](../deployment/deploy_manually.md#第二步启动-be-服务)を参照してください。

### CNポート

CNデプロイ用のインスタンスでは、次のポートを確認する必要があります：

- `9060`：CN Thriftサーバーポート（`be_port`）
- `8040`：CN HTTPサーバーポート（`be_http_port`）
- `9050`：CNハートビートサービスポート（`heartbeat_service_port`）
- `8060`：CN bRPCポート（`brpc_port`）
- `9070`：CN（v3.0のBE）の追加エージェントサービスポート（ストアと計算の分離クラスタ）（`starlet_port`）

CNインスタンスで次のコマンドを実行してこれらのポートが使用されているかどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のいずれかのポートが使用されている場合、CNノードのデプロイ時に置き換えに使用できるポートを指定する必要があります。詳細な説明については、[StarRocksをデプロイする- CNサービスを起動する](../deployment/deploy_manually.md#第三步可选启动-cn-服务)を参照してください。

## ホスト名

StarRocksクラスタで[FQDNアクセスを有効にする](../administration/enable_fqdn.md)場合は、各インスタンスにホスト名を設定する必要があります。

各インスタンスの **/etc/hosts** ファイルで、クラスタ内の他のインスタンスのIPアドレスと対応するホスト名を指定する必要があります。

> **注意**
>
> **/etc/hosts** ファイルのすべてのIPアドレスは一意である必要があります。

## JDKの設定

StarRocksは、インスタンス上のJava依存関係を特定するために環境変数 `JAVA_HOME` に依存しています。

次のコマンドを実行して環境変数 `JAVA_HOME` を確認します：

```Bash
echo $JAVA_HOME
```

次の手順に従って `JAVA_HOME` を設定します：

1. **/etc/profile** ファイルで `JAVA_HOME` を設定します：

   ```Bash
   sudo vi /etc/profile
   # JDKがインストールされているパスで <path_to_JDK> を置き換えてください。
   export JAVA_HOME=<path_to_JDK>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 変更を有効にします：

   ```Bash
   source /etc/profile
   ```

次のコマンドを実行して変更が正常に適用されたかどうかを確認します：

```Bash
java -version
```

## CPUスケーリングガバナー

この設定項目は**オプションの設定項目**です。CPUがScaling Governorをサポートしていない場合は、この項目をスキップできます。

CPUスケーリングガバナーは、CPUの電力消費モードを制御するために使用されます。CPUがこの設定項目をサポートしている場合は、より良いCPUパフォーマンスを得るために `performance` に設定することをお勧めします：

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## メモリの設定

### メモリオーバーコミット

メモリオーバーコミットにより、オペレーティングシステムはプロセスに追加のメモリリソースを割り当てることができます。メモリオーバーコミットを有効にすることをお勧めします。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### 透過的なヒュージページ

Transparent Huge Pagesはデフォルトで有効になっています。これはメモリの割り当てに干渉し、パフォーマンスの低下を引き起こすため、この機能を無効にすることをお勧めします。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### スワップスペース

スワップスペースを無効にすることをお勧めします。

スワップスペースを確認して無効にする手順は次のとおりです：

1. スワップスペースを無効にします。

   ```SQL
   swapoff /<path_to_swap_space>
   ```

2. **/etc/fstab** ファイルからスワップスペースの情報を削除します。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. スワップスペースが無効になっていることを確認します。

   ```Bash
   free -m
   ```

### スワッピネス

スワッピネスはパフォーマンスに影響を与えるため、スワッピネスを無効にすることをお勧めします。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## ストレージの設定

選択したストレージメディアに応じて適切なスケジューリングアルゴリズムを決定することをお勧めします。

現在使用しているスケジューリングアルゴリズムを確認するには、次のコマンドを使用できます：

```Bash
cat /sys/block/${disk}/queue/scheduler
# 例：cat /sys/block/vdb/queue/scheduler を実行します
```

SATAディスクにはmq-deadlineスケジューリングアルゴリズムを、NVMeまたはSSDディスクにはkyberスケジューリングアルゴリズムを使用することをお勧めします。

### SATA

mq-deadlineスケジューリングアルゴリズムはSATAディスクに適しています。

この設定を一時的に変更するには：

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

変更を永続的に有効にするには、この設定を変更した後に次のコマンドを実行します：

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSDおよびNVMe

kyberスケジューリングアルゴリズムはNVMeまたはSSDディスクに適しています。

この設定を一時的に変更するには：

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

システムがSSDおよびNVMeのkyberスケジューリングアルゴリズムをサポートしていない場合は、none（またはnoop）スケジューリングアルゴリズムを使用することをお勧めします。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

変更を永続的に有効にするには、この設定を変更した後に次のコマンドを実行します：

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

ファイアウォールを有効にしている場合は、FE、BE、およびBrokerの内部ポートを開く必要があります。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG変数

次のコマンドを使用してLANG変数を手動で確認および設定する必要があります：

```Bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

## タイムゾーン

実際のタイムゾーンに基づいてこの設定を行ってください。

以下の例では、タイムゾーンを `/Asia/Shanghai` に設定しています。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

## ulimitの設定

**最大ファイルディスクリプタ**と**最大ユーザープロセス**の値が小さすぎる場合、StarRocksの実行に問題が発生する可能性があります。

### 最大ファイルディスクリプタ

次のコマンドを使用して最大ファイルディスクリプタ数を設定できます：

```Bash
ulimit -n 655350
```

### 最大ユーザープロセス

次のコマンドを使用して最大ユーザープロセス数を設定できます：

```Bash
ulimit -u 40960
```

## ファイルシステムの設定

ext4またはXFSジャーナルファイルシステムを使用することをお勧めします。マウントタイプを確認するには、次のコマンドを実行できます：

```Bash
df -Th
```

## ネットワークの設定

### tcp_abort_on_overflow

バックグラウンドプロセスが処理できない新しい接続がオーバーフローした場合、システムが新しい接続をリセットできるようにします：

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

リッスンソケットキューの最大接続要求数を `1024` に設定します：

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTPの設定

トランザクションの一貫性を確保するために、StarRocksクラスタの各ノード間で時間同期を設定する必要があります。インターネット時間サービスであるpool.ntp.orgを使用するか、オフライン環境に組み込まれたNTPサービスを使用することができます。たとえば、クラウドサービスプロバイダが提供するNTPサービスを使用できます。

1. NTPタイムサーバーが存在するかどうかを確認します。

   ```Bash
   rpm -qa | grep ntp
   ```

2. 存在しない場合は、次のコマンドを実行してNTPタイムサーバーをインストールします。

   ```Bash
   sudo yum install ntp ntpdate && \
   sudo systemctl start ntpd.service && \
   sudo systemctl enable ntpd.service
   ```

3. NTPサービスを確認します。

   ```Bash
   systemctl list-unit-files | grep ntp
   ```

4. NTPサービスの接続と監視の状態を確認します。

   ```Bash
   netstat -tlunp | grep ntp
   ```

5. サービスがNTPサーバーと同期しているかどうかを確認します。

   ```Bash
   ntpstat
   ```

6. ネットワーク内のNTPサーバーを確認します。

   ```Bash
   ntpq -p
   ```

## 高並行設定

StarRocksクラスタの負荷が高い場合は、次の設定を行うことをお勧めします：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```

```