---
displayed_sidebar: "Japanese"
---

# ルーチンのロードを表示

## 例

1. テスト1という名前のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。その結果は1行以上です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. テスト1という名前の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. example_db内のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。その結果は1行以上です。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. example_db内のすべての実行中のルーチンインポートジョブを表示します。

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. example_db内の名前がtest1で現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. example_db内のテスト1という名前のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。その結果は1行以上です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```