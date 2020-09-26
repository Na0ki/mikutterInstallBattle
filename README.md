# MikutterInstallBattle
macOS Sierra での mikutter のインストール手順。  
前提条件として以下のものが必要であるため、事前にインストールして再起動しておくこと。  
なおXQuartzのインストール後は再起動します。
* [homebrew](https://brew.sh/index_ja.html)
* [XQuartz](https://www.xquartz.org/)


## 目次
<!-- TOC -->

- [MikutterInstallBattle](#mikutterinstallbattle)
    - [目次](#%E7%9B%AE%E6%AC%A1)
    - [インストール環境](#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E7%92%B0%E5%A2%83)
    - [mikutter のダウンロー<br>ド](#mikutter-%E3%81%AE%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BCbr%E3%83%89)
    - [ruby の用意](#ruby-%E3%81%AE%E7%94%A8%E6%84%8F)
    - [mikutter の依存ライブラリのインストール](#mikutter-%E3%81%AE%E4%BE%9D%E5%AD%98%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)
    - [日本語入力](#%E6%97%A5%E6%9C%AC%E8%AA%9E%E5%85%A5%E5%8A%9B)
    - [Special Thanks](#special-thanks)

<!-- /TOC -->

## インストール環境  
あくまで参考です。

| name     | version          |
| -------- | ---------------- |
| macOS    | 10.13.2          |
| Homebrew | 1.5.0-6-gd14fd49 |
| XQuartz  | 2.7.11           |

## mikutter のダウンロー<br>ド  
1. mikutterユーザーのほとんどは `git` の `develop` を使っているんだ。（当社比）
    ```
    $ brew install git
    ```
1. mikutter をインストールしたいディレクトリにいき、以下のコマンドでダウンロードする。
    ```
    $ git clone git://toshia.dip.jp/mikutter.git
    $ git checkout develop
    ```

## ruby の用意  
1. macOS にはデフォルトで `ruby` が入っていますが、ここでは任意のバージョンにするために　`rbenv` をインストールする。
    ```shell
    $ brew install rbenv
    ```
1. `rbenv` をインストールしたら、使っているシェルの `rc` ファイルに以下を追記する。( `.bashrc`, `.zshrc` など )
    ```
    export PATH=$HOME/.rbenv/bin:$PATH
    eval "$(rbenv init - zsh)"
    ```
1. `rbenv` を使って `ruby` をインストールします。  
    * オプション  
        のちのちpryとか使いたい人はpry日本語文字化け回避のため以下を実行する。
        ```shell
        $ brew install readline
        $ brew link readline --force
        $ RUBY_CONFIGURE_OPTS="--with-readline-dir=$(brew --prefix readline)"
        ```
    ruby のインストール
    ```shell
    $ rbenv install 2.5.0
    $ cd [mikutterのダウンロードディレクトリ]
    $ rbenv local 2.5.0
    ```

## mikutter の依存ライブラリのインストール  
以下、全て mikutter のダウンロードディレクトリにいる前提で進める。  
`$ cd /path/to/mikutter`
1. Xcode のコマンドラインツールをインストール  
    そういえば、homebrew のインストール時に入れてたというやつ。
    すでに入ってる人は次の手順に進む。  
    これがないと `nokogiri` やらのインストールに失敗する。（ thx [@rettar5](https://twitter.com/rettar5/status/871323979079835648) ）
    ```
    $ xcode-select --install
    ```
1. bundler をインストール
    ```shell
    $ gem install bundler
    ```
1. `gtk+` と `cairo` のインストール  
    この手順はshibafu528氏が新たにまとめてくれました。  
    下記を参考にしてください。  
    https://www.shibafu528.info/2020/09/macmikutter.html
1. `pango` のインストール
    ```
    $ brew install pango
    ```
1. 依存 `gem` のインストール
    ```
    $ bundle install
    ```

## 日本語入力
1. MacUIM のインストール

    以下の GitHub のページから MacUIM をインストールする。
    * [MacUIM](https://github.com/e-kato/macuim/releases)

    インストーラがある Latest を使う。  
    インストールしようとするとセキュリティがうんちゃらと出るのでノリと雰囲気で乗り切る。（設定のセキュリティを見るのです）  
    なお、インストールし次第ログアウトを迫られるので、ログアウトしても問題ないようにしておくと吉。

1. 起動設定の追加

    MacUIM の起動設定を書いたりする。  
    * `/Library/LaunchAgents/` に [mikutter_env.plist](./config/mikutter_env.plist) を設置する。  
    パスの関係上 `sudo` で設置する必要がある。
    * 以下のコマンドを実行してロードに必要なスクリプトを作る。
        ```
        $ mkdir -p ~/.xinitrc.d
        $ touch ~/.xinitrc.d/uim-xim.sh
        ```
    * 上で作った `uim-xim.sh` に以下を記述して保存する。
        ```shell
        #!/bin/sh
        /Library/Frameworks/UIM.framework/Versions/Current/bin/uim-xim &
        ```
    * `uim-xim.sh` に実行権限を与える
        ```shell
        $ chmod +x ~/.xinitrc.d/uim-xim.sh
        ```
    * シェルの `rc` ファイルに以下の3行を追加します。
        ```
        export LANG=ja_JP.UTF-8
        export XMODIFIERS=@im=uim
        export GTK_IM_MODULE=uim
        ```
    * MacUIM の設定
        1. macOS のシステム環境設定を開き、MacUIM の設定を開く（ちょっと時間がかかる）
        1. 一般タブの入力方式を `mozc(ja)` にする
        1. uimタブの全体設定をする
            * 標準入力方式を設定する

                全体設定の `標準の入力方式を指定` にチェックし、`標準の入力方式` を `mozc` にする。
                ![標準の入力方式の指定](./resource/uim-general.png)
            * 遅延ローディングの無効化

                `高速起動のための遅延ローディングを有効にする` のチェックを外す。
                ![遅延ローディング](./resource/uim-general-loading.png)

        1. macOS を再起動して終わり

            .　　　　　∞  
        　　　　　∫  
        　　,';:☜;.`,ਊ,,;';,;☞,.՞  
        　　՞  
        　　　　　　　　　お わ り

## インストール手順変更履歴
1. `libidn` のインストール  
    2020/09/26時点の最新のmikutterではtwitterプラグインを使用する際にも `libidn` には依存しなくなったため、下記のコマンド実行を手順から除外した。
    ```
    $ brew install libidn
    ```
1. `gtk+` と `cairo` のインストール  
    2020/09/26時点ではこの手順では足らなかったため、代わりにshibafu528の記事へのリンクに替えた。
    1. `XQuartz` 向けに `Homebrew` のパッケージを少しいじる。
        ```shell
        $ brew edit gtk+
        ```
        エディタが起動するので次のパラメータの行をコメントアウトする。
        ```
        "--with-gdktarget=quartz"
        ```
    1. `cairo` のインストール
        ```shell
        $ brew install cairo --with-x11
        ```
    1. `gtk+` のインストール
        ```
        $ brew install gtk+ --build-from-source
        ```

## Special Thanks
* 参考にした
    - [Mavericksで動いてたmikutterをYosemiteでも動くようにする方法](http://moguno.hatenablog.jp/entry/2014/11/23/095157) by moguno
    - [mikutter Advent Calendar 2013 Day2](http://akkiesoft.hatenablog.jp/entry/20131202/1385969580) by Akkiesoft
    - [@rettar5 のインストールバトル実況](https://twitter.com/rettar5/status/871323979079835648) by rettar5
    - [Macを買ってmikutterを入れたときのメモ](https://www.shibafu528.info/2020/09/macmikutter.html) by shibafu528

* 林檎社

    意味不明な挙動を連発することで私に OS のクリーンインストールを促し、本ドキュメントの作成に最適な環境にすることを強いてくれたことに多大な感謝をいたします。
