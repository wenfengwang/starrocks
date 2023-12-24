---
displayed_sidebar: English
---

# 显示文件

您可以执行SHOW FILE语句来查看数据库中存储的文件信息。

:::提示

对于文件所属的数据库具有任何权限的用户都可以执行此操作。如果文件属于某个数据库，那么所有可以访问该数据库的用户都可以使用这个文件。

:::

## 语法

```SQL
SHOW FILE [FROM database]
```

该语句返回的文件信息如下：

- `FileId`：文件的全局唯一标识符。

- `DbName`：文件所属的数据库。

- `Catalog`：文件所属的类别。

- `FileName`：文件的名称。

- `FileSize`：文件的大小，单位为字节。

- `MD5`：用于检查文件的消息摘要算法。

## 例子

查看存储在`my_database`中的文件。

```SQL
SHOW FILE FROM my_database;
```