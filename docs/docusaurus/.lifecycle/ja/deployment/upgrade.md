---
displayed_sidebar: "Japanese"
---

# StarRocksのアップグレード

このトピックでは、StarRocksクラスターのアップグレード方法について説明します。

## 概要

アップグレードを行う前に、このセクションの情報を確認し、推奨されるアクションを実行してください。

### StarRocksのバージョン

StarRocksのバージョンは、**Major.Minor.Patch**の形式で表されます。例えば、`2.5.4`のようです。最初の数字はStarRocksのメジャーバージョンを表し、2番目の数字はマイナーバージョンを、3番目の数字はパッチバージョンを表します。StarRocksでは、特定のバージョンに対して長期サポート（LTS）が提供されます。これらのサポート期間は半年以上続きます。

| **StarRocksのバージョン** | **LTSバージョンですか** |
| --------------------- | ---------------------- |
| v2.0.x                | いいえ                     |
| v2.1.x                | いいえ                     |
| v2.2.x                | いいえ                     |
| v2.3.x                | いいえ                     |
| v2.4.x                | いいえ                     |
| v2.5.x                | はい                    |
| v3.0.x                | いいえ                     |
| v3.1.x                | いいえ                     |

### アップグレードパス

- **パッチバージョンのアップグレード**

  StarRocksクラスターをパッチバージョンごとにアップグレードすることができます。例えば、v2.2.6から直接v2.2.11にアップグレードできます。

- **マイナーバージョンのアップグレード**

  StarRocks v2.0以降、StarRocksクラスターをマイナーバージョンごとにアップグレードすることができます。例えば、v2.2.xから直接v2.5.xにアップグレードすることができます。ただし、互換性と安全性のために、StarRocksクラスターを**連続的にマイナーバージョンから別のマイナーバージョンへ**アップグレードすることを強くお勧めします。例えば、StarRocks v2.2クラスターをv2.5にアップグレードするには、次の順序でアップグレードする必要があります：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **メジャーバージョンのアップグレード**

  StarRocksクラスターをv3.0にアップグレードするためには、まずv2.5にアップグレードする必要があります。

### アップグレード手順

StarRocksは**ローリングアップグレード**をサポートしており、これによりサービスを停止することなくクラスターをアップグレードすることができます。BEとCNはFEとの互換性があるため、**まずBEおよびCNをアップグレードし、次にFEを** アップグレードする必要があります。逆の順序でアップグレードすると、FEとBE/CNの間で互換性がなくなり、それによってサービスがクラッシュする可能性があります。FEノードの場合、Leader FEノードをアップグレードする前にすべてのFollower FEノードをまずアップグレードする必要があります。

## 開始する前に

準備中、マイナーまたはメジャーバージョンのアップグレードを行う場合は、互換性設定を行う必要があります。また、クラスター全体のノードをアップグレードする前に、FEおよびBEの1台でアップグレードの可用性テストを実行する必要があります。

### 互換性設定を実行する

StarRocksクラスターを後のマイナーまたはメジャーバージョンにアップグレードする場合は、互換性設定を行う必要があります。一般的な互換性設定に加えて、具体的な設定はアップグレード元のStarRocksクラスターのバージョンによって異なります。

- **一般的な互換性設定**

StarRocksクラスターをアップグレードする前に、タブレットクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

アップグレード後、すべてのBEノードの状態が`Alive`である場合は、タブレットクローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.0から後のバージョンにアップグレードする場合**

StarRocks v2.0クラスターをアップグレードする前に、次のBE構成およびシステム変数を設定する必要があります。

1. BE構成項目`vector_chunk_size`を変更している場合は、アップグレードする前にそれを`4096`に設定する必要があります。これは静的パラメータであるため、BE構成ファイル**be.conf**で変更を行い、変更を有効にするためにノードを再起動する必要があります。
2. システム変数`batch_size`をグローバルで`4096`以下に設定してください。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## BEをアップグレードする

アップグレードの可用性テストに合格したら、まずクラスター内のBEノードをアップグレードできます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>にBEノードの展開ディレクトリを置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 元のデプロイメントファイルを新しいバージョンのファイルで置き換えます。

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

5. 他のBEノードをアップグレードするために上記の手順を繰り返します。

## CNをアップグレードする

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>にCNノードの展開ディレクトリを置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 元のデプロイメントファイルを新しいバージョンのファイルで置き換えます。

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
   ps aux | grep starrocks_be
   ```

5. 他のCNノードをアップグレードするために上記の手順を繰り返します。

## FEをアップグレードする

すべてのBEノードおよびCNノードをアップグレードした後、FEノードをアップグレードできます。まずFollower FEノードをアップグレードし、その後Leader FEノードをアップグレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>にFEノードの展開ディレクトリを置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 元のデプロイメントファイルを新しいバージョンのファイルで置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 他のFollower FEノードをアップグレードし、最後にLeader FEノードをアップグレードします。

   > **注意**
   >
   > StarRocksクラスターをv2.5からv3.0にアップグレードした後、それを再びv3.0にアップグレードする場合、いくつかのFollower FEにメタデータのアップグレードの失敗を避けるためには、これらの手順に従う必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのFollower FEに同期されるのを待ちます。
   >
   > イメージファイルが同期されたかどうかは、Leader FEのログファイル **fe.log** を見て確認することができます。"push image.* from subdir [] to other nodes. totally xx nodes, push successful xx nodes"という記録は、イメージファイルが正常に同期されたことを示します。