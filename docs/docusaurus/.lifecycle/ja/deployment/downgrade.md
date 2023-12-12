---
displayed_sidebar: "Japanese"
---

# StarRocksのダウングレード

このトピックでは、StarRocksクラスターのダウングレード方法について説明します。

StarRocksクラスターをアップグレードした後に例外が発生した場合、クラスターを早く回復させるために、それを以前のバージョンにダウングレードすることができます。

## 概要

ダウングレードする前に、このセクションの情報を確認してください。必要なアクションを実行してください。

### ダウングレードパス

- **パッチバージョンのダウングレードの場合**

  例えば、v2.2.11から直接v2.2.6にダウングレードするなど、パッチバージョンを横断してStarRocksクラスターをダウングレードすることができます。

- **マイナーバージョンのダウングレードの場合**

  互換性と安全性のため、**マイナーバージョンから別のマイナーバージョンへの連続したダウングレード**を強くお勧めします。例えば、StarRocks v2.5クラスターをv2.2にダウングレードするには、次の順序でダウングレードする必要があります: v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **メジャーバージョンのダウングレードの場合**

  StarRocks v3.0クラスターをv2.5.3以降のバージョンにのみダウングレードすることができます。

  - v3.0ではBDBライブラリがアップグレードされますが、BDBJEはロールバックできません。ダウングレード後は、v3.0のBDBライブラリを使用する必要があります。
  - 新しいRBAC特権システムがv3.0でデフォルトで使用されます。ダウングレード後にRBAC特権システムを使用することができます。

### ダウングレード手順

StarRocksのダウングレード手順は、[アップグレード手順](../deployment/upgrade.md#upgrade-procedure) の逆の順番です。そのため、**まずFEをダウングレードし、その後BEとCNをダウングレードする** 必要があります。間違った順番でダウングレードすると、FEとBE/CNの間の非互換性が発生し、サービスがクラッシュする可能性があります。FEノードの場合、リーダーFEノードをダウングレードする前に、まずすべてのフォロワーFEノードをダウングレードする必要があります。

## 開始前に

準備中に、マイナーまたはメジャーバージョンのダウングレードを行う場合は、互換性構成を実行する必要があります。また、クラスター内のすべてのノードをダウングレードする前に、1つのFEまたはBEでダウングレードの可用性テストを実行する必要があります。

### 互換性構成を実行する

StarRocksクラスターを以前のマイナーやメジャーバージョンにダウングレードする場合、互換性構成を実行する必要があります。汎用互換性設定に加えて、具体的な設定は、ダウングレード元のStarRocksクラスターバージョンによって異なります。

- **汎用互換性構成**

StarRocksクラスターをダウングレードする前に、タブレット・クローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

ダウングレード後、すべてのBEノードの状態が`Alive`になった場合は、タブレット・クローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.2およびそれ以降のバージョンからのダウングレードの場合**

FE設定項目`ignore_unknown_log_id`を`true`に設定します。これは静的パラメータですので、FE構成ファイル**fe.conf**で変更し、ノードを再起動して変更を有効にする必要があります。ダウングレードと最初のチェックポイントが完了した後、`false`にリセットしてノードを再起動できます。

- **FQDNアクセスを有効にしている場合**

FQDNアクセスを有効にしている（v2.4以降でサポート）場合、v2.4より前のバージョンにダウングレードする必要がある場合は、ダウングレードする前にIPアドレスアクセスに切り替える必要があります。詳細な手順については、[FQDNのロールバック](../administration/enable_fqdn.md#rollback) を参照してください。

## FEのダウングレード

互換性構成と可用性テストが完了したら、FEノードをダウングレードできます。まず、フォロワーFEノードをダウングレードし、次にリーダーFEノードをダウングレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードのデプロイディレクトリに置き換えます。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. **bin**、**lib**、**spark-dpp**の元のデプロイメントファイルを、以前のバージョンのファイルに置き換えます。

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
   > StarRocks v3.0 を v2.5 にダウングレードする場合、デプロイメントファイルを置き換えた後に、次の手順に従う必要があります:
   >
   > 1. v3.0のデプロイメントのファイル **fe/lib/starrocks-bdb-je-18.3.13.jar** を v2.5のデプロイメントのディレクトリ **fe/lib** にコピーします。
   > 2. ファイル **fe/lib/je-7.\*.jar** を削除します。

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を他のフォロワーFEノード、最後にリーダーFEノードのダウングレードに繰り返してください。

   > **注意**
   >
   > StarRocks v3.0 を v2.5 にダウングレードする場合、ダウングレード後に次の手順に従う必要があります:
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。
   >
   > このコマンドを実行しないと、ダウングレード操作の一部が失敗する可能性があります。ALTER SYSTEM CREATE IMAGE は、v2.5.3以降でサポートされています。

## BEのダウングレード

FEノードをダウングレードした後、クラスター内のBEノードをダウングレードできます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードのデプロイディレクトリに置き換えます。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. **bin** および **lib**の元のデプロイメントファイルを、以前のバージョンのファイルに置き換えます。

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

5. 上記の手順を他のBEノードに繰り返してください。

## CNのダウングレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードのデプロイディレクトリに置き換えます。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. **bin** および **lib**の元のデプロイメントファイルを、以前のバージョンのファイルに置き換えます。

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

5. 上記の手順を他のCNノードに繰り返してください。