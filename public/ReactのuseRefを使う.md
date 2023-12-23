---
title: ReactのuseRefを使う
tags:
  - React
private: false
updated_at: '2023-12-22T14:06:50+09:00'
id: 7101bb558d0cf6984ed2
organization_url_name: null
slide: false
ignorePublish: false
---

## useRefの基本的な使い方

コンポーネントに ref を追加するには、コンポーネント内で、`useRef` フックを呼び出し、第一引数に参照したい初期値を渡します。
変数 ref は意図的に書き換えが可能な変数となっています。

https://ja.react.dev/learn/escape-hatches

```tsx
const ref = useRef(0);
```

このように宣言した変数 ref は、中身は以下のようなオブジェクトとなり、以下のように参照できます。
```tsx
{
  current: 0
}

ref.current;        // 0

ref.current = 2;    // 2
```

また、`useRef` の型引数に追加したい値の型を指定できます。

```tsx
const countRef = useRef<number>(0);                 // number型
const idRef = useRef<string>("");                   // string型
const inputRef = useRef<HTMLInputElement>(null);    // inputタグのrefに代入できる
```

## 値を保持・参照する

https://ja.react.dev/reference/react/useRef#troubleshooting

`useRef` によって生成された変数は、その変数に変化があっても再レンダーは走らず、また、コンポーネントに再レンダーが走った時にはその値を保持し続けます。

### let変数・state変数との違い
`let` による変数定義でも書き換え可能な変数を作成できますが、`useRef` によって作成した変数との違いは、コンポーネントの再レンダーによって初期化されるかがあります。

`useState` によって宣言した変数は set 関数でしか書き換えができません。また、state 変数に変更があればコンポーネントには再レンダーが走ります。

```tsx
export const Counter: React.FC = () => {

  let count = 0;
  const incrementLet = () => {
    count = count + 1;

    console.log(count, "let変数");
  };

  const [countState, setCountState] = useState(0);
  const incrementCountState = () => {
    setCountState(countState + 1);
    console.log(countState, "state変数");
  };

  const countRef = useRef<number>(0);
  const incrementCountRef = () => {
    countRef.current = countRef.current + 1;
    console.log(countRef.current, "useRef変数");
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={incrementLet}>INCREMENT</button>
      <p>{countState}</p>
      <button onClick={incrementCountState}>INCREMENT</button>
      <p>{countRef.current}</p>
      <button onClick={incrementCountRef}>INCREMENT</button>
    </div>
  );
};
```
上のコードでは `let`、`useState`、`useRef` によって宣言した変数をそれぞれに対応するボタンのクリックでインクリメントします。
このコードの場合、ボタンのクリックによってブラウザ上の見た目が変わるのは、`useState` で宣言した変数をインクリメントした時だけです。なぜなら state 変数に変化が生じたことでコンポーネントが再レンダリングされたからです。
`let` 、`useRef` で宣言した変数は内部的にはインクリメントしていることがログからわかります。
2 つの違いとしては、state 変数をインクリメントした後で `let` で宣言した変数は値が 0 になりますが、`useRef` での場合再レンダー前の値を引き継ぎます。

:::note
React のドキュメントには  `レンダー中に current の値を読み取る（または書き込む）べきではない` とあります。
上記のコードでは `countRef.current` を画面に表示していますが、本来であれば画面に表示させるべきではありません。
:::

レンダー中に書き込み・読み込みをしてはいけないので、代わりにイベントハンドラから読み書きをします。

```tsx
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ You can read or write refs in effects
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ You can read or write refs in event handlers
    doSomething(myOtherRef.current);
  }
  // ...
}

```

### 再描画を抑制する

input タグに ref を渡すことで入力フォームの入力時の再レンダリングを抑制できます。
このようにフォームの入力値を DOM で扱うコンポーネントを `非制御コンポーネント` といいます。

```tsx
const UncontrolledForm: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);

   return (
    <div>
      <input ref={inputRef} />
      <button
        onClick={() => {
          console.log(inputRef.current.value);
        }}
      >
        click
      </button>
    </div>
  );
};
```

input タグに ref を渡すので `useRef` の型引数を `HTMLInputElement` にします。
入力した値は `inputRef.current.value` で参照できます。

## DOMを操作する

https://ja.react.dev/learn/manipulating-the-dom-with-refs

React で操作できない `useRef` で場合 DOM API を使って操作できます。
<!-- textlint-disable -->
例： 
<!-- textlint-disable -->
- フォーカスを当てる
- スクロールする
- サイズ位置を測定する

https://developer.mozilla.org/ja/docs/Web/API/HTML_DOM_API

ボタンのクリックで入力フォームにフォーカスを当てる例のコードを示します。
```tsx
const Component: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleClick = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <button onClick={handleClick}>フォーカスを当てる</button>
      <input ref={inputRef} />
    </div>
  );
};
```

## forwardRef

https://ja.react.dev/reference/react/forwardRef

基本的に `useRef` によって宣言した変数は自コンポーネントのみで利用でき、子コンポーネントに渡すには明示的にコンポーネントを `forwardRef` でラップする必要があります。

```tsx
const MyInput = forwardRef<HTMLInputElement, ComponentPropsWithRef<"input">>(
  (props, ref) => <input type="text" {...props} ref={ref} />
);

const Component: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <MyInput ref={inputRef} />
    </div>
  );
};
```
