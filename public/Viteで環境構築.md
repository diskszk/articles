---
title: Viteで環境構築
tags:
  - vite
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: ae1ab798c01d83ba519a
organization_url_name: null
slide: false
ignorePublish: false
---

## Viteとは

詳しくは公式の[Overview](https://vitejs.dev/guide/#overview)に任せるとしますが、Vite を使えば素早く立ち上がる開発サーバーを使って、より速くより無駄のない開発体験を享受できます。


## ビルドツール

Vite とはビルドツールです。
yarn build や yarn dev コマンドを入力すると build が行われてビルド結果を build(あるいは dist 等)のディレクトリに吐き出されたり、開発サーバーが立ち上がりブラウザに表示されます。  
create-react-app でプロジェクトを作った際にはデフォルトでは react-scripts が使われており、手動でプロジェクトを作った際には webpack 等を導入します。
Vite を使うと build 処理が速く、すぐに開発サーバーが立ち上がります。


## Viteの環境構築

React の create-react-app, Next.js の create-next-app 同様にテンプレートを生成してくれるのでコマンド 1 つ(+ 質問に答える)だけで開発環境を構築できます。

公式の[Getting Started](https://vitejs.dev/guide/#getting-started)を参考に以下の手順で開発環境を構築できます。

``` 
$ yarn create vite

# Vite + React + Typescript 環境のプロジェクトを作りたい場合
$ yarn create vite my-app --template react-ts
```

上記 2 つの場合コマンドライン上に TypeScript を使うか等の質問が表示されるので答えていけばプロジェクトが生成されます。

プロジェクトの生成が終わったらディレクトリを移動して以下のコマンドで開発サーバーが立ち上がります。

```
yarn install
yarn run dev
```
