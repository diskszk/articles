---
title: zshのプロンプトをいい感じにしたい
tags:
  - Zsh
  - shell
private: false
updated_at: '2023-11-21T17:10:31+09:00'
id: ea6f5e09d9a64b84308e
organization_url_name: null
slide: false
ignorePublish: false
---
## 記事の内容
プロンプトを初期状態のからいい感じにする。

## 初期状態から変更する

![screenshot 2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/e111da35-c349-7b46-e14f-c76e48afaaec.png)

初期状態では `ユーザー名@ホスト名 現在いるディレクトリ % ` と表示されます。
おそらくホスト名は `ユーザー名-no-MacBook-Air` などとなっているのではないでしょうか。モノクロなのも相まってかなり見にくいです。

プロンプトの情報はシェル変数 `PROMPT` に保存されており、`echo $PROMPT` とコマンド入力するとその中身を教えてくれます。

```sh: shell
username@hostname ~ % echo $PROMPT 
%n@%m %1~ %# 
```
zsh 特有の Prompt Expansion が用いられていて分かりにくいですが、以下の変数を表示しています。

https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html

|Prompt Expansion |意味  |
|---|---|
|%n  |ユーザー名 |
|%m  |ホスト名  |
|%1~ | 現在いるディレクトリ|
|%# | 管理者権限の場合「＃」、それ以外の場合「%」|


次にプロンプトを変更する方法です。シェル変数 `PROMPT` の値を書き換えれば OK です。

```sh: shell
export $PROMPT="hoge@fuga > "
hoge@fuga > echo $PROMPT
# ↓ 出力結果
hoge@fuga > 
# プロンプト
hoge@fuga > 
```

ただしこれだけではターミナルを再起動すると元の状態に戻ります。

zsh の設定を~ディレクトリ配下にある `~/.zshrc` という設定ファイルに書き込み、保存することで再起動しても設定が保たれます。

たとえば。`.zshrc` に以下の行を追加すれば、ターミナルアプリを開いたときに挨拶をしてくれます。

```.zshrc
export $PROMPT="hoge@fuga > "
echo "hello!!"

# zshのPrompt Expansionを表示する場合 ``print -P`` を用いる
print -P "hell %n !!"
```
.zshrc を書き換えたら `source ~/.zshrc` とコマンド入力して `.zshrc` を読み込みむことでプロンプトの表示が更新されます。

```sh: shell
hoge@fuga > source ~/.zshrc
hello!!
hell username !!
oge@fuga >
```

## Prompt Expansionを試してみる

色々な表示方法を試してみたい場合、毎回 `.zshrc` を編集して保存とすると面倒なので、プロンプト上で一時的に変更する方法を試してみます。

https://qiita.com/syoshika_/items/0211c873475eb0d59e23

こちらの記事を参考にいろんな表示方法を試してみたいのでコマンドラインに入力していきます。
左から、%d: カレントディレクトリ、 %n ユーザー名、 %D: 日付(YY-MM-DD), %T: 時刻(hh:mm), %#: ルートユーザーの場合 `#`、それ以外の場合 `%` となります。

```sh: shell
hoge@fuga > export PROMPT="%%d: %d , %%n: %n, %%D: %D, %%T: %T %%#: %#"
%d: /Users/username , %n: username, %D: 23-11-20, %T: 22:19 %#: %
```

単に `print -P "%%d: %d"` のようにコマンド入力すれば以下のような出力結果になります。

```sh
%D: 23-11-20
```
## 色をつける

`%F{色名}hoge%f` というように `%F` と `%F` で囲むことで色を付けることができます。
`%K%k` で囲むことで背景色を変更します。

![screenshot 3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/80ba66e6-4921-07a3-f826-8ee2199df0c8.png)


## Gitリポジトリの状態を表示する

https://zenn.dev/sprout2000/articles/bd1fac2f3f83bc

こちらの記事の[6. プロンプトへ Git リポジトリの状態を表示する](https://zenn.dev/sprout2000/articles/bd1fac2f3f83bc#6.-%E3%83%97%E3%83%AD%E3%83%B3%E3%83%97%E3%83%88%E3%81%B8-git-%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%AE%E7%8A%B6%E6%85%8B%E3%82%92%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B) を参考に Git リポジトリの状態を表示する処理を追加していきます。

### インストール
```sh: shell
brew install zsh-git-prompt
```

`~/.zshrc` を開いて以下のコードを貼り付けます。

```.zshrc
+ source $(brew --prefix)/opt/zsh-git-prompt/zshrc.sh
+ export PROMPT='hoge@huga %~ $(git_super_status) > '
```

結果
![screenshot 4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/ae7734ff-94cf-ba40-826e-2e8d5b35d2f9.png)
左が追加前、右が追加後です。


## さいごに
さいごにたどり着いた zsh の設定を残してこの記事の結びとさせていただきます。
``` .zshrc
export LANG=ja_JP.UTF-8

# Gitリポジトリの状態を表示する https://github.com/olivierverdier/zsh-git-prompt
source $(brew --prefix)/opt/zsh-git-prompt/zshrc.sh

# Gitリポジトリ以外ではGitリポジトリの状態を表示しない
git_prompt() {
 if [ "$(git rev-parse --is-inside-work-tree 2> /dev/null)" = true ]; then
   export PROMPT="[%B%F{red}%n@%m%f%b:%F{green}%~%f]%F{cyan} $(git_super_status)%f"$'\n'"> " 
 else
   export PROMPT="[%B%F{red}%n@%m%f%b:%F{green}%~%f]%F{cyan}%f"$'\n'"> " 
 fi
}
precmd() {
 git_prompt
}

# 補完機能を有効にする
autoload -Uz compinit
compinit

# 自動補完を補助する https://github.com/zsh-users/zsh-autosuggestions
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh

# エイリアス
alias la='ls -a'
alias ll='ls -l'
alias python="python3"

# 補完のリストの、選択している部分を塗りつぶす
zstyle ':completion:*' menu select

# 入力ミスに対応する。
setopt correct

# 直前のコマンドと同じなら、履歴に残さない
setopt hist_ignore_dups

# 高機能なワイルドカード展開を使用する
setopt extended_glob

# Ctrl+J でコマンドライン入力中に改行を入れる
bindkey '^J' self-insert
```
