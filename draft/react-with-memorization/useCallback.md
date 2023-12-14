## useCallback

`useCallback` は、レンダー間で関数定義をキャッシュするための `React フック` です。
`useCallback` でラップした関数をキャッシュに保存し、再レンダリング時にはキャッシュから前の関数を取り出します。

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

親コンポーネントだけが再レンダーされます。

これは props として渡される関数が初回レンダー時と 2 回目以降で `===` であるかどうかによって決まります。
処理が一緒の関数であっても `===` の関係にはなりません。

![screenshot 2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/a20a3834-3c5d-b3a0-a0e2-4ab5cd6919f9.png)

`useCallback` を使わない場合、レンダリングごとに新しく生成された関数同士を比較し、異なる関数が渡ってきたとみなします。すなわち、前回と異なる props を検知して再レンダーが走るのです。
これは関数に限ったことではなく、オブジェクトを props 引数として渡す際にも生じます。

