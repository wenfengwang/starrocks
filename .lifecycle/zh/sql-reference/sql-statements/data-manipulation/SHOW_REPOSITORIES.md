---
displayed_sidebar: English
---

# 查看仓库

## 描述

查看在 StarRocks 中创建的仓库。

## 语法

```SQL
SHOW REPOSITORIES
```

## 返回值

|返回|说明|
|---|---|
|RepoId|存储库 ID。|
|RepoName|存储库名称。|
|CreateTime|存储库创建时间。|
|IsReadOnly|如果存储库是只读的。|
|位置|远程存储系统中存储库的位置。|
|代理|用于创建存储库的代理。|
|ErrMsg|连接到存储库期间出现错误消息。|

## 示例

示例1：查看在 StarRocks 中创建的仓库。

```SQL
SHOW REPOSITORIES;
```
