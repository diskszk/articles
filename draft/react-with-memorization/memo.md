## memo

`useMemo` 同様、React においてパフォーマンスを向上させることができる機能です。
`memo` は `React` の API であり、コンポーネントを `memo` でラップすることで、コンポーネントをキャッシュに保存します。
親コンポーネントで再レンダーが走っても受け取る props 引数に変更がなければ、コンポーネントは再レンダーを行いません。

### 構文

```tsx
const MemorizedComponent: React.FC<Props> = React.memo(({props}) => {
  return (
    // 
  )
});
```
```tsx
const Component: React.FC<Props> = ({props}) => {
  return (
    // 
  );
}
const MemorizedComponent = React.memo(Component);
```

### 再レンダーが発生する場合

親コンポーネントから以前と異なる props が渡ってきた時に再レンダーが発生します。
memo 化されているコンポーネントで自身の state が変更された時は再レンダーが発生します。
また、コンポーネント内部で呼ばれているコンテクストが変更された場合には再レンダーが発生します。

### どういうときに使うか

コンポーネントが同一 props で頻繁に再レンダーされ、かつ、`レンダリングコストが高いコンポーネント` に有用です。

### サンプルコード

```tsx
const App: React.FC = () => {
  console.log("App");
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

![heavy.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/9b6bb24a-3119-14c2-0069-e04e2f6e687f.gif)

`Heavy` コンポーネントの再レンダーコストが高いため、入力フォームへ入力する時にもたつきます。

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

![memorized_heavy.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5d6e7e13-c6ab-8196-ef0b-20d20f5438d0.gif)

`memo` でラップし `Heavy` コンポーネントをメモ化することでもたつきがなくなります。