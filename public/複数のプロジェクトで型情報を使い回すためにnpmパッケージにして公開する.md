---
title: 複数のプロジェクトで型情報を使い回すためにnpmパッケージにして公開する
tags:
  - npm
  - TypeScript
  - npmパッケージ
private: false
updated_at: '2022-11-26T13:25:07+09:00'
id: dd5bc72c48f377dc184e
organization_url_name: null
slide: false
ignorePublish: false
---
## 目的

複数のプロジェクトでTypeScriptの型情報を使い回したかった。
その際、以下のようにimportして使えるようにしたかった。

```App.tsx
import { FC } from "react";

const App: FC = () => (
  //
)
```

## どうやったか

npmパッケージにして公開し、それぞれのプロジェクトでパッケージをinstallして使うようにしました。

## 使用ライブラリのバージョン
- node: v16.x
- typescript: v4.8.x

## 何をしたか

### npmパッケージにするプロジェクトを作成
``$ mkdir npm-package-sample && cd $_``
``$ yarn init``

プロジェクト初期化の際にいろいろ質問されるので答えていきます。

最終的にpackage.jsonは以下のようになります。

```package.json
{
  "name": "npm-package-sample",
  "version": "0.0.0",
  "main": "dist/index.js",
  "license": "MIT",
  "description": "説明",
  "author": "your-name",
  "private": false,
  "files": [
    "dist"
  ]
}
```

- versionはひとまず0.0.0にしておきます
- mainはエントリーポイントになります。TypeScriptを使うのでbuild先のディレクトリのエントリーポイントを指定します。
- npmパッケージとして公開するのでprivateはfalseにします
- filesは公開するディレクトリを指定します。こちらもTypeScriptを使うのでdistにします
- name, description, author 等は適宜入力します

gitを使うのであればgit init や .gitignoreファイルなどを用意します。

### TypeScriptを入れる
`$ yarn add -D typescript @types/node`
`$ yarn tsc --init`

### package.jsonにscriptsを追加
このあたりがあると良いでしょう

```package.json
"scripts": {
  "build": "tsc",
  "prepublishOnly": "yarn build"
}
```

`prepublishOnly`を書いておけばnpm publishを実行する前にbuildをしてくれます

### ソースファイル作成

srcディレクトリに、tsファイルを作成します。
src/index.tsにあるものをbuildし、公開する予定です。

では実際にindex.tsに型定義を書いていきます。
```index.ts
type Animal = {
  name: string;
  age: number;
};

export { Animal };
```

これでbuildでき、dist/index.jsが作成されたことが確認できたらいよいよ公開していきます。

### npmパッケージを公開する

  1. アカウントを作成
    NPMのアカウントがない場合、 [NPM](https://www.npmjs.com/) でアカウントの作成をします

  2. コマンドラインでログインする
    `npm login`
    こちらで先程作成したアカウント情報を入力しログインします。
    ログインできたかを確認するには`npm whoami` と入力すれば確認できます。

  3. ログインできたら`npm publish`のコマンド入力でパッケージを公開します。
    バージョンを聞かれるのでひとまず0.1.0としておきます。
    エラーが起きず無事終了すればnpmパッケージに公開できています。

  4. npmからログアウトしておく
     `npm logout`でログアウトできるのでログアウトしておきます。

### 追記
編集して再度publishする際にはpackage.jsonのversioinを変更しないとエラーになります。

6. 確認の為使ってみる
別のTypeScriptプロジェクトに移動して先程公開したパッケージを`yarn add`コマンドを使ってインストールします。
`$ yarn add 作成したnpmパッケージ名`

package.jsonを確認するとdependenciesに追加されています。
次に、適当なtsファイルを開いてimportしてみます。

```sample.ts
import { Animal } from "npm-package-sample";

const animal: Animal = {
  name: "John",
  age: 6
}
```

importでき、型アノテーションがつくことを確認できました。

これで複数のTypeScriptプロジェクトで同じ型を使い回すことができます。

## 終わりに

今回npmパッケージを通して公開し、importして型を使うことにしましたが、もしかしたらもっといい方法があるのでは？とも思っています...
もし何かご存じの方いらっしゃいましたら教えていただけますと幸いです。
その他なんでも是非コメントいただけたら嬉しいです。
