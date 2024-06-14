---
title: Radix UIを使ってレスポンシブデザインに対応する
tags:
  - CSS
  - React
  - Radix
private: false
updated_at: '2024-06-14T17:36:49+09:00'
id: 89c33609ff3450a8d7b0
organization_url_name: null
slide: false
ignorePublish: false
---

## Radix UI とは

https://www.radix-ui.com/

Radix UI とはヘッドレス UI ライブラリです。  
ヘッドレス UI とは、スタイルを持たない、すなわち機能とアクセシビリティのみを持った UI コンポーネントを提供する仕組みです。

この記事では Radix UI を使ってレスポンシブデザインに対応したコンポーネントを作る方法を紹介します。

## コンポーネント作成

Radix UI を install してコンポーネントを作ります。  
例として Vite と React で作ったアプリに web ページの `Header` コンポーネントを作っていきます。

### Radix UIを導入

```
yarn add @radix-ui/themes
yarn add @radix-ui/react-icons
```

``` main.ts
+ import '@radix-ui/themes/styles.css';
```

私が感じている Radix UI のデメリットなのですが、Flex コンポーネントが多用されがちでそれぞれのコンポーネントが何のために書かれているのかが一目では分かりづらいです。今回は `id` 要素を使って明示的に名前をつけています。  

```tsx Header.tsx 
import { GitHubLogoIcon } from "@radix-ui/react-icons";
import {
  Flex,
  Heading,
  IconButton,
  TabNav,
  Box,
  Button,
} from "@radix-ui/themes";

export const Header: React.FC = () => (
  <header>
    <Flex id="header-container" align="center" height="112px" justify="between" px="24px">
      <Fle id="header-logo-container" gapX="8px">
        <Button asChild color="gray" variant="ghost">
          <a aria-label="Homeへのリンク" href="/">
            <Box m="auto">
              <img
                alt="アイコン画像"
                height="48px"
                src="path/to/icon.png"
                width="48px"
              />
            </Box>
            <Heading align="center" as="h1" size="8" weight="regular">
              Tech Blog.
            </Heading>
          </a>
        </Button>
      </Flex>
      <Flex id="navigation-container" align="center" gap="16px">
        <TabNav.Root>
          <TabNav.Link href="/blog">Blog</TabNav.Link>
          <TabNav.Link href="/profile">Profile</TabNav.Link>
        </TabNav.Root>
        <IconButton asChild color="gray" variant="ghost">
          <a
            aria-label="GitHubリポジトリへのリンク"
            href="https://github.com/user/repo"
            rel="noopener noreferrer"
            target="_blank"
          >
            <GitHubLogoIcon color="black" height="36px" width="36px" />
          </a>
        </IconButton>
      </Flex>
    </Flex>
  </header>
);
```

`@radix-ui/themes` が提供するコンポーネントで直接 `height`, `width` をなどの style を調整しながら、Flex コンポーネントの `direction`, `align`, `justify` を駆使して Header コンポーネントのレイアウトを調整できます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/59ff7d33-ffe2-dfe9-96d2-dde389efd9c9.png)

PC では問題ないように見えますが、developer tool を使ってモバイル用の画面を確認するとレイアウトが崩れていることが確認できます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/a1fd23cb-7235-a985-8da3-b67e89c0a026.png)

## レスポンシブ対応

https://developer.mozilla.org/ja/docs/Learn/CSS/CSS_layout/Responsive_Design

以下の対応をしていきます。

- PC とモバイルで header の高さを変更する
- ナビゲーションの表示/非表示を切り替える
- ハンバーガーメニューアイコンの表示/非表示を切り替える

また、今回それぞれのブレイクポイントを簡易的に以下のようにします。
- モバイルビュー: 0px ~ 1023px
- PC ビュー 1024px ~


### PC とモバイルで header の高さを変更する

https://www.radix-ui.com/themes/docs/theme/breakpoints

コンポーネントに直接各ブレークポイントを超えた際のスタイルを適用できるので書いていきます。

``` diff_tsx
<header>
-  <Flex id="header-container" align="center" height="112px" justify="between" pt="8px" px="24px">
+  <Flex id="header-container" align="center" justify="between" pt="8px" px="24px"
+      height={{
+        initial: "64px",
+        md: "112px"
+      }}
+    >
    // 省略
</ Flex>
</header>
```

`initial (0px)` のときは `height` が 64 px で、画面の width が `md (1024px)` を超えた時 `height` が 112px になります。

PC ビュー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/7cff89e1-e8f6-273b-ef22-a4b28e4584a1.png)

モバイルビュー
![screenshot 8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/70cf7ef0-df05-13ab-6cb3-1da7352617e1.png)

### ナビゲーションの表示/非表示を切り替える

ナビゲーション (アプリ内のリンク、 GitHub へのリンク)の表示非表示を切り替えます。  
PC ビューの時は表示し、モバイルビューでは表示しないようにします。


width が `md (1024px)` より小さい場合に `navigation-container` へ `display: "none"` を追加します。  
`header-container` の height と同様に Radix UI の　Breakpoints を使ってそれぞれの値を設定します。

``` diff_tsx
<Flex id="navigation-container" align="center" gap="16px"
+    display={{
+      initial: "none",
+      md: "flex"
+    }}
+  >
```

PC ビュー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5e1969f0-f1cf-f2a7-8625-fe0eaf42475a.png)

モバイルビュー
![screenshot 10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/dbda45d1-a434-09cf-6eee-5c1b8f92c3e8.png)


### ハンバーガーメニューアイコンの表示/非表示を切り替える

モバイルビューの時にヘッダーのナビゲーションの代わりにハンバーガーメニューを実装するとして、開閉するアイコンボタン作成します。  
ナビゲーションとは逆で PC ビューの時は非表示、モバイルビューでは表示するようにします。  

`header-container` の子要素として、 `heder-logo-container` の前(左に表示する) へ `hamburger-menu-icon-container` を作成します。

``` diff_tsx
+ import { HamburgerMenuIcon } from "@radix-ui/react-icons";

/* 省略 */

<Flex id="header-container" /* 省略 */>
+  <Flex id="hamburger-menu-icon-container"
+    display={{
+      initial: "flex",
+      sm: "none",
+    }}
+  >
+    <IconButton onClick={handleClickMenuIcon} >
+      <HamburgerMenuIcon />
+    </IconButton>
+  </Flex>
+  <Fle id="header-logo-container" /* 省略 */>
    ・・・
```

PC ビュー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5e1969f0-f1cf-f2a7-8625-fe0eaf42475a.png)

モバイルビュー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/f8d72ed6-01a3-25e5-b8a1-84ea56412d2a.png)

これでモバイルビュー時にのみハンバーガーメニューアイコンを表示できました。
