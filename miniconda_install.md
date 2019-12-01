# QIIME2: Wet研究者も使える微生物叢解析ツール

本ページは実験医学2019年12月号（羊土社）「クローズアップ実験法」の補足ページとなります。  


### MacにMiniconda（4.7.12）をインストールする方法
condaの公式サイトのインストール手順に沿います（Macにインストールしますが、Linuxでも手順はほぼ同様です）。
- 使用PCminiconda: MacBook Air (13-inch, 2017), macOS Mojave (10.14.6)
- [公式HP](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)にアクセスし、ダウンロードページに移動する。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_accessHP.png)
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_download.png)
- インストールファイルをダウンロードする。Macにはbashとpkgの2ファイル形式のインストーラーが用意されています。pkgファイルは一般的なアプリのようにインストールできますが、今回はターミナル操作に慣れる意味も込めてbashファイルを用いてインストールします。「Miniconda3 MacOSX 64-bit bash」をクリックすると自動的にダウンロードが始まります（今回は「ダウンロード」ディレクトリにダウンロードしたことにします）。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_download2.png)
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_download3.png)
- ターミナルの起動。アプリケーション->ユーティリティ->ターミナルをクリックして起動します。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_terminal.png)
- ダウンロードが正常に行われたかを確認します。下記のshasumコマンドを実行すると64桁の英数字が出力されます。この値とインストーラーページの「SHA256 hash」が同じであればファイルが正常にダウンロードされたことを示しています。もし、値が異なるようでしたらもう一度ダウンロードし直してください。
```sh
cd Downloads/
shasum -a 256 Miniconda3-latest-MacOSX-x86_64.sh 
```
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_hash.png)

- ターミナル上で下記コマンドを実行するとインストールが始まります。
```sh
bash Miniconda3-latest-MacOSX-x86_64.sh
```
- 「Please, press ENTER to continue」と尋ねられますので、Enter(Return)を押します。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_install1.png)
- Minicondaの使用ライセンスが表示されます。下矢印キーなどでスクロールしてください。最後までスクロールすると「Do you accept the license terms ?」と尋ねられますので、「yes」と入力しEnter（Return）を押します。インストールする場所を尋ねられますので、問題なければEnter（Return）を押します。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_install2.png)
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_install3.png)
- 様々なパッケージがインストールされ始めます。そばらくすると「Do you wish the installer ~」と尋ねられますので「yes」と入力しEnter（Return）を押します。インストールが完了すると「Thank you for installing Miniconda3!」と表示されます。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_install4.png)
- インストールを反映するため、一度ターミナルを閉じて再び開きます。
- 下記のように「conda list」とターミナル上で入力しEnter（Return）を押したときに、パッケージの一覧が表示されればMinicondaは正常にインストールされています。
![](https://github.com/t-tsukimi/Experimental_Medicine2019.12/blob/master/image/miniconda_mac_install5.png)