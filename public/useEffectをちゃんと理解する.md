---
title: useEffectをちゃんと理解する
tags:
  - React
  - react-hooks
  - useEffect
private: false
updated_at: '2023-11-21T18:12:25+09:00'
id: 333511fb97d24f52a439
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
## 記事を聞くに至った経緯
プログラミングに触れて初めて自分から勉強しにいったのが React でした。当時はとりあえず動くものをつくるので精一杯だった為 useEffect をなんとなく使うことはできるようにはなりましたが、ちゃんと理解はしておらず、そのままここまで来てしまいました。仕事で痛い目見る前にちゃんとしておこうと思い、改めて勉強しなおすに至りました。

## 目的
- useEffect をいまだになんとなくで使っているので理解を深めたい
- useEffect を使いこなしてバグのないコードを書けるようになりたい

# 分からなかったこと

- そもそも何をしているのか？
- 副作用
- ライフサイクル
- useEffect はいつ実行される？
- クリーンアップ
- dependencies(第二引数)に何を入れればいかわからない
  - [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)に脳死でしたがっていた

これらの疑問を解決していきましょう！


## そもそも何をしているのか？
まずは[React公式ドキュメント](https://ja.reactjs.org/docs/hooks-effect.html)ですね。

> 関数コンポーネント内で副作用を実行することができるようになります

**副作用**について詳しくは後述しますが、useEffect を用いることで、コンポーネントのレンダー後に実行される様々な種類の副作用を実行できるようになります。つまり、useEffect を使うことで、レンダー後に何かの処理をしないといけないという事を React に伝えます。

> 関数コンポーネントというのは、JavaScriptの関数の書き方でReactコンポーネントを表現する方法です。別の方法にclassコンポーネントがありますが、私は関数コンポーネントでしか書いたことがありません。
useEffect は関数コンポーネントで使える react-hooks の 1 つです。  
class コンポーネントでは 1 コンポーネントにつき componentDidMount,componentDidUpdate などの副作用のコードを 1 つしか書けず、関心事が分離できなません。対して関数コンポーネントでは、useEffect を複数書く事ができるので関心ごとの分離がしやすくなりました。

具体的に何をしているかというと、基本的に以下の流れで処理が走ります。

1. useState の set 関数等によって関数の値が書き換える
1. DOM を書き換える
1. 画面を再レンダリングする
1. レンダリング**後**に useEffect の第一引数に渡した処理を実行する

```tsx
補足
useEffect(() => {  
  // 何らかの処理  
} ,[dependencies])"  
における  
 "() => {  
  // 何らかの処理  
　}"  
の箇所が第一引数のコールバック関数です。
```


## 副作用

プログラミングにおける「副作用」とは、ある機能がプログラム上のデータを変化させ、それ以降の演算の結果に影響を与えること。つまり数学的関数は本来計算結果を返すことが目的であるが、その過程で周囲の状態を変化させてしまうことです。  
**「薬を飲んだら眠くなる」のような副作用**とは異なる意味を持ちます。  
必ずしも悪影響をもたらすという訳ではありません。大切なのは、**適切な場所に配置する事**です。

<!-- textlint-disable -->
副作用が無い関数・副作用がある関数の例を挙げてみます。
<!-- textlint-enable -->

```副作用のない関数.ts
function double(x: number) :number {
	return 2 * x;
}
```

```副作用のある関数.ts
let num = 0;
// グローバル変数numを参照し、それを増加させて返す関数
function add(x: number): void {
	num = num + x;
}
```

副作用の無い関数には参照透過性があります。
> 参照透過性
    - 同じ条件を与えれば結果は同じになる
    - 他のいかなる機能の結果にも影響を与えない

逆に副作用のある関数には参照透過性がなく、機能(関数)の外側の状態を変化させています。上の add 関数は実行するたびに異なる結果となるので副作用があると言えます。

以上はプログラミングにおける副作用です。フロントエンドでの具体的な話も挙げていきます。

<!-- textlint-disable -->
React における副作用とは、何らかの処理の結果、DOM が書き換えられ再レンダリングが生じることです。

簡単な例ですと、ボタンを押すとカウントの値が変化し、画面に表示する数値が変わるコードがあります。
<!-- textlint-enable -->

```Counter.tsx
const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount((prevCount) => prevCount +1);
  };

  return (
    <div>
      <p>
        カウントは{count}です。
      </p>
      <button onClick={handleClick}>+</button>
    </div>
  );
}
```
<!-- textlint-disable -->
ボタンを押下する事で handleClick 関数が実行され、count の値が変わり DOM が書き換えられ、再レンダーが走ります。  
handleClick が副作用のある関数です。
<!-- textlint-enable -->

> リアクティブプログラミング
React は元々、コンポーネントに渡された引数によって仮想 DOM を操作し、渡されたハンドラによって状態を更新するという考えで作られていて、それがまさにリアクティブプログラミングだったために React という名称がつけらています。  
あるコンポーネントの引数やイベントによって他の状態も変化するという構造です。


## ライフサイクル

React におけるライフサイクルとは、以下の 3 つの期間に分けられます。
- Mounting
  - コンポーネントが生成されて、レンダリングされるまでの期間

<!-- textlint-disable -->
- Updating
  - コンポーネントが管理するデータがユーザーによって更新される期間
<!-- textlint-enable -->

- Unmounting
  - コンポーネントが不要となり、破棄するための期間

このようなマウント処理、アンマウント処理の繰り返しをライフサイクルと呼びます。  
それぞれのタイミングで React コンポーネントがレンダリングされます。

## useEffectはいつ実行されるのか

useEffect は第二引数に何を指定するかによって実行されるタイミングが異なります。

- 第二引数に何も指定しない場合
レンダリング(Mounting, Updating, Unmounting)毎に実行します。
つまり、コンポーネント内で値が変わり、再レンダリングされるような場合には再レンダリング毎に実行します。

- 第二引数に空の配列を指定した場合
初回レンダリング時のみ実行されます。

- 第二引数の配列に 1 つ以上の値が指定されている場合
渡された値がレンダリング前後で変更がなければ処理をスキップし、再レンダリング後に変更していれば実行します。

  
コードで見てみます。

```Counter.tsx
const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  const [isOpen, setIsOpen] = useState(false);

  const countUp = () => {
    setCount((prevCount) => prevCount + 1);
  };
  // useEffect_1
  useEffect(() => {
    console.log("再レンダリングされるたび実行");
  });
  // useEffect_2
  useEffect(() => {
    console.log("初回レンダリング時のみ実行");
  },[])
  // useEffect_3
  useEffect(() => {
    console.log("countの値が変わるたび実行");
  }, [count]);

  return (
    <>
      {console.log("----レンダリング----")}

      <button onClick={countUp}>+</button>
      <p>count: {count}</p>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? "close" : "open"}
      </button>
    </>
  );
};
```

![useEffectDemo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/efe98c26-b4d5-6ef0-9e1d-4e5d9ff44f95.gif)



count の値を変更する時。

1. 「+ボタン」の押下により countUp 関数を呼ぶ
1. count の値が変わる
1. 再レンダリングされる
1. 再レンダリング後に useEffect_1 が呼ばれる
1. レンダリング前と再レンダリング後で count の値が異なるから、useEffect_3 が呼ばれる	

isOpen の値を変更する時。

1. open or close ボタンの押下により handleToggle 関数を呼ぶ
1. isOpen の値が変わる
1. 再レンダリングされる
1. 再レンダリング後に useEffect_1 が呼ばれる
1. レンダリング後と再レンダリング後で count の値は同じなので useEffect_3 はスキップされる


## クリーンアップ

何らかの外部データソースへの購読をセットアップしたい場合や、setTimeout 関数などのタイマー関数を利用する場合、メモリリークが発生しないようにクリーンアップが必要となります。

useEffect では、副作用関数内でクリーンアップ関数を return する事で、マウント時に実行した処理を、2 度目以降のレンダリング時に前回の副作用を消す事ができます。

なぜクリーンアップが必要になるのか、こちらもサンプルコードをみてみましょう。  

```Timer.tsx
const Timer: React.FC<{
  setIsDisplay: (value: React.SetStateAction<boolean>) => void;
}> = ({ setIsDisplay }) => {
  const [count, setCount] = useState(10);


  useEffect(() => {
    console.log("再レンダー");
    if (count < 0) {
      setIsDisplay(false);
      return;
    }

    setInterval(() => {
      setCount((prev) => prev - 1);
    }, 1000);
  }, [count, setIsDisplay]);

  return (
    <p>{count}秒後にunMountします</p>
  );
};
```

App コンポーネントで Timer コンポーネントの表示/非表示をコントロールします。

```App.tsx
const App: React.FC = () => {
  const [isDisplay, setIsDisplay] = useState(true);
  const handleToggleDisplay = () => {
    setIsDisplay(!isDisplay);
  };

   return (
    <div>
      <span>
        コンポーネントを
        <button onClick={handleToggleDisplay}>
          {isDisplay ? "Unmount" : "Mount"}
        </button>
      </span>
      {isDisplay && <Timer setIsDisplay={setIsDisplay} />}
    </div>
  );
}
```

こちらをブラウザで実行すると、Mount ボタン押下後に Timer コンポーネントがマウントされ、レンダー後に useEffect 内の処理が実行されます。  
このコードだと、count の値が変わるたびに useEffect の関数が実行されます。つまり、setTimeout 関数が並行して呼び出されてしまい、スクリーンショットのように 1 秒毎に減少する count 数の異なるバグが生じます。  

![badTimer.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/f5537584-ba93-07b0-24b5-ae26debaac2c.gif)


また、ブラウザの開発者コンソールでメモリリークが発生している事が確認できます。

![errorlog.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5f739441-9408-f035-78e5-04b41335182b.jpeg)


これらのバグを生まないコードを書いていきましょう。

```Timer.tsx
const Timer: React.FC<{
  setIsDisplay: (value: React.SetStateAction<boolean>) => void;
}> = ({ setIsDisplay }) => {
  const [count, setCount] = useState(10);

  useEffect(() => {
    console.log("再レンダー");
    if (count < 0) {
      setIsDisplay(false);
      return;
    }

    const doInterval = setInterval(() => {
      setCount((prev) => prev - 1);
    }, 1000);

    return () => {
      console.log("前回のintervalをクリーンアップします");
      clearInterval(doInterval);
    };
  }, [count, setIsDisplay]);

  return (
    <p>{count}秒後にunMountします</p>
  );
};
```
変更を加えた点としては、useEffect 内でクリーンアップ関数を追加しています。
クリーンアップ関数では再レンダー時に前回の setInterval 関数を破棄し、再レンダー後に新しい setInterval 関数が実行されます。これによってちゃんと 1 秒ごとに count が 1 ずつ減少するようになり、メモリリークも回避できます。

![goodTimer.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/b1ce88f1-228d-aa68-d2a2-e362cef21d96.gif)



## dependencies

useEffect の第二引数には適切な値を指定しなければバグを生みます。よくあるバグとしては無限レンダリングが挙げられます。では、どういう場合に無限レンダリングが生じるかというと、useEffect 内の処理で変更させる値を第二引数に指定している場合です。

```無限レンダリングが生じるuseEffect.tsx
  useEffect(() => {
    console.log("countを更新", count);
    setCount((prev) => prev + 1);
  }, [count]);
```

何故無限レンダリングが生じてしまうかというと、以下の 2〜4 が繰り返されてしまうからです。  

1. 初回レンダリング時に useEffect が呼び出され、count の値を変更する
1. レンダリング前後で count の値が変わっているので useEffect が呼び出される
1. useEffect が呼び出され count の値を変更する
1. 再レンダリングする

**useEffect内で変更させたい値は第二引数には含めない**ようにしましょう。

記事の冒頭で、[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)に脳死でしたがっていてまずいと述べましたが、こちらのルールは React 公式ドキュメンとでも推奨しており、必ず入れるようにしています。これに限った話では無いですが、**理屈を分かった上で使うのが大切**だという事です。

# 最後に

長文になってしまいましたが最後まで読んでいただきありがとうございます。  
何か間違っている点や補足などございましたらコメントお願いします！
