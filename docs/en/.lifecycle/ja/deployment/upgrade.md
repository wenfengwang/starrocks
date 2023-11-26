---
displayed_sidebar: "Japanese"
---

# StarRocksのアップグレード

このトピックでは、StarRocksクラスタのアップグレード方法について説明します。

## 概要

アップグレード前に、このセクションの情報を確認し、推奨されるアクションを実行してください。

### StarRocksのバージョン

StarRocksのバージョンは、**Major.Minor.Patch**の形式で表されます。たとえば、`2.5.4`です。最初の数字はStarRocksのメジャーバージョンを表し、2番目の数字はマイナーバージョンを表し、3番目の数字はパッチバージョンを表します。StarRocksは特定のバージョンに対して長期サポート（LTS）を提供しています。サポート期間は半年以上続きます。

| **StarRocksのバージョン** | **LTSバージョンですか** |
| --------------------- | ---------------------- |
| v2.0.x                | いいえ                  |
| v2.1.x                | いいえ                  |
| v2.2.x                | いいえ                  |
| v2.3.x                | いいえ                  |
| v2.4.x                | いいえ                  |
| v2.5.x                | はい                   |
| v3.0.x                | いいえ                  |
| v3.1.x                | いいえ                  |

### アップグレードパス

- **パッチバージョンのアップグレードの場合**

  StarRocksクラスタをパッチバージョンごとにアップグレードすることができます。たとえば、v2.2.6から直接v2.2.11にアップグレードすることができます。

- **マイナーバージョンのアップグレードの場合**

  StarRocks v2.0以降、StarRocksクラスタをマイナーバージョンごとにアップグレードすることができます。たとえば、v2.2.xから直接v2.5.xにアップグレードすることができます。ただし、互換性と安全性のために、StarRocksクラスタを**連続してマイナーバージョンから別のマイナーバージョンにアップグレードする**ことを強くお勧めします。たとえば、StarRocks v2.2クラスタをv2.5にアップグレードするには、次の順序でアップグレードする必要があります：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **メジャーバージョンのアップグレードの場合**

  StarRocksクラスタをv3.0にアップグレードするには、まずv2.5にアップグレードする必要があります。

### アップグレード手順

StarRocksは**ローリングアップグレード**をサポートしており、サービスを停止することなくクラスタをアップグレードすることができます。設計上、BEとCNはFEと互換性があります。したがって、クラスタをアップグレードする際には、まずBEとCNをアップグレードし、その後FEをアップグレードする必要があります。逆の順序でアップグレードすると、FEとBE/CNの非互換性が生じ、サービスがクラッシュする可能性があります。FEノードの場合、リーダーFEノードをアップグレードする前に、まずすべてのフォロワーFEノードをアップグレードする必要があります。

## 開始する前に

準備中に、マイナーまたはメジャーバージョンのアップグレードを行う場合は、互換性の設定を行う必要があります。また、クラスタ内のすべてのノードをアップグレードする前に、FEとBEのいずれかでアップグレードの可用性テストを実行する必要があります。

### 互換性の設定を行う

StarRocksクラスタを後のマイナーまたはメジャーバージョンにアップグレードする場合は、互換性の設定を行う必要があります。一般的な互換性の設定に加えて、詳細な設定はアップグレード元のStarRocksクラスタのバージョンによって異なります。

- **一般的な互換性の設定**

StarRocksクラスタをアップグレードする前に、タブレットのクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

アップグレード後、すべてのBEノードのステータスが「Alive」になったら、タブレットのクローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.0から後のバージョンにアップグレードする場合**

StarRocks v2.0クラスタをアップグレードする前に、次のBE設定とシステム変数を設定する必要があります。

1. BE設定項目`vector_chunk_size`を変更している場合は、アップグレード前に`4096`に設定する必要があります。これは静的パラメータであるため、BEの設定ファイル**be.conf**で変更し、ノードを再起動して変更を有効にする必要があります。
2. システム変数`batch_size`をグローバルに`4096`以下に設定します。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## BEのアップグレード

アップグレードの可用性テストに合格したら、まずクラスタ内のBEノードをアップグレードできます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードのデプロイディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 新しいバージョンのデプロイファイルである**bin**と**lib**の元のデプロイファイルを置き換えます。

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

5. 上記の手順を他のBEノードのアップグレードにも繰り返します。

## CNのアップグレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードのデプロイディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 新しいバージョンのデプロイファイルである**bin**と**lib**の元のデプロイファイルを置き換えます。

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

5. 上記の手順を他のCNノードのアップグレードにも繰り返します。

## FEのアップグレード

すべてのBEノードとCNノードをアップグレードした後、FEノードをアップグレードすることができます。まず、フォロワーFEノードをアップグレードし、その後にリーダーFEノードをアップグレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードのデプロイディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 新しいバージョンのデプロイファイルである**bin**、**lib**、**spark-dpp**の元のデプロイファイルを置き換えます。

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

5. 上記の手順を他のフォロワーFEノードのアップグレードにも繰り返し、最後にリーダーFEノードをアップグレードします。

   > **注意**
   >
   > StarRocksクラスタをv2.5からv3.0にアップグレードした後、クラスタをダウングレードし、再度v3.0にアップグレードする場合、一部のフォロワーFEのメタデータのアップグレードの失敗を回避するために、次の手順に従う必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。
   >
   > イメージファイルが同期されたかどうかは、リーダーFEのログファイル**fe.log**を表示して確認することができます。"push image.* from subdir [] to other nodes. totally xx nodes, push successful xx nodes"というログの記録があれば、イメージファイルが正常に同期されていることを示しています。
