---
displayed_sidebar: English
---

# 显示仓库

## 描述

查看在 StarRocks 中创建的存储库。

## 语法

```SQL
SHOW REPOSITORIES
```

## 返回

| **返回** | **描述**                                          |
| ---------- | -------------------------------------------------------- |
| RepoId     | 存储库 ID。                                           |
| RepoName   | 存储库名称。                                         |
| CreateTime | 存储库创建时间。                                |
| IsReadOnly | 存储库是否为只读。                          |
| Location   | 存储库在远程存储系统中的位置。 |
| Broker     | 用于创建存储库的代理。                    |
| ErrMsg     | 连接到存储库时出现的错误消息。       |

## 例子

示例 1：查看在 StarRocks 中创建的存储库。

```SQL
SHOW REPOSITORIES;
```
