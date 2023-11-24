---
title: MacBookを手に入れたので環境構築をする
tags:
  - MacBook
private: false
updated_at: '2023-11-24T09:26:06+09:00'
id: 4ad4d592d400a94644aa
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

先日新しく手に入れた MacBook Air での初期設定の備忘録。
使用しているアプリやツールの説明とインストール方法を記していきます。

### スペック
<!-- textlint-disable -->
OS: macOS Ventura
shell: zsh
キーボード： US 配列
<!-- textlint-enable -->

:::note warn
ツールをダウンロードする際に入力したコマンドを載せていますが、公式サイトと照らし合わせて入力するようにしてください。
:::

## Chrome

https://www.google.com/intl/ja_jp/chrome/

Web サイトを閲覧するのに必要な Web ブラウザです。同一の Google アカウントに紐付いていれば別端末とブックマークや履歴を共有できるので便利です。

Mac ではデフォルトでは Safari が入っていますが、Chrome を使いたいのでダウンロードします。
デフォルトのブラウザに設定するのも簡単で、Chrome を開くとデフォルトのブラウザに設定するか聞かれるので、設定するを選択すれば Chrome 上でデフォルトブラウザの設定を行なってくれます。

## ライブ変換をOFF

https://support.apple.com/ja-jp/guide/japanese-input-method/jpim10265/mac

ライブ変換を ON にしていると、日本語を入力中に自動で漢字に変換します。使い勝手が微妙なので OFF にしています。

一度日本語入力モードに切り替え、画面右上のメニューバーから「あ」をクリックし、「ライブ変換」のチェックを外します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/d1c1d835-415b-1991-1c7a-cb55411c038b.png)

## ファンクションキー/地球儀キーでの言語切り替えをOFF

後述する `Karabiner-Elements` を使って、スペースキーの左右のコマンドキーで入力ソースの切り替えを行うので OFF にしました。

https://zenn.dev/prgskater/articles/00bfb00bcf3df3

`システム環境設定` → `キーボード` → `地球儀キーを押して` の右のセレクトボックスを `入力ソースを変更` から `何もしない` へと変更します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/3d1acffa-cb43-53e5-fb82-5079e9371566.png)

## スクリーンショットの設定

### 保存先を変更

デフォルトだとデスクトップに保存されるので保存先を変更します。
`finder` からアプリケーション → ユーティリティ → スクリーンショットを選択し開きます。

オプションから任意の保存先を選択します。

![スクリーンショット 2023-11-14 15.41.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/c8b1671c-900e-aee8-42ca-e9551c7aa9b3.png)


### ファイル名を変更

デフォルトでは `スクリーンショット 2023-11-14 15.41.08` のように `スクリーンショット` + `日付` + `時刻` になっています。
全角文字が入っていると悪いことが起きそうなのと、見分けやすいようにスクリーンショットを撮影した際に付けられるファイル名を変更します。

ターミナルアプリを開き以下のコマンドを入力します。

```sh: shell
// ファイル名の日本語での`スクリーンショット`の部分を `screenshot` へ変更する
$ defaults write com.apple.screencapture name screenshot
// ファイル名から `日付` + `時刻` を除き、連番にする
$ defaults write com.apple.screencapture include-date -bool false
```

結果として以下のようなファイル名になります。
- screenshot
- screenshot 1
- screenshot 2

### スクリーンショットをクリップボードに即時保存する

https://zenn.dev/masaaania/articles/fabfe969f4f0e3#discuss

こちらの記事を参考にしました。
かいつまんで説明すると、 macOS にプリインストールされている ` Automator` というアプリケーションを使って、スクリーンショットを撮った時に実行するプログラムを書くことができます。これによりスクリーンショットを撮った後にクリップボードへと保存をします。


## Karabiner-Elements

https://karabiner-elements.pqrs.org/

Karabiner-Elements は macOS のキーボードをカスタマイズするためのツールです。
すべて自分好みにカスタマイズできますし、用意されている `complex rules` を import すれば難しい設定をせずに使うこともできます。

### インストール方法

`Karabiner-Elements` の Web サイトからアプリケーションを入手できます。

https://karabiner-elements.pqrs.org/docs/getting-started/installation/

### カスタマイズ

#### 「caps lock」キーを「control」キーにする

1. `Karabiner-Elements` を開き「Simple Modifications」の「Add item」をクリックします

![スクリーンショット 2023-11-14 16.03.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/75a72d80-46ed-cf9a-c10e-48389d87782d.png)

1. 「Modifier keys」から、2 つあるセレクトボックスの左に「caps_lock」と、右に「left_control」を選択します

![スクリーンショット 2023-11-14 16.05.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/640d0144-185a-abfb-9a68-0ed3ad56403c.png)

![スクリーンショット 2023-11-14 16.05.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/ee9f5002-3e3a-7b6a-c364-e0191ca6fc66.png)

これにより「caps lock」キーを押した時に「control」キーとして反応するようになります。

#### For Japanese （日本語環境向けの設定） (rev 6)をインポートする

https://karabiner-elements.pqrs.org/docs/manual/configuration/configure-complex-modifications/

上記説明を参考にします。 Step2 の検索バーに `For Japanese` と入力し `For Japanese （日本語環境向けの設定） (rev 6)` の横の `import` をクリックし、あとは上記説明通り進めます。
Step4 でそれぞれのルールを enable(適用する)を決められるので、気に入ったルールがあれば取り入れてみましょう。

私の場合、`右コマンドキーを単体で押したときに、かなキーを送信、左コントロールキーを単体で押したときに、英数キーを送信する。 (rev 2)` を追加して、US 配列キーボードでのスペースキーの左右のコマンドキーの入力によってそれぞれ入力ソースを切り替えられるようにしました。

## Alfred 5

https://www.alfredapp.com/

「Alfred」とは macOS で使えるランチャーアプリです。備え付けの「spotlight」のようなものです。

Web ページから dmg ファイルをダウンロードできます。

使い方の一例ですが、デフォルトの設定で `option` + `space` を同時に押すことで使用でき、検索バーに起動したいアプリ名を入力すると起動してくれます。

## Homebrew

https://brew.sh

macOS または Linux 用のパッケージマネージャーです。Homebrew を使うことにより、macOS に初期から入っていない Python, Node.js といったパッケージや Google Chrome, Slack や Zoom といったアプリケーションをインストールできます。

公式に掲載されているコマンドでインストールできます。

```sh: shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

パスワードを聞かれるので Mac のログインパスワードを入力します。

M2 チップの Mac だと手動で PATH を通す必要があります。

```sh: shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/{User name}/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

私は `Homebrew` を使って `git`, `asdf`, `hyper` などをインストールしました。

## git

https://git-scm.com/

言わずと知れたソースコードのバージョン管理システムです。

https://git-scm.com/download/mac

インストールに `Homebrew` を使います。

```sh: shell
$ brew install git
```

### ユーザー設定

`git config` コマンドを使って、Git コミットと関連付けられている名前を変更できます。 
ユーザー名と Email アドレスを設定します。

```sh: shell
$ git config --global user.name "test"
$ git config --global user.email "test@example.com"
```

設定されたことを確認します。
```sh: shell
$ git config user.name 
> test
$ git config user.email
> test@example.com
```


## asdf

https://asdf-vm.com/

`asdf` はツールのバージョン管理ツールです。すべてのツールのバージョン定義は 1 つのファイル（.tool-versions）に含まれており、プロジェクトの Git リポジトリにチェックインしてチームで共有することで、全員がまったく同じバージョンのツールを使っていることを確認できるようになります。

`asdf` を使って `node` や `golang` などをインストールして使えるようにできます。

`Homebrew` を使ってインストールしました。

```sh: shell
brew install asdf
```

PATH を通します。

```sh: shell
echo -e "\n. $(brew --prefix asdf)/libexec/asdf.sh" >> ${ZDOTDIR:-~}/.zshrc
```

## Node.js

`JavaScript` のランタイムである `Node.js` をインストールします。
これがないと `JavaScript` をサーバーで動かすことができません。

`asdf` を使い `Node.js` をインストールします。

1. 依存関係をインストールインストールする
    ```sh: shell
    $ brew install gpg gawk
    ```

1. プラグインをインストールする

    https://github.com/asdf-vm/asdf-nodejs

    ```sh: shell
    $ asdf plugin add nodejs https://github.com/asdf-vm/asdf-nodejs.git
    ```

1. 最新安定バージョンの確認

    ```sh: shell
    $ asdf latest nodejs
    ```
1. 最新安定バージョンをインストール

    ```sh: shell
    $ asdf install nodejs latest
    ```

1. インストールした Node.js をグローバルに設定
    ```sh: shell
    $ asdf global nodejs latest
    ```

global には `Node.js` の最新版を設定しましたが特定のリポジトリで別のバージョンを使うのも簡単です。

1. インストール可能なバージョンを調べる

    ```sh: shell
    $ asdf list-all nodejs
    ```

1. `18.18.2` をインストールする
    ```sh: shell
    $ asdf install nodejs 18.18.2
    ```

1. 設定したリポジトリへ移動し、 Node.js のバージョンを変更する

    ```sh: shell
    $ cd /path/to
    $ asdf local nodejs 18.18.2
    ```

## yarn

https://chore-update--yarnpkg.netlify.app/ja/

 `npm` と互換性がある Node.js のパッケージマネージャーです。

公式では `Homebrew` を使ってインストールしていますが、私は `asdf` を使いました。
`asdf` を使う場合の説明は `asdf-yarn` の GitHub リポジトリの README にあります。

https://github.com/twuni/asdf-yarn

```sh: shell
$ asdf plugin-add yarn
$ asdf install yarn latest
$ asdf global yarn latest
```

## shell のカスタマイズ

macOS のデフォルトシェルである `zsh` を使っています。

:::note warn
筆者は shell に zsh を使っています。 bash など他の shell を使っている場合参照するファイルや記法が異なります。
:::

以下のコマンドで現在使っているシェルが何であるかを確認できます。

```sh: shell
$ echo $SHELL                                                                              
> /bin/zsh
```

詳しい zsh のカスタマイズを別記事に起こしました。

https://qiita.com/diskszk/items/ea6f5e09d9a64b84308e

## Docker

https://docs.docker.com/desktop/install/mac-install/

Docker の公式サイトから `Docker Desktop for Mac with Apple sillicon` を選択してダウンロードしました。
適宜自分のマシンに合ったものを選択しましょう。
