---
displayed_sidebar: Chinese
---

# FQDN アクセスの有効化

この記事では、StarRocks ノードへの完全修飾ドメイン名（fully qualified domain name、FQDN）に基づくアクセスを有効にする方法について説明します。FQDN は、インターネット上でアクセス可能な特定のエンティティの**完全なドメイン名**を指します。FQDN はホスト名とドメイン名の二つの部分で構成されています。

StarRocks v2.4 以前のバージョンでは、各ノードへのアクセスは IP アドレスを通じてのみサポートされていました。FQDN を使用してクラスタにノードを追加しても、その FQDN は強制的に IP アドレスに変換されていました。クラスタ内の一部のノードの IP アドレスを変更すると、そのノードにアクセスできなくなる可能性があります。v2.4 で、StarRocks は各ノードをその IP アドレスから切り離しました。FQDN アクセスを有効にすると、ノードの FQDN を直接使用して対応するノードを管理できます。

## 前提条件

StarRocks クラスタで FQDN アクセスを有効にするには、以下の要件を満たしている必要があります：

- クラスタ内のすべてのマシンにはホスト名が設定されている必要があります。

- クラスタ内の各マシンの **/etc/hosts** ファイルには、他のマシンの IP アドレスと FQDN を指定する必要があります。

- **/etc/hosts** ファイルには重複する IP アドレスが存在してはなりません。

## 新しいクラスタでの FQDN アクセスの有効化

新しいクラスタの FE ノードは、起動時にデフォルトで IP アドレスアクセスが有効になっています。FQDN アクセスを有効にした新しいクラスタをデプロイする場合は、**初回のクラスタ起動時**に以下のコマンドで FE ノードを起動するだけです：

```Shell
sh bin/start_fe.sh --host_type FQDN --daemon
```

`--host_type` 属性は、そのノードのアクセス方法を指定するために使用されます。有効な値には `FQDN` と `IP` があります。この属性はノードを初めて起動する時にのみ指定する必要があります。

StarRocks のデプロイ方法の詳細については、[StarRocks の手動デプロイ](../deployment/deploy_manually.md)を参照してください。

新しいクラスタの BE ノードは、FE のメタデータで定義された `BE Address` に基づいて自身を FQDN または IP アドレスで識別するため、BE ノードを起動する際に `--host_type` を指定する必要はありません。例えば、`BE Address` に BE ノードの FQDN が記録されている場合、その BE ノードはこの FQDN を使用して自身を識別します。

## 既存のクラスタでの FQDN アクセスの有効化

既存のクラスタで FQDN アクセスを有効にするには、まずクラスタを v2.4.0 またはそれ以上のバージョンに**アップグレード**する必要があります。

### FE ノードでの FQDN アクセスの有効化

FE ノードで FQDN アクセスを有効にする場合、まずすべての Follower FE ノードを変更し、最後に Leader FE ノードを変更する必要があります。

> **注意**
>
> クラスタに少なくとも 3 つの Follower FE ノードがない場合は、FE ノードで FQDN アクセスを有効にすることはできません。

#### Follower FE ノードの変更

Leader FE ノードを変更する前に、すべての Follower FE ノードを最初に変更してください。

1. FE ノードのデプロイディレクトリに入り、以下のコマンドを実行して FE ノードを停止します。

    ```Shell
    sh bin/stop_fe.sh --daemon
    ```

2. MySQL クライアントを使用して以下のステートメントを実行し、FE ノードの `Alive` ステータスが `false` になるまで確認します。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

3. 以下のステートメントを実行して、FE ノードの IP アドレスを FQDN に変更します。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 以下のコマンドを実行して FE ノードを起動します。

    ```Shell
    sh bin/start_fe.sh --host_type FQDN --daemon
    ```

    `--host_type` 属性は、そのノードのアクセス方法を指定するために使用されます。有効な値には `FQDN` と `IP` があります。この属性は変更後の初回起動時にのみ指定する必要があります。

5. FE ノードの `Alive` ステータスが `true` になるまで確認します。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

6. 現在の FE ノードの `Alive` ステータスが `true` になったら、上記の手順を繰り返して他の Follower FE ノードで FQDN アクセスを有効にします。

#### Leader FE ノードの変更

すべての Follower FE ノードを変更し、正常に再起動した後、Leader FE ノードで FQDN アクセスを有効にすることができます。

> **説明**
>
> Leader FE ノードで FQDN アクセスを有効にする前に、新しいノードを追加するために使用される FQDN は依然として IP アドレスに変換されます。FQDN アクセスが有効な Leader FE ノードが再選出された後、新しいノードを追加するための FQDN は IP アドレスに変換されなくなります。

1. Leader FE ノードのデプロイディレクトリに入り、以下のコマンドを実行してノードを停止します。

    ```Shell
    sh bin/stop_fe.sh --daemon
    ```

2. 以下のステートメントを実行して、クラスタが新しい Leader FE ノードを選出したかどうかを確認します。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    `Alive` と `Role` が `LEADER` である任意の FE ノードが正常に動作している Leader FE ノードです。

3. 以下のステートメントを実行して、FE ノードの IP アドレスを FQDN に変更します。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 以下のコマンドを実行して FE ノードを起動します。

    ```Shell
    sh bin/start_fe.sh --host_type FQDN --daemon
    ```

    `--host_type` 属性は、そのノードのアクセス方法を指定するために使用されます。有効な値には `FQDN` と `IP` があります。この属性は変更後の初回起動時にのみ指定する必要があります。

5. FE ノードの `Alive` ステータスが `true` になるまで確認します。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    `Alive` ステータスが `true` になった場合、その FE ノードは正常に変更され、Follower FE ノードとしてクラスタに追加されました。

### BE ノードでの FQDN アクセスの有効化

以下のステートメントを実行して、BE ノードで FQDN アクセスを有効にします。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **説明**
>
> BE ノードで FQDN アクセスを有効にした後、その BE ノードを再起動する必要はありません。

## ロールバック

FQDN アクセスを有効にした StarRocks クラスタを、FQDN アクセスをサポートしていない以前のバージョンにロールバックする場合は、まずクラスタ内のすべてのノードを IP アドレスアクセスに変更する必要があります。[既存のクラスタでの FQDN アクセスの有効化](#既存のクラスタでの-fqdn-アクセスの有効化)の手順を参照し、以下のコマンドに変更してください：

- FE ノードで IP アドレスアクセスを有効にする：

```SQL
ALTER SYSTEM MODIFY FRONTEND HOST "<fe_hostname>" TO "<fe_ip>";
```

- BE ノードで IP アドレスアクセスを有効にする：

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_hostname>" TO "<be_ip>";
```

変更が完了したら、変更を有効にするためにクラスタを再起動する必要があります。

## よくある質問

**Q: FE ノードで FQDN アクセスを有効にした際にエラーが発生しました："required 1 replica. But none were active with this master"。どう対処すれば良いですか？**

A: クラスタに少なくとも 3 つの Follower FE ノードがあることを確認してください。そうでない場合、FE ノードで FQDN アクセスを有効にすることはできません。

**Q: FQDN アクセスを有効にしたクラスタに新しいノードを IP アドレスを使って追加することはできますか？**

A: はい、できます。
