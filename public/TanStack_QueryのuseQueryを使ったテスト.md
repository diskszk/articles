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

## 結論
記事に起こそうとしていたこと、「`useQuery` を使ってテストを書くことが一筋縄じゃ行かない」は勘違いでした。

## はじめに
`@tanstack/react-query` の `useQuery` を使って WebAPI などの外部リソースからデータを fetch した際のテストにつまずくことがあったので、解消法を共有しようと思い執筆に至ります。
また、テストの書き方の違いなどについても `useQuery` を使わない場合と比較して見ていきます。

## 環境

```json
{
  "react": "^18.2.0 ",
  "typescript": "^5.2.2",
  "vite": "^4.4.9",
  "vitest": "0.29.8",
  "@tanstack/react-query": "4.28.0",
  "@testing-library/react": "14.0.0",
  "@testing-library/react-hooks": "^8.0.1",
  "axios": "^1.5.1",
  "msw": "^1.3.2",
}
```

## Reactの機能使う場合の実装+テスト

`useSyncExternalStore` を使って fetch したデータをコンポーネントにレンダリングします。
(`useEffect` でなく `useSyncExternalStore` を使う理由や、`useSyncExternalStore` については当記事では言及しません)

https://react.dev/reference/react/useSyncExternalStore


### コンポーネントのコード

こちらの記事を参考に React コンポーネントからのデータフェッチを実装しました。

https://zenn.dev/kazuma1989/articles/a30ba6e29b5b4c

```tsx Todos.tsx
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
      .get<Todo[]>("/api/todos", { signal: controller.signal })
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

### テストコード

単純に fetch したデータをリスト形式で表示できていることのテストを行います。

```ts Todos.spec.tsx
import { test } from "vitest";

test("todos2件を表示すること", () => {
  // 
});
```

テストに使うデータを WebAPI から直接受け取るのではなく、fetch をインターセプトし、規定の値を返すよう `msw` を使用します。

```sh
$ yarn add msw --dev
```

テストコード内で `msw` を使いモック用のデータを作成します。

```ts Todos.spec.tsx
import { test } from "vitest";
import { rest } from "msw";
import { setupServer } from "msw/node";

const server = setupServer();

beforeAll(() => server.listen());
afterEach(() => {
  cleanup();
  server.resetHandlers();
});
afterAll(() => server.close());

test("todos2件を表示すること", () => {

  server.use(
    rest.get("/api/todos", (req, res, ctx) => {
      return res(
        ctx.status(200),
        ctx.json([
          {
            userId: 1,
            id: 1,
            title: "delectus aut autem",
            completed: false,
          },
          {
            userId: 1,
            id: 2,
            title: "quis ut nam facilis et officia qui",
            completed: false,
          },
        ]),
      );
    }),
  );

  // 以下にテストを書く
});
```

テストを書きます。 `react-testing-library` を使った基本的なテストです。
`screen.find(All)ByXXX` を使うことで非同期通信の完了・レンダリングを待った上で要素を取得します。

```ts Todos.spec.tsx
import { test } from "vitest";
import { render, screen, cleanup } from "@testing-library/react";
import { Todos } from "./Todos";
import { rest } from "msw";
import { setupServer } from "msw/node";

const server = setupServer();

beforeAll(() => server.listen());
afterEach(() => {
  cleanup();
  server.resetHandlers();
});
afterAll(() => server.close());

test("todos2件を表示すること", async () => {

  server.use(
    rest.get("/api/todos", (req, res, ctx) => {
      // 省略
    }),
  );

  render(<Todos />);

  const listItem = await screen.findAllByRole("listitem");

  expect(listItem).toHaveLength(2);
});
```

![ 2023-10-11 21.31.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/cb5170e8-6f8d-48b3-2d51-2d134be33f44.png)

## useQueryを使う場合の実装とテスト
前置きが長くなってしまいましたが、いよいよ本題です。
続いて `useQuery` を使って fetch したデータをレンダリングする際のテストを見ていきます。

### コンポーネントのコード

```tsx TodosWithUseQuery.tsx
import { useQuery } from "@tanstack/react-query";
import axios from "axios";
import { Todo } from "./types";

export const TodosWithUseQuery: React.FC = () => {
  const { data } = useQuery<Todo[]>(["todos"], () =>
    axios.get<Todo[]>("/api/todos").then(({ data }) => data),
  );

  return (
    <div>
      <h2>todos with useQuery</h2>
      <ul>
        {data?.map((todo) => (
          <li key={todo.id}>
            <p>title: {todo.title}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

`useQuery` を使うことで格段に見やすくなりました。

### テストコード

#### NG
テストは先程と全く同じで、 `render` するコンポーネントの差し替えと `useQuery` を使用するための設定を追加します。

```ts spec.tsx

```