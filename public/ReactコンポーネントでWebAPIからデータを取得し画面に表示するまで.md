---
title: ReactコンポーネントでWebAPIからデータを取得し画面に表示するまで
tags:
  - React
  - react-hooks
private: false
updated_at: '2022-08-07T16:09:53+09:00'
id: e9f4664a3093d001f596
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
先日useEffectについてまとめた記事を書きました。本記事では、[こちらの記事](https://qiita.com/diskszk/items/c895c6f28ad4e565b67e)に収まりきらなかった、useEffectを使って初回レンダリング時にWebAPIからデータを取得して表示する法方と勉強し始めの頃にハマったことを文章に起こします。

この記事ではWebAPIから投稿データを取得し、リスト形式に並べてレンダリングするまでを、筆者の過去の失敗を交え解説していきます。

今回WebAPIから何らかのデータを取得するにあたって、[JSONPlaceholder](https://jsonplaceholder.typicode.com/)というサービスを利用しました。手軽にREST API を試せるのでおすすめです。

![ 2021-09-25 15.31.04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/446c362b-8371-1d78-f182-b2eac24ba946.png)


## 事前準備

### 型定義

WebAPIから取得したデータから利用する要素だけを型定義します。

```types.ts
export type Post = {
  id: string;
  body: string;
  title: string;
};
```

### fetch用関数

fetchメソッドを使って `https://jsonplaceholder.typicode.com/posts/?_limit=10` にあるリソースを見に行きます。postsを最大10件取得する関数です。
こちらの関数をこの記事の例では使いまわしていきます。

```fetchPost.ts
const fetchPosts = async (): Promise<Post[]> => {
  console.log("fetching-start");

  const res = await fetch(
    "https://jsonplaceholder.typicode.com/posts/?_limit=10"
  );

  console.log("fetching-finish");
  return res.json();
};
```

fetch　は「目的の場所に行って取ってくる」という意味があるので外部リソースから取得する時によく使われます。

## 実践編

### バッドパターン

fetch結果がレンダリングされません。

```BadPattern.tsx
const PostList: React.FC = () => {
  let posts: Post[] = [];
  fetchPosts().then((data) => {
    posts = data;
  });

  return (
    <div>
      <p>データ一覧</p>
      <ul>
        {console.log("render")}
        {posts.map((post) => {
          return (
            <li key={post.id}>
              <h1>{post.title}</h1>
              <p>{post.body}</p>
            </li>
          );
        })}
      </ul>
    </div>
  );
};
```

みるからに酷いコードですね...

ですが私は昔本当にこのコードを書いていたのです。本当か疑わしくなってきました。

ブラウザで確認すると、データ一覧の下に投稿データが表示される想定でしたが1件も表示されません。

![emptyresult.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/dea7ca09-8f24-135f-b4f1-47c734502287.png)


コードはさておき、なぜレンダリングされないのかを処理の順番を追って見ていきます。

こちらは単に非同期通信処理のハンドリングがされていないからです。
コンポーネント内での処理の順番としては、

1. fetchPosts関数を呼び出し'fetching-start'のログが吐かれ、WebAPIとの通信が始まります
1. 画面がレンダリングされる
1. WebAPIとの通信が終わり、'fetching-finish'のログが測れる

![unasync.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/e8ce5a76-a18d-6fe5-80ab-ba1150400149.png)


2.のレンダリング時にはまだ投稿データの取得が完了しておらず、ローカル変数postsは空のままなので空のデータがレンダリングされるからです。
なのでuseEffectを使って投稿データをローカル変数に格納し終えたら再度レンダリングしてもらうようにします。


### OK パターン

あくまでも私がこれで大丈夫というコードです。動きはします。

```OKPattern.tsx
const PostList: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    (async function () {
      const data = await fetchPosts();

      setPosts(data);
      setIsLoading(false);
    })();
  }, []);

  return (
    <div>
      {console.log("render")}
      <p>データ一覧</p>
      {isLoading && <h2>Loading...</h2>}
      <ul>
        {posts.map((post) => {
          return (
            <li key={post.id}>
              <h1>{post.title}</h1>
              <p>{post.body}</p>
            </li>
          );
        })}
      </ul>
    </div>
  );
};
```

レンダリングの順番をわかりやすくするためにWebAPIとの通信中とのことをユーザーに教えるLoading状態をコンポーネントに追加しました。

ブラウザで確認してみると

![rendering.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/ab872e63-3c04-0f3f-f2d4-8db3cf179ede.gif)


ブラウザのリロードボタンを押すと始めにLoading... が表示されて、少しすると投稿データが表示されます。

処理の順番を追っていきます。

1. 初回レンダリング処理が走ります。
  - この時点で画面にレンダリングされるのはLoading...です
1. 初回レンダリング後にuseEffect内部の処理が始まります
1. `const data = await fetchPosts()` の行で WebAPIとの通信が始まり、'fetching-start'のログが吐かれます
  - 非同期処理なのでfetchPosts()の処理が終了するまで次の処理を待ってくれます
1. WebAPIから投稿データを取得する処理が完了したので'fetching-finish'のログが吐かれます
1. ReactコンポーネントのuseEffectに戻り、取得した投稿データをローカル変数postsにセットします
1. useEffect内で画面に表示するデータが変わったので再レンダリング処理が始まります

開発者コンソールを見てみると以下のようになります。

![ 2021-09-25 16.37.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/337913c2-0fa0-19dc-1ce5-d2e8a8e665b4.png)


最後の'render'が二回出てくるのは'posts'と'isLoading'の二つの変数が変更しているからですね。


## 最後に

改めて過去のコードを眺めると酷いもんだなと思えるようになってきたのは成長の証だと捉えて精進していきます...

以上です。最後まで読んでいただきありがとうございます！
