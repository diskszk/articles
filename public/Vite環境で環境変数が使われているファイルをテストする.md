---
title: Vite環境で環境変数が使われているファイルをテストする
tags:
  - TypeScript
  - Jest
  - React
  - vite
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: ed6362e35e15f2fd790e
organization_url_name: null
slide: false
ignorePublish: false
---
今回構築した環境でのライブラリのバージョンは以下のとおりです。
```
- react: 18.0.0
- typescript: 4.6.3
- vite: 2.9.9
- jest: 28.1.0
- ts-jest: 28.0.2
- vite-plugin-env-compatible: 1.1.1
```

## Vite + React + TypeScript のテンプレート作成

```
$ yarn create vite vite-test-sample --template react-ts
$ cd vite-test-sample
$ yarn install
```


## テストツール導入

jest を導入し、config ファイルを作っていきます。
ts-jest は TypeScript で書かれたファイルを jest に認識させるため使います。

```
$ yarn add -D jest ts-jest @types/jest
$ yarn ts-jest config:init
```

## テストコード

まずはテストができる確認をするため通るテストを書いてみます。

``` ts
// add.ts
function add(x: number, y: number): number {
  return x + y;
}

// add.spec.ts
describe("add", () => {
  test("1 + 2 = 3", () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

add.ts と add.spec.ts の 2 つのファイルを作り、package.json に test のスクリプトを追加し、yarn test してみるとテストが通る事を確認できます。

次に環境変数を含むコードのテストを書いていきます。

### Viteで環境変数を使う

Vite で環境変数を使うにはプロジェクトのルートに。env とういファイルを作成しその中に `VITE_` のプレフィックスを付けて環境変数を宣言し、`import.meta.env. + 環境変数名` で使用できます。
詳しくは[こちら](https://ja.vitejs.dev/guide/env-and-mode.html)を参照してください。

``` ts
// .env
VITE_SOME_KEY=abcd

// testingWithEnvironmentVariable.ts
function testingWithEnvironmentVariable(): string {

  // 環境変数を使う処理
  const env = import.meta.env.VITE_SOME_KEY;

  return "この関数では環境変数が使われています";
}


// testingWithEnvironmentVariable.spec.ts
describe("testingWithEnvironmentVariable", () => {
  test("環境変数を使った関数をテストする", () => {
    expect(testingWithEnvironmentVariable()).toBe(
      "この関数では環境変数が使われています"
    );
  });
});

```

上記のコードでテストを走らせるとテストは失敗します。

``` エラーログ
The 'import.meta' meta-property is only allowed when the '--module' option is 'es2020', 'es2022', 'esnext', 'system', 'node12', or 'nodenext'.
```

こちらのエラーの原因は以下の様子です。
- import.meta は ESModule でしか動作しない
- es2020 以降でしか動作しない
- ts-jest を使ってテストファイルを CommonJS 形式に変換してからテストしている

つまり、ts-jest によって CommonJS 形式に変換されたテストファイルでは import.meta での環境変数を使えないようです。

当記事では、環境変数の呼び出し方を import.meta ではなく process.env を使うという解決方法を試してみます。

[vite-plugin-env-compatible](https://github.com/IndexXuan/vite-plugin-env-compatible)というプラグインを使用して process.env の形式で環境変数を使えるようにしていきます。

プラグインをインストールし、vite.config.ts に変更を加えます。
```
$ yarn add -D vite-plugin-env-compatible
```

```vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
+ import env from "vite-plugin-env-compatible";

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [
    react(),
+   env({ prefix: "VITE",  mountedPath: "process.env" }) 
  ],
});
```

9 行目の prefix で環境変数名のプレフィックスを指定します。デフォルトでは VUE_APP となっていますが、Vite で使うので VITE と指定します。

環境変数を使っている箇所を process.env の形式に修正します。

```testingWithEnvironmentVariable.ts
function testingWithEnvironmentVariable(): string {

  // 環境変数を使う処理
-  const env = import.meta.env.VITE_SOME_KEY;
+  const env = process.env.VITE_SOME_KEY;

  return "この関数では環境変数が使われています";
}
```

この状態で yarn test を走らせると無事テストは成功します。

## おわりに

当記事では `テスト対象ファイルで環境変数が使われているとテストできない -> import.metaではなくprocess.envを使う` という方法を取りましたが、環境変数に型をつける方法や、テストをする際に ESModule のままテストできる方法、または他の解決策など何かご存知の方いらっしゃいましたらコメントを頂けると幸いです。
