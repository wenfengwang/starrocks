---
displayed_sidebar: "Japanese"
---

# ルーチンのロードを表示

## 例

1. テスト1という名前のルーチンのすべてのインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。 結果は1行以上です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 現在実行中のルーチンインポートジョブを名前test1で表示する

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. example_db内のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。 結果は1行以上です。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. example_db内で実行中のすべてのルーチンインポートジョブを表示します

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. example_db内の名前がtest1の現在実行中のルーチンインポートジョブを表示します

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. example_db内の名前がtest1のルーチンインポートジョブすべて（停止またはキャンセルされたジョブを含む）を表示します。 結果は1行以上です。 

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```