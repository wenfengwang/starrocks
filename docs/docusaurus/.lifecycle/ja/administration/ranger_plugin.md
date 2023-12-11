---
displayed_sidebar: "Japanese"
---

# Apache Rangerを使用して権限を管理する

[Apache Ranger](https://ranger.apache.org/)は、ユーザーがビジュアルウェブページを通じてアクセスポリシーをカスタマイズできるセントラルセキュリティ管理フレームワークを提供します。これにより、どのロールがどのデータにアクセスできるかを決定し、Hadoopエコシステムのさまざまなコンポーネントおよびサービスに対して細かいデータアクセス制御を実行できます。

Apache Rangerには次のコアモジュールがあります。

- Ranger Admin: 組み込みのウェブページを持つRangerのコアモジュール。ユーザーはこのページまたはRESTインターフェースを介してセキュリティポリシーを作成および更新できます。Hadoopエコシステムのさまざまなコンポーネントのプラグインは、定期的にこれらのポリシーをポーリングおよび取得します。
- Agent Plugin: Hadoopエコシステムに組み込まれたコンポーネントのプラグイン。これらのプラグインは定期的にRanger Adminからセキュリティポリシーを取得し、ローカルファイルに保存します。ユーザーがコンポーネントにアクセスすると、対応するプラグインは構成されたセキュリティポリシーに基づいてリクエストを評価し、認証結果を対応するコンポーネントに送信します。
- User Sync: ユーザーおよびユーザーグループの情報を取得し、ユーザーおよびユーザーグループの権限データをRangerのデータベースに同期させるために使用されます。

StarRocks v3.1では、ネイティブのRBAC特権システムに加えて、Apache Rangerを使用してアクセス制御をサポートし、より高度なデータセキュリティを提供しています。

このトピックでは、StarRocksとApache Rangerのアクセス制御方法と統合プロセスについて説明します。データセキュリティを管理するためにRangerでセキュリティポリシーを作成する方法については、[Apache Ranger公式ウェブサイト](https://ranger.apache.org/)を参照してください。

## アクセス制御方法

Apache Rangerと統合されたStarRocksは、以下のアクセス制御方法を提供します。

- StarRocks ServiceをRangerに作成し、アクセス制御を実装します。ユーザーがStarRocksの内部テーブル、外部テーブル、その他のオブジェクトにアクセスする場合、StarRocks Serviceで構成されたアクセスポリシーに従ってアクセス制御が行われます。
- ユーザーが外部データソースにアクセスする場合、Apache Ranger上の外部サービス(例: Hiveサービス)を再利用してアクセス制御を行うことができます。StarRocksは異なるExternal Catalogsに対応するRangerサービスをマッチングし、外部テーブルへのアクセス制御を実装します。

StarRocksがApache Rangerと統合された後、以下のアクセス制御パターンを達成できます。

- Apache Rangerを使用して、StarRocksの内部テーブル、外部テーブル、およびすべてのオブジェクトへのアクセスを一元的に管理する。
- Apache Rangerを使用して、StarRocksの内部テーブルとオブジェクトへのアクセスを管理します。External Catalogsについては、Ranger上の対応する外部サービスのポリシーを再利用してアクセス制御を行います。
- 外部カタログへのアクセスはApache Rangerを使用して管理し、外部データソースに対応するServiceを再利用します。StarRocks RBAC特権システムを使用して、StarRocksの内部テーブルとオブジェクトへのアクセスを管理します。

**認証プロセス**

- ユーザー認証にLDAPを使用し、LDAPユーザーを同期し、それらのためのアクセスルールを構成することもできます。StarRocksはLDAPを使用したユーザーログイン認証を完了することもできます。
- ユーザーがクエリを開始すると、StarRocksはクエリステートメントを解析し、ユーザー情報と必要な権限をApache Rangerに渡します。Rangerは対応するServiceで構成されたアクセスポリシーに基づいて、ユーザーが必要な権限を持っているかどうかを判定し、認証結果をStarRocksに返します。ユーザーがアクセス権を持っている場合、StarRocksはクエリデータを返します。持っていない場合、エラーを返します。

## 必要条件

- Apache Ranger 2.0.0 以降がインストールされていること。Apache Rangerのインストール方法については、[Rangerクイックスタート](https://ranger.apache.org/quick_start_guide.html)を参照してください。
- すべてのStarRocks FEマシンがApache Rangerにアクセスできること。各FEマシンで次のコマンドを実行して確認できます。

   ```SQL
   telnet <ranger-ip> <ranger-host>
   ```

   `Connected to <ip>`が表示された場合、接続に成功しています。

## 統合手順 

### ranger-starrocks-pluginのインストール

現在、StarRocksは以下をサポートしています。

- Apache Rangerを使用してアクセスポリシー、マスクポリシー、および行レベルのフィルタポリシーを作成できます。
- Ranger監査ログ。

1. Ranger Adminディレクトリ`ews/webapp/WEB-INF/classes/ranger-plugins`に`starrocks`フォルダを作成します。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. [`plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar`](https://www.starrocks.io/download/community)と[`mysql-connector-j`](https://dev.mysql.com/downloads/connector/j/)をダウンロードし、それらを`starrocks`フォルダに配置します。

3. Ranger Adminを再起動します。

   ```SQL
   ranger-admin restart
   ```

### Ranger AdminでStarRocks Serviceを構成する

1. [`ranger-servicedef-starrocks.json`](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json)をStarRocks FEマシンまたはRangerマシンの任意のディレクトリにコピーします。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. Ranger管理者として以下のコマンドを実行してStarRocks Serviceを追加します。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. `http://<ranger-ip>:<ranger-host>/login.jsp`にアクセスして、Apache Rangerページにログインします。STARROCKSサービスがページに表示されます。

   ![home](../assets/ranger_home.png)

4. **STARROKCS**の後のプラス記号(`+`)をクリックしてStarRocks Serviceを構成します。

   ![service detail](../assets/ranger_service_details.png)

   ![property](../assets/ranger_properties.png)

   - `Service Name`: サービス名を入力してください。
   - `Display Name`: STARROCKSのサービスの下で表示する名前。指定しない場合、`Service Name`が表示されます。
   - `Username`および`Password`: オブジェクト名を自動入力する際に使用されるFEのユーザー名とパスワード。これらのパラメータはStarRocksとRanger間の接続には影響しません。オートコンプリートを使用したい場合は、少なくとも1つのユーザーを`db_admin`ロールで設定してください。
   - `jdbc.url`: StarRocks FEのIPアドレスとポートを入力してください。

   次の図は、構成の例を示します。

   ![example](../assets/ranger_show_config.png)

   次の図は、追加されたサービスを示しています。

   ![added service](../assets/ranger_added_service.png)

5. **Test connection**をクリックして接続をテストし、接続が成功したら保存します。
6. StarRocksクラスタの各FEマシンで、`fe/conf`フォルダに[`ranger-starrocks-security.xml`](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml)を作成し、内容をコピーします。次の2つのパラメータを変更し、変更を保存する必要があります。

   - `ranger.plugin.starrocks.service.name`: ステップ4で作成したStarRocks Service名に変更してください。
   - `ranger.plugin.starrocks.policy.rest the url`: Ranger Adminのアドレスに変更してください。

   他の設定を変更する場合は、Apache Rangerの公式ドキュメントを参照してください。たとえば、`ranger.plugin.starrocks.policy.pollIntervalM`を変更してポリシーの変更の取得間隔を変更することができます。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- StarRocks Service名に変更してください。
           <description>
               このStarRocksインスタンスのポリシーを含むRangerサービスの名前
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- Ranger Adminのアドレスに変更してください。
           <description>
               Ranger AdminのURL
           </description>
       </property>   
   ...
   ```

7. すべてのFE構成ファイルに設定`access_control = ranger`を追加します。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. すべてのFEマシンを再起動します。

   ```SQL
   -- FEフォルダに移動します。
   cd..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 他のサービスを再利用して外部テーブルへのアクセス制御を行う

External Catalogの場合、外部サービス（例: Hive Service）を再利用してアクセス制御を行うことができます。StarRocksは異なるRanger外部サービスを異なるカタログにマッチングできます。ユーザーが外部テーブルにアクセスする場合、システムは外部テーブルに対応するRangerサービスのアクセスポリシーに基づいてアクセス制御を実行します。

1. HiveのRanger設定ファイル`ranger-hive-security.xml`をすべてのFEマシンの`fe/conf`フォルダにコピーします。
2. すべてのFEマシンを再起動します。
3. External Catalogを構成します。

   - External Catalogを作成する際に、プロパティ`"ranger.plugin.hive.service.name"`を追加します。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
```
            "hive.metastore.uris" = "thrift://172.26.195.10:9083",
            "ranger.plugin.hive.service.name" = "hive_catalog_1"
        )
      ```

   - 既存のExternal Catalogにこのプロパティを追加することもできます。
  
       ```SQL
       ALTER CATALOG hive_catalog_1
       SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
       ```

​    この操作により、既存のカタログの認証方法がRangerベースの認証に変更されます。

## 次に何をするか

StarRocksサービスを追加した後、サービスをクリックして、サービスにアクセス制御ポリシーを作成し、異なるユーザーまたはユーザーグループに異なる権限を割り当てることができます。ユーザーがStarRocksデータにアクセスする際、これらのポリシーに基づいてアクセス制御が実施されます。