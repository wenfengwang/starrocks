---
displayed_sidebar: "Japanese"
---

# Apache Rangerを使用してアクセス権限を管理する

[Apache Ranger](https://ranger.apache.org/)は、ユーザーがビジュアルなウェブページを通じてアクセスポリシーをカスタマイズできる、集中型のセキュリティ管理フレームワークを提供しています。これにより、Hadoopエコシステム内のさまざまなコンポーネントやサービスに対して、どのロールがどのデータにアクセスできるかを決定し、細かい粒度のデータアクセス制御を行うことができます。

Apache Rangerには、以下のコアモジュールがあります：

- Ranger Admin：組み込みのウェブページを備えたRangerのコアモジュールです。ユーザーはこのページまたはRESTインターフェースを介してセキュリティポリシーを作成および更新することができます。Hadoopエコシステムのさまざまなコンポーネントのプラグインは、定期的にこれらのポリシーをポーリングおよびプルします。
- Agent Plugin：Hadoopエコシステムに組み込まれたコンポーネントのプラグインです。これらのプラグインは、定期的にRanger Adminからセキュリティポリシーをプルし、ローカルファイルにポリシーを保存します。ユーザーがコンポーネントにアクセスすると、対応するプラグインは設定されたセキュリティポリシーに基づいてリクエストを評価し、認証結果を対応するコンポーネントに送信します。
- User Sync：ユーザーおよびユーザーグループの情報をプルし、ユーザーおよびユーザーグループの許可データをRangerのデータベースに同期するために使用されます。

StarRocks v3.2では、ネイティブのRBAC特権システムに加えて、Apache Rangerを介したアクセス制御もサポートされており、より高いレベルのデータセキュリティを提供しています。

このトピックでは、StarRocksとApache Rangerのアクセス制御方法と統合プロセスについて説明します。データセキュリティを管理するためにRangerでセキュリティポリシーを作成する方法については、[Apache Ranger公式ウェブサイト](https://ranger.apache.org/)を参照してください。

## アクセス制御方法

Apache Rangerと統合されたStarRocksは、以下のアクセス制御方法を提供します：

- StarRocks ServiceをRangerに作成して、アクセス制御を実装します。ユーザーがStarRocksの内部テーブル、外部テーブル、またはその他のオブジェクトにアクセスする場合、StarRocks Serviceで構成されたアクセスポリシーに基づいてアクセス制御が行われます。
- ユーザーが外部データソースにアクセスする場合、Apache Ranger上の外部サービス（Hiveサービスなど）を再利用してアクセス制御を行うことができます。StarRocksは、異なるExternal Catalogsに対して異なるRangerサービスをマッチングし、外部データソースに対応するRangerサービスに基づいてアクセス制御を実装します。

StarRocksがApache Rangerと統合されると、次のアクセス制御パターンを実現することができます：

- Apache Rangerを使用して、StarRocksの内部テーブル、外部テーブル、およびすべてのオブジェクトへのアクセスを一元管理します。
- Apache Rangerを使用して、StarRocksの内部テーブルとオブジェクトへのアクセスを管理します。External Catalogsの場合は、Ranger上の対応する外部サービスのポリシーを再利用してアクセス制御を行います。
- Apache Rangerを使用して、外部データソースに対するアクセスを管理します。StarRocksの内部テーブルとオブジェクトへのアクセスは、StarRocksのRBAC特権システムを使用して管理します。

**認証プロセス**

- ユーザー認証にLDAPを使用し、Rangerを使用してLDAPユーザーを同期し、それらにアクセスルールを設定することもできます。StarRocksは、LDAPを介してユーザーログイン認証を完了することもできます。
- ユーザーがクエリを開始すると、StarRocksはクエリステートメントを解析し、ユーザー情報と必要な特権をApache Rangerに渡します。Rangerは、対応するServiceで構成されたアクセスポリシーに基づいて、ユーザーが必要な特権を持っているかどうかを判断し、認証結果をStarRocksに返します。ユーザーがアクセス権を持っている場合、StarRocksはクエリデータを返します。アクセス権がない場合、StarRocksはエラーを返します。

## 前提条件

- Apache Ranger 2.0.0以降がインストールされていること。Apache Rangerのインストール方法については、[Rangerクイックスタートガイド](https://ranger.apache.org/quick_start_guide.html)を参照してください。
- すべてのStarRocks FEマシンがApache Rangerにアクセスできること。各FEマシンで次のコマンドを実行して、接続が成功するかどうかを確認できます：

   ```SQL
   telnet <ranger-ip> <ranger-host>
   ```

   `Connected to <ip>`と表示された場合、接続は成功しています。

## 統合手順

### ranger-starrocks-pluginのインストール

現在、StarRocksは以下をサポートしています：

- Apache Rangerを介してアクセスポリシー、マスキングポリシー、および行レベルのフィルタポリシーを作成します。
- Ranger監査ログ。

1. Ranger Adminディレクトリの`ews/webapp/WEB-INF/classes/ranger-plugins`に`starrocks`フォルダを作成します。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. `plugin-starrocks/target/ranger-starrocks-plugin- 3.0.0 -SNAPSHOT.jar`と`mysql-connector-j`をダウンロードし、`starrocks`フォルダに配置します。

   ```SQL
   cd {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   wget xxxx
   wget xxx
   ```

3. Ranger Adminを再起動します。

   ```SQL
   ranger-admin restart
   ```

### Ranger AdminでStarRocks Serviceを構成する

1. `ranger-servicedef-starrocks.json`をStarRocks FEマシンまたはRangerマシンの任意のディレクトリにコピーします。Rangerプロジェクトの`/agents-common/src/main/resources/service-defs`ディレクトリに`ranger-servicedef-starrocks.json`が格納されています。

   ```SQL
   vim ranger-servicedef-starrocks.json
   -- JSONファイルの内容をコピーして保存します。
   ```

2. Ranger管理者として以下のコマンドを実行して、StarRocks Serviceを追加します。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. `http://<ranger-ip>:<ranger-host>/login.jsp`にアクセスして、Apache Rangerページにログインします。StarRocks Serviceがページに表示されます。

   ![home](../assets/ranger_home.png)

4. **STARROKCS**の後にあるプラス記号（`+`）をクリックして、StarRocks Serviceを構成します。

   ![service detail](../assets/ranger_service_details.png)
   ![property](../assets/ranger_properties.png)

   - `Username`と`Password`：ポリシー作成時にオブジェクト名の自動補完を行うために使用されます。これらのパラメータは、StarRocksとRangerの接続に影響しません。自動補完を使用する場合は、少なくとも1つの`db_admin`ロールがアクティブ化されたユーザーを構成してください。
   - `jdbc.url`：StarRocksクラスタのIPとポートを入力します。

   以下の図は、構成の例を示しています。

   ![example](../assets/ranger_show_config.png)

   以下の図は、追加されたサービスを示しています。

   ![added service](../assets/ranger_added_service.png)

5. **Test connection**をクリックして接続をテストし、接続が成功したら保存します。
6. StarRocksクラスタの各FEマシンで、`fe/conf`フォルダに`ranger-starrocks-security.xml`を作成し、内容をコピーします。次の2つのパラメータを変更し、変更内容を保存する必要があります：
   a. `ranger.plugin.starrocks.service.name`：Step 4で作成したStarRocks Serviceの名前に変更します。
   b. `ranger.plugin.starrocks.policy.rest.url`：Ranger Adminのアドレスに変更します。

   他の設定を変更する場合は、Apache Rangerの公式ドキュメントを参照してください。たとえば、`ranger.plugin.starrocks.policy.pollIntervalM`を変更してポリシーの変更の間隔を変更することができます。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- StarRocks Serviceの名前に変更します。
           <description>
               Name of the Ranger service containing policies for this StarRocks instance
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- Ranger Adminのアドレスに変更します。
           <description>
               URL to Ranger Admin
           </description>
       </property>   
   ...
   ```

7. すべてのFEマシンを再起動します。

   ```SQL
   -- FEフォルダに移動します。
   cd..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 他のサービスを再利用して外部テーブルへのアクセスを制御する

External Catalogの場合、アクセス制御に外部サービス（Hiveサービスなど）を再利用することができます。StarRocksは、異なるRanger外部サービスを異なるCatalogにマッチングすることができます。ユーザーが外部テーブルにアクセスする場合、システムは外部テーブルに対応するRanger Serviceのアクセスポリシーに基づいてアクセス制御を実装します。

1. HiveのRanger設定ファイル`ranger-hive-security.xml`をすべてのFEマシンの`fe/conf`フォルダにコピーします。
2. すべてのFEマシンを再起動します。
3. External Catalogを構成します。

   - External Catalogを作成する場合は、プロパティ`"ranger.plugin.hive.service.name"`を追加します。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
            "hive.metastore.uris" = "thrift://172.26.195.10:9083",
            "ranger.plugin.hive.service.name" = "hive_catalog_1"
        )
      ```

   - 既存のExternal Catalogにこのプロパティを追加することもできます。

       ```SQL
       ALTER CATALOG hive_catalog_1
       SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
       ```

   これにより、既存のCatalogの認証方法がRangerベースの認証に変更されます。

## 次の手順

StarRocks Serviceを追加した後、サービスにアクセス制御ポリシーを作成し、異なるユーザーまたはユーザーグループに異なる権限を割り当てることができます。ユーザーがStarRocksデータにアクセスする際には、これらのポリシーに基づいてアクセス制御が実装されます。
