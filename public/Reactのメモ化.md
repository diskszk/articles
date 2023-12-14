---
title: Reactのメモ化
tags:
  - React
private: false
updated_at: '2023-12-14T09:00:00+09:00'
id: 9443470b25fcf6a3b6ef
organization_url_name: null
slide: false
ignorePublish: false
---

React のメモ化について調べたことと使い方、使うべきところをまとめます。

## メモ化とは

React においてメモ化をすることでキャッシュを再利用してコンポーネントの不要な再レンダーを防ぎます。
変数、関数、コンポーネントそれぞれに別の方法でメモ化できます。

## useMemo

`useMemo` は、レンダー間で計算結果をキャッシュするための React フックです。
`useMemo` でラップした関数の結果をキャッシュに保存し、再レンダリング時にはキャッシュから前の結果を取り出します。

### 構文

```tsx
const result = useMemo(() => fn, [依存配列]);
```

### サンプルコード

親コンポーネントである `Appコンポーネント` と、その子コンポーネントである `TodoListコンポーネント` があるとします。
`Appコンポーネント` には入力フォームがあり、入力フォームの値が変更されれば `Appコンポーネント` および子コンポーネントである `TodoListコンポーネント` で再レンダーが走ります。
この時に `TodoListコンポーネント` 内で重い計算処理をする関数が呼ばれているとします。通常であれば再レンダーが走るたび重い計算をする関数はその処理を実行します。

```App.tsx
const App: React.FC = memo(() => {
  const { todos } = useTodos();
  const [val, setVal] = useState("");
  return (
    <div>
      <input
        type="text"
        value={val}
        onChange={(ev) => setVal(ev.currentTarget.value)}
      />
      <TodoList todos={todos} />
    </div>
  );
});
```

```TodoList.tsx
const TodoList: React.FC<{ todos: Todo[] }> = ({ todos }) => {
  const calculatedTodos = heavyCalculate(todos);    // 重い計算

  return (
    <ul>
      {calculatedTodos.map((todo) => (
        <li key={todo.id}>
          <p>{todo.body}</p>
        </li>
      ))}
    </ul>
  );
};
```

![without_usememo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/35ddcea5-c7f1-c685-d6b0-b73a303cb836.gif)

メモ化をしないままだと、`Appコンポーネント` の入力フォームに何かを入力するたびに、 `TodoListコンポーネント` で再レンダーが走り、そのたびに重い計算が行われてしまいます。
ここで `useMemo` を使ってメモ化をしてあげます。

```TodoList.tsx
const TodoList: React.FC<{ todos: Todo[] }> = ({ todos }) => {
-  const calculatedTodos = heavyCalculate(todos); 
+  const calculatedTodos = useMemo(() => heavyCalculate(todos), [todos]);

  return (
    <ul>
      {calculatedTodos.map((todo) => (
        <li key={todo.id}>
          <p>{todo.body}</p>
        </li>
      ))}
    </ul>
  );
};
```

`useMemo` は、第一引数に処理関数、第二引数に依存配列を記載します。
`useMemo` は第二引数の依存配列が変化していない場合は、以前のレンダーで保存された値を返し、変化している場合は処理関数を再度実行し、結果をかえします。

上のコードでは第二引数の `todos` に変更がない限り `calculatedTodos` 変数にはキャッシュから値が格納されます。
入力フォームの変更があり、`TodoListコンポーネント` で再レンダーが走ったとしても、`useMemo` でラップされた関数はキャッシュされた値を返します。そうすることでつど実行されていた重い処理が一度目のレンダリングだけで済み、パフォーマンスが改善されます。

![usememo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/c83befff-9e6e-3c0f-d177-fe03f13a7b4c.gif)

### どういうときに使うか

- 値の計算コストが高い時

https://ja.react.dev/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive

操作によって 1ms 秒以上かかるケースがメモ化する 1 つの目安となります。

ただし、なんでもかんでもメモ化する必要はありません。`useMemo` 自体のオーバーヘッドの問題もそうですし、コードが読みづらくなります。
また、依存配列の記載を間違えば正しく動かなくなる可能性もあります。

ドキュメントに書いてあるその他の使うべきケースを紹介します。

https://ja.react.dev/reference/react/useMemo#should-you-add-usememo-everywhere

> - `useMemo` で行う計算が著しく遅く、かつ、その依存値がほとんど変化しない場合
> - 計算した値を、memo でラップされたコンポーネントの props に渡す場合。この場合は、値が変化していない場合には再レンダーをスキップしたいでしょう。メモ化することで、依存値が異なる場合にのみコンポーネントを再レンダーさせることができます
> - その値が、後で何らかのフックの依存値として使用されるケース。例えば、別の useMemo の計算結果がその値に依存している場合や、useEffect がその値に依存している場合などです

## memo (React.memo)

https://ja.react.dev/reference/react/memo

`useMemo` 同様、React においてパフォーマンスを向上させることができる機能です。
`memo` は React の API であり、コンポーネントを `memo` でラップすることで、コンポーネントをキャッシュに保存します。
親コンポーネントで再レンダーが走った際に受け取る props 引数の変更がなければ、コンポーネントは再レンダーを行いません。

### 構文

```tsx
const MemorizedComponent: React.FC<Props> = React.memo(({props}) => {
  return (
    // 
  );
});

または

const Component: React.FC<Props> = ({props}) => {
  
  return (
    // 
  );
}
const MemorizedComponent = React.memo(Component);
```

### 再レンダーが発生する場合

親コンポーネントから以前と異なる props が渡ってきた時に再レンダーが発生します。
メモ化されているコンポーネントで自身の state が変更された時は再レンダーが発生します。
また、コンポーネント内部で呼ばれているコンテクストが変更された場合には再レンダーが発生します。

### どういうときに使うか

コンポーネントが同一 props で頻繁に再レンダーされ、かつ、 **レンダリングコストが高いコンポーネント** に有用です。

### サンプルコード

```tsx
const Heavy: React.FC<{label: string}> = ({ label }) => {
  /* 
    とても重い描画処理を行う 
  */

  return (
    <p>{label}</p>
  );
};
```

```tsx
const Parent: React.FC = () => {
  console.log("Parent");
  const [val, setVal] = useState("");

  return (
    <div>
      <input onChange={(ev) => setVal(ev.target.value)} value={val} />
      <Heavy label="Heavy Component" />
    </div>
  );
};
```

`App` コンポーネントには入力フォームがあり、入力フォームとは無関係の高コストのレンダーロジックがある `Heavy` コンポーネントを呼びだすとします。
入力フォームに何かを入力するたびに `App` コンポーネントが再レンダーし、子コンポーネントである `Heavy` コンポーネントでも再レンダーが走ります。

![heavy.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/321f0098-89e5-d50d-4560-7b91e27fd802.gif)

`Heavy` コンポーネントの再レンダーコストが高いため、入力フォームへ入力する時にもたつきが発生しています。


続いて `memo` でラップします。

```tsx
+ const MemorizedHeavy = React.memo(Heavy);

const Parent: React.FC = () => {
  console.log("Parent");
  const [val, setVal] = useState("");

  return (
    <div>
      <input onChange={(ev) => setVal(ev.target.value)} value={val} />
-       <Heavy label="Heavy Component" />
+       <MemorizedHeavy label="Heavy Component" />
    </div>
  );
};
```

![memorized_heavy.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/2df6ce9a-0809-7f64-470c-e3b8e9ea59ff.gif)

`memo` でラップし `Heavy` コンポーネントをメモ化することでもたつきがなくなります。

## useCallback

https://ja.react.dev/reference/react/useCallback

`useCallback` は、レンダー間で関数定義をキャッシュするための React フックです。
`useCallback` でラップした関数をキャッシュに保存し、再レンダリング時にはキャッシュから関数を取り出します。

```tsx
const cachedFn = useCallback(fn, [依存配列])
```

`useMemo` との違いは、`useMemo` は変数をキャッシュするのに対して `useCallback` は関数をキャッシュします。

`useCallback` は 2 回目以降のレンダリング時に依存配列を比較し、前回と同じ場合、キャッシュした関数を返し、変化がある場合は今回のレンダーで渡された関数を返します。

### どういうときに使うか
基本的には `memo` とセットで使用します。
メモ化した子コンポーネントに props として関数を渡す時の挙動の違いを見てみます。

```tsx
const Child: React.FC<{ handleClick: () => void }> = React.memo(
  ({ handleClick }) => {
    console.log("Child");
    return <button onClick={handleClick}>ChildA</button>;
  }
);
```
メモ化したコンポーネントを props に関数を含めて呼び出します。
まずは、`useCallback` を使わない場合からです。

```tsx
const Parent: React.FC = () => {
  console.log("Parent");
  const [count, updateCount] = useState(0);
 
  const handleClick = () => {
    updateCount((count) => count + 1);
  };

  return (
    <div>
      <div>{count}</div>
      <Child handleClick={handleClick} />
    </div>
  );
};
```

![without-memo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/1cff1534-d38d-f61c-99f7-f5d51612b897.gif)

親コンポーネントの state が変わるたびに子コンポーネントにも再レンダーが走ります。

`handleClick` 関数を `useCallback` でラップしてメモ化した上で子コンポーネントへ渡します。

```tsx
const Parent: React.FC = () => {
  console.log("Parent");
  const [count, updateCount] = useState(0);
  const handleClick = useCallback(() => {
    updateCount((count) => count + 1);
  }, []);

  return (
    <div>
      <div>{count}</div>
      <Child handleClick={handleClick} />
    </div>
  );
};
```

![useCallback.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/0cecc0e7-dea1-bd35-ec5d-2eaed65606b2.gif)

親コンポーネントだけが再レンダーされるようになります。


これは props として渡される関数が初回レンダー時と 2 回目以降で `Object.is` で同一であるか判定します。
処理が一緒の関数であっても同一にはなりません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/161ad619-77fc-17bb-c432-69a1a3b0a059.png)

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/is

`useCallback` を使わない場合、レンダリングごとに新しく生成された関数同士を比較し、異なる関数が渡ってきたとみなします。すなわち、前回と異なる props を検知して再レンダーが走るのです。
これは関数に限ったことではなく、オブジェクトを props 引数として渡す際にも生じます。

## まとめ

今回改めて React におけるメモ化を調べてみて、後々パフォーマンスチューニングをするつもりでもなく、とりあえずで `useCallback` を使っていたなと実感しました。
また、メモ化を使う場面は限られているということを知りました。とはいえ必要な知識ではあるので今後必要に応じてパフォーマンスを良くするためにメモ化を使っていきたいです。




