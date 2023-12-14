```yaml
---
displayed_sidebar: "Chinese"
---

# 显示文件

您可以执行SHOW FILE语句来查看存储在数据库中的文件的信息。

## 语法

```SQL
SHOW FILE [FROM database]
```

该语句返回的文件信息如下：

- `FileId`：文件的全局唯一ID。

- `DbName`：文件所属的数据库。

- `Catalog`：文件所属的类别。

- `FileName`：文件的名称。

- `FileSize`：文件的大小。单位为字节。

- `MD5`：用于检查文件的消息摘要算法。

## 示例

查看存储在`my_database`中的文件。

```SQL
SHOW FILE FROM my_database;
```