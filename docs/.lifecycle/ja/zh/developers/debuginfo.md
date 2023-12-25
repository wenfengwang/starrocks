---
displayed_sidebar: Chinese
---

# debuginfo ファイルを使用したデバッグ

StarRocks のバイナリパッケージのディスク使用量を減らすために、バージョン 2.5 から、StarRocks BE のバイナリに含まれるデバッグ情報 debuginfo ファイルを分離し、別途ダウンロードできるようにしました。

一般ユーザーにとって、この変更は日常使用には影響しません。引き続きバイナリパッケージのみをダウンロードして、デプロイやアップグレードを行うことができます。debuginfo パッケージは、開発者が GDB を使用してプログラムをデバッグする際にのみ必要です。[StarRocks 公式ウェブサイト](https://www.starrocks.io/download/community)から対応するバージョンの debuginfo パッケージをダウンロードできます。

![debuginfo](../assets/debug_info.png)

## 注意事項

GDB 12.1 以上のバージョンを使用してデバッグすることを推奨します。

## 使用方法

1. debuginfo パッケージをダウンロードして解凍します。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **説明**
    >
    > 上記のコマンドラインで `<sr_ver>` を StarRocks の対応するバージョン番号に置き換えてください。

2. GDB debug を行う際に debug info をインポートします。

    - 方法一

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    上記の操作により、debug info と実行ファイルが自動的に関連付けられ、GDB debug 時に自動的に関連付けてロードされます。

    - 方法二

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

perf や pstack については、追加操作なしで直接使用できます。
