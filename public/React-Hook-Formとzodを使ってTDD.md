---
title: React-Hook-Formとzodを使ってTDD
tags:
  - TDD
  - テスト
  - React
  - react-hook-form
private: false
updated_at: '2023-07-14T22:21:01+09:00'
id: 154590fbcda7f9a72dfa
organization_url_name: null
---

## TDD とは

https://ja.wikipedia.org/wiki/%E3%83%86%E3%82%B9%E3%83%88%E9%A7%86%E5%8B%95%E9%96%8B%E7%99%BA

テスト駆動開発 (Test Driven Development : TDD) は、ソフトウェア開発プロセスの 1 つで、テストを先に書いてから実装を進める手法です。このプロセスに沿って開発を進めることで、バグの早期発見やコード品質の向上、可読性の高いコードの実装を目的としています。

具体的には、以下のような手順で開発が進められます。

1. テストケースを書く
最初に、書きたい機能（メソッドやクラスなど）のテストケースを書きます。このテストケースでは、想定される入力に対して、どのような結果が返ってくるべきかを定義します。このテストケース自体は、まだ通らないものになります。

2. テストが通る最小限の実装をする
次に、テストを通すために最小限の実装をします。この実装は、機能に必要な最小限の処理のみを実装します。

3. テストが通るリファクタリングを行う
テストが通る最小限の実装ができたら、その後にリファクタリングを行い、コード品質を向上させます。このリファクタリングでは、可読性や保守性の高いコードにすることが主な目的となります。

4. 1-3 を繰り返す
次に、新しいテストケースを追加して、上記の手順を繰り返します。このように、テストケースを書いてから実装を進めていくことで、バグを早期発見でき、また可読性や保守性に優れたコードを実装できます。

このように、TDD は品質の高いソフトウェアを開発するための手法のひとつとなっています。

## 開発環境
node: 18.14.0
react: 18.2.0
typescript: 5.1.6
vite: 4.4.2
jest: 29.6.1
testing-library/react: 14.0.0
react-hook-form: 7.45.1
zod: 3.21.4

vite-cli から react-ts のテンプレーとを使って react 環境を作りました。

### React Hook Form とは

https://www.react-hook-form.com

React でのフロントエンド開発でフォームを扱いやすくするためのライブラリです。フォームのバリデーションや送信などの機能をサポートしており、Hooks を利用することで簡単に実装できます。また、スケーラビリティに優れ高速で動作するという特徴もあります。

### zod とは

https://zod.dev

TypeScript での型バリデーションをより簡単に行うことができるライブラリです。主に、入力データのバリデーションやサーバーから返ってきた JSON データの型チェックに利用されます。例えば、API のリクエストパラメータのバリデーションやフォームの入力値チェックなどに使われることが多いです。また、zod はオブジェクトのネストやオプションのバリデーションなどをサポートしており、複雑なオブジェクトを扱う場合にも便利です。

## 仕様

今回はタイトルと本文入力して記事を作成する記事作成フォームを作成します。  

1. 仕様を定義する
それぞれのフィールドの最大・最小文字数、および送信ボタンの状態を定義します。

- タイトル
  - 最小 1 文字
  - 最大 25 文字
  - 1 行のみ
- 本文
  - 最小 1 文字
  - 最大 140 文字
  - 複数行表示する
  - 初期状態は 4 行表示する

- 送信ボタンの状態
  - 初期表示は非活性である
  - タイトルか本文のどちらかの入力を変更した場合、活性になる
  - 送信中は非活性である

送信ボタンがクリックされた時の処理についても定義します。
- タイトルに入力された文字数が 0 文字、または、25 文字より大きい場合、 submit 処理は実行しない
- 本文に入力された文字数が 0 文字、または、140 文字より大きい場合、 submit 処理は実行しない
- 送信後は入力フォームを初期化する

送信処理の成功 / 失敗の通知について定義します。
- 送信が完了したら、送信が完了した旨のメッセージを snackbar に表示する
- 送信中にエラーが発生したら、エラーが発生した旨を snackbar に表示する
  - snackbar は 3000mm 秒後非表示になる

1. TODO　リストを作成する

TDD の流儀に習い TODO リストを作成します。

- [ ] タイトルか本文に 1 文字以上入力すると、送信ボタンが活性になる
- [ ] 初期事表示では送信ボタンが非活性
- [ ] 送信ボタンが非活性から活性に切り替わる
- [ ] タイトルの入力欄にバリデーションエラーメッセージを表示する
- [ ] 本文の入力欄にバリデーションエラーメッセージを表示する
- [ ] submit 関数が実行することを確認する
- [ ] バリデーションエラーがある場合、submit 関数は実行しない
- [ ] 送信中は非活性である
- [ ] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

## 実装

### ボタンの活性 | 非活性のテスト
一番手をつけやすそうな `初期事表示では送信ボタンが非活性` から書いていきます。
`CreatePostForm.spec.tsx` というファイルを作成します。

```tsx CreatePostForm.spec.tsx
import { screen, render } from "@testing-library/react";

test("初期事表示では送信ボタンが非活性", () => {
  render(<CreatePostForm />);

  expect(screen.getByRole("button", { name: "送信" })).toBeDisabled();
});
```

この状態でテストを実行してみます。もちろん落ちます。

![01NG.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/94462ab6-7a4e-7d1f-99bd-268e7f50aab6.png)

まずはこれで OK です。

次に、テストを Pass するために `CreatePostForm.tsx` を仮実装します。
``` tsx CreatePostForm.tsx
export const CreatePostForm: React.FC = () => {
  return (
    <div>
      <button disabled>送信</button>
    </div>
  );
};
```

テストファイルでコンポーネントをインポートしてからテストを実行します。

![02OK.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/d9b858f4-4a19-dc91-876f-ee65063b2255.png)

テストが無事通りました！これで TODO リストの `初期事表示では送信ボタンが非活性` は OK です。


- [x] タイトルか本文に 1 文字以上入力すると、送信ボタンが活性になる
- [x] 初期事表示では送信ボタンが非活性
- [ ] 送信ボタンが非活性から活性に切り替わる
- [ ] タイトルの入力欄にバリデーションエラーメッセージを表示する
- [ ] 本文の入力欄にバリデーションエラーメッセージを表示する
- [ ] submit 関数が実行することを確認する
- [ ] バリデーションエラーがある場合、submit 関数は実行しない
- [ ] 送信中は非活性である
- [ ] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### 入力フォームのテスト

次は `タイトルか本文に 1 文字以上入力すると、送信ボタンが活性になる` をやっていきます。

まずはテストを追加します。
```tsx CreatePostForm.spec.tsx
// input への入力をエミュレートする
import userEvent from "@testing-library/user-event";
const user = userEvent.setup();

test("タイトルに1文字以上入力すると、送信ボタンが活性になる", async () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });

  await user.type(title, "テスト用タイトル");

  expect(screen.getByRole("button", { name: "送信" })).toBeEnabled();
});

test("本文に1文字以上入力すると、送信ボタンが活性になる", async () => {
  render(<CreatePostForm />);

  const body = screen.getByRole("textbox", { name: "本文" });

  await user.type(body, "テスト用本文");

  expect(screen.getByRole("button", { name: "送信" })).toBeEnabled();
});
```

テストコードを `screen.getByRole("textbox", { name: "タイトル" })` 、プロダクトコード側を `<input aria-label="タイトル" ... />` として、 aria-label の値を screen.getByRole("textbox", {name: ""}) の name に指定することで　input タグを取得できます。

ひとまず入力フォームを実装します。本文の入力フォームは複数行かつ、初期状態で 4 行表示したいので textarea タグを用います。

``` tsx CreatePost.tsx
export const CreatePostForm: React.FC = () => {
  return (
    <div>
      <form>
        <input type="text" aria-label="タイトル" />
        <textarea aria-label="本文" rows={4} />
        <button disabled>送信</button>
      </form>
    </div>
  );
};
```

React Hook Form と zod を用いて入力フォームを作っていきます。
```tsx
import z from "zod";
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  title: z.string(),
  body: z.string(),
}).strict();

type Input = z.infer<typeof schema>;
```

zod を用いて記事作成入力フォームの項目を定義します。
必要最低限 `{ title: z.string(), body: z.string() }` と書くことで、 title, body それぞれが string 型であるということを定義します。


次に React Hook Form を用いて入力フォームを作っていきます。
formState.isDirty は、ユーザーがいずれかの入力を変更した後に true へと変わるので、これを利用して button の disabled 属性を切り替えます。

```tsx CreatePost.tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

export const CreatePostForm: React.FC = () => {
  const {
    register,
    formState: { isDirty },
  } = useForm<Input>({
    resolver: zodResolver(schema),
    defaultValues: {
      title: "",
      body: "",
    },
  });
  return (
    <div>
      <form>
        <input
          type="text"
          aria-label="タイトル"
          {...register("title")}
        />
        <textarea aria-label="本文" rows={4} {...register("body")} />
        <button disabled={!isDirty}>送信</button>
      </form>
    </div>
  );
};
```

![03isDirtyOK.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/d98967b2-72dd-199a-fea7-34da6ace3ce9.png)

テストが通りました！
これで `タイトルか本文に 1 文字以上入力すると、送信ボタンが活性になる` と `送信ボタンが非活性から活性に切り替わる` は OK です。

- [x] 送信ボタンが非活性から活性に切り替わる
- [ ] タイトルの入力欄にバリデーションエラーメッセージを表示する
- [ ] 本文の入力欄にバリデーションエラーメッセージを表示する
- [ ] submit 関数が実行することを確認する
- [ ] バリデーションエラーがある場合、submit 関数は実行しない
- [ ] 送信中は非活性である
- [ ] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### バリデーションのテスト

次は zod のバリデーションを用いて `タイトルの入力欄にバリデーションエラーメッセージを表示する` のテストを書きます。

```tsx spec.tsx
test("タイトルが入力されていない場合、エラーメッセージを表示する", async () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });

  // タイトル入力欄をクリックし、フォーカスをあわせる
  await user.click(title);

  // フォーカスを外しバリデーションエラーをチェックする
  await user.tab();

  await waitFor(() => {
    expect(title).toBeInvalid();
  });
  expect(title).toHaveErrorMessage("タイトルは必ず入力してください。");
});
test("タイトルに26文字入力された場合、エラーメッセージを表示する", async () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });

  await user.click(title);
  await user.type(title, "A".repeat(26));

  await user.tab();

  await waitFor(() => {
    expect(title).toBeInvalid();
  });
  expect(title).toHaveErrorMessage("タイトルは25文字以内で入力してください。");
});
```

次にバリデーションとエラー時のメッセージを作成します。
```ts
const schema = z
  .object({
    title: z
      .string()
+      .min(1, "タイトルは必ず入力してください。")
+      .max(25, "タイトルは25文字以内で入力してください。"),
    body: z.string(),
  })
  .required()
  .strict();
```

タイトル入力フォームにバリデーションとバリデーションエラーメッセージを追加します。

```tsx
export const CreatePostForm: React.FC = () => {
+  const errorMessageId = useId();

  const {
    register,
    formState: { isDirty, errors },
  } = useForm<Input>({
    resolver: zodResolver(schema),
    defaultValues: {
      title: "",
      body: "",
    },
+    mode: "onBlur",  // フォーカスが外れたときにバリデーションをチェックする
  });

  return (
    ...
     <input
          type="text"
          aria-label="タイトル"
          aria-invalid={!!errors.title}
          aria-errormessage={errorMessageId}
          {...register("title")}
        />
        {/* バリデーションエラー時に表示するエラーメッセージ */}
        {errors.title && (
          <p role="alert" id={errorMessageId}>
            {errors.title?.message}
          </p>
        )}
      ...
  );
}
```

テスト結果は OK です。

![ 2023-07-13 16.47.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/15afb446-e55e-a27e-7627-98ce5c454f6f.png)

同じ要領で本文にもバリデーションとエラーメッセージを作成します。

```tsx spec.tsx
test("本文が入力されていない場合、エラーメッセージを表示する", async () => {
  render(<CreatePostForm />);

  const body = screen.getByRole("textbox", { name: "本文" });

  // 本文入力欄をクリックし、フォーカスをあわせる
  await user.click(body);

  // フォーカスを外しバリデーションエラーをチェックする
  await user.tab();

  await waitFor(() => {
    expect(body).toBeInvalid();
  });
  expect(body).toHaveErrorMessage("本文は必ず入力してください。");
});
test("本文に141文字入力された場合、エラーメッセージを表示する", async () => {
  render(<CreatePostForm />);

  const body = screen.getByRole("textbox", { name: "本文" });

  await user.click(body);

  await user.type(body, "A".repeat(141));
  await user.tab();

  await waitFor(() => {
    expect(body).toBeInvalid();
  });
  expect(body).toHaveErrorMessage("本文は140文字以内で入力してください。");
});
```

```ts
const schema = z
  .object({
    title: ...,
    body: z
      .string()
      .min(1, "本文は必ず入力してください。")
      .max(140, "本文は140文字以内で入力してください。"),
  })
  .required()
  .strict();
```

```tsx
export const CreatePostForm: React.FC = () => {
  ...
  return (
    <div>
      <form>
       ...
        <textarea
          aria-label="本文"
          rows={4}
          aria-invalid={!!errors.body}
          aria-errormessage={errorMessageId + 1}
          {...register("body")}
        />
        {/* バリデーションエラー時に表示するエラーメッセージ */}
        {errors.body && (
          <p role="alert" id={errorMessageId + 1}>
            {errors.body?.message}
          </p>
        )}
        <button disabled={!isDirty}>送信</button>
      </form>
    </div>
  );
};
```

これでテストは通ります。

![ 2023-07-13 19.44.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/61816c54-59f8-9be7-d1fc-1ab34458959f.png)

バリデーションエラーが発生しない正常型のテストも書いていきます。
境界値をみるのがよさそうなので、タイトルの場合 1 文字の入力と 25 文字の入力を、 本文は 1 文字の入力と 140 文字の入力をテストします。

```tsx
test("タイトルに1文字入力された場合、バリデーションエラーが発生しない", async () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });
  await user.type(title, "A");
  await user.tab();

  expect(title).not.toBeInvalid();
});
test("タイトルに25文字入力された場合、バリデーションエラーが発生しない", async () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });
  await user.type(title, "A".repeat(25));
  await user.tab();

  expect(title).not.toBeInvalid();
});
test("本文に1字入力された場合、バリデーションエラーが発生しない", async () => {
  render(<CreatePostForm />);

  const body = screen.getByRole("textbox", { name: "本文" });
  await user.type(body, "A");
  await user.tab();

  expect(body).not.toBeInvalid();
});
test("本文に140字入力された場合、バリデーションエラーが発生しない", async () => {
  render(<CreatePostForm />);

  const body = screen.getByRole("textbox", { name: "本文" });
  await user.type(body, "A".repeat(140));
  await user.tab();

  expect(body).not.toBeInvalid();
});
```

プロダクトコードを変えなくてもテストは通ります。

![validTestOK.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/0a78996d-169b-0aba-988f-9bb15fa01b66.png)

- [x] タイトルの入力欄にバリデーションエラーメッセージを表示する
- [x] 本文の入力欄にバリデーションエラーメッセージを表示する
- [ ] submit 関数が実行することを確認する
- [ ] バリデーションエラーがある場合、submit 関数は実行しない
- [ ] 送信中は非活性である
- [ ] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### テストコードのリファクタリング

次のテストへと行く前にテストコードを整理します。
`render(<CreatePostForm />);` や、 `screen.getByRole("textbox", {name: "タイトル" })` , `screen.getByRole("textbox", { name: "本文" })` がたびたび出現するので、これらのコードを一まとまりにします。

```tsx
const setup = () => {
  render(<CreatePostForm />);

  const title = screen.getByRole("textbox", { name: "タイトル" });
  const body = screen.getByRole("textbox", { name: "本文" });
  const submitButton = screen.getByRole("button", { name: "送信" });

  const typeTitle = async (value: string) => {
    await user.type(title, value);
  };

  const typeBody = async (value: string) => {
    await user.type(body, value);
  };

  const clickSubmitButton = async () => {
    await user.click(submitButton);
  };

  return {
    element: {
      title,
      body,
      submitButton,
    },
    action: {
      typeTitle,
      typeBody,
      clickSubmitButton,
    },
  };
};
```

ただし、過剰にコードをまとめるとテストコードの可読性が低下する可能性すらあるので、そこに十分気をつけるべきです。

### submit 関数が実行するかのテスト

続いては `submit 関数が実行することを確認する` と `バリデーションエラーがある場合、submit 関数は実行しない` をテストしていきます。  
jest の mock 関数を用意し、 mock 関数が呼び出されたことをテストします。

ここで設計を一部変更します。submit 関数を props 引数にもたせ、submit 関数の定義はコンポーネントの呼び出し元に任せます。
こうすることで、submit 関数の処理に関するテストをコンポーネント内のテストから省くことができ、テストを容易にします。

```tsx
// CreatePostForm.spec.tsx
const mockFn = jest.fn();

afterEach(() => {
  mockFn.mockClear();
});

const setup = () => {
  render(<CreatePostForm handleSubmitPost={mockFn} />);
  ...
}

// CreatePostForm.tsx
type Props = {
  handleSubmitPost: SubmitHandler<{
    body: string;
    title: string;
  }>;
};

export const CreatePostForm: React.FC<Props> = ({ handleSubmitPost }) => {
  ...
  return (
    <form onSubmit={handleSubmit(handleSubmitPost)}>
    ...
    </form>
  )
};
```

`バリデーションエラーがある場合、submit 関数は実行しない` からテストしていきます。
では改めて、どのような条件のときにバリデーションエラーが発生するかですが、以下の条件の場合ですね。
- タイトル
  - 未入力
  - 25 文字より多い
- 本文
  - 未入力
  - 140 文字より多い

また、タイトル・本文の両方が見入力の場合、送信ボタンは非活性であるのでこの場合のテストは行いません。

なので、これらのテストケースを作ります。

```
test("タイトルが入力されておらず、本文が正常に入力された場合、submit関数は実行しない");
test("タイトルに26文字入力され、本文が正常に入力された場合、submit関数は実行しない");
test("本文が入力されておらず、タイトルが正常に入力された場合、submit関数は実行しない");
test("本文に141文字入力され、タイトルが正常に入力された場合、submit関数は実行しない");
```

ここでいう `正常に入力された場合` というのはこれらのテストと逆の条件になりますが、まだ、`正常に入力された場合` のテストをしていませんでした。
`submit 関数が実行することを確認する` をテストしがてら、`正常に入力された場合` のテストを先に書いていきます。

```tsx
test("タイトルに`テスト用タイトル`、本文に`テスト用本文`と入力された時、バリデーションエラーが発生しない", async () => {
  const { element, action } = setup();

  await action.typeTitle("テスト用タイトル");
  await action.typeBody("テスト用本文");

  await user.tab();

  expect(element.title).not.toBeInvalid();
  expect(element.body).not.toBeInvalid();
});
test("タイトルに`テスト用タイトル`、本文に`テスト用本文`と入力され、送信ボタンがクリックされた時、submit関数を実行する", async () => {
  const { action } = setup();

  await action.typeTitle("テスト用タイトル");
  await action.typeBody("テスト用本文");

  await action.clickSubmitButton();

  expect(mockFn).toHaveBeenCalled();
});
```

実行すると無事テストが通るので、タイトルに `テスト用タイトル` 、本文に `テスト用本文` と入力すれば、バリデーションエラーは発生しないということになります。

`バリデーションエラーがある場合、submit 関数は実行しない` に戻り、テストを書いていきます。

```tsx
test("タイトルが入力されておらず、本文が正常に入力された場合、submit関数は実行しない", async () => {
  const { element, action } = setup();

  await action.typeBody("テスト用本文");
  expect(element.submitButton).toBeEnabled();

  await action.clickSubmitButton();
  expect(mockFn).not.toHaveBeenCalled();
});
test("タイトルに26文字入力され、本文が正常に入力された場合、submit関数は実行しない", async () => {
  const { element, action } = setup();

  await action.typeTitle("A".repeat(26));
  await action.typeBody("テスト用本文");
  expect(element.submitButton).toBeEnabled();

  await action.clickSubmitButton();
  expect(mockFn).not.toHaveBeenCalled();
});
test("本文が入力されておらず、タイトルが正常に入力された場合、submit関数は実行しない", async () => {
  const { element, action } = setup();

  await action.typeTitle("テスト用タイトル");
  expect(element.submitButton).toBeEnabled();

  await action.clickSubmitButton();
  expect(mockFn).not.toHaveBeenCalled();
});
test("本文に141文字入力され、タイトルが正常に入力された場合、submit関数は実行しない", async () => {
  const { element, action } = setup();

  await action.typeTitle("テスト用タイトル");
  await action.typeBody("A".repeat(141));
  expect(element.submitButton).toBeEnabled();

  await action.clickSubmitButton();
  expect(mockFn).not.toHaveBeenCalled();
});
```

これらのテストもテストコードを書いただけで通ります。

![ 2023-07-13 20.19.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/e9e2d14f-4f30-a161-f92d-351f16f51a28.png)

- [x] submit 関数が実行することを確認する
- [x] バリデーションエラーがある場合、submit 関数は実行しない
- [ ] 送信中は非活性である
- [ ] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### 入力フォームの残りのテスト

入力フォームのテストは残り `送信中は送信ボタンは非活性` 、`送信後は入力フォームは初期化する` の 2 つとなります。

```tsx
test("送信中は送信ボタンは非活性である", async () => {
  const { element, action } = setup();

  await action.typeTitle("テスト用タイトル");
  await action.typeBody("テスト用本文");

  await action.clickSubmitButton();
  expect(element.submitButton).toBeDisabled();
});
test("送信後は入力フォームは初期化する", async () => {
  const { element, action } = setup();

  await action.typeTitle("テスト用タイトル");
  await action.typeBody("テスト用本文");

  await action.clickSubmitButton();

  expect(element.title).toHaveValue("");
  expect(element.body).toHaveValue("");
});
```

これらのテストが通るようにプロダクトコードを実装していきます。

```tsx
export const CreatePostForm: React.FC<Props> = ({ handleSubmitPost }) => {
  const errorMessageId = useId();

  const {
    register,
    handleSubmit,
+    reset,
    formState: {
      isDirty,
      errors,
+      isSubmitting,
+      isSubmitSuccessful,
    },
  } = useForm<Input>({
    resolver: zodResolver(schema),
    defaultValues: {
      title: "",
      body: "",
    },
    mode: "onBlur",
  });

+  useEffect(() => {
+    if (isSubmitSuccessful) {
+      reset();
+    }
+  }, [isSubmitSuccessful, reset]);

  return (
    <div>
      <form onSubmit={handleSubmit(handleSubmitPost)}>
        <input
          type="text"
          aria-label="タイトル"
          aria-invalid={!!errors.title}
          aria-errormessage={errorMessageId}
          {...register("title")}
        />
        {errors.title && (
          <p role="alert" id={errorMessageId}>
            {errors.title?.message}
          </p>
        )}
        <textarea
          aria-label="本文"
          rows={4}
          aria-invalid={!!errors.body}
          aria-errormessage={errorMessageId + 1}
          {...register("body")}
        />
        {errors.body && (
          <p role="alert" id={errorMessageId + 1}>
            {errors.body?.message}
          </p>
        )}
-        <button disabled={!isDirty}>送信</button>
+        <button disabled={isSubmitting || !isDirty}>送信</button>
      </form>
    </div>
  );
};
```

これでテストは通るようになりました。

![reset.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/275a7b17-8a6e-c2ff-05e6-f2163c0e31d3.png)

- [x] 送信中は非活性である
- [x] 送信後は入力フォームを初期化する
- [ ] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### snackbar を用意する

記事の作成の成否を表示する snackbar を用意します。

`CreatePostForm` コンポーネントと `snackbar` コンポーネントをラップする `CreatePostPage` を作成します。

```tsx CreatePostPage
import { CreatePostForm } from "./CreatePostForm";

export const CreatePostPage: React.FC = () => {

  return (
    <div>
      <CreatePostForm handleSubmitPost={() => void 0} />
    </div>
  );
};
```

今回は `CreatePostPage` コンポーネントで snackbar の state 管理をします。
```tsx 
import { useEffect } from "react";

type Props = {
  message: string;
  isOpen: boolean;
  setOpen: React.Dispatch<React.SetStateAction<boolean>>;
};

export const Snackbar: React.FC<Props> = ({ message, isOpen, setOpen }) => {
  useEffect(() => {
    const timeout = setTimeout(() => {
      setOpen(false);
    }, 3000);   // 3000ms後に非表示にする
    return () => clearTimeout(timeout);
  }, [isOpen, setOpen]);

  if (!isOpen) {
    return null;
  }

  return <div role="alert">{message}</div>;
};
```

```tsx
import { useState } from "react";
import { CreatePostForm } from "./CreatePostForm";
import { Snackbar } from "./Snackbar";

export const CreatePostPage: React.FC = () => {
  const [isOpen, setOpen] = useState(false);
  const [message, setMessage] = useState("");

  return (
    <div>
      <CreatePostForm handleSubmitPost={() => void 0} />
      <Snackbar message={message} isOpen={isOpen} setOpen={setOpen} />
    </div>
  );
};
```

- [x] snackbar を用意する
- [ ] 記事作成が失敗した旨を snackbar に表示する
- [ ] 記事作成が成功した旨を snackbar に表示する

### カスタムフックス

続いて WebAPI とのつなぎ込みの処理を書いていきます。

`useCreatePost.spec.tsx`, `useCreatePost.ts` の 2 つのファイルを作成します。
この時、ブラウザの開発者ツールなどから、送信ボタンの非活性を無効化されたとしても、不正な値がサーバーへ送信されるのを防ぐためのテストと実装を書いていきます。

エラーが発生する場合のテストを書きます。
```tsx
test("タイトルが0文字の場合、エラーが発生する");
test("タイトルが26文字の場合、エラーが発生する");
test("本文が0文字の場合、エラーが発生する");
test("本文が141文字の場合、エラーが発生する");
```

カスタムフックでバリデーションを行うとして、 testing-library を使ってカスタムフックスをテストしていきます。
```tsx
const { result } = renderHook(() => useCreatePost());

test("タイトルが0文字の場合、エラーが発生する", async () => {
  await expect(
    async () =>
      await result.current.createPost({ title: "", body: "テスト用本文" })
  ).rejects.toThrow("タイトルに不正な値が入力されています。");
});
```
useCreatePost に createPost 関数を定義し、 HTTP POST メソッドを使い、 WebAPI に記事の情報を送信します。

```tsx
function useCreatePost() {
  const createPost = async (data: Input) => {
    if (data.title.length < 1 || data.title.length > 25) {
      throw new Error("タイトルに不正な値が入力されています。");
    }
    if (data.body.length < 1 || data.body.length > 140) {
      throw new Error("本文に不正な値が入力されています。");
    }

    try {
      await axios.post("api/posts", JSON.stringify(data), {
        headers: {
          "Content-Type": "application/json",
        },
      });
      return;
    } catch {
      throw new Error("エラーが発生しました。");
    }
  };

  return { createPost };
}
```

テストは OK ですね。

![ 2023-07-14 16.36.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/3b0d2a67-b50e-e7a1-5f53-28f635e8df08.png)

### snackbarに記事の作成の成否を表示する

いままで作成したコンポーネント、関数を組み合わせて、記事作成の成否のテストを行います。

`CreatePostPage.spec.tsx` ファイルを作成し、テストを書いていきます。

ひとまず、検証結果だけをテストコードに記載します。
```tsx
const user = userEvent.setup();

test("記事の作成中にエラーが発生する場合、記事の作成に失敗した旨のメッセージを表示する", async () => {
  // arrange
  // act

  // assert
  expect(screen.getByRole("alert")).toHaveTextContent("エラーが発生しました。");
});
```

このテストを実現するには、`CreatePostPage` コンポーネントを使う必要があります。
`CreatePostPage` 内で `CreatePostForm` コンポーネントを呼び出しているので、入力フォームを使えます。

setup 関数で、`CratePostPage` の呼び出しと、バリデーションエラーが発生しないフォームを入力します。

```tsx
const setup = async () => {
  render(<CreatePostPage />);

  const title = screen.getByRole("textbox", { name: "タイトル" });
  const body = screen.getByRole("textbox", { name: "本文" });

  await user.type(title, "テスト用タイトル");
  await user.type(body, "テスト用本文");

  const submitButton = screen.getByRole("button", { name: "送信" });

  const clickSubmitButton = async () => {
    await user.click(submitButton);
  };

  return { clickSubmitButton };
};
```

テストケースで setup を実行し、テストの準備と実行を省略します。

```tsx
test("記事の作成中にエラーが発生する場合、記事の作成に失敗した旨のメッセージを表示する", async () => {
  const { clickSubmitButton } = await setup();

  await clickSubmitButton();

  expect(screen.getByRole("alert")).toHaveTextContent("エラーが発生しました。");
});
```

このままでは正常に関数が終了し、記事作成が成功するので、 msw を使ってエラーの発生をモックします。

#### msw を導入する

https://mswjs.io/docs/getting-started/install

公式ドキュメントに従い、msw を用意します。

```ts mocks/server.ts
import { rest } from "msw";
export const handlers = [
  rest.post("api/posts", (_req, res, ctx) => {
    return res(ctx.status(201));
  }),
];

export const server = setupServer(...handlers);
```

テストコードでエラーを発生させます。  
msw を使って、意図的にエラーを起こし、エラー時のテストを書きます。

```tsx
import { server } from "@/mocks/server";
import { rest } from "msw";

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test("記事の作成中にエラーが発生する場合、記事の作成に失敗した旨のメッセージを表示する", async () => {
  server.use(
    rest.post("api/posts", (_req, res, ctx) => {
      // 400エラー(Bad Request)を発生させる
      return res(ctx.status(400));
    })
  );
  const { clickSubmitButton } = await setup();

  await clickSubmitButton();

  expect(screen.getByRole("alert")).toHaveTextContent("エラーが発生しました。");
});
```

まだ、実装がないのでテストは通りません。

### CreatePostPage コンポーネントの実装

`CreatePostForm` でクリック時に実行する関数を定義します。
こちらでは、 `useCreatePost` 関数で定義した、 `createPost` を使います。
カスタムフックスにロジックを譲ることで、処理が別れ、テストもしやすくなります。

```tsx
export const CreatePostPage: React.FC = () => {
  const { createPost } = useCreatePost();

  const handleSubmitPost = useCallback(
    async (data: Input) => {
      await createPost(data);
    },
    [createPost]
  );

  return (
    <div>
      <CreatePostForm handleSubmitPost={handleSubmitPost} />
      ...
    </div>
  );
}
```

続いて、成否のメッセージを作成した snackbar に表示する処理を書きます。
createPost 関数が成功したら、関数が異常終了したら、エラーを snackbar に表示します。

```tsx
export const CreatePostPage: React.FC = () => {
  const [isOpen, setOpen] = useState(false);
  const [message, setMessage] = useState("");

  const { createPost } = useCreatePost();
  const handleSubmitPost = useCallback(
    async (data: Input) => {
      try {
        await createPost(data);
      } catch (err) {
        setMessage(err.message);
        setOpen(true);
      }
    },
    [createPost]
  );

  return (
    <div>
      <CreatePostForm handleSubmitPost={handleSubmitPost} />
      <Snackbar message={message} isOpen={isOpen} setOpen={setOpen} />
    </div>
  );
}
```
`記事の作成中にエラーが発生する場合、記事の作成に失敗した旨のメッセージを表示する` のテストは通りました。

![ 2023-07-14 19.34.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/5eaab24c-45d0-cad8-447d-9023d8e9612c.png)

続いて、同じようにして、`記事の作成が成功した場合、記事の作成に成功した旨のメッセージを表示する` のテストを書いていきます。

```tsx test

test("記事の作成が成功した場合、記事の作成に成功した旨のメッセージを表示する", async () => {
  const { clickSubmitButton } = await setup();

  await clickSubmitButton();

  expect(screen.getByRole("alert")).toHaveTextContent(
    "記事の作成に成功しました。"
  );
});
```

```tsx
export const CreatePostPage: React.FC = () => {
  const [isOpen, setOpen] = useState(false);
  const [message, setMessage] = useState("");

  const { createPost } = useCreatePost();
  const handleSubmitPost = useCallback(
    async (data: Input) => {
      try {
        await createPost(data);
+        setMessage("記事の作成に成功しました。");
+        setOpen(true);
      } catch (err) {
        ...
      }
    },
    [createPost]
  );

  return (
    ...
  );
}
```

テストは通ります。

![ 2023-07-14 19.38.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/73534a09-80ec-856a-cc90-67aaf24cdb46.png)

これで一連のテストは全部終了です。
最後に全部のテストが通ることを確認します。

![testAllGreen.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/dfd7af16-dfd6-2c5b-80bf-832409cca407.png)

- [x] 記事作成が失敗した旨を snackbar に表示する
- [x] 記事作成が成功した旨を snackbar に表示する

## 最後に

コードが量・文章ともに多くなり、読みづらい文章になってしまいましたが、これで一通りの TDD での入力フォーム開発は完了です。

こちらが今回の記事で作成したコードになります。

https://github.com/diskszk/testing-sandbox/tree/main/src/tdd-with-rhf-zod
