---
displayed_sidebar: "Japanese"
---

# Apache Rangerを使用して権限を管理する

[Apache Ranger](https://ranger.apache.org/)は、ユーザーがビジュアルWebページを通じてアクセスポリシーをカスタマイズすることを可能にする、中央集権的なセキュリティ管理フレームワークを提供します。これにより、どのロールがどのデータにアクセスできるかを決定し、Hadoopエコシステムのさまざまなコンポーネントやサービスに対して細かくデータアクセス制御を行うことができます。

Apache Rangerには、次のコアモジュールがあります。

- Ranger Admin：組み込みのWebページを持つRangerのコアモジュールです。ユーザーはこのページまたはRESTインターフェースを通じてセキュリティポリシーを作成および更新することができます。Hadoopエコシステムのさまざまなコンポーネントのプラグインは、定期的にこれらのポリシーを取得して引き出します。
- エージェントプラグイン：Hadoopエコシステムに埋め込まれたコンポーネントのプラグインです。これらのプラグインは、定期的にRanger Adminからセキュリティポリシーを取得し、それをローカルファイルに保存します。ユーザーがコンポーネントにアクセスすると、対応するプラグインは構成されたセキュリティポリシーに基づいてリクエストを評価し、認証結果を対応するコンポーネントに送信します。
- ユーザー同期：ユーザーやユーザーグループ情報を取得し、ユーザーやユーザーグループの権限データをRangerのデータベースに同期するために使用されます。

StarRocks v3.1では、ネイティブのRBAC特権システムに加えて、Apache Rangerを介したアクセスコントロールもサポートされており、より高いレベルのデータセキュリティが提供されています。

このトピックでは、StarRocksとApache Rangerの権限制御方法と統合プロセスについて説明します。データセキュリティを管理するためにRangerでセキュリティポリシーを作成する方法については、[Apache Ranger公式ウェブサイト](https://ranger.apache.org/)を参照してください。

## 権限制御方法

Apache Rangerと統合したStarRocksでは、次の権限制御方法が提供されます。

- RangerでStarRocksサービスを作成して権限制御を実装する。ユーザーがStarRocks内部テーブル、外部テーブル、または他のオブジェクトにアクセスする場合、StarRocks Serviceで構成されたアクセスポリシーに従ってアクセス制御が実行されます。
- ユーザーが外部データソースにアクセスする場合、Apache Ranger上の外部サービス（例：Hiveサービス）を再利用してアクセス制御を行うことができます。StarRocksは、異なるExternal Catalogに対応するRangerサービスをマッチングし、外部データソースに対応するRangerサービスに基づいてアクセス制御を実施します。

StarRocksがApache Rangerと統合されると、以下のアクセス制御パターンを実現できます。

- Apache Rangerを使用して、StarRocksの内部テーブル、外部テーブル、およびすべてのオブジェクトへの統一されたアクセスを管理する。
- Apache Rangerを使用して、StarRocksの内部テーブルとオブジェクトへのアクセスを管理する。外部カタログについては、対応するRanger上の外部サービスのポリシーを再利用してアクセス制御を行う。
- Apache Rangerを使用して、外部データソースに対するアクセスを管理する。外部データソースに対応するServiceを再利用してアクセス制御を行い、StarRocksのRBAC特権システムを使用して、StarRocksの内部テーブルとオブジェクトへのアクセスを管理する。

**認証プロセス**

- ユーザー認証にLDAPを使用し、Rangerを使用してLDAPユーザーを同期し、それらにアクセスルールを設定することもできます。StarRocksはLDAPを介したユーザーログイン認証も行うことができます。
- ユーザーがクエリを開始すると、StarRocksはクエリ文を解析し、ユーザー情報と必要な特権をApache Rangerに渡します。Rangerは、対応するServiceで構成されたアクセスポリシーに基づいてユーザーが必要な特権を持っているかどうかを判断し、認証結果をStarRocksに返します。ユーザーがアクセス権を持っている場合、StarRocksはクエリデータを返し、持っていない場合はエラーを返します。

## 前提条件

- Apache Ranger 2.0.0以降がインストールされている必要があります。Apache Rangerのインストール手順については、[Rangerクイックスタート](https://ranger.apache.org/quick_start_guide.html)を参照してください。
- すべてのStarRocks FEマシンがApache Rangerにアクセスできる必要があります。それぞれのFEマシンで次のコマンドを実行してアクセスを確認できます。

   ```SQL
   telnet <ranger-ip> <ranger-host>
   ```

   `<ip>`に「Connected to」が表示された場合、接続に成功しています。

## 統合手順

### ranger-starrocks-pluginのインストール

現在、StarRocksは次の機能をサポートしています。

- Apache Rangerを介してアクセスポリシー、マスキングポリシー、および行レベルフィルタポリシーを作成します。
- Ranger監査ログ。

1. Ranger Adminディレクトリ`ews/webapp/WEB-INF/classes/ranger-plugins`に`starrocks`フォルダを作成します。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. [`plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar`](https://www.starrocks.io/download/community)および[`mysql-connector-j`](https://dev.mysql.com/downloads/connector/j/)をダウンロードし、それらを`starrocks`フォルダに配置します。

3. Ranger Adminを再起動します。

   ```SQL
   ranger-admin restart
   ```

### Ranger AdminでStarRocks Serviceを構成する

1. [`ranger-servicedef-starrocks.json`](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json)をStarRocks FEマシンまたはRangerマシンの任意のディレクトリにコピーします。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. Ranger管理者として次のコマンドを実行して、StarRocks Serviceを追加します。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. `http://<ranger-ip>:<ranger-host>/login.jsp`にアクセスしてApache Rangerページにログインします。ページ上にSTARROCKSサービスが表示されます。

   ![home](../assets/ranger_home.png)

4. **STARROKCS**の後ろにあるプラス記号（`+`）をクリックして、StarRocks Serviceを構成します。

   ![service detail](../assets/ranger_service_details.png)

   ![property](../assets/ranger_properties.png)

   - `Service Name`：サービス名を入力してください。
   - `Display Name`：STARROCKSの下で表示したい名前。指定しない場合、`Service Name`が表示されます。
   - `Username`および`Password`：ポリシーを作成する際にオブジェクト名をオートコンプリートするために使用されるFEのユーザー名とパスワード。この2つのパラメータはStarRocksとRangerの接続に影響しません。オートコンプリートを使用する場合は、少なくとも`db_admin`ロールをアクティブにしたユーザーを1人以上構成してください。
   - `jdbc.url`：StarRocks FEのIPアドレスとポートを入力します。

   以下は構成例です。

   ![example](../assets/ranger_show_config.png)

   次の図は追加されたサービスを示しています。

   ![added service](../assets/ranger_added_service.png)

5. **Test connection**をクリックして接続をテストし、接続が成功したら保存します。
6. StarRocksクラスタの各FEマシンにおいて、`fe/conf`フォルダに[`ranger-starrocks-security.xml`](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml)を作成し、その内容をコピーします。次の2つのパラメータを変更し、変更を保存します。

   - `ranger.plugin.starrocks.service.name`：ステップ4で作成したStarRocks Serviceの名前に変更します。
   - `ranger.plugin.starrocks.policy.rest the url`：Ranger Adminのアドレスに変更します。

   その他の構成を変更する必要がある場合は、Apache Rangerの公式ドキュメントを参照してください。たとえば、`ranger.plugin.starrocks.policy.pollIntervalM`を変更してポリシー変更の間隔を変更することができます。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- StarRocks Serviceの名前に変更します。
           <description>
               StarRocksインスタンスのポリシーを含むRangerサービスの名前
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- Ranger Adminのアドレスに変更します。
           <description>
               Ranger AdminへのURL
           </description>
       </property>   
   ...
   ```

7. すべてのFE構成ファイルに`access_control=ranger`の設定を追加します。

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

## 外部テーブルへのアクセス制御のための他のサービスの再利用

External Catalogについては、外部サービス（Hive Serviceなど）を再利用してアクセス制御を行うことができます。StarRocksは、異なるRanger外部サービスを異なるカタログにマッチングしサポートしています。外部テーブルにアクセスする場合、システムは外部テーブルに対応するRanger Serviceのアクセスポリシーに基づいてアクセス制御を実施します。

1. HiveのRanger構成ファイル`ranger-hive-security.xml`をすべてのFEマシンの`fe/conf`フォルダにコピーします。
2. すべてのFEマシンを再起動します。
3. External Catalogを構成します。

   - External Catalogを作成する際、プロパティに`"ranger.plugin.hive.service.name"`を追加します。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
```markdown
      "hive.metastore.uris" = "thrift://172.26.195.10:9083",
      "ranger.plugin.hive.service.name" = "hive_catalog_1"
    )

- 既存の外部カタログにもこのプロパティを追加できます。

   ```SQL
   ALTER CATALOG hive_catalog_1
   SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
   ```

この操作により、既存のカタログの認証方式がRangerベースの認証に変更されます。

## 次の手順

StarRocksサービスを追加した後、サービスをクリックして、サービスのアクセス制御ポリシーを作成し、異なるユーザーまたはユーザーグループに異なる権限を割り当てることができます。ユーザーがStarRocksデータにアクセスする際には、これらのポリシーに基づいてアクセス制御が実装されます。
```