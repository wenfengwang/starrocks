---
displayed_sidebar: "Chinese"
---

# 显示例行载入

## 示例

1. 显示所有名为test1的例行导入作业（包括已停止或取消的作业）。结果为一个或多个行。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 显示当前正在运行的名为test1的例行导入作业

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. 显示所有在example_db中的例行导入作业（包括已停止或取消的作业）。结果为一个或多个行。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. 显示在example_db中正在运行的所有例行导入作业

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. 显示在example_db中名为test1的当前正在运行的例行导入作业

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. 在example_db中显示所有名为test1的例行导入作业（包括已停止或取消的作业）。结果为一个或多个行。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```