---
displayed_sidebar: "Japanese"
---

# デバッグ情報ファイルを使用する

## 変更の説明

v2.5以降、BEのデバッグ情報ファイルは、StarRocksインストールパッケージから取り除かれ、インストールパッケージのサイズとスペースの使用量を削減します。[StarRocksのウェブサイト](https://www.starrocks.io/download/community)で2つのパッケージを見ることができます。

![debuginfo](../assets/debug_info.png)

この図では、「デバッグシンボルファイルを取得」をクリックして、デバッグ情報パッケージをダウンロードできます。「StarRocks-2.5.10.tar.gz」はインストールパッケージであり、このパッケージをダウンロードするには**ダウンロード**をクリックします。

この変更は、StarRocksのダウンロード動作や使用に影響しません。クラスター展開およびアップグレードのためには、インストールパッケージのみをダウンロードできます。デバッグ情報パッケージは、GDBを使用してプログラムをデバッグする開発者のためのものです。

## 注意事項

デバッグには、GDB 12.1以降が推奨されます。

## デバッグ情報ファイルの使用方法

1. デバッグ情報パッケージをダウンロードして展開します。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注記**
    >
    > `<sr_ver>`をダウンロードしたいStarRocksインストールパッケージのバージョン番号で置き換えてください。

2. GDBデバッグを実行する際に、デバッグ情報ファイルをロードします。

    - **方法1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    この操作により、デバッグ情報ファイルが実行可能ファイルに関連付けられます。

    - **方法2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

デバッグ情報ファイルはperfやpstackともうまく機能します。追加の操作なしでperfやpstackを直接使用できます。