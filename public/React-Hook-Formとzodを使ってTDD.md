---
title: React-Hook-Formとzodを使ってTDD
tags:
  - 'react'
private: true
updated_at: ''
id: null
organization_url_name: null
code_path: /Users/suzukidaisuke/work/products/testing-sandbox/src/tdd-with-rhf_zod
---

## TDD とは
TODO: なにか参照して言及する

## 

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

TODO: React Hook Form　の説明、 zod の説明

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

また、送信ボタンがクリックされた時の処理についても定義します。
- タイトルに入力された文字数が 0 文字、または、25 文字より大きい場合、 submit 処理は実行しない
- 本文に入力された文字数が 0 文字、または、140 文字より大きい場合、 submit 処理は実行しない
- 送信後は入力フォームは初期化する

また、ブラウザの開発者コンソールから送信ボタンの非活性を取り消されてクリックされるなどして、不正な値がサーバーに渡るのを防ぐため以下の条件も追加します。
( これらは送信ボタンがクリックされて実行する関数内に実装します。 )
- タイトルに入力された文字数が 0 文字、または、25 文字より大きい場合、トーストにエラーを表示する
- 本文に入力された文字数が 0 文字、または、140 文字より大きい場合、トーストにエラーを表示する

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
- [ ] 記事を WebAPI に送信する関数を用意する
- [ ] 関数内でエラーを起こす
- [ ] トーストを用意する
- [ ] 記事作成が失敗した旨をトーストに表示する
- [ ] 記事作成が成功した旨をトーストに表示する

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

### 関数のテスト

続いて `記事を WebAPI に送信する関数を用意する` こちらの TODO をクリアしていきます。



実際に WebAPI サーバーを作らずにテストだけを行いたいので、 msw を使って WebAPI サーバをモックします。

#### msw を導入する

https://mswjs.io/docs/getting-started/install

公式ドキュメントに習って