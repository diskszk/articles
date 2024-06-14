---
title: Markdownのリンクを別タブで開くようにする
tags:
  - JavaScript
  - Markdown
private: false
updated_at: '2024-06-14T13:42:23+09:00'
id: 9cd3ceeadd162935bae8
organization_url_name: null
slide: false
ignorePublish: false
---

Markdown で書いたドキュメントのリンクから別タブで開くようにするための処理を書いていきます。


## Markdown を表示する

index.html に JavaScript を使って Markdown を表示します。

``` html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <!-- head -->
  </head>
  <body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js"></script>
    <script>
      const markdown = `
## Markdown ドキュメント

### リンク一覧

[私のQiita](https://qiita.com/diskszk)  
[私のGitHub](https://github.com/diskszk)
`;

      (function () {
        const div = document.createElement("div");
        div.innerHTML = marked(markdown);
        document.body.appendChild(div);
      })();
    </script>
  </body>
</html>
```

index.html ファイルをブラウザで開くと Markdown を表示していることが確認できます。

![screenshot 3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/f1a00a91-1946-ac21-8cd3-6e2ce71adfea.png)

developer tool で確認すると、当然 href 属性しか設定されていません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/d63a175a-d527-71b8-0fcb-324cfc709b68.png)

## aタグに属性を追加する

リンクをクリックした時に別タブで開かせたいので、 a タグに `target="_black"` と `rel="noopener noreferrer` の属性を追加する処理を書いていきます。

``` html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <!-- head -->
  </head>
  <body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js"></script>
    <script>
      const markdown = `
    <!-- markdown の内容 -->
`;

      (function () {
        const div = document.createElement("div");
        div.innerHTML = marked(markdown);
        document.body.appendChild(div);
      })();

+      document.addEventListener("DOMContentLoaded", function () {
+        const links = document.querySelectorAll("a");
+        links.forEach(function (link) {
+          link.setAttribute("target", "blank");
+          link.setAttribute("rel", "noopener noreferrer");
+        });
+      });
    </script>
  </body>
</html>
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/027ce529-d3dc-ea27-a027-5abc620f6c13.png)

これでリンクをクリックした時に別タブで開くようになりました。

## 参考
https://webitworks.jp/target_blank_noreferrer_noopener/

https://rubirubi.hateblo.jp/entry/how-to-target_new_blank-in-markdown-and-open-a-link-in-a-separate-window
