---
displayed_sidebar: "Japanese"
---

# 環境設定の確認

このトピックでは、StarRocksをデプロイする前に確認して設定する必要のあるすべての環境とシステムの設定項目をリストアップしています。これらの設定項目を適切に設定することで、StarRocksクラスタは高可用性とパフォーマンスを持つようになります。

## ポート

StarRocksは異なるサービスに特定のポートを使用します。これらのポートが他のサービスと競合していないか、各インスタンスで確認してください。

### FEポート

FEデプロイメントに使用するインスタンスでは、次のポートを確認する必要があります：

- `8030`：FE HTTPサーバーポート（`http_port`）
- `9020`：FE Thriftサーバーポート（`rpc_port`）
- `9030`：FE MySQLサーバーポート（`query_port`）
- `9010`：FE内部通信ポート（`edit_log_port`）

FEインスタンスで以下のコマンドを実行して、これらのポートが使用中かどうかを確認します：

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

上記のポートのいずれかが使用中の場合は、代替ポートを見つけて後でFEノードをデプロイする際に指定する必要があります。詳細な手順については、[StarRocksのデプロイ - リーダーFEノードの起動](../deployment/deploy_manually.md#step-1-start-the-leader-fe-node)を参照してください。

### BEポート

BEデプロイメントに使用するインスタンスでは、次のポートを確認する必要があります：

- `9060`：BE Thriftサーバーポート（`be_port`）
- `8040`：BE HTTPサーバーポート（`be_http_port`）
- `9050`：BEハートビートサービスポート（`heartbeat_service_port`）
- `8060`：BE bRPCポート（`brpc_port`）

BEインスタンスで以下のコマンドを実行して、これらのポートが使用中かどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のポートのいずれかが使用中の場合は、代替ポートを見つけて後でBEノードをデプロイする際に指定する必要があります。詳細な手順については、[StarRocksのデプロイ - BEサービスの起動](../deployment/deploy_manually.md#step-2-start-the-be-service)を参照してください。

### CNポート

CNデプロイメントに使用するインスタンスでは、次のポートを確認する必要があります：

- `9060`：CN Thriftサーバーポート（`be_port`）
- `8040`：CN HTTPサーバーポート（`be_http_port`）
- `9050`：CNハートビートサービスポート（`heartbeat_service_port`）
- `8060`：CN bRPCポート（`brpc_port`）

CNインスタンスで以下のコマンドを実行して、これらのポートが使用中かどうかを確認します：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

上記のポートのいずれかが使用中の場合は、代替ポートを見つけて後でCNノードをデプロイする際に指定する必要があります。詳細な手順については、[StarRocksのデプロイ - CNサービスの起動（オプション）](../deployment/deploy_manually.md#step-3-optional-start-the-cn-service)を参照してください。

## ホスト名

StarRocksクラスタで[FQDNアクセスを有効にする](../administration/enable_fqdn.md)場合は、各インスタンスにホスト名を割り当てる必要があります。

各インスタンスのファイル**/etc/hosts**に、クラスタ内の他のすべてのインスタンスのIPアドレスと対応するホスト名を指定する必要があります。

> **注意**
>
> ファイル**/etc/hosts**内のすべてのIPアドレスは一意である必要があります。

## JDKの設定

StarRocksは、インスタンス上のJava依存関係を検出するために環境変数`JAVA_HOME`を使用します。

環境変数`JAVA_HOME`を確認するには、次のコマンドを実行します：

```Bash
echo $JAVA_HOME
```

次の手順に従って`JAVA_HOME`を設定します：

1. ファイル**/etc/profile**で`JAVA_HOME`を設定します：

   ```Bash
   sudo vi /etc/profile
   # <path_to_JDK>をJDKがインストールされているパスに置き換えてください。
   export JAVA_HOME=<path_to_JDK>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 変更を有効にします：

   ```Bash
   source /etc/profile
   ```

変更が正しく反映されたかを確認するには、次のコマンドを実行します：

```Bash
java -version
```

## CPUスケーリングガバナー

この設定項目は**オプション**です。CPUがスケーリングガバナーをサポートしていない場合は、スキップしてください。

CPUスケーリングガバナーは、CPUのパワーモードを制御します。CPUがサポートしている場合は、CPUのパフォーマンスを向上させるために`performance`に設定することをおすすめします：

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## メモリ設定

### メモリオーバーコミット

メモリオーバーコミットは、オペレーティングシステムがプロセスにメモリリソースをオーバーコミットすることを許可します。メモリオーバーコミットを有効にすることをおすすめします。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### 透過的な巨大ページ

透過的な巨大ページはデフォルトで有効になっています。メモリアロケータと干渉する可能性があるため、パフォーマンスの低下を引き起こす可能性があるため、この機能を無効にすることをおすすめします。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### スワップスペース

スワップスペースを無効にすることをおすすめします。

次の手順に従ってスワップスペースを確認して無効にします：

1. スワップスペースを無効にします。

   ```SQL
   swapoff /<path_to_swap_space>
   ```

2. 設定ファイル**/etc/fstab**からスワップスペースの情報を削除します。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. スワップスペースが無効になっていることを確認します。

   ```Bash
   free -m
   ```

### スワッピネス

パフォーマンスに対する影響を排除するため、スワッピネスを無効にすることをおすすめします。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## ストレージ設定

使用するストレージメディアに応じて適切なスケジューラアルゴリズムを選択することをおすすめします。

次のコマンドを実行して、使用しているスケジューラアルゴリズムを確認できます：

```Bash
cat /sys/block/${disk}/queue/scheduler
# 例：cat /sys/block/vdb/queue/schedulerを実行します
```

SATAディスクにはmq-deadlineスケジューラアルゴリズムが適しています。SSDおよびNVMeディスクにはkyberスケジューラアルゴリズムが適しています。

### SATA

mq-deadlineスケジューラアルゴリズムはSATAディスクに適しています。

この項目を一時的に変更するには、次のコマンドを実行します：

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

この項目を変更した後、次のコマンドを実行して変更を永続化します：

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSDおよびNVMe

kyberスケジューラアルゴリズムはNVMeまたはSSDディスクに適しています。

この項目を一時的に変更するには、次のコマンドを実行します：

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

システムがSSDおよびNVMeのkyberスケジューラをサポートしていない場合は、none（またはnoop）スケジューラを使用することをおすすめします。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

この項目を変更した後、次のコマンドを実行して変更を永続化します：

```Bash
chmod +x /etc/rc.d/rc.local
```

## SELinux

SELinuxを無効にすることをおすすめします。

```Bash
sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sed -i 's/SELINUXTYPE/#SELINUXTYPE/' /etc/selinux/config
setenforce 0 
```

## ファイアウォール

ファイアウォールが有効な場合は、FEノード、BEノード、およびBrokerの内部ポートを開く必要があります。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG変数

LANG変数を確認および手動で設定するには、次のコマンドを実行します：

```Bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

## タイムゾーン

実際のタイムゾーンに応じてこの項目を設定します。

次の例では、タイムゾーンを`/Asia/Shanghai`に設定しています。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

## ulimit設定

**最大ファイルディスクリプタ数**と**最大ユーザープロセス数**の値が異常に小さい場合、StarRocksで問題が発生する可能性があります。

### 最大ファイルディスクリプタ数

次のコマンドを実行して、最大ファイルディスクリプタ数を設定できます：

```Bash
ulimit -n 655350
```

### 最大ユーザープロセス数

次のコマンドを実行して、最大ユーザープロセス数を設定できます：

```Bash
ulimit -u 40960
```

## ファイルシステムの設定

ext4またはxfsジャーナリングファイルシステムを使用することをおすすめします。マウントタイプを確認するには、次のコマンドを実行します：

```Bash
df -Th
```

## ネットワークの設定

### tcp_abort_on_overflow

システムが現在新しい接続のオーバーフローであふれている場合、新しい接続をリセットするようにシステムに許可します：

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

リスニングソケットに対してキューに入ることができる接続要求の最大数を`1024`に指定します：

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTPの設定

トランザクションの線形一貫性を確保するために、StarRocksクラスタ内のノード間で時刻同期を設定する必要があります。pool.ntp.orgが提供するインターネット時刻サービスを使用するか、オフライン環境に組み込まれたNTPサービスを使用することができます。たとえば、クラウドサービスプロバイダが提供するNTPサービスを使用することができます。

1. NTPタイムサーバーが存在するかどうかを確認します。

   ```Bash
   rpm -qa | grep ntp
   ```

2. NTPサービスが存在しない場合は、NTPサービスをインストールします。

   ```Bash
   sudo yum install ntp ntpdate && \
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

6. ネットワーク内のすべての設定済みNTPサーバーの状態を確認します。

   ```Bash
   ntpq -p
   ```

## 高並行性の設定

StarRocksクラスタの負荷が高い場合は、次の設定を行うことをおすすめします：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```
