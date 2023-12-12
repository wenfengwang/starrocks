---
displayed_sidebar: "Japanese"
---

# 拡大縮小

このトピックでは、StarRocksのノードを拡大縮小する方法について説明します。

## FEの拡大縮小

StarRocksには、FollowerとObserverの2種類のFEノードがあります。Followerは選挙投票および書き込みに関与します。Observerはログを同期し読み取り性能を向上させるために使用されます。

> * Follower FEの数（リーダーを含む）は奇数でなければならず、高可用性（HA）モードを形成するために3つのFEを展開することが推奨されます。
> * FEが高可用性展開（1つのリーダーと2つのFollower）されている場合、良好な読み取り性能のためにObserver FEを追加することを推奨します。
> * 通常、1つのFEノードは10〜20のBEノードと連携することができます。FEノードの合計数は10未満であることを推奨します。ほとんどの場合、3つで十分です。

### FEの拡大

FEノードを展開しサービスを開始した後、次のコマンドを実行してFEノードを拡大します。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### FEの縮小

FEの縮小は拡大と同様です。FEを縮小するには、次のコマンドを実行します。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

拡大と縮小の後、`show proc '/frontends';` を実行してノード情報を表示できます。

## BEの拡大縮小

BEを拡大または縮小した後、StarRocksは全体のパフォーマンスに影響を与えることなく自動的に負荷分散を行います。

### BEの拡大

BEを拡大するには、次のコマンドを実行します。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

BEの状態を確認するには、次のコマンドを実行します。

~~~sql
show proc '/backends';
~~~

### BEの縮小

BEノードを縮小するには、`DROP`および`DECOMMISSION`の2つの方法があります。

`DROP`はBEノードを直ちに削除し、失われた重複はFEのスケジューリングによって補われます。 `DECOMMISSION`は重複がまず補われ、その後BEノードが削除されることを確認します。 `DECOMMISSION`はよりフレンドリーであり、BEの縮小に推奨されます。

両方の方法のコマンドは似ています:

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

BEを削除する操作は危険なので、実行する前に2回確認する必要があります

* `alter system drop backend "be_host:be_heartbeat_service_port";`