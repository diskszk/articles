
## useMemo

`useMemo` は、レンダー間で計算結果をキャッシュするための `React フック` です。
`useMemo` でラップした関数の結果をキャッシュに保存し、再レンダリング時にはキャッシュから前の結果を取り出します。

### 構文

```tsx
const result = useMemo(() => fn, [依存配列]);
```

### サンプルコード

親コンポーネントである `Appコンポーネント` と、その子コンポーネントである `TodoListコンポーネント` があるとします。
`Appコンポーネント` には入力フォームがあり、入力フォームの値が変更されれば `Appコンポーネント` および子コンポーネントである `TodoListコンポーネント` で再レンダーが走ります。
この時に、`TodoListコンポーネント` 内で重い計算処理をする関数が呼ばれているとします。通常であれば再レンダーが走るたび重い処理を行う関数はその処理を実行します。

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
  const calculatedTodos = heavyCalculate(todos);

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

![without_usememo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/e56ffe73-b65c-9cd5-c9fa-9c4e54942fdc.gif)

memo 化をしないままだと、`Appコンポーネント` の入力フォームに何かを入力するたびに、 `TodoListコンポーネント` で再レンダーが走り、そのたびに重い計算が行われてしまいます。
ここで `useMemo` を使って memo 化をしてあげます。

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

![usememo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5c085cfb-68b7-1fb3-61f2-a19747d66964.gif)

### どういうときに使うか

- 値の計算コストが高い時

https://ja.react.dev/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive

操作によって 1ms 秒以上かかるケースが memo 化する 1 つの目安となります。

ただし、なんでもかんでも memo 化する必要はありません。`useMemo` 自体のオーバーヘッドの問題もそうですし、コードが読みづらくなります。
また、依存配列の記載を間違えば正しく動かなくなる可能性もあります。

ドキュメントに書いてあるその他の使うべきケースを紹介します。
https://ja.react.dev/reference/react/useMemo#should-you-add-usememo-everywhere

> - `useMemo` で行う計算が著しく遅く、かつ、その依存値がほとんど変化しない場合
> - 計算した値を、memo でラップされたコンポーネントの props に渡す場合。この場合は、値が変化していない場合には再レンダーをスキップしたいでしょう。メモ化することで、依存値が異なる場合にのみコンポーネントを再レンダーさせることができます
> - その値が、後で何らかのフックの依存値として使用されるケース。例えば、別の useMemo の計算結果がその値に依存している場合や、useEffect がその値に依存している場合などです



