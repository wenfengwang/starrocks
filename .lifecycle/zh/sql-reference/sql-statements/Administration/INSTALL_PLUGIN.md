---
displayed_sidebar: English
---

# 安装插件

## 说明

此语句用于安装插件。

:::提示

该操作需要 **SYSTEM** 级别的 **PLUGIN** 权限。您可以依照 [GRANT](../account-management/GRANT.md) 中的指南来授予相应权限。

:::

## 语法

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

支持以下三种来源类型：

```plain
1. An absolute path that directs to a zip file
2. An absolute path that directs to a plugin directory 
3. A http or https download link that directs to a zip file
```

PROPERTIES 支持对插件的某些配置进行设置，例如设定 zip 文件的 md5sum 值等。

## 示例

1. 从本地 zip 文件安装插件：

   ```sql
   INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
   ```

2. 从本地路径安装插件：

   ```sql
   INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
   ```

3. 下载并安装插件：

   ```sql
   INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
   ```

4. 下载并安装插件，同时设定 zip 文件的 md5sum 值：

   ```sql
   INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
   ```
