---
title: inputタグのreadonly属性のテスト
tags:
  - HTML
  - React
  - react-testing-library
private: false
updated_at: '2023-11-21T18:12:26+09:00'
id: 3a4044107ac86d53c60d
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに
<!-- textlint-disable -->
タイトルの通り input タグで入力フォームを作る際の readonly 属性のテストにつまずくことがあったので、解決方法を共有できればなと思います。
<!-- textlint-enable -->

## 環境
```json
{
  "react": "^18.2.0",
  "@mui/material": "5.14.13",
  "@testing-library/react": "14.0.0",
  "typescript": "^5.2.2",
  "vite": "^4.4.9",
  "vitest": "0.29.8"
}
```
テストフレームワークに `vitest` と `react-testing-library` を使用しています。

https://codesandbox.io/p/sandbox/testing-input-form-readonly-282z8n?file=%2Fsrc%2FInputForm.test.tsx%3A1%2C1

## テストコード

一番最初に `readonly` 属性のテスト時の検証方法から述べます。

### readonly属性がある(trueの)場合

```App.test.tsx
render(
  <input label="入力フォーム" readonly />
);

// HTML要素を取得する
const input = screen.getByRole("textbox");

// HTML要素に`readonly`属性が付与されていることを期待する
expect(input).toHaveAttribute("readonly");
```

上記のテストコードで `<input readonly />` のように `readonly` 属性がある場合、テストが通ります。

### readonly属性がない(falseの)場合

`screen.debug()` を使って `readonly` 属性が `true` の場合と `false` の場合を見てみます。

```tsx
render(
  <input label="入力フォーム" readonly />
);

screen.debug();
```
![ 2023-10-15 20.54.27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/0975196a-5b98-5b70-1450-53a5b55986ca.png)


```tsx
render(
  <input label="入力フォーム" readonly={false} />
);

screen.debug();
```
![ 2023-10-15 20.55.07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/c81e1ade-95c2-79c6-34a2-0f38cb0f3516.png)

`readonly` 属性がある場合、 `readonly=""` となり、 `readonly` が `false` の場合、そもそも `readonly` 属性が存在しないこととみなされます。
なので `expect(input.getAttribute("readonly")).toBeFalsy()` というように真偽値で取得できません。 `.not.toHaveAttribute("readonly")` を使うことで `readonly=false` のテストできます。


## MUIの<TextField />を使う場合

MUI の [TextField](https://mui.com/material-ui/react-text-field/) を使う場合、`<TextField />` に `readonly` 属性を渡しても入力不可能な状態にはなりません。

```tsx
render(<TextField label="input" readOnly={true} />);
screen.debug();
```

![ 2023-10-15 21.24.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/6db6c0be-e423-b1ec-6cf1-10ae811582f4.png)

input タグのや要素の div タグに `readonly` 属性が付与され、 input タグには存在していません。

解決するには `<TextField />` の `inputProps` に `readOnly` を加えて渡せば OK です。

```tsx
<TextField label="入力フォーム" inputProps={{ readOnly: true }} />
```

この場合も input タグと同様のテストで検証できます。

```tsx
test("readOnly:true", () => {
    render(<TextField label="input" inputProps={{ readOnly: true }} />);

    const input = screen.getByRole("textbox");

    expect(input).toHaveAttribute("readonly");
  });
```
