---
title: MacBookを手に入れたので環境構築する
tags:
  - 'MacBook'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

## 概要

先日新しく手に入れた MacBook Air での初期設定の備忘録。

### スペック
<!-- textlint-disable -->
OS: macOS Ventura
shell: zsh
キーボード： US 配列
<!-- textlint-enable -->

<!-- textlint-disable -->
:::note warn
ツールをダウンロードする際に入力したコマンドを載せていますが、公式サイトへのリンクを貼りますので、正しいページであることをご確認の上、照らし合わせて入力することを推奨します。
:::
<!-- textlint-enable -->

<!-- textlint-disable -->
:::note warn
筆者は shell に zsh を使っています。 bash など他の shell を使っている場合参照するファイルや記法が異なるのでご自身でご確認ください。
:::
<!-- textlint-enable -->

## ライブ変換を OFF

https://support.apple.com/ja-jp/guide/japanese-input-method/jpim10265/mac

<!-- TODO: before after img -->

一度日本語入力モードに切り替え、画面右上のメニューバーから「あ」をクリックし、「ライブ変換」のチェックを外します。

<!-- TODO: img -->

## ファンクションキー/地球儀キーでの言語切り替えを OFF

https://zenn.dev/prgskater/articles/00bfb00bcf3df3

後述する `Karabiner-Elements` を使って、スペースキーの左右のコマンドキーで入力ソースの切り替えを行うので OFF にしました。

## スクリーンショットの設定

https://next-blog.net/mac-screenshots/

### 保存先を変更

デフォルトだとデスクトップに保存されるので保存先を変更します。
finder からアプリケーション → ユーティリティ → スクリーンショットを選択し開きます。

<!-- TODO: img -->

オプションから任意の保存先を選択します。

### ファイル名を変更

デフォルトでは `スクリーンショット 2023-11-14 15.41.08` のように `スクリーンショット` + `日付` + `時刻` になっています。
全角文字が入っていると悪いことが起きそうなのと、見分けやすいようにスクリーンショットを撮影した際に付けられるファイル名を変更します。

```
// ファイル名の日本語での`スクリーンショット`の部分を `screenshot` へ変更する
$ defaults write com.apple.screencapture name screenshot
// ファイル名から `日付` + `時刻` を除き、連番にする
$ defaults write com.apple.screencapture include-date -bool false
```

- screenshot.png
- screenshot 1.png
- screenshot 2.png

## 画面録画を GIF 形式にする

<!-- TODO: 設定する -->

## Chrome

https://www.google.com/intl/ja_jp/chrome/

Web サイトを閲覧するのに必要な Web ブラウザです。同一の Google アカウントに紐付いていれば別端末とブックマークや履歴を共有できるので便利です。

Mac ではデフォルトでは Safari が入っていますが、Chrome を使いたいのでダウンロードします。
デフォルトのブラウザに設定するのも簡単で、Chrome を開くとデフォルトのブラウザに設定するか聞かれるので、設定するを選択すれば Chrome 上でデフォルトブラウザの設定を行なってくれます。

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

1. 「Modifier keys」から「caps_lock」と「left_control」を選択します

#### For Japanese （日本語環境向けの設定） (rev 6)をインポートする

https://karabiner-elements.pqrs.org/docs/manual/configuration/configure-complex-modifications/

上記説明を参考にします。 Step2 の検索バーに `For Japanese` と入力し `For Japanese （日本語環境向けの設定） (rev 6)` の横の `import` をクリックし、あとは上記説明通り進めます。
Step4 でそれぞれのルールを enable(適用する)を決められるので、気に入ったルールがあれば取り入れてみましょう。

私の場合、`右コマンドキーを単体で押したときに、かなキーを送信、左コントロールキーを単体で押したときに、英数キーを送信する。 (rev 2)` を追加して、US 配列キーボードでのスペースキーの左右のコマンドキーの入力によってそれぞれ入力ソースを切り替えられるようにしました。

#### デバイスごとの設定

<!-- TODO: 設定する -->

## Alfred 5

https://www.alfredapp.com/

「Alfred」とは macOS で使えるランチャーアプリです。備え付けの「spotlight」のようなものです。

Web ページから dmg ファイルをダウンロードできます。

## DeepL

## homebrew

## git

https://git-scm.com/download/mac

## asdf

## node

## yarn

## Visual Studio Code

## hyper

## shell のカスタマイズ

## Docker

## Slack
