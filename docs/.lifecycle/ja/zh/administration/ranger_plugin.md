---
displayed_sidebar: Chinese
---

# Apache Ranger を使用した権限管理

[Apache Ranger](https://ranger.apache.org/) は、集中式のセキュリティ管理フレームワークを提供し、ユーザーは視覚化された Web ページを通じてさまざまなアクセスポリシーをカスタマイズし、どのロールがどのデータにアクセスできるかを決定し、Hadoop エコシステムの各コンポーネントとサービスに対して細かいデータアクセス制御を行い、データのセキュリティとコンプライアンスを確保します。

Apache Ranger は以下のコアモジュールを提供します：

- Ranger Admin：Ranger のコアモジュールで、Web インターフェースが内蔵されており、ユーザーはこのインターフェースまたは REST API を通じてセキュリティポリシーを作成および更新できます。Hadoop エコシステムの各コンポーネントのプラグインは、これらのポリシーを定期的にポーリングし、取得します。
- Agent Plugin：Hadoop エコシステムのコンポーネントに組み込まれたプラグインで、定期的に Ranger Admin からセキュリティポリシーを取得し、ローカルファイルに保存します。ユーザーがコンポーネントにアクセスする際、プラグインはセキュリティポリシーに基づいてリクエストを評価し、結果を対応するコンポーネントにフィードバックします。
- User Sync：ユーザーとユーザーグループの情報を取得し、ユーザーとユーザーグループの権限データを Ranger のデータベースに同期します。

StarRocks 3.1 以降のバージョンでは、ネイティブの RBAC 権限システムに加えて、Apache Ranger を介したアクセス制御もサポートし、より高いレベルのデータセキュリティを提供します。

この記事では、StarRocks と Apache Ranger の統合後のアクセス制御方法と統合プロセスについて説明します。Ranger 上でアクセスポリシーを作成してデータセキュリティを管理する方法については、[Apache Ranger 公式サイト](https://ranger.apache.org/)を参照してください。

## アクセス制御方法

StarRocks が Apache Ranger と統合された後、以下のアクセス制御方法を実現できます：

- Ranger で StarRocks Service を作成し、アクセス制御を実施します。ユーザーが StarRocks の内部テーブル、外部テーブル、またはその他のオブジェクトにアクセスする際、StarRocks Service に設定されたアクセスポリシーに基づいてアクセス制御が行われます。
- External Catalog については、Ranger 上の外部サービス（例：Hive Service）を再利用してアクセス制御を実施できます。StarRocks は異なる Catalog に対して異なる Ranger サービスをマッチさせることをサポートしています。ユーザーが外部データソースにアクセスする際、直接データソースに対応するサービスに基づいてアクセス制御が行われます。

Apache Ranger の統合により、以下のアクセス制御モードを実現できます：

- Ranger を使用してすべての権限を管理し、StarRocks Service 内で内部テーブル、外部テーブル、およびすべてのオブジェクトを一元管理します。
- Ranger を使用してすべての権限を管理します。内部テーブルおよび内部オブジェクトについては StarRocks Service 内で管理し、External Catalog については追加の作成を行わず、対応する外部データソースの Ranger Service を直接再利用します。
- External Catalog については Ranger を使用して権限を管理し、対応する外部データソースの Ranger Service を再利用します。内部オブジェクトおよび内部テーブルについては StarRocks 内部で権限付与を行います。

**アクセス制御プロセス：**

- ユーザー認証については、LDAP を介して行うことも選択できます。Ranger は LDAP ユーザーを同期し、それらに対して権限ルールを設定できます。StarRocks も LDAP を介してユーザーログイン認証を完了できます。
- ユーザーがクエリを発行する際、StarRocks はクエリ文を解析し、ユーザー情報と必要な権限を Ranger に渡します。Ranger は対応するサービス内で作成されたアクセスポリシーに基づいてユーザーがアクセス権を持っているかどうかを判断し、StarRocks に認証結果を返します。ユーザーにアクセス権がある場合、StarRocks はクエリデータを返します。ユーザーにアクセス権がない場合、StarRocks はエラーを返します。

## 前提条件

- Apache Ranger 2.1.0 以上のバージョンが既にデプロイされていること。詳細なデプロイ手順については、[クイックスタート](https://ranger.apache.org/quick_start_guide.html)を参照してください。
- StarRocks のすべての FE マシンが Ranger にアクセスできることを確認してください。FE ノードのマシンで以下のコマンドを実行して判断できます：

  ```SQL
  telnet <ranger-ip> <ranger-port>
  ```

  `Connected to <ip>` と表示された場合、接続に成功しています。

## 統合プロセス

### ranger-starrocks-plugin のインストール

現在 StarRocks は以下をサポートしています：

- Ranger を通じて Access policy、Masking policy、Row-level filter policy を作成します。
- Ranger の監査ログをサポートします。

1. Ranger Admin の `ews/webapp/WEB-INF/classes/ranger-plugins` ディレクトリに `starrocks` フォルダを作成します。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. [plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar](https://www.starrocks.io/download/community) と [mysql-connector-j](https://dev.mysql.com/downloads/connector/j/) をダウンロードし、`starrocks` フォルダに配置します。

3. Ranger Admin を再起動します。

   ```SQL
   ranger-admin restart
   ```

### Ranger Admin で StarRocks Service を設定

1. [ranger-servicedef-starrocks.json](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json) を StarRocks FE マシンまたは Ranger クラスタマシンの任意のディレクトリにコピーします。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. Ranger の管理者アカウントを使用して以下のコマンドを実行し、StarRocks Service を追加します。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. Ranger のインターフェース `http://<ranger-ip>:<ranger-port>/login.jsp` にログインします。STARROCKS サービスが表示されていることが確認できます。

   ![ホーム](../assets/ranger_home.png)

4. **STARROCKS** のプラス記号 (`+`) をクリックして StarRocks Service の情報を設定します。

   ![サービス構成](../assets/ranger_service_details.png)

   ![プロパティ](../assets/ranger_properties.png)

   - `Service Name`：サービス名、必須項目です。
   - `Display Name`：STARROCKS の下に表示されるサービス名。指定しない場合は `Service Name` が表示されます。
   - `Username` と `Password`：FE のアカウントとパスワード。後で Policy を作成する際のオブジェクト名の自動補完に使用されますが、StarRocks と Ranger の接続性には影響しません。自動補完機能を使用する場合は、少なくとも `db_admin` ロールがデフォルトでアクティブなユーザーを一人設定してください。
   - `jdbc.url`：StarRocks クラスタの FE の IP とポートを記入します。

   下の画像は記入例を示しています。

   ![例示](../assets/ranger_show_config.png)

   下の画像はページ上で設定されたサービスを示しています。

   ![サービス](../assets/ranger_added_service.png)

5. **Test connection** をクリックして接続テストを行い、成功したら保存します。
6. StarRocks クラスタの各 FE マシンで、`fe/conf` フォルダに [ranger-starrocks-security.xml](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml) を作成し、内容をコピーしてください。以下の2箇所を変更して保存する必要があります：

   - `ranger.plugin.starrocks.service.name` を作成した StarRocks Service の名前に変更します。
   - `ranger.plugin.starrocks.policy.rest.url` を Ranger Admin のアドレスに変更します。

   他の設定を変更する必要がある場合は、Ranger の公式ドキュメントに従って対応する変更を行うことができます。例えば、`ranger.plugin.starrocks.policy.pollIntervalMs` を変更して権限変更の取得時間を変更することができます。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- StarRocks Service の名前に変更します。
           <description>
               Name of the Ranger service containing policies for this StarRocks instance
           </description>
       </property>
   ...


   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- Ranger Admin のアドレスに変更します。
           <description>
               URL to Ranger Admin
           </description>
       </property>   
   ...
   ```

7. すべての FE の設定ファイルを変更し、`access_control=ranger` を追加します。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. すべての FE を再起動します。

   ```SQL
   -- FE ディレクトリに戻ります
   cd..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 外部テーブルの認証に他の Service を再利用

External Catalog については、外部サービス（例：Hive Service）を再利用してアクセス制御を実施できます。StarRocks は異なる Catalog に対して異なる Ranger サービスをマッチさせることをサポートしています。ユーザーが外部テーブルにアクセスする際、対応するテーブルのサービスに基づいてアクセス制御が行われます。

1. Hive の Ranger 関連設定ファイル (`ranger-hive-security.xml`) をすべての FE マシンの `fe/conf` フォルダにコピーします。
2. すべての FE を再起動します。
3. Catalog を設定します。

   External Catalog を作成する際に、PROPERTIES に `"ranger.plugin.hive.service.name"` を追加します。

    ```SQL
      CREATE EXTERNAL CATALOG hive_catalog_1
      PROPERTIES (
          "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
          "ranger.plugin.hive.service.name" = "hive_catalog_1"
      )
    ```

   既存の External Catalog にこの属性を追加することもできます。既存の Catalog を Ranger 認証を通じて変換します。

    ```SQL
      ALTER CATALOG hive_catalog_1
      SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
    ```

## 次のステップ

StarRocks Service を追加した後、そのサービスをクリックして権限ポリシーを作成し、異なるユーザーやユーザーグループに異なる権限を割り当てることができます。その後、ユーザーが StarRocks のデータにアクセスする際、これらのポリシーに基づいてアクセス制御が行われます。
