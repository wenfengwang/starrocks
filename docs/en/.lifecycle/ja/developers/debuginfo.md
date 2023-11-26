---
displayed_sidebar: "Japanese"
---

# デバッグにはdebuginfoファイルを使用します

## 説明の変更

v2.5以降、StarRocksのインストールパッケージからBEのdebuginfoファイルが削除され、インストールパッケージのサイズとスペース使用量が削減されました。[StarRocksのウェブサイト](https://www.starrocks.io/download/community)で2つのパッケージを確認できます。

![debuginfo](../assets/debug_info.png)

この図では、`Get Debug Symbol files`をクリックしてdebuginfoパッケージをダウンロードすることができます。`StarRocks-2.5.10.tar.gz`はインストールパッケージであり、このパッケージをダウンロードするには**Download**をクリックします。

この変更は、StarRocksのダウンロード動作や使用に影響を与えません。クラスター展開とアップグレードのためには、インストールパッケージのみをダウンロードすることができます。debuginfoパッケージは、開発者がGDBを使用してプログラムをデバッグするためのものです。

## 注意事項

デバッグにはGDB 12.1以降が推奨されます。

## debuginfoファイルの使用方法

1. debuginfoパッケージをダウンロードして展開します。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注意**
    >
    > `<sr_ver>`をダウンロードしたいStarRocksのインストールパッケージのバージョン番号に置き換えてください。

2. GDBデバッグを実行する際に、debuginfoファイルをロードします。

    - **方法1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    この操作により、デバッグ情報ファイルが実行可能ファイルに関連付けられます。

    - **方法2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

debuginfoファイルはperfとpstackともうまく動作します。追加の操作なしでperfとpstackを直接使用することができます。
