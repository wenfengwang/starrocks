---
displayed_sidebar: English
---

# スケールインとスケールアウト

このトピックでは、StarRocksのノードをスケールインおよびスケールアウトする方法について説明します。

## FEのスケールインとスケールアウト

StarRocksには、FollowerとObserverの2種類のFEノードがあります。Followerは選挙投票と書き込みに関与します。Observerはログの同期と読み取りパフォーマンスの拡張のみに使用されます。

> * Follower FE（リーダーを含む）の数は奇数でなければならず、3つを配置してハイアベイラビリティ（HA）モードを形成することを推奨します。
> * FEがハイアベイラビリティ展開（1つのリーダー、2つのフォロワー）の場合、読み取りパフォーマンスを向上させるためにObserver FEを追加することを推奨します。* 通常、1つのFEノードは10〜20のBEノードと連携できます。FEノードの総数は10未満が望ましいです。ほとんどのケースでは3つで十分です。

### FEのスケールアウト

FEノードをデプロイしてサービスを開始した後、以下のコマンドを実行してFEをスケールアウトします。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### FEのスケールイン

FEのスケールインはスケールアウトと同様です。以下のコマンドを実行してFEをスケールインします。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

スケールアウトとスケールインの後、`show proc '/frontends';`を実行してノード情報を確認できます。

## BEのスケールインとスケールアウト

BEがスケールインまたはスケールアウトされた後、StarRocksは全体的なパフォーマンスに影響を与えずに自動的にロードバランシングを行います。

### BEのスケールアウト

以下のコマンドを実行してBEをスケールアウトします。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

以下のコマンドを実行してBEのステータスを確認します。

~~~sql
show proc '/backends';
~~~

### BEのスケールイン

BEノードをスケールインするには、`DROP`と`DECOMMISSION`の2つの方法があります。

`DROP`はBEノードを即座に削除し、失われたレプリカはFEによって補完されます。`DECOMMISSION`は、レプリカが補完された後にBEノードを削除するため、よりフレンドリーです。BEのスケールインには`DECOMMISSION`を推奨します。

両方の方法のコマンドは以下の通りです。

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

バックエンドの削除は危険な操作ですので、実行する前に二度確認してください。

* `alter system drop backend "be_host:be_heartbeat_service_port";`
