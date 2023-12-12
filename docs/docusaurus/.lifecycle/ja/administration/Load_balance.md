---
displayed_sidebar: "Japanese"
---

# ロードバランシング

複数のFEノードを展開する場合、ユーザーは高可用性を実現するためにFEの上にロードバランシングレイヤーを展開することができます。

以下はいくつかの高可用性オプションです：

## コードアプローチ

1つの方法は、アプリケーションレイヤーで再試行とロードバランシングを実行するためのコードを実装することです。たとえば、接続が切断された場合、他の接続で自動的に再試行します。このアプローチでは、複数のFEノードアドレスを構成する必要があります。

## JDBCコネクタ

JDBCコネクタは自動再試行をサポートしています：

~~~sql
jdbc:mysql:loadbalance://[host1][:port],[host2][:port][,[host3][:port]]...[/[database]][?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]
~~~

## ProxySQL

ProxySQLは、読み書きの分離、クエリのルーティング、SQLのキャッシュ、動的なロード構成、フェイルオーバー、およびSQLフィルタリングをサポートするMySQLプロキシレイヤーです。

StarRocks FEは接続とクエリのリクエストを受け取る責任があり、水平方向に拡張可能で高可用性です。しかし、FEでは自動的なロードバランシングを実現するために、その上にプロキシレイヤーを設定する必要があります。セットアップの手順については以下を参照してください：

### 1. 関連する依存関係のインストール

~~~shell
yum install -y gnutls perl-DBD-MySQL perl-DBI perl-devel
~~~

### 2. インストールパッケージのダウンロード

~~~shell
wget https://github.com/sysown/proxysql/releases/download/v2.0.14/proxysql-2.0.14-1-centos7.x86_64.rpm
~~~

### 3. 現在のディレクトリに展開する

~~~shell
rpm2cpio proxysql-2.0.14-1-centos7.x86_64.rpm | cpio -ivdm
~~~

### 4. 設定ファイルの変更

~~~shell
vim ./etc/proxysql.cnf 
~~~

ユーザーがアクセス権限を持っているディレクトリに移動します（絶対パス）：

~~~vim
datadir="/var/lib/proxysql"
errorlog="/var/lib/proxysql/proxysql.log"
~~~

### 5. 開始

~~~shell
./usr/bin/proxysql -c ./etc/proxysql.cnf --no-monitor
~~~

### 6. ログイン

~~~shell
mysql -u admin -padmin -h 127.0.0.1 -P6032
~~~

### 7. グローバルログの設定

~~~shell
SET mysql-eventslog_filename='proxysql_queries.log';
SET mysql-eventslog_default_log=1;
SET mysql-eventslog_format=2;
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
~~~

### 8. リーダーノードに挿入

~~~sql
insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.92.139', 8533);
~~~

### 9. オブザーバーノードを挿入

~~~sql
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.139', 9931);
insert into mysql_servers(hostgroup_id, hostname, port) values(2, '172.26.34.140', 9931);
~~~

### 10. 設定の読み込み

~~~sql
load mysql servers to runtime;
save mysql servers to disk;
~~~

### 11. ユーザー名とパスワードの設定

~~~sql
insert into mysql_users(username, password, active, default_hostgroup, backend, frontend) values('root', '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29', 1, 1, 1, 1);
~~~

### 12. 設定の読み込み

~~~sql
load mysql users to runtime; 
save mysql users to disk;
~~~

### 13. プロキシルールへの書き込み

~~~sql
insert into mysql_query_rules(rule_id, active, match_digest, destination_hostgroup, mirror_hostgroup, apply) values(1, 1, '.', 1, 2, 1);
~~~

### 14. 設定の読み込み

~~~sql
load mysql query rules to runtime; 
save mysql query rules to disk;
~~~