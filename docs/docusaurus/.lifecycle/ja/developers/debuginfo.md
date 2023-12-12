---
displayed_sidebar: "Japanese"
---

# デバッグ情報ファイルを使用する

## 変更の説明

v2.5以降、BEのデバッグ情報ファイルは、StarRocksのインストールパッケージから削除され、インストールパッケージのサイズとスペース使用量を削減しています。[StarRocksのウェブサイト](https://www.starrocks.io/download/community)で2つのパッケージを確認できます。

![debuginfo](../assets/debug_info.png)

この図において、`デバッグシンボルファイルを取得`をクリックすると、デバッグ情報パッケージをダウンロードできます。`StarRocks-2.5.10.tar.gz`がインストールパッケージであり、このパッケージをダウンロードするには**ダウンロード**をクリックします。

この変更は、StarRocksのダウンロード動作や使用に影響を与えません。クラスター展開やアップグレードのためには、インストールパッケージだけをダウンロードすることができます。デバッグ情報パッケージは、GDBを使用してプログラムをデバッグするためのものです。

## 注意事項

デバッグにはGDB 12.1またはそれ以降が推奨されています。

## デバッグ情報ファイルの使用方法

1. デバッグ情報パッケージをダウンロードして展開します。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注意**
    >
    > `<sr_ver>`をダウンロードしたいStarRocksのインストールパッケージのバージョン番号に置き換えてください。

2. GDBデバッグを実行する際に、デバッグ情報ファイルを読み込みます。

    - **方法1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    この操作により、デバッグ情報ファイルが実行可能ファイルに関連付けられます。

    - **方法2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

デバッグ情報ファイルはperfやpstackともうまく機能します。追加の操作なしにperfとpstackを直接使用することができます。