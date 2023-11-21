---
title: 複数のプロジェクトで型情報を使い回すためにnpmパッケージにして公開する
tags:
  - npm
  - TypeScript
  - npmパッケージ
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: dd5bc72c48f377dc184e
organization_url_name: null
slide: false
ignorePublish: false
---
## 目的

- 複数のプロジェクトで TypeScript の型情報を使い回したい
- その際、以下のように import して使えるようにしたい

```App.tsx
import { FC } from "react";

const App: FC = () => (
  //
)
```

## どうやったか

npm パッケージにして公開し、それぞれのプロジェクトでパッケージを install して使うようにしました。

## 使用ライブラリのバージョン
- node: v16.x
- typescript: v4.8.x

## 何をしたか

### npmパッケージにするプロジェクトを作成
``$ mkdir npm-package-sample && cd $_``
``$ yarn init``

プロジェクト初期化の際にいろいろ質問されるので答えていきます。

最終的に package.json は以下のようになります。

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

- version はひとまず 0.0.0 にしておきます
- main はエントリーポイントになります。TypeScript を使うので build 先のディレクトリのエントリーポイントを指定します
- npm パッケージとして公開するので private は false にします
- files は公開するディレクトリを指定します。こちらも TypeScript を使うので dist にします
- name, description, author 等は適宜入力します

git を使うのであれば git init や .gitignore ファイルなどを用意します。

### TypeScriptを入れる
`$ yarn add -D typescript @types/node`
`$ yarn tsc --init`

### package.jsonにscriptsを追加
このあたりがあると良いでしょう。

```package.json
"scripts": {
  "build": "tsc",
  "prepublishOnly": "yarn build"
}
```

`prepublishOnly` を書いておけば npm publish を実行する前に build をしてくれます。

### ソースファイル作成

src ディレクトリに、ts ファイルを作成します。
src/index.ts にあるものを build し、公開する予定です。

では実際に index.ts に型定義を書いていきます。
```index.ts
type Animal = {
  name: string;
  age: number;
};

export { Animal };
```

これで build でき、dist/index.js が作成されたことが確認できたらいよいよ公開していきます。

### npmパッケージを公開する

  1. アカウントを作成
    NPMのアカウントがない場合、 [NPM](https://www.npmjs.com/) でアカウントの作成をします

  2. コマンドラインでログインする
    `npm login`
    こちらで先程作成したアカウント情報を入力しログインします。
    ログインできたかを確認するには`npm whoami` と入力すれば確認できます。

  3. ログインできたら `npm publish` のコマンド入力でパッケージを公開します
    バージョンを聞かれるのでひとまず0.1.0としておきます。
    エラーが起きず無事終了すればnpmパッケージに公開できています。

  4. npm からログアウトしておく
     `npm logout` でログアウトできるのでログアウトしておきます

### 追記
編集して再度 publish する際には package.json の versioin を変更しないとエラーになります。

6. 確認の為使ってみる
別の TypeScript プロジェクトに移動して先程公開したパッケージを `yarn add` コマンドを使ってインストールします。
`$ yarn add 作成したnpmパッケージ名`

package.json を確認すると dependencies に追加されています。
次に、適当な ts ファイルを開いて import してみます。

```sample.ts
import { Animal } from "npm-package-sample";

const animal: Animal = {
  name: "John",
  age: 6
}
```

import でき、型アノテーションがつくことを確認できました。

これで複数の TypeScript プロジェクトで同じ型を使い回すことができます。

## 終わりに

今回 npm パッケージを通して公開し、import して型を使うことにしましたが、もしかしたらもっといい方法があるのでは？とも思っています。..
もし何かご存じの方いらっしゃいましたら教えていただけますと幸いです。
その他なんでも是非コメントいただけたら嬉しいです。
