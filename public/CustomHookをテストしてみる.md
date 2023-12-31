---
title: Custom Hook をテストしてみる
tags:
  - テスト
  - React
  - react-testing-library
  - reacthooks
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: ea4c3900fccb3202dab3
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Custom Hook を使うと面倒くさい事この上ない React をテストする時、ロジック部分を分離して小さくテストしやすくなりそうだなと思い、やってみることにしました。

今回は一番よくあるカウンターのデモと、実際の開発でもよく使うであろう input での入力の例を用いていきます。

[testing-library/react](https://testing-library.com/docs/react-testing-library/intro/)と[testing-library/react-hooks](https://react-hooks-testing-library.com/)を使ってテストを書いていきます。

#### GitHub
https://github.com/testing-library/react-testing-library

https://github.com/testing-library/react-hooks-testing-library



最終的な成果物リポジトリは[こちら](https://github.com/diskszk/hooks-test)です。

### カウンターの例

では早速、カウンターのデモを書いていきます。
ボタンを押すと state の値が 1 増えるというシンプルなものです。


#### reactコンポーネントにべた書きの場合

```Counter.tsx
export const Counter: React.FC = () => {
  const { count, setCount } = useState(0);

  const handleIncrement = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  },[]);

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={handleIncrement}>add</button>
    </div>
  );
};
```

#### Custom Hookに分離

```useCounter.ts
type ReturnType = {
  count: number;
  handleIncrement: () => void;
};

export const useCounter = (): ReturnType => {
  const [count, setCount] = useState(0);
  const handleIncrement = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  return {
    count,
    handleIncrement,
  };
};
```

```Counter.tsx
export const Counter: React.FC = () => {
  const { count, handleIncrement } = useCounter();

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={handleIncrement}>add</button>
    </div>
  );
};

```

useCounter.ts で `count` 変数と `handleIncrement` 関数を定義しました。
これらを Counter.tsx で呼びだす事で `count` と `handleIncrement` を利用してレンダリングや件数の実行を行なっています。

#### Custom Hooksのテスト

```useCounter.spec.ts
import { act, RenderResult, renderHook } from "@testing-library/react-hooks";
import { useCounter, ReturnType } from "../hooks/useCounter";

describe("useCounter", () => {
  let result: RenderResult<ReturnType>;

  beforeEach(() => {
    result = renderHook(() => useCounter()).result;
  });

  it("countの初期値は0である", () => {
    expect(result.current.count).toBe(0);
  });

  it("handleIncrementを1度呼んだ後、countの値は1である", () => {
    act(() => {
      result.current.handleIncrement();
    });
    expect(result.current.count).toBe(1);
  });
});

```

`react-testing-library` を使ってテストを実装しています。
`react-testing-library` の `renderHook` 関数を使う事で hooks のテストができるようになります。便利です。

ここでいう `handleIncrement` のような、state を更新する処理を act の中で呼ぶ必要があります。


### テキスト入力の例

次は react でよく使う input タグに文字を入力する際の Custom Hooks の例を書いていきます。
せっかくなので `firstName` と `lastName` の 2 つの state を扱うことにします。
追加で `firstName` には 8 文字以内、`lastName` には 6 文字以内というそれぞれ異なる簡単なバリデーションをつけてみます。
それぞれ input タグを通して入力した文字列を受け取ります。

#### reactコンポーネントにべた書き

```TextInput.tsx
export const TextInput: React.FC = () => {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  const handleChangeFirstName = useCallback(
    (ev: React.ChangeEvent<HTMLInputElement>) => {
      // バリデーション 8文字まで
      if (firstName.length >= 8) {
        return;
      }
      setFirstName(ev.target.value);
    },
    [firstName]
  );
  const handleChangeLastName = useCallback(
    (ev: React.ChangeEvent<HTMLInputElement>) => {
      //  バリデーション 6文字まで
      if (lastName.length >= 6) {
        return;
      }
      setLastName(ev.target.value);
    },
    [lastName]
  );

  return (
    <div>
      <input type="text" value={firstName} onChange={handleChangeFirstName} />
      <input type="text" value={lastName} onChange={handleChangeLastName} />
    </div>
  );
};

```

state が 3 つ 4 つとさらに増えていったら書くの結構大変そうですし、バリデーションも全部べた書きにしたらだいぶ辛くなりそうです。

#### Custom Hookに分離
```useTextInput.ts
export type ReturnType = [
  string,
  (ev: React.ChangeEvent<HTMLInputElement>) => void
];

export const useTextInput = (maxLength: number): ReturnType => {
  const [value, setValue] = useState("");

  const handleChange = useCallback(
    (ev: React.ChangeEvent<HTMLInputElement>) => {
      if (value.length >= maxLength) {
        return;
      }
      setValue(ev.target.value);
    },
    [value, maxLength]
  );

  return [value, handleChange];
};
```

```TextInput.tsx

export const TextInput: React.FC = () => {
  const [firstName, handleChangeFirstName] = useTextInput(8);
  const [lastName, handleChangeLastName] = useTextInput(6);

  return (
    <div>
      <input type="text" value={firstName} onChange={handleChangeFirstName} />
      <input type="text" value={lastName} onChange={handleChangeLastName} />
    </div>
  );
};
```

useCounter.ts で `value` 変数と `handleChange` 関数を定義しました。
react コンポーネントでの記述量がグッと減りました。
また、useTextInput の引数に設定した数値がそれぞれの最大入力可能文字数になります。

#### 関数のreturn
`useCounter.ts` と `useTextInput` で return の時の形が異なります。オブジェクトとして return する場合、呼び出し先で使える変数名・関数名は基本同じ名前でしか呼び出します。

```
const { count, handleIncrement } = useCounter();
```
一方、配列にいれて return することで、呼び出し先で使う時に別の名前をつけることができます。

```
const [ name, handleChangeName ] = useTextInput();
const [ counter, setCounter] = useState<number>(0);
const [ name, setName ] = useState<string>("");
```

前者の場合、使いたいものだけを呼び出せますが、後者だと順番に応じたものを呼びだすことになるので、例えば useTextInput から `handleChangeName` だけを呼び出したい時は以下のようにします。
```
const [ , handleChangeName ] = useTextInput();
```

#### Custom Hooksのテスト

```useTextInput.spec.tsx

describe("UseTextInput", () => {
  let result: RenderResult<ReturnType>;
  beforeEach(() => {
    result = renderHook(() => useTextInput(10)).result;
  });

  test("初期値は空文字である", () => {
    const [value] = result.current;
    expect(value).toBe("");
  });

  test("入力値が反映される", () => {
    const { container } = render(<input type="text" {...[result.current]} />);

    const input = container.querySelector("input");

    fireEvent.change(input!, { target: { value: "test" } });

    expect(input!.value).toBe("test");
  });
});
```

`useTextInput.spec.tsx` では DOM のテストと組み合わせて Custom Hook をテストしていきます。
テスト用の input を作成し `useTextInput` で作った値と関数を持たせます。

ここのやり方に関してはもっといい方法あるよ！これはよくないよ！!などの意見ありましたら是非お願いします！!

## さいごに

今後取っていきたい設計方針としてはこんな感じです。
- react コンポーネントから Custom Hook に分離する
- Custom Hook のユニットテストはどんどんやっていく
- DOM のユニットテストはリソースと相談しつつ頑張っていきます

テストと仲良くなって安全なプログラムを書けるように精進していきます。
