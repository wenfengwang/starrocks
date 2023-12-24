---
displayed_sidebar: English
---

# 使用 Apache Ranger 管理权限

[Apache Ranger](https://ranger.apache.org/) 提供了一个集中式安全管理框架，允许用户通过可视化网页自定义访问策略。这有助于确定哪些角色可以访问哪些数据，并对 Hadoop 生态系统中的各种组件和服务进行细粒度的数据访问控制。

Apache Ranger 提供以下核心模块：

- Ranger Admin：Ranger 的核心模块，内置网页。用户可以在此页面上或通过 REST 接口创建和更新安全策略。Hadoop 生态系统中各个组件的插件会定期轮询并提取这些策略。
- Agent Plugin：嵌入在 Hadoop 生态系统中的组件插件。这些插件会定期从 Ranger Admin 中提取安全策略，并将这些策略存储在本地文件中。当用户访问某个组件时，对应的插件会根据配置的安全策略对请求进行评估，并将认证结果发送给相应的组件。
- User Sync：用于拉取用户和用户组信息，将用户和用户组的权限数据同步到 Ranger 的数据库中。

StarRocks v3.1 除了原生的 RBAC 权限系统外，还支持通过 Apache Ranger 进行访问控制，提供更高级别的数据安全性。

本文介绍 StarRocks 和 Apache Ranger 的权限控制方法和集成流程。关于如何在 Ranger 上创建安全策略来管理数据安全，请参见 [Apache Ranger 官网](https://ranger.apache.org/)。

## 权限控制方式

StarRocks 与 Apache Ranger 集成，提供以下权限控制方式：

- 在 Ranger 中创建 StarRocks Service，实现权限控制。当用户访问 StarRocks 内部表、外部表或其他对象时，会根据 StarRocks Service 中配置的访问策略进行访问控制。
- 当用户访问外部数据源时，Apache Ranger 上的外部服务（如 Hive 服务）可以复用进行访问控制。StarRocks 可以将 Ranger 服务与不同的外部目录进行匹配，并基于数据源对应的 Ranger 服务实现访问控制。

StarRocks 与 Apache Ranger 集成后，您可以实现以下访问控制模式：

- 使用 Apache Ranger 统一管理对 StarRocks 内表、外表和所有对象的访问。
- 使用 Apache Ranger 管理对 StarRocks 内部表和对象的访问。对于外部目录，在 Ranger 上重用相应外部服务的策略进行访问控制。
- 使用 Apache Ranger 通过重用与外部数据源对应的服务来管理对外部目录的访问。使用 StarRocks RBAC 权限系统管理对 StarRocks 内部表和对象的访问。

**身份验证过程**

- 您也可以使用 LDAP 进行用户认证，然后使用 Ranger 同步 LDAP 用户并为其配置访问规则。StarRocks 也可以通过 LDAP 完成用户登录认证。
- 当用户发起查询时，StarRocks 会解析查询语句，并将用户信息和所需权限传递给 Apache Ranger。Ranger 根据对应 Service 中配置的访问策略判断用户是否具有所需的权限，并将认证结果返回给 StarRocks。如果用户有访问权限，StarRocks 会返回查询数据；否则，StarRocks 将返回错误。

## 先决条件

- 已安装 Apache Ranger 2.1.0 或更高版本。有关如何安装 Apache Ranger 的说明，请参阅 Ranger [快速入门](https://ranger.apache.org/quick_start_guide.html)。
- 所有 StarRocks FE 机器都可以访问 Apache Ranger。您可以通过在每台 FE 机器上运行以下命令来检查这一点：

   ```SQL
   telnet <ranger-ip> <ranger-host>
   ```

   如果显示 `Connected to <ip>`，则表示连接成功。

## 集成过程

### 安装 ranger-starrocks-plugin

目前，StarRocks 支持：

- 通过 Apache Ranger 创建访问策略、屏蔽策略和行级筛选策略。
- Ranger 审核日志。

1. 在 Ranger Admin 目录 `ews/webapp/WEB-INF/classes/ranger-plugins` 中创建 `starrocks` 文件夹。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. 下载 [plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar](https://www.starrocks.io/download/community) 和 [mysql-connector-j](https://dev.mysql.com/downloads/connector/j/)，并将它们放在 `starrocks` 文件夹中。

3. 重新启动 Ranger Admin。

   ```SQL
   ranger-admin restart
   ```

### 在 Ranger Admin 上配置 StarRocks 服务

1. 将 [ranger-servicedef-starrocks.json](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json) 复制到 StarRocks FE 机器或 Ranger 机器的任意目录下。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. 以 Ranger 管理员身份执行以下命令，添加 StarRocks Service。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. 访问 `http://<ranger-ip>:<ranger-host>/login.jsp` Apache Ranger 页面。页面上将显示 STARROCKS 服务。

   ![home](../assets/ranger_home.png)

4. 点击 **STARROKCS** 后面的加号（+）来配置 StarRocks Service。

   ![service detail](../assets/ranger_service_details.png)

   ![property](../assets/ranger_properties.png)

   - `Service Name`：必须输入服务名称。
   - `Display Name`：要在 STARROCKS 下为服务显示的名称。如果未指定，`Service Name` 将显示。
   - `Username` 和 `Password`：FE 用户名和密码，用于在创建策略时自动完成对象名称。这两个参数不会影响 StarRocks 和 Ranger 之间的连通性。如果要使用自动完成功能，请至少为一个用户配置 `db_admin` 已激活的角色。
   - `jdbc.url`：输入 StarRocks FE 的 IP 地址和端口。

   配置示例如下图所示。

   ![example](../assets/ranger_show_config.png)

   添加的服务如下图所示。

   ![added service](../assets/ranger_added_service.png)

5. 单击 **测试连接**，测试连通性，连接成功后保存。
6. 在 StarRocks 集群的每台 FE 机器上，在 `fe/conf` 文件夹中创建 [ranger-starrocks-security.xml](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml) 并复制内容。您必须修改以下两个参数并保存修改：

   - `ranger.plugin.starrocks.service.name`：修改步骤 4 中创建的 StarRocks 服务名称。
   - `ranger.plugin.starrocks.policy.rest.url`：更改为 Ranger 管理员的地址。

   如果需要修改其他配置，请参考 Apache Ranger 的官方文档。例如，您可以修改 `ranger.plugin.starrocks.policy.pollIntervalM` 以更改拉取策略更改的时间间隔。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- 将其更改为 StarRocks 服务名称。
           <description>
               包含此 StarRocks 实例策略的 Ranger 服务的名称
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- 将其更改为 Ranger Admin 地址。
           <description>
               到 Ranger Admin 的 URL
           </description>
       </property>   
   ...
   ```

7. 将配置添加到所有 FE 配置文件中，`access_control = ranger`。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. 重启所有 FE 机器。

   ```SQL
   -- 切换到 FE 文件夹。
   cd..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 重用其他服务来控制对外部表的访问

对于外部目录，您可以复用外部服务（例如 Hive 服务）进行访问控制。StarRocks 支持为不同的 Catalog 匹配不同的 Ranger 外部服务。当用户访问外部表时，系统会根据外部表对应的 Ranger Service 的访问策略实现访问控制。

1. 将 Hive 的 Ranger 配置文件 `ranger-hive-security.xml` 复制到所有 FE 机器的 `fe/conf` 文件夹中。
2. 重启所有 FE 机器。
3. 配置外部目录。

   - 创建外部目录时，请添加属性 `"ranger.plugin.hive.service.name"`。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
            "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
            "ranger.plugin.hive.service.name" = "hive_catalog_1"
        )
      ```

   - 还可以将此属性添加到现有外部目录。
  
       ```SQL
       ALTER CATALOG hive_catalog_1
       SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
       ```

此操作将现有目录的身份验证方法更改为基于 Ranger 的身份验证。

## 下一步做什么

添加 StarRocks 服务后，您可以单击该服务，为该服务创建访问控制策略，并为不同的用户或用户组分配不同的权限。当用户访问 StarRocks 数据时，会根据这些策略实施访问控制。
