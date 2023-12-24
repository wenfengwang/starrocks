---
displayed_sidebar: English
---

# 显示例行导入作业

## 示例

1. 显示名称为 test1 的所有例行导入作业（包括已停止或已取消的作业）。结果是一行或多行。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 显示当前正在运行的例行导入作业，名称为 test1

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. 在 example_db 中显示所有例行导入作业（包括已停止或已取消的作业）。结果是一行或多行。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. 在 example_db 中显示所有正在运行的例行导入作业

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. 在 example_db 中显示当前正在运行的名为 test1 的例行导入作业

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. 在 example_db 中显示名称为 test1 的所有例行导入作业（包括已停止或已取消的作业）。结果是一行或多行

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```
