---
title: useEffectをちゃんと理解する
tags:
  - React
  - react-hooks
  - useEffect
private: false
updated_at: '2022-10-26T09:02:45+09:00'
id: 333511fb97d24f52a439
organization_url_name: null
---
# はじめに
## 記事を聞くに至った経緯
プログラミングに触れて初めて自分から勉強しにいったのがReactでした。当時はとりあえず動くものをつくるので精一杯だった為useEffectをなんとなく使うことはできるようにはなりましたが、ちゃんと理解はしておらず、そのままここまで来てしまいました。仕事で痛い目見る前にちゃんとしておこうと思い、改めて勉強しなおすに至りました。

## 目的
- useEffectをいまだになんとなくで使っているので理解を深めたい
- useEffectを使いこなしてバグのないコードを書けるようになりたい

# 分からなかったこと

- そもそも何をしているのか？
- 副作用
- ライフサイクル
- useEffectはいつ実行される?
- クリーンアップ
- dependencies(第二引数)に何を入れればいかわからない
  - [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)に脳死で従っていた

これらの疑問を解決していきましょう！


## そもそも何をしているのか？
まずは[React公式ドキュメント](https://ja.reactjs.org/docs/hooks-effect.html)ですね。

> 関数コンポーネント内で副作用を実行することができるようになります

**副作用**について詳しくは後述しますが、useEffectを用いることで、コンポーネントのレンダー後に実行される様々な種類の副作用を実行できるようになります。つまり、useEffectを使うことで、レンダー後に何かの処理をしないといけないという事をReactに伝えます。

> 関数コンポーネントというのは、JavaScriptの関数の書き方でReactコンポーネントを表現する方法です。別の方法にclassコンポーネントがありますが、私は関数コンポーネントでしか書いたことがありません。
useEffectは関数コンポーネントで使えるreact-hooksの1つです。  
classコンポーネントでは1コンポーネントにつきcomponentDidMount,componentDidUpdateなどの副作用のコードを1つしか書けず、関心事が分離できなません。対して関数コンポーネントでは、useEffectを複数書く事ができるので関心ごとの分離がしやすくなりました。

具体的に何をしているかというと、基本的に以下の流れで処理が走ります。

1. useStateのset関数等によって関数の値が書き換える
1. DOMを書き換える
1. 画面を再レンダリングする
1. レンダリング**後**にuseEffectの第一引数に渡した処理を実行する

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

プログラミングにおける「副作用」とは、ある機能がプログラム上のデータを変化させ、それ以降の演算の結果に影響を与えること、つまり、数学的関数は本来計算結果を返すことが目的であるが、その過程で周囲の状態を変化させてしまうことです。  
**「薬を飲んだら眠くなる」のような副作用**とは異なる意味を持ちます。  
必ずしも悪影響をもたらすという訳ではありません。大切なのは、**適切な場所に配置する事**です。

副作用の無い関数・副作用のある関数の例を挙げてみます。

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

逆に副作用のある関数には参照透過性がなく、機能(関数)の外側の状態を変化させています。上のadd関数は実行するたびに異なる結果になるので副作用があると言えます。

以上はプログラミングにおける副作用です。フロントエンドでの具体的な話も挙げていきます。

Reactにおける副作用とは、なんらかの処理の結果、DOMが書き換えられ再レンダリングが生じることです。

簡単な例ですと、ボタンを押すとカウントの値が変化し、画面に表示する数値が変わるコードがあります。

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

ボタンを押下する事でhandleClick関数が実行され、countの値が変わりDOMが書き換えられ、再レンダーが走ります。  
handleClickが副作用のある関数です。

> リアクティブプログラミング
Reactは元々、コンポーネントに渡された引数によって仮想DOMを操作し、渡されたハンドラによって状態を更新するという考えで作られていて、それがまさにリアクティブプログラミングだったためにReactという名称がつけらています。  
あるコンポーネントの引数やイベントによって他の状態も変化するという構造です。


## ライフサイクル

Reactにおけるライフサイクルとは、以下の三つの期間に分けられます。
- Mounting
  - コンポーネントが生成されて、レンダリングされるまでの期間

- Updating
  - コンポーネントが管理するデータがユーザーによって更新される期間

- Unmounting
  - コンポーネントが不要となり、破棄するための期間

このようなマウント処理、アンマウント処理の繰り返しをライフサイクルと呼びます。  
それぞれのタイミングでReactコンポーネントがレンダリングされます。

## useEffectはいつ実行されるのか

useEffectは第二引数に何を指定するかによって実行されるタイミングが異なります。

- 第二引数に何も指定しない場合
レンダリング(Mounting, Updating, Unmounting)毎に実行します。
つまり、コンポーネント内で値が変わり、再レンダリングされるような場合には再レンダリング毎に実行します。

- 第二引数に空の配列を指定した場合
初回レンダリング時のみ実行されます。

- 第二引数の配列に1つ以上の値が指定されている場合
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



countの値を変更する時

1. 「+ボタン」の押下によりcountUp関数を呼ぶ
1. count の値が変わる
1. 再レンダリングされる
1. 再レンダリング後にuseEffect_1が呼ばれる
1. レンダリング前と再レンダリング後でcountの値が異なるから、useEffect_3が呼ばれる	

isOpenの値を変更する時

1. open or close ボタンの押下によりhandleToggle関数を呼ぶ
1. isOpenの値が変わる
1. 再レンダリングされる
1. 再レンダリング後にuseEffect_1が呼ばれる
1. レンダリング後と再レンダリング後でcountの値は同じなのでuseEffect_3はスキップされる


## クリーンアップ

何らかの外部データソースへの購読をセットアップしたい場合や、setTimeout関数などのタイマー関数を利用する場合、メモリリークが発生しないようにクリーンアップが必要となります。

useEffectでは、副作用関数内でクリーンアップ関数をreturnする事で、マウント時に実行した処理を、２度目以降のレンダリング時に前回の副作用を消す事ができます。

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

AppコンポーネントでTimerコンポーネントの表示/非表示をコントロールします。

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

こちらをブラウザで実行すると、Mountボタン押下後にTimerコンポーネントがマウントされ、レンダー後にuseEffect内の処理が実行されます。  
このコードだと、countの値が変わるたびにuseEffectの関数が実行されます。つまり、setTimeout関数が並行して呼び出されてしまい、スクリーンショットのように1秒毎に減少するcount数が異なるバグが生じます。  

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
変更を加えた点としては、useEffect内でクリーンアップ関数を追加しています。
クリーンアップ関数では再レンダー時に前回のsetInterval関数を破棄し、再レンダー後に新しいsetInterval関数が実行されます。これによってちゃんと1秒ごとにcountが1ずつ減少するようになり、メモリリークも回避できます。

![goodTimer.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/b1ce88f1-228d-aa68-d2a2-e362cef21d96.gif)



## dependencies

useEffectの第二引数には適切な値を指定しなければバグを生みます。よくあるバグとしては無限レンダリングが挙げられます。では、どういう場合に無限レンダリングが生じるかというと、useEffect内の処理で変更させる値を第二引数に指定している場合です。

```無限レンダリングが生じるuseEffect.tsx
  useEffect(() => {
    console.log("countを更新", count);
    setCount((prev) => prev + 1);
  }, [count]);
```

何故無限レンダリングが生じてしまうかというと  

1. 初回レンダリング時にuseEffectが呼び出され、countの値を変更する
1. レンダリング前後でcountの値が変わっているのでuseEffectが呼び出される
1. useEffectが呼び出されcountの値を変更する
1. 再レンダリングする

の2~4が繰り返されてしまうからです。  
**useEffect内で変更させたい値は第二引数には含めない**ようにしましょう。

記事の冒頭で、[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)に脳死で従っていてまずいと述べましたが、こちらのルールはReact公式ドキュメンとでも推奨しており、必ず入れるようにしています。これに限った話では無いですが、**理屈を分かった上で使うのが大切**だという事です。

# 最後に

長文になってしまいましたが最後まで読んでいただきありがとうございます。  
何か間違っている点や補足などございましたらコメントお願いします！
