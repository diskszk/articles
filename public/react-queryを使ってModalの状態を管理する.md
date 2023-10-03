---
title: '@tanstack/react-queryを使ってModalの状態を管理する'
tags:
  - React
  - 状態管理
  - react-query
private: false
updated_at: '2023-06-06T18:27:07+09:00'
id: b754a53aff015587940b
organization_url_name: null
slide: false
ignorePublish: false
---

Reactのフロントエンドアプリケーションで、@tanstack/react-query (以下react-query)を使った状態管理の方法を紹介し、メッセージを表示するモーダルを作ってみようと思います。

https://tanstack.com/query/latest/


## 開発環境
node: 16.15.0
PC: mac(ver.11.7.1)

``` json
  "vite": "^4.3.9"
  "react": "^18.2.0",
  "react-dom": "^18.2.0"
  "typescript": "^5.0.2",
  "@tanstack/react-query": "^4.29.12",
  "@mui/material": "^5.13.4",
  "@emotion/react": "^11.11.0",
  "@emotion/styled": "^11.11.0",
```

## react-queryで状態管理を行う方法

### 初期設定

パッケージをインストールする

`$ yarn add -D @tanstack/react-query`

Providerを設定する

``` main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";
+ import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

+ const client = new QueryClient({
+   defaultOptions: {
+     queries: {
+       enabled: false,
+     },
+   },
+ });

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
+   <QueryClientProvider client={client}>
     <App />
+    </QueryClientProvider>
  </React.StrictMode>
);
```

QueryClientインスタンスを作成するときに、 `defaultOptions.queries.enabled: false` を設定することで、データfetchを抑制します。
なので、react-queryを使ってデータfetchをしたい場合には、後述する方法で設定を行うようにしてください。

### react-queryで状態管理を行う方法

#### データを設定する

``` type.tsx
type UserState = {
  name: string;
};
```

``` tsx
const queryClient = useQueryClient()

queryClient.setQueryData<UserState>(["user-state"], {name: "John"})
```

#### データを取り出す

``` tsx 
const data = useQuery<UserState>(["user-state"]).data;
console.log(data && data.user); // John
```


Reactコンポーネントあるいはカスタムフック関数内でこのように書くことで、異なるコンポーネント間であっても `{name: "John"}` のデータを設定し、取り出すことができます。


このとき重要なのは、`queryClient.setQueryData` と `useQuery` の第一引数である `QueryKey` の値が同じであることです。


https://tanstack.com/query/v4/docs/react/reference/QueryClient#queryclientgetquerydata


また、先述したreact-queryでデータfetchを行う場合には、main.tsxの QueryClientに `defaultOptions,queries.enabled: false` の設定を書かず、取り出すときに以下のように設定すればデータfetchは抑制されます。
``` tsx
const data = useQuery(["query-key"], {
  enabled: false,
}).data;
```

## メッセージモーダルの状態を管理する

### MessageModalコンポーネント
material-ui を使って、MessageModalコンポーネントを作ります。

https://mui.com/material-ui/react-modal/

``` tsx 
import { Box, Button, Typography, Modal } from "@mui/material";

export const  MessageModal = () => {
  return (
    <div>
      <Button onClick={handleOpen}>Open modal</Button>
      <Modal
        open={open}
        onClose={handleClose}
      >
        <Box>
          <Typography variant="h6" component="h2">
            Message Modal
          </Typography>
        </Box>
      </Modal>
    </div>
  );
}
```

![sample01.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/59ff1891-51e8-9faf-1144-30261d660cb6.gif)


今はまだMessageModalコンポーネント内で状態を管理しています。

### 状態を管理するカスタムフックを作る

MessageModalの開/閉の状態と、表示する文言をカスタムフック内で管理できるようにします。

``` ts
type ModalState = {
  isOpen: boolean;
  message: string;
}
```

また、モーダルを開く際には何かしらの文言が設定されていたほうがいいので、モーダルを開く際に同時に表示する文言も設定します。

カスタムフックの戻り値の型が以下のようになればOKです。

``` ts 
type ReturnType = {
  isOpen: boolean;
  message: string;
  open: (message: string) => void;
  close: () => void;
}
```

モーダルを表示するか否かの変数 `isOpen` とモーダルに表示する文言 `message` 及び、開/閉する関数 `open`, `close` を呼び出せるカスタムフックを作ります。

``` ts
export function useModalState(): ReturnType {
  const queryClient = useQueryClient();

  // initialDataを設定しないと戻り値の型がundefinedになる
  const { message, isOpen } = useQuery<ModalState>(["modal-state"], {
    initialData: {
      isOpen: false,
      message: "",
    },
  }).data;
  const open = (message: string) => {
    queryClient.setQueryData<ModalState>(["modal-state"], {
      isOpen: true,
      message,
    });
  };

  const close = () => {
    queryClient.setQueryData<ModalState>(["modal-state"], {
      isOpen: false,
      message: "",
    });
  };

  return { isOpen, message, open, close };
}
```

こうしてできたカスタムフックを、MessageModalコンポーネントと、MessageModalを表示させたいコンポーネントで使います。

``` tsx
export const MessageModal = () => {
  const { isOpen, message, close } = useModalState();
  const handleClose = () => close();

  if (!isOpen) {
    return null;
  }

  return (
    <div>
      <Modal
        open={isOpen}
        onClose={handleClose}
      >
        <Box sx={style}>
          <Typography  variant="h6" component="h2">
            {message}
          </Typography>
          <Button onClick={handleClose}>モーダルを閉じる</Button>
        </Box>
      </Modal>
    </div>
  );
};
```

``` tsx
import { MessageModal } from "./MessageModal";
import { useModalState } from "./useModalMessage";


const ChildA: React.FC = () => {
  const { open } = useModalState();

  return (
    <div>
      <p>A コンポーネント</p>
      <button onClick={() => open("This is A Component.")}>
        モーダルを開く
      </button>
    </div>
  );
};

const ChildB: React.FC = () => {
  const { open } = useModalState();

  return (
    <div>
      <p>B コンポーネント</p>
      <button onClick={() => open("This is B Component.")}>
        モーダルを開く
      </button>
    </div>
  );
};

function App() {
  const { open } = useModalState();

  return (
    <>
      <MessageModal />
      <div>
        <h1>Hello, React</h1>

        <button onClick={() => open("モーダルを開きました。")}>
          モーダルを開く
        </button>
        <ChildA />
        <ChildB />
      </div>
    </>
  );
}
```

![changed.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/7bf390dc-f808-7ce7-b5d8-36e0b0b8e8d4.gif)

以上によりMessageModalの状態を他のコンポーネントから管理すつことができるようになり、楽にMessageModalの開/閉が行えるよになりました。
