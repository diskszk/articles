---
title: React-Hook-Formとzodを使ってバリデーションのテストを行う
tags:
  - testing
  - React
  - storybook
  - react-hook-form
  - zod
private: false
updated_at: '2023-07-13T00:03:16+09:00'
id: a50b72e5dd89f933e1bd
organization_url_name: null
---

React-Hook-Form で作成した入力フォームにバリデーションエラーが表示されることのテストを行います。

## 開発環境
node: 18.14.0
react: 18.2.0
typescript: 5.1.6
vite: 4.4.2
jest: 29.6.1
testing-library/react: 14.0.0
react-hook-form: 7.45.1
zod: 3.21.4
storybook: 7.0.26

vite-cli から react-ts のテンプレートを使って react 環境を作りました。

## コード

https://github.com/diskszk/testing-sandbox/tree/main/src/rhf-zod-validations

## 仕様
フィールドの最大・最小文字数を定義します。
- タイトル
  - 必須 ( 最小 1 文字 )
  - 最大 25 文字

## 入力フォームの実装

```tsx
import { useForm } from "react-hook-form";

export const CreatePostForm: React.FC = () => {
  const {
    register,
    formState: { errors },
  } = useForm({
    defaultValues: {
      title: "",
    },
  });

  return (
    <div>
      <form>
        <input
          type="text"
          aria-label="タイトル"
          aria-invalid={!!errors.title}
          {...register("title")}
        />
        // バリデーションエラー時に表示するエラーメッセージ
        {errors.title && (
          <p role="alert">
            {errors.title?.message}
          </p>
        )}
      </form>
    </div>
  );
};
```

 この状態だとバリデーションルール、エラーメッセージが定義されていないため、上記の仕様をもとに、 zod でバリデーションを作っていきます。

```ts
import z from "zod";

const schema = z
  .object({
    title: z
      .string()
      .min(1, "タイトルは必ず入力してください。")
      .max(25, "タイトルは25文字以内で入力してください。")
  })
  .required()
  .strict();

type Input = z.infer<typeof schema>;
```

先ほど作成した入力フォームに追加します。
useForm のオプションに `{mode: "onBlur"} ` を追加し、フォーカスが外れたときにバリデーションをかけるようにします。

```tsx CreatePostForm.tsx
export const CreatePostForm: React.FC = () => {
  const errorMessageId = useId();

  const {
    register,
    formState: { errors },
  } = useForm<Input>({
    resolver: zodResolver(schema),
    defaultValues: {
      title: "",
    },
    mode: "onBlur", 
  });

  return (
    <div>
      <form>
        <label>
          タイトル:
          <input
            type="text"
            aria-label="タイトル"
            aria-errormessage={errorMessageId}
            aria-invalid={!!errors.title}
            {...register("title")}
          />
          {errors.title && (
            <p role="alert" id={errorMessageId}>
              {errors.title?.message}
            </p>
          )}
        </label>
      </form>
    </div>
  );
};
```

また、上記 input タグの aria-errormessage と p タグの `id` 要素に生成した `id` を含ませることで、input 要素とエラー文言要素を紐づかせ、テストコードで `toHaveErrorMessage(errorMessage)` を参照できるようにします。こちらの記事を参考にさせていただいております。

https://zenn.dev/takepepe/articles/useid-for-a11y

## Story の作成

Storybook を使って stories を作成します。
`$ npx storybook@latest init` とコマンド入力すると `Would you like to install it?` と聞かれるので `y` と入力して、必要なライブラリをインストールします。
すべてインストールが済んだら Storybook の準備は完了です。

```tsx CreatePostForm.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { CreatePostForm } from "./CreatePostForm";

const meta: Meta<typeof CreatePostForm> = {
  component: CreatePostForm,
};

export default meta;

type Story = StoryObj<typeof CreatePostForm>;

export const EmptyTitle: Story = {
  play: async ({
    canvasElement,
  }: {
    canvasElement: HTMLElement;
  }): Promise<void> => {
    const canvas = within(canvasElement);

    // タイトル入力欄にフォーカスを当てる
    await canvas.getByRole("textbox", { name: "タイトル" }).focus();

    // 何も入力せずにフォーカスを外す
    await userEvent.tab();
  },
};
```

## テスト作成

Storybook をテストに使うので、以下のライブラリを追加します。  
`$ yarn add -D @storybook/jest @storybook/testing-library`  

テストファイルを作成します。

```tsx CreatePostForm.spec.tsx
import { render, screen, waitFor } from "@testing-library/react";
import { composeStories } from "@storybook/react";
import * as stories from "./CreatePostForm.stories";

test("タイトルが入力されていない場合、エラーメッセージを表示する", async () => {
  const { EmptyTitle } = composeStories(stories);

  const { container } = render(<EmptyTitle />);

  await EmptyTitle.play({ canvasElement: container });

  const title = screen.getByRole("textbox", { name: "タイトル" });

  await waitFor(() => {
    expect(title).toBeInvalid();
  });
  expect(title).toHaveErrorMessage("タイトルは必ず入力してください。");
});
```

テストの実行結果は Pass となります。

![testOK.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/19f6571b-9d2a-d885-cd58-11a87a79e679.png)

同じ要領で 26 文字入力された場合のテストを書いていきます。
```tsx CreatePostForm.stories.tsx
export const OverTitleLength: Story = {
  play: async ({
    canvasElement,
  }: {
    canvasElement: HTMLElement;
  }): Promise<void> => {
    const canvas = within(canvasElement);

    await userEvent.type(
      canvas.getByRole("textbox", { name: "タイトル" }),
      "A".repeat(26)
    );

    await userEvent.tab();
  },
};
```

```tsx CreatePostForm.spec.tsx
test("タイトルに26文字入力された場合、エラーメッセージを表示する", async () => {
  const { OverTitleLength } = composeStories(stories);
  const { container } = render(<OverTitleLength />);

  await OverTitleLength.play({ canvasElement: container });

  const title = screen.getByRole("textbox", { name: "タイトル" });

  await waitFor(() => {
    expect(title).toBeInvalid();
  });
  expect(title).toHaveErrorMessage("タイトルは25文字以内で入力してください。");
});
```

テストの実行結果は Pass となります。

![testOkBoth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/1ecd1cb9-8c9f-2e40-6d8b-921d7b40cee3.png)
