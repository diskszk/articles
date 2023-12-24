---
title: フロントエンドにおけるJavaScriptのビルド
tags:
  - JavaScript
  - フロントエンド
private: false
updated_at: '2023-12-25T00:15:01+09:00'
id: d39640af191bc2292fa9
organization_url_name: null
slide: false
ignorePublish: false
---

## ビルドとは

人間が読み書きしやすいように設計されたプログラミング言語で書いたソースコードを、機械が読みやすい(素早く読み込める) ように変換することです。

> ビルドとは、組み立てる、建てる、築く、作り上げる、などの意味を持つ英単語。ソフトウェアの分野では、プログラミング言語で書かれたソースコードなどを元に実行可能ファイルや配布パッケージを作成する処理や操作のこと。
>
> 出典: https://e-words.jp/w/%E3%83%93%E3%83%AB%E3%83%89.html


似た用語に `コンパイル` 、`トランスパイル` がありますが、基本的な違いとして成果物が異なります。コンパイルはソースコードから直接実行可能ファイルを生成し、トランスパイルはソースコードから新しいコードを生成します。

## JavaScriptのビルド

そもそも JavaScript はスクリプト言語であり、ブラウザ上で動くコードとは別に、サーバー上で動かすことも可能です。 JavaScript を使えばフロントエンドアプリも WebAPI といったサーバー開発、 CLI アプリの開発も行えます。

ブラウザであれば、 js ファイルに処理を書き、 HTML ファイルの script タグから読み込むことで利用できます。
```index.html
...
<button id="my-button">クリック</button>
...
<script src="main.js"></script>
...
```
```main.js
const button = document.getByElementById("my-button");
button.addEventListener('click', () => {
  /* 処理の内容 */
});
```

上記のようなコードはコンパイルをしなくても動かせます。
では、なぜフロントエンド開発にビルド・コンパイルをする必要があるのかを解説していきます。

### bundle
複数のファイルをまとめて 1 つのファイルにすることを bundle と言います。
bundle をする理由としましては、ブラウザからアプリケーションを使うときに js ファイルをダウンロードします。このときに、ファイル数が少なければダウンロード時間が短く住むため、パフォーマンスの為に bundle を行います。  
あらかじめ 1 つのファイルにまとめて書いてしまっては、可読性・保守性が低くなることは目に見えています。なので、 bundle を行うことで、書くときはエンジニアが書きやすいように書いて、本番環境で実行するときはユーザーが使いやすいようにしてくれます。

### トランスパイル
JavaScript を実行する各種ブラウザも日に日に開発が進められ、新しい機能を実装していきます。  
しかし、それぞれ別の団体が実装しているので Chrome の最新版で使える JavaScript の機能を firefox では使えない可能性があります。
また、新しい JavaScript の機能が古いブラウザでは使えないといったこともあります。
そのような差異を吸収するために、トランスパイルを行います。

トランスパイルとは、ソースコードを別のソースコードに変換することです。最新のコードを解析し、古い構文を使って書き換えることで、古いエンジンでも動作するようにできます。

なので新しい JavaScript の機能を古いブラウザでも使いたい場合、一度古いブラウザでも動作する構文に書き換えることで可能にします。その作業をトランスパイルすると言います。

#### JSX
react でフロントエンドを書くときに JSX という構文を使います。見た目は HTML に近いですが、中身は単なる JavaScript です。しかし、JSX 構文はそのままで動作しませんし、ブラウザで読み込むこともできません。  
babel や rollup を使って JSX 構文を解析し、 ブラウザで動作する  JavaScript のコードへとトランスパイルできます。

[babel.js](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=MYewdgzgLgBAKiAJiCMC8MAUAHATibCASnQD4YBvAKBhlwFMoBXXMLGmASAB5EBLAG6kOtWtyYAbYaJmU8BCADooSFIoC2AQ2yZMK5CTTlMI2WIl8YAa3oBPNBX0hFfRAF9Sj1cr5Q-E-jduAHoLaTMYIiI3UzFgyXC4_iEOIk4qNyA&debug=false&forceAllTransforms=false&modules=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.23.6&externalPlugins=&assumptions=%7B%7D)の web サイトで実際に触ってみることもできます。

```Todo.jsx
const Todos = (props) => {
  return (
  	<div>
      <ul>
        {props.todos.map((todo) => (
          <li key={todo.id}>{todo.titile}</li>
        ))}
      </ul>
    </div>
  )	
}
```

```Todo.js
import { jsx as _jsx } from "react/jsx-runtime";
const Todos = props => {
  return /*#__PURE__*/_jsx("div", {
    children: /*#__PURE__*/_jsx("ul", {
      children: props.todos.map(todo => /*#__PURE__*/_jsx("li", {
        children: todo.titile
      }, todo.id))
    })
  });
};
```
このようにコードが変換されます。

### minify
ファイルを圧縮することです。minify を行うことでファイルサイズを縮小させ、ページの表示速度を向上させることができます。
改行やスペース、インデントなどコードを書く・読む際には絶対に必要な要素を省き人間よりもマシンに素早く読ませるコードへと変換します。
[UglyJS](https://skalman.github.io/UglifyJS-online/) の web ページで試すことができます。

## jsxファイルをビルドする
jsx が書かれたファイルをビルドして使えるようになるまでに、どのような手順を踏むかを説明していきます。
※ 特定のライブラリのケースというよりは一般的にこうビルドされるというイメージを持っていただけたらなと思います。

1. jsx ファイルをトランスパイル
  - `.jsx` ファイル群から `.js` ファイル群へ
1. ファイル群を bundle する
  - 複数の `.js` ファイルから `bundle.js` といった複数ファイルがひとまとめになったファイルへ
1. ファイルを minify する
  - ひとまとめになった `.js` ファイルから余白や改行・インデントを取り除く


