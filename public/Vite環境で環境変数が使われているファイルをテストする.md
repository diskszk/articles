---
title: Vite環境で環境変数が使われているファイルをテストする
tags:
  - TypeScript
  - Jest
  - React
  - vite
private: false
updated_at: '2022-08-07T16:52:19+09:00'
id: ed6362e35e15f2fd790e
organization_url_name: null
slide: false
ignorePublish: false
---
今回構築した環境でのライブラリのバージョンは以下の通りです
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

jestを導入し、configファイルを作っていきます。
ts-jestはTypeScriptで書かれたファイルをjestに認識させるために使います。

```
$ yarn add -D jest ts-jest @types/jest
$ yarn ts-jest config:init
```

## テストコード

まずはテストができる確認をするため通るテストを書いてみます

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

add.tsとadd.spec.tsの2つのファイルを作り、package.jsonにtestのスクリプトを追加し、yarn testしてみるとテストが通る事が確認できます。

次に環境変数を含むコードのテストを書いていきます。

### Viteで環境変数を使う

Viteで環境変数を使うにはプロジェクトのルートに.envとういファイルを作成しその中に`VITE_`のプレフィックスを付けて環境変数を宣言し、`import.meta.env. + 環境変数名`で使用できます。
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
- import.metaはESModuleでしか動作しない
- es2020以降でしか動作しない
- ts-jestを使ってテストファイルをCommonJS形式に変換してからテストしている

つまり、ts-jestによってCommonJS形式に変換されたテストファイルではimport.metaでの環境変数を使えないようです。

当記事では、環境変数の呼び出し方をimport.metaではなくprocess.envを使うという解決方法を試してみます。

[vite-plugin-env-compatible](https://github.com/IndexXuan/vite-plugin-env-compatible)というプラグインを使用してprocess.envの形式で環境変数を使えるようにしていきます。

プラグインをインストールし、vite.config.tsに変更を加えます。
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

9行目のprefixで環境変数名のプレフィックスを指定します。デフォルトではVUE_APPとなっていますが、Viteで使うのでVITEと指定します。

環境変数を使っている箇所をprocess.envの形式に修正します

```testingWithEnvironmentVariable.ts
function testingWithEnvironmentVariable(): string {

  // 環境変数を使う処理
-  const env = import.meta.env.VITE_SOME_KEY;
+  const env = process.env.VITE_SOME_KEY;

  return "この関数では環境変数が使われています";
}
```

この状態でyarn testを走らせると無事テストは成功します。

## おわりに

当記事では `テスト対象ファイルで環境変数が使われているとテストできない -> import.metaではなくprocess.envを使う` という方法を取りましたが、環境変数に型をつける方法や、テストをする際にESModuleのままテストできる方法、または他の解決策など何かご存知の方いらっしゃいましたらコメントを頂けると幸いです。
