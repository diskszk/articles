---
title: TanStack QueryのuseQueryを使ったテスト
tags:
  - 'react'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

## はじめに
`@tanstack/react-query` の `useQuery` を使って WebAPI 等の外部リソースからデータを fetch した際のテストにつまずくことがあったので、解消法を共有しようと思い執筆に至ります。

## 環境

