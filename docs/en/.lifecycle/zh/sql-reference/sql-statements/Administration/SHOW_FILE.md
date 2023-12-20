---
displayed_sidebar: English
---

# 显示文件

您可以执行 SHOW FILE 语句来查看存储在数据库中的文件的信息。

:::tip

对文件所属数据库具有任何权限的用户都可以执行此操作。如果文件属于某个数据库，那么所有有权访问该数据库的用户都可以使用这个文件。

:::

## 语法

```SQL
SHOW FILE [FROM database]
```

该语句返回的文件信息包括：

- `FileId`：文件的全局唯一标识符。

- `DbName`：文件所属的数据库名称。

- `Catalog`：文件所属的分类。

- `FileName`：文件的名称。

- `FileSize`：文件的大小，单位为字节。

- `MD5`：用于校验文件的消息摘要算法。

## 示例

查看 `my_database` 中存储的文件。

```SQL
SHOW FILE FROM my_database;
```