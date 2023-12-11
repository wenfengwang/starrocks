---
displayed_sidebar: "Japanese"
---

# スケールインとアウト

このトピックでは、StarRocksのノードをスケールインおよびスケールアウトする方法について説明します。

## FEのスケールインとアウト

StarRocksには、FollowerとObserverの2種類のFEノードがあります。Followerは選挙投票や書き込みに関与します。Observerはログを同期し、読み取りパフォーマンスを向上させるために使用されます。

> * フォロワーFE（リーダーを含む）の数は奇数でなければならず、3つのノードを展開して高可用性（HA）モードを構築することを推奨します。
> * FEが高可用性展開（1つのリーダー、2つのフォロワー）の場合、より良い読み取りパフォーマンスのためにObserver FEを追加することを推奨します。* 通常、1つのFEノードは10〜20のBEノードと連携できます。FEノードの総数は10より少ないことが推奨されます。ほとんどの場合、3つで十分です。

### FEのスケールアウト

FEノードを展開してサービスを開始した後、次のコマンドを実行してFEをスケーリングアウトします。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### FEのスケールイン

FEのスケールインはスケールアウトと同様です。次のコマンドを実行してFEをスケーリングインします。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

拡張と収縮の後、ノードの情報は`show proc '/frontends';`を実行することで表示できます。

## BEのスケールインとアウト

BEがスケーリングインまたはスケーリングアウトされると、StarRocksは全体のパフォーマンスに影響を与えることなく自動的に負荷分散を行います。

### BEのスケールアウト

BEをスケーリングアウトするには、次のコマンドを実行します。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

BEの状態を確認するには、次のコマンドを実行します。

~~~sql
show proc '/backends';
~~~

### BEのスケールイン

BEノードをスケーリングインする方法は2つあります - `DROP`と`DECOMMISSION`。

`DROP`はBEノードをただちに削除し、失われた重複はFEのスケジューリングによって補われます。 `DECOMMISSION`はまず重複が補われ、その後BEノードを削除します。 `DECOMMISSION`はよりフレンドリーであり、BEのスケールインには推奨されます。

両方の方法のコマンドは類似しています:

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

BEを削除する操作は危険なので、実行する前に2回確認する必要があります

* `alter system drop backend "be_host:be_heartbeat_service_port";`