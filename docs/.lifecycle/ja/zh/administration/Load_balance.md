---
displayed_sidebar: Chinese
---

# ロードバランシング

この記事では、複数の FE ノードの上にロードバランシング層をデプロイして StarRocks の高可用性を実現する方法について説明します。

## コードによるロードバランシング

アプリケーション層のコードでリトライとロードバランシングを行うことができます。特定の接続がダウンした場合、コードはシステムが他の接続で自動的にリトライするように制御する必要があります。この方法を使用する場合、複数の StarRocks フロントエンドノードのアドレスを設定する必要があります。

## JDBC Connector によるロードバランシング

MySQL JDBC Connector を使用して StarRocks に接続する場合、JDBC の自動リトライ機能を利用してリトライとロードバランシングを行うことができます。

```sql
jdbc:mysql:loadbalance://[host1][:port],[host2][:port][,[host3][:port]]...[/[database]][?propertyName1=propertyValue1[&propertyName2=propertyValue2]...]
```

## ProxySQL によるロードバランシング

ProxySQL は、柔軟で強力な MySQL プロキシレイヤーで、リードライト分離、クエリルーティング、SQL キャッシュ、動的な設定のロード、フェイルオーバー、SQL フィルタリングなどの機能を実現できます。

StarRocks の FE プロセスは、ユーザーの接続とクエリリクエストを受け付ける責任があり、それ自体が水平方向にスケーリング可能で、高可用性クラスタとしてデプロイすることができます。自動的な接続ロードバランシングを実現するために、複数の FE ノード上にプロキシ層を設置する必要があります。

1. 関連する依存関係をインストールします。

    ```shell
    yum install -y gnutls perl-DBD-MySQL perl-DBI perl-devel
    ```

2. ProxySQL のインストールパッケージをダウンロードして解凍します。

    ```shell
    wget https://github.com/sysown/proxysql/releases/download/v2.0.14/proxysql-2.0.14-1-centos7.x86_64.rpm
    rpm2cpio proxysql-2.0.14-1-centos7.x86_64.rpm | cpio -ivdm
    ```

    > 説明：他のバージョンの ProxySQL をダウンロードすることもできます。

3. 設定ファイル **/etc/proxysql.cnf** を編集します。

    以下の設定項目をアクセス権を持つディレクトリ（絶対パス）に変更します。

    ```plain text
    datadir = "/var/lib/proxysql"
    errorlog = "/var/lib/proxysql/proxysql.log"
    ```

4. ProxySQL を起動します。

    ```shell
    ./usr/bin/proxysql -c ./etc/proxysql.cnf --no-monitor
    ```

5. StarRocks にログインします。

    ```shell
    mysql -u admin -padmin -h 127.0.0.1 -P6032
    ```

    > 注意：
    >
    > ProxySQL にはデフォルトで2つのポートが含まれており、`6032` は ProxySQL の管理ポート、`6033` は ProxySQL のトラフィック転送ポート、つまりサービス提供ポートです。

6. グローバルログを設定します。

    ```sql
    SET mysql-eventslog_filename='proxysql_queries.log';
    SET mysql-eventslog_default_log=1;
    SET mysql-eventslog_format=2;
    LOAD MYSQL VARIABLES TO RUNTIME;
    SAVE MYSQL VARIABLES TO DISK;
    ```

7. マスターノードとオブザーバーノードを挿入し、設定を読み込みます。

    ```sql
    insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.92.139', 9030);
    insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.34.139', 9030);
    insert into mysql_servers(hostgroup_id, hostname, port) values(1, '172.26.34.140', 9030);
    load mysql servers to runtime;
    save mysql servers to disk;
    ```

8. ユーザー名とパスワードを設定し、設定を読み込みます。

    ```sql
    insert into mysql_users(username, password, active, default_hostgroup, backend, frontend) 
    values('root', '*FAAFFE644E901CFAFAEC7562415E5FAEC243B8B2', 1, 1, 1, 1);
    load mysql users to runtime; 
    save mysql users to disk;
    ```

    > 注意：ここでの `password` は暗号化された値を入力します。例えば、root ユーザーのパスワードが `root123` の場合、`password` には `*FAAFFE644E901CFAFAEC7562415E5FAEC243B8B2` を入力します。`password()` 関数を使用して具体的な暗号化された値を取得できます。
    >
    > 例：
    >
    > ```plain text
    > mysql> select password('root123');
    > +---------------------------------------------+
    > | password                                   |
    > +---------------------------------------------+
    > | *FAAFFE644E901CFAFAEC7562415E5FAEC243B8B2   |
    > +---------------------------------------------+
    > 1 row in set (0.01 sec)
    > ```

9. プロキシルールを書き込み、設定を読み込みます。

    ```sql
    insert into mysql_query_rules(rule_id, active, match_digest, destination_hostgroup, mirror_hostgroup, apply) values(1, 1, '.', 1, 2, 1);
    load mysql query rules to runtime; 
    save mysql query rules to disk;
    ```

上記の手順を完了した後、ProxySQL を介して `6033` ポートでデータベース操作を行うことができます。

```shell
mysql -u admin -padmin -P6033 -h 127.0.0.1 -e"select * from db_name.table_name"
```

結果が正常に返された場合、ProxySQL を介して StarRocks に成功して接続したことを意味します。
