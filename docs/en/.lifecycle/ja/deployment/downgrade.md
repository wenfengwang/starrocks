---
displayed_sidebar: "Japanese"
---

# StarRocksのダウングレード

このトピックでは、StarRocksクラスタをダウングレードする方法について説明します。

StarRocksクラスタをアップグレードした後に例外が発生した場合、クラスタを早く回復するために、以前のバージョンにダウングレードすることができます。

## 概要

ダウングレードする前に、このセクションの情報を確認してください。推奨されるアクションを実行してください。

### ダウングレードパス

- **パッチバージョンのダウングレードの場合**

  StarRocksクラスタをパッチバージョン間でダウングレードすることができます。たとえば、v2.2.11から直接v2.2.6にダウングレードすることができます。

- **マイナーバージョンのダウングレードの場合**

  互換性と安全性のために、StarRocksクラスタを**連続するマイナーバージョンからダウングレードすることを強くお勧めします**。たとえば、StarRocks v2.5クラスタをv2.2にダウングレードする場合、次の順序でダウングレードする必要があります：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **メジャーバージョンのダウングレードの場合**

  StarRocks v3.0クラスタをv2.5.3以降のバージョンにのみダウングレードすることができます。

  - StarRocksはv3.0でBDBライブラリをアップグレードします。ただし、BDBJEはロールバックできません。ダウングレード後は、v3.0のBDBライブラリを使用する必要があります。
  - 新しいRBAC特権システムは、v3.0にアップグレード後にデフォルトで使用されます。ダウングレード後にRBAC特権システムを使用することができます。

### ダウングレード手順

StarRocksのダウングレード手順は、[アップグレード手順](../deployment/upgrade.md#upgrade-procedure)の逆の順序です。したがって、**FEを最初にダウングレードし、次にBEとCNをダウングレードする必要があります**。間違った順序でダウングレードすると、FEとBE/CNの非互換性が生じ、サービスがクラッシュする可能性があります。FEノードの場合、リーダーFEノードをダウングレードする前に、まずすべてのフォロワーFEノードをダウングレードする必要があります。

## 開始する前に

準備中に、マイナーまたはメジャーバージョンのダウングレードを行う場合は、互換性の設定を実行する必要があります。また、クラスタ内のすべてのノードをダウングレードする前に、FEまたはBEのいずれかでダウングレードの可用性テストを実行する必要があります。

### 互換性の設定を実行する

StarRocksクラスタを以前のマイナーまたはメジャーバージョンにダウングレードする場合、互換性の設定を実行する必要があります。一般的な互換性の設定に加えて、詳細な設定はダウングレード元のStarRocksクラスタのバージョンによって異なります。

- **一般的な互換性の設定**

StarRocksクラスタをダウングレードする前に、タブレットのクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

ダウングレード後、すべてのBEノードのステータスが`Alive`になった場合は、タブレットのクローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.2以降からのダウングレードの場合**

FEの設定項目`ignore_unknown_log_id`を`true`に設定します。これは静的なパラメータなので、FEの設定ファイル**fe.conf**で変更し、ノードを再起動して変更を有効にする必要があります。ダウングレードと最初のチェックポイントが完了した後、`false`にリセットし、ノードを再起動することができます。

- **FQDNアクセスを有効にしている場合**

FQDNアクセスを有効にしている場合（v2.4以降でサポート）、v2.4より前のバージョンにダウングレードする必要がある場合は、ダウングレード前にIPアドレスアクセスに切り替える必要があります。詳細な手順については、[FQDNのロールバック](../administration/enable_fqdn.md#rollback)を参照してください。

## FEのダウングレード

互換性の設定と可用性テストが完了したら、FEノードをダウングレードすることができます。まず、フォロワーFEノードをダウングレードし、次にリーダーFEノードをダウングレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードのデプロイディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 元のデプロイファイル（**bin**、**lib**、**spark-dpp**）を以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

   > **注意**
   >
   > StarRocks v3.0をv2.5にダウングレードする場合、デプロイファイルを置き換えた後、次の手順に従う必要があります：
   >
   > 1. v3.0のデプロイファイルの**fe/lib/starrocks-bdb-je-18.3.13.jar**をv2.5のデプロイディレクトリの**fe/lib**にコピーします。
   > 2. **fe/lib/je-7.\*.jar**ファイルを削除します。

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して、他のフォロワーFEノード、最後にリーダーFEノードをダウングレードします。

   > **注意**
   >
   > StarRocks v3.0をv2.5にダウングレードする場合、ダウングレード後に次の手順に従う必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。
   >
   > このコマンドを実行しないと、一部のダウングレード操作が失敗する可能性があります。ALTER SYSTEM CREATE IMAGEは、v2.5.3以降でサポートされています。

## BEのダウングレード

FEノードをダウングレードした後、クラスタ内のBEノードをダウングレードすることができます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードのデプロイディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 元のデプロイファイル（**bin**、**lib**）を以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. BEノードを起動します。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. BEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して、他のBEノードをダウングレードします。

## CNのダウングレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードのデプロイディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 元のデプロイファイル（**bin**、**lib**）を以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. CNノードを起動します。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. CNノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep  starrocks_be
   ```

5. 上記の手順を繰り返して、他のCNノードをダウングレードします。
