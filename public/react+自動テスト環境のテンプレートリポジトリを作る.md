---
title: react+自動テスト環境のテンプレートリポジトリを作る
tags:
  - React
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: e790fba7ddf5a916b277
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに
フロントエンドの自動テストまわりの記事を書くことが増えてきたので、 react + 自動テストのテンプレートリポジトリを作ろうと思い至りました。
<!-- textlint-disable -->
また、毎回環境構築から開始すると記事を書くことのハードルが高くなるので、テンプレートを活用してさらに記事の記事の更新頻度を高められたらと思います。
<!-- textlint-enable -->

## テンプレートリポジトリとは
既存のリポジトリをテンプレート化して、あなたや他の人が、同じディレクトリ構造、ブランチ、ファイルを持つ新しいリポジトリ作成できるようにできる GitHub の機能です。

## リポジトリの紹介

https://github.com/diskszk/react-testing-boilerplate

今回作成したテンプレートリポジトリは以下のライブラリを使用しており、 フロントエンドの開発環境および、自動テスト環境を備えています。

```json
{
  "react": "^18.2.0",
  "typescript": "^5.0.2",
  "vite": "^4.4.5",
  "vitest": "^0.34.6",
  "@testing-library/react": "^14.0.0",
  "@testing-library/react-hooks": "^8.0.1",
  "@testing-library/user-event": "^14.5.1",
  "axios": "^1.5.1",
  "msw": "^1.3.2",
  "eslint": "^8.45.0",
}
```

## テンプレートリポジトリの作成

https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-template-repository

テンプレートリポジトリの作成方法はとても簡単で、 GitHub 上でテンプレート化したいリポジトリのヘッダーから `Settings` を選択します。

![ 2023-10-05 15.06.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/f0994cba-5ea7-4c8d-555e-c022fcb46fec.png)

`Template repository` のチェックボックスにチェックを入れればテンプレート化できます。

## テンプレートリポジトリを使う

https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template

使用したいテンプレートのリポジトリへアクセスし、 `Use this template` を選択します。

![ 2023-10-05 15.08.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/b48cfa94-5348-4a06-2dee-63f901d25221.png)

GitHub Cli を使う場合は `--template <repository>` とコマンド入力すれば　GitHub 上にテンプレートを元にしたリポジトリを作成できます。

例として、今回私が作成したテンプレートリポジトリを使う場合は以下のコマンドになります。

```bash
$ gh repo create my-repo --private --template https://github.com/diskszk/react-testing-boilerplate.git
```
