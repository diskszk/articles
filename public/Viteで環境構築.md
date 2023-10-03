---
title: Viteで環境構築
tags:
  - vite
private: false
updated_at: '2022-05-18T18:53:39+09:00'
id: ae1ab798c01d83ba519a
organization_url_name: null
slide: false
ignorePublish: false
---

## Viteとは

詳しくは公式の[Overview](https://vitejs.dev/guide/#overview)に任せるとしますが、Viteを使えば素早く立ち上がる開発サーバーを使って、より速くより無駄のない開発体験を享受できます。


## ビルドツール

Viteとはビルドツールです。
yarn build や yarn devコマンドを入力するとbuildが行われてビルド結果をbuild(あるいはdist等)のディレクトリに吐き出されたり、開発サーバーが立ち上がりブラウザに表示されます。  
create-react-appでプロジェクトを作った際にはデフォルトではreact-scriptsが使われており、手動でプロジェクトを作った際にはwebpack等を導入します。
Viteを使うとbuild処理が速く、すぐに開発サーバーが立ち上がります。


## Viteの環境構築

Reactのcreate-react-app, Next.jsのcreate-next-app 同様にテンプレートを生成してくれるのでコマンド一つ(+ 質問に答える)だけで開発環境を構築できます。

公式の[Getting Started](https://vitejs.dev/guide/#getting-started)を参考に以下の手順で開発環境を構築できます。

``` 
$ yarn create vite

# Vite + React + Typescript 環境のプロジェクトを作りたい場合
$ yarn create vite my-app --template react-ts
```

上記二つの場合コマンドライン上にTypescriptを使うか等の質問が表示されるので答えていけばプロジェクトが生成されます。

プロジェクトの生成が終わったらディレクトリを移動して以下のコマンドで開発サーバーが立ち上がります。

```
yarn install
yarn run dev
```
