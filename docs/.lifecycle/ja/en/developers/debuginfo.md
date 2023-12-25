---
displayed_sidebar: English
---

# デバッグ用の debuginfo ファイルを使用する

## 変更の説明

v2.5 以降、BE の debuginfo ファイルは、インストールパッケージのサイズとスペースの使用量を削減するために StarRocks インストールパッケージから削除されました。[StarRocks のウェブサイト](https://www.starrocks.io/download/community)で二つのパッケージを確認できます。

![debuginfo](../assets/debug_info.png)

この図では、`Get Debug Symbol files` をクリックして debuginfo パッケージをダウンロードできます。`StarRocks-2.5.10.tar.gz` はインストールパッケージで、**Download** をクリックしてこのパッケージをダウンロードできます。

この変更は、StarRocks のダウンロードや使用に影響を与えません。クラスタのデプロイメントやアップグレードのためだけにインストールパッケージをダウンロードすることができます。debuginfo パッケージは、GDB を使用してプログラムをデバッグする開発者のためのものです。

## 注意事項

デバッグには GDB 12.1 以降のバージョンを推奨します。

## debuginfo ファイルの使用方法

1. debuginfo パッケージをダウンロードして解凍します。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注記**
    >
    > `<sr_ver>` をダウンロードしたい StarRocks インストールパッケージのバージョン番号に置き換えてください。

2. GDB デバッグを行う際に debuginfo ファイルを読み込みます。

    - **方法 1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    この操作で、デバッグ情報ファイルが実行ファイルに関連付けられます。

    - **方法 2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

debuginfo ファイルは perf と pstack ともに適切に機能します。perf と pstack は追加操作なしで直接使用できます。
