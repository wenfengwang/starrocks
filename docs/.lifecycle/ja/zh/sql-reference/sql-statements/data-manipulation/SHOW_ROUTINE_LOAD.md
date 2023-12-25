---
displayed_sidebar: Chinese
---

# SHOW ROUTINE LOAD

## 機能

routine load タスクの情報を確認します。

## 例

1. 名前が test1 のすべてのルーチンインポートジョブを表示します（停止またはキャンセルされたジョブを含む）。結果は1行または複数行になります。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 名前が test1 の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. example_db における、すべてのルーチンインポートジョブを表示します（停止またはキャンセルされたジョブを含む）。結果は1行または複数行になります。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. example_db における、現在実行中のすべてのルーチンインポートジョブを表示します。

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. example_db における、名前が test1 の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. example_db における、名前が test1 のすべてのルーチンインポートジョブを表示します（停止またはキャンセルされたジョブを含む）。結果は1行または複数行になります。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```
