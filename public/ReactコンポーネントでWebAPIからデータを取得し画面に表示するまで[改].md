---
title: 'ReactコンポーネントでWebAPIからデータを取得し画面に表示するまで[改]'
tags:
  - React
private: false
updated_at: '2023-10-16T15:57:04+09:00'
id: 6549584a3843781aac79
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

昔こんな記事を書きました。

https://qiita.com/diskszk/items/e9f4664a3093d001f596

しかし、外部リソースからデータを fetch する目的であれば、必ずしも `useEffect` を使う必要はなく、むしろ別の方法のほうがいいとも言われています。

https://react.dev/learn/synchronizing-with-effects

ということなので、こちらの記事を参考にしつつ、`useEffect` を使ってデータフェッチしている処理から `useEffect` を取り除いていきます。

https://zenn.dev/kazuma1989/articles/a30ba6e29b5b4c

## useEffectを使用する場合

```tsx
type Todo = {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
};

const fetchPosts = async (): Promise<Post[]> => {
  const res = await fetch(
    "https://jsonplaceholder.typicode.com/posts/?_limit=10"
  );
  return res.json();
};

const PostList: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    (async function () {
      const data = await fetchPosts();

      setPosts(data);
    })();
  }, []);

  return (
    <div>
      <h2>todos</h2>
      <ul>
        {todos?.map((todo) => (
          <li key={todo.id}>
            <p>title: {todo.title}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

コンポーネントがレンダリングされる際に一度だけ `useEffect` 内の処理が実行され、WebAPI からデータをフェッチします。

## useEffectを使用しない場合

`useSyncExternalStore` を使用します。

```tsx
import { useCallback, useRef, useSyncExternalStore } from "react";
import axios from "axios";

type Todo = {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
};

export const Todos: React.FC = () => {
  const data$ = useRef<Todo[]>();

  const subscribe = useCallback((onStoreChange: () => void): (() => void) => {
    const controller = new AbortController();

    axios
      .get<Todo[]>("/https://jsonplaceholder.typicode.com/posts/?_limit=10", { signal: controller.signal })
      .then((res) => res.data)
      .then((data) => {
        data$.current = data;

        onStoreChange();
      });

    return () => controller.abort();
  }, []);

  const todos = useSyncExternalStore(subscribe, () => data$.current);

  return (
    <div>
      <h2>todos</h2>
      <ul>
        {todos?.map((todo) => (
          <li key={todo.id}>
            <p>title: {todo.title}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

