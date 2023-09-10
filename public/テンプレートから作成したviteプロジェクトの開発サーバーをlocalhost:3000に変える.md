---
title: 'テンプレートから作成したviteプロジェクトの開発サーバーをlocalhost:3000に変える'
tags:
  - JavaScript
  - vite
private: false
updated_at: '2023-07-18T16:14:40+09:00'
id: 8c89fa9391fc62522325
organization_url_name: null
slide: false
---
vite のバージョン 3 以降を使う場合、開発サーバーを起動させると、デフォルトで `http://127.0.0.1:5173` に開発サーバーが立ち上がります。

https://v3.vitejs.dev/guide/migration.html#dev-server-changes

これ自体は vite がそうしていることなので、なんの問題もありません。
もし、いままで慣れ親しんだ `http://localhost:3000` を使いたい場合や、その他の理由があってポート番号を変えたい場合などは、以下の手順で変更できます。

## 実行環境
- node: 18.14.X

## 実行手順
"example" という名前で vite プロジェクトを作成します。  
`$ yarn create vite example --template react-ts`
フレームワークに react　を使う設定でプロジェクトを初期化します。

作業ディレクトリに移動し、ライブラリをインストールします。
```
$ cd ./example
$ yarn install
```


vite.config.ts を開き、`defineConfig` に `server.port` を追加します。

```vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
+  server: {
+    port: 3000,
+  },
});
```
`server.port` にポート番号を振ることで、指定したポート番号を使うことができます。

開発サーバーを起動し、 `http:localhost:3000` にブラウザでアクセスすると、無事アプリケーションが動いていることが確認できます。

```
$ yarn dev
```

![ 2023-07-18 14.59.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/ab0e867e-25df-ea2f-5e84-98e4fa975ba9.png)

![ 2023-07-18 15.01.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/e91e53c0-8b92-9b74-c745-159f74b56524.png)

