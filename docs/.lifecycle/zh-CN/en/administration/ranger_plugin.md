---
displayed_sidebar: "Chinese"
---

# 使用Apache Ranger管理权限

[Apache Ranger](https://ranger.apache.org/)提供了一个集中的安全管理框架，允许用户通过可视化网页自定义访问策略。这有助于确定哪些角色可以访问哪些数据，并对Hadoop生态系统中的各种组件和服务进行细粒度的数据访问控制。

Apache Ranger提供以下核心模块：

- Ranger Admin：带有内置网页的Ranger的核心模块。用户可以在此页面上创建和更新安全策略，也可以通过REST接口。Hadoop生态系统的各个组件的插件定期轮询和拉取这些策略。
- Agent Plugin：嵌入在Hadoop生态系统中的组件的插件。这些插件定期从Ranger Admin中拉取安全策略，并将策略存储在本地文件中。当用户访问组件时，相应的插件根据配置的安全策略评估请求，并将认证结果发送到相应的组件。
- User Sync：用于拉取用户和用户组信息，并将用户和用户组的权限数据同步到Ranger的数据库。

除了本地RBAC权限系统外，StarRocks v3.1还支持通过Apache Ranger进行访问控制，提供更高级别的数据安全。

该主题描述了StarRocks与Apache Ranger的权限控制方法和集成过程。有关如何在Ranger上创建安全策略以管理数据安全的信息，请参阅[Apache Ranger官方网站](https://ranger.apache.org/)。

## 权限控制方法

集成了Apache Ranger的StarRocks提供了以下权限控制方法：

- 在Ranger中创建StarRocks服务以实现权限控制。当用户访问StarRocks内部表、外部表或其他对象时，将根据StarRocks服务中配置的访问策略执行访问控制。
- 当用户访问外部数据源时，可以重用Apache Ranger上的外部服务（例如Hive服务）进行访问控制。StarRocks可以将不同的Ranger服务与不同的外部目录进行匹配，并基于与外部表对应的Ranger服务的访问策略进行访问控制。

经过StarRocks与Apache Ranger的集成后，可以实现以下访问控制模式：

- 使用Apache Ranger统一管理对StarRocks内部表、外部表和所有对象的访问。
- 使用Apache Ranger管理对StarRocks内部表和对象的访问。对于外部目录，重用Ranger上的相应外部服务的策略进行访问控制。
- 使用Apache Ranger管理对外部目录的访问，重用与外部数据源对应的服务。使用StarRocks的RBAC权限系统管理对StarRocks内部表和对象的访问。

**认证过程**

- 您也可以使用LDAP进行用户认证，然后使用Ranger同步LDAP用户并为他们配置访问规则。StarRocks也可以通过LDAP完成用户登录认证。
- 当用户发起查询时，StarRocks解析查询语句，将用户信息和所需权限传递给Apache Ranger。Ranger根据相应服务中配置的访问策略确定用户是否具有所需权限，并将认证结果返回给StarRocks。如果用户有权限，StarRocks将返回查询数据；否则，StarRocks将返回错误。

## 先决条件

- 已安装Apache Ranger 2.0.0或更高版本。有关如何安装Apache Ranger的说明，请参阅[Ranger快速入门](https://ranger.apache.org/quick_start_guide.html)。
- 所有StarRocks FE机器可以访问Apache Ranger。您可以通过在每个FE机器上运行以下命令来检查：

   ```SQL
   telnet <ranger-ip> <ranger-host>
   ```

   如果显示`Connected to <ip>`，则连接成功。

## 集成流程

### 安装ranger-starrocks-plugin

当前，StarRocks支持：

- 通过Apache Ranger创建访问策略、掩码策略和行级过滤策略。
- Ranger审计日志。

1. 在Ranger Admin目录`ews/webapp/WEB-INF/classes/ranger-plugins`中创建`starrocks`文件夹。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. 下载[`plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar`](https://www.starrocks.io/download/community)和[`mysql-connector-j`](https://dev.mysql.com/downloads/connector/j/)，并将它们放在`starrocks`文件夹中。

3. 重新启动Ranger Admin。

   ```SQL
   ranger-admin restart
   ```

### 在Ranger Admin上配置StarRocks服务

1. 将[`ranger-servicedef-starrocks.json`](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json)复制到StarRocks FE机器或Ranger机器的任意目录。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. 以Ranger管理员身份运行以下命令添加StarRocks服务。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. 访问`http://<ranger-ip>:<ranger-host>/login.jsp`以登录到Apache Ranger页面。在页面上会出现STARROCKS服务。

   ![home](../assets/ranger_home.png)

4. 单击**STARROKCS**后面的加号（`+`）配置StarRocks服务。

   ![service detail](../assets/ranger_service_details.png)

   ![property](../assets/ranger_properties.png)

   - `Service Name`：必须输入一个服务名称。
   - `Display Name`：您希望在STARROCKS下显示的服务名称。如果未指定，则将显示`Service Name`。
   - `Username`和`Password`：FE用户名和密码，用于在创建策略时自动完成对象名称。这两个参数不影响StarRocks和Ranger之间的连接。如果要使用自动完成，请至少配置一个具有`db_admin`角色激活的用户。
   - `jdbc.url`：输入StarRocks FE IP地址和端口。

   以下图片显示了一个配置示例。

   ![example](../assets/ranger_show_config.png)

   以下图片显示了添加的服务。

   ![added service](../assets/ranger_added_service.png)

5. 单击**Test connection**来测试连接，连接成功后保存。
6. 在StarRocks集群的每个FE机器上，在`fe/conf`文件夹创建[`ranger-starrocks-security.xml`](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml)并复制内容。您必须修改以下两个参数并保存修改：

   - `ranger.plugin.starrocks.service.name`：改为Step 4中创建的StarRocks服务名称。
   - `ranger.plugin.starrocks.policy.rest.url`：改为Ranger Admin的地址。

   如果需要修改其他配置，请参考Apache Ranger的官方文档。例如，您可以修改`ranger.plugin.starrocks.policy.pollIntervalM`以更改策略更改的间隔。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- 将其更改为StarRocks服务名称。
           <description>
               包含此StarRocks实例策略的Ranger服务的名称
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- 将其更改为Ranger Admin地址。
           <description>
               到Ranger Admin的URL
           </description>
       </property>   
   ...
   ```

7. 在所有FE配置文件中添加配置`access_control = ranger`。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. 重新启动所有FE机器。

   ```SQL
   -- 切换到FE文件夹。
   cd..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 重用其他服务以控制对外部表的访问

对于外部目录，可以重用外部服务（例如Hive服务）进行访问控制。StarRocks支持将不同的Ranger外部服务与不同的目录进行匹配。当用户访问外部表时，系统将基于与外部表对应的Ranger服务的访问策略进行访问控制。

1. 将Hive的Ranger配置文件`ranger-hive-security.xml`复制到所有FE机器的`fe/conf`文件夹。
2. 重新启动所有FE机器。
3. 配置外部目录。

   - 在创建外部目录时，添加属性`"ranger.plugin.hive.service.name"`。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
"hive.metastore.uris" = "thrift://172.26.195.10:9083",
"ranger.plugin.hive.service.name" = "hive_catalog_1"
)


- 您还可以将此属性添加到现有的外部目录。

    ```SQL
    ALTER CATALOG hive_catalog_1
    SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
    ```

​    此操作会将现有目录的身份验证方法更改为基于Ranger的身份验证。

## 下一步操作

添加StarRocks服务后，您可以单击该服务以为该服务创建访问控制策略，并为不同用户或用户组分配不同权限。当用户访问StarRocks数据时，访问控制将根据这些策略进行实施。