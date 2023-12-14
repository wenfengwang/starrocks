---
displayed_sidebar: "中文"
---

# 显示例行加载

## 功能

查看例行加载任务的信息。

## 示例

1. 显示名称为 test1 的所有例行加载任务（包括已停止或取消的任务）。结果为一行或多行。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 显示名称为 test1 的当前正在运行的例行加载任务

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. 显示 example_db 下，所有的例行加载任务（包括已停止或取消的任务）。结果为一行或多行。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. 显示 example_db 下，所有正在运行的例行加载任务

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. 显示 example_db 下，名称为 test1 的当前正在运行的例行加载任务

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. 显示 example_db 下，名称为 test1 的所有例行加载任务（包括已停止或取消的任务）。结果为一行或多行。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```