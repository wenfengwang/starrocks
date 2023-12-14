---
displayed_sidebar: "Chinese"
---

# 显示仓库

## 描述

查看在StarRocks中创建的仓库。

## 语法

```SQL
SHOW REPOSITORIES
```

## 返回

| **返回**   | **描述**                                                   |
| ---------- | -------------------------------------------------------- |
| RepoId     | 仓库ID。                                                   |
| RepoName   | 仓库名称。                                                 |
| CreateTime | 仓库创建时间。                                             |
| IsReadOnly | 仓库是否为只读。                                           |
| Location   | 仓库在远程存储系统中的位置。                              |
| Broker     | 用于创建仓库的Broker。                                     |
| ErrMsg     | 连接仓库时的错误信息。                                     |

## 示例

示例1：查看在StarRocks中创建的仓库。

```SQL
SHOW REPOSITORIES;
```