```yaml
---
displayed_sidebar: "中文"
---

# 显示仓库

## 功能

显示当前已创建的存储库。

## 语法

```SQL
SHOW REPOSITORIES
```

## 返回

| **返回**   | **说明**                |
| ---------- | ----------------------- |
| RepoId     | 存储库 ID。             |
| RepoName   | 存储库名称。            |
| CreateTime | 存储库创建时间。        |
| IsReadOnly | 是否为只读存储库。      |
| Location   | 远程存储系统路径。      |
| Broker     | 用于创建存储库的 Broker。|
| ErrMsg     | 存储库连接错误信息。    |

## 示例

示例一：显示已创建的存储库。

```SQL
SHOW REPOSITORIES;
```