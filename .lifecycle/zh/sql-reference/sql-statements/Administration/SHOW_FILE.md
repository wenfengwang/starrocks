---
displayed_sidebar: English
---

# 查看文件

您可以执行 SHOW FILE 语句来查看存储在数据库中的文件信息。

:::提示

任何对文件所属数据库有权限的用户都可以进行此操作。如果文件隶属于某个数据库，那么所有能够访问该数据库的用户均可使用这个文件。

:::

## 语法

```SQL
SHOW FILE [FROM database]
```

此语句返回的文件信息包括：

- FileId：文件的全球唯一标识符。

- DbName：文件所属的数据库名称。

- Catalog：文件所属的分类。

- FileName：文件的名字。

- FileSize：文件的大小，单位为字节。

- MD5：用来校验文件的消息摘要算法。

## 示例

查看存储在 my_database 中的文件。

```SQL
SHOW FILE FROM my_database;
```
