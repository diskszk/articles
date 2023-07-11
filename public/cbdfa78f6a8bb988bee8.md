---
title: useContext × useReducerを使ってタスク管理アプリをテストファーストで作る
tags:
  - Jest
  - React
  - react-testing-library
  - useContext
  - useReducer
private: false
updated_at: '2022-08-07T18:00:48+09:00'
id: cbdfa78f6a8bb988bee8
organization_url_name: null
---
テストを書きながら状態管理にuseContextとuseReducerを使ってタスク管理アプリを作っていきます。

## 作るアプリの概要

よくあるタスク管理アプリです。  
ReactのuseContextとuseReducerで状態管理をしています。  
以下のイメージのようにタスク入力フォームと、タスクの数だけタスクの内容、doneボタン、削除ボタンを表示します。
doneボタンをクリックするとタスクの内容に取り消し線が引かれ、もう一度doneボタンをクリックすると取り消し線が消えます。  
削除ボタンをクリックするとタスクが削除されます。  

![ 2022-08-07 17.09.42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/639130/37f1c041-0cbd-a185-79ba-4df87de04f91.png)

テスト駆動開発の手法に乗っ取り、Red（失敗）、Green（成功）、Refactor（リファクタリング） のサイクルで開発していきます。  
DBなどの外部のリソースに保存しないので、サーバーを再起動したりブラウザをリロードすると初期化されます。  

## 使用ライブラリ・バージョン

- typescript: 4.6.X
- vite: 2.9.X
- react: 18.0.X
- react-dom: 18.0.X
- jest: 28.1.X
- ts-jest: 28.0.X
- testing-library/jest-dom: 5.16.X
- testing-library/react: 13.3.X
- testing-library/user-event: 14.2.X

## repository

https://github.com/diskszk/testing-sandbox/tree/main/src/todoList


### 要件・仕様
- ユーザーから文字列の入力を受け取る入力フォームを持つ
  - 「タスクを入力してください」の文言のラベルテキストを持つ入力フォームを表示する
- 作成ボタンを持つ
  - 入力フォームの内容が空の状態の場合、作成ボタンは非活性である
  - 入力フォームに何かしら入力してある状態の場合、作成ボタンは活性である
  - 作成ボタンをクリックすると入力フォームの内容が空になる
- タスクが存在しない場合、「タスクが存在しません。」という文言を表示する
- タスクが存在する場合、タスクの数だけ以下を表示する
  - タスク内容
  - doneボタン
  - 削除ボタン
- done状態のタスクは取り消し線を表示する
- タスク入力フォームにテキストを入力し、作成ボタンをクリックするとタスクリストに入力したテキストのタスクを表示する
  - 新たに作られたタスクに取り消し線が引かれていない
- doneボタンをクリックすると取り消し線を表示する
- 削除をクリックするとタスを削除する
- タスクが削除され、0件になった場合、「タスクが存在しません。」という文言を表示する

これらをテストしていきます!

## 環境構築
viteのcliを使ってReact TypeScriptのテンプレートを作ります
`$ yarn create vite sample-app --template react-ts`
`$ cd sample-app`
`$ yarn install`

これだけでReact環境が出来上がりました。試しに開発サーバーを立てて確認してみてください。

create-react-app と異なり、デフォルトでjestや@testing-library/react等はインストールされていないので手動でインストールしていきます。

`$ yarn add -D jest @types/jest ts-jest @testing-library/jest-dom @testing-library/react @testing-library/user-event jest-environment-jsdom`

これらのライブラリをインストールした後、jest.config.jsを作ります。

`yarn ts-jest config:init`

自動生成されたjest.config.jsの内容を変更します。
```jest.config.js
/** @type {import('ts-jest/dist/types').InitialOptionsTsJest} */
module.exports = {
  preset: "ts-jest",
-  testEnvironment: "node",
+  testEnvironment: "jsdom",
};
```
testEnvironmentにjsdomを使うことでReactなど本来ブラウザで実行されるコードをテストできるようにします。

## reducerのロジックのテスト

今回reducerが持つロジックは以下の通りです。
- タスクを追加する
- 特定のタスクの持つ"完了"の状態を変更する
- 特定のタスクを削除する

### 要件と仕様をテストコードに起こす

```ts
describe("reducer", () => {
  test("ADD_TASK: タスクを追加する", () => {
    const result = reducer([], {  // reducerの第一引数が初期値になる
      type: "ADD_TASK",
      payload: { id: 1, text: "sample", done: false },
    });

    expect(result).toEqual([{ id: 1, text: "sample", done: false }]);
  });

  test("CHANGE_DONE: タスクの持つdoneの状態を変更する", () => {
    const result = reducer([{ id: 1, text: "sample", done: false }], {
      type: "CHANGE_DONE",
      payload: 1,
    });

    expect(result).toEqual([{ id: 1, text: "sample", done: true }]);
  });

  test("DELETE_TASK: タスクを削除する", () => {
    const result = reducer([{ id: 1, text: "sample", done: false }], {
      type: "DELETE_TASK",
      payload: 1,
    });

    expect(result).toEqual([]);
  });
});
```

この状態でtestを実行するともちろんエラーになります。これがTDDの第一歩です!  
次に、テストを成功させる為だけのコードを書いていきます。  

### 実装(Green)

```ts
// 型定義
type Task = {
  id: number;
  text: string;
  done: boolean;
};
```

```ts
type AddTaskAction = {
  type: typeof "ADD_TASK";
  payload: Task;
};
type ChangeDoneAction = {
  type: typeof "CHANGE_DONE";
  payload: number;
};
type DeleteTaskAction = {
  type: typeof "DELETE_TASK";
  payload: number;
};
type Action = AddTaskAction | ChangeDoneAction | DeleteTaskAction;

export const reducer: Reducer<Task[], Action> = (state, action): Task[] => {
  switch (action.type) {
    case "ADD_TASK": {
      return [{ id: 1, text: "sample", done: false }];
    }
    case "CHANGE_DONE": {
      return [{ id: 1, text: "sample", done: true }];
    }
    case "DELETE_TASK": {
      return [];
    }
  }
};
```

テストコードの方にimport文を追加すればtestが通るようになります。これでGreen(成功)になりました。  
しかし、この状態だと実際にアプリケーションとして使うことはできませんので、リファクタリングして実用可能なコードにしていきます。

### 実装(リファクタリング)

```ts
export const reducer: Reducer<Task[], Action> = (state, action): Task[] => {
  switch (action.type) {
    case "ADD_TASK": {
      return [...state, { ...action.payload }];
    }
    case "CHANGE_DONE": {
      return state.map((item) =>
        item.id === action.payload ? { ...item, done: !item.done } : item
      );    }
    case "DELETE_TASK": {
      return state.filter(({ id }) => id !== action.payload);
    }
  }
};
```

## useContextのテスト
useContextを使ってコンポーネントのどこからでもタスクの配列を参照・追加・値の変更・削除をできるようにします。そのためにReactContextを用いたhooksのテストをしていきます。

### 要件と仕様をテストに起こす
reducerのテストでロジックは検証したので、hooksを使ってタスクの配列を参照できることをテストします。

```ts
describe("useTasks", () => {
  let renderHookResult: RenderHookResult<{state: Task[]}, null>;

  test("初期値は空の配列である", () => {
    const { result } = renderHookResult;
    expect(result.current.state).toEqual([]);
  });
});
```

### 実装(Green)

通るテストを書いていきます。  
まずはuseContextを用いたProvider関数を作っていきます。  
このProvider関数はReactコンポーネントで、childrenをレンダーすると同時にタスクの状態を子コンポーネントで使えるようにします。

```tsx
type TasksDispatch = {
  addTask: (task: Task) => void;
  changeDone: (id: number) => void;
  deleteTask: (id: number) => void;
};

const TasksStateContext = createContext<Task[]>([]);
const TasksDispatchContext = createContext<TasksDispatch>({
  addTask: () => void 0,
  changeDone: () => void 0,
  deleteTask: () => void 0,
});

type Props = {
  children: ReactNode;
};

export const TasksProvider: React.FC<Props> = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, []);

  const addTask = useCallback(
    (task: Task) => dispatch({ type: ADD_TASK, payload: task }),
    []
  );
  const changeDone = useCallback(
    (id: number) => dispatch({ type: CHANGE_DONE, payload: id }),
    []
  );
  const deleteTask = useCallback(
    (id: number) => dispatch({ type: DELETE_TASK, payload: id }),
    []
  );

  return (
    <TasksStateContext.Provider value={state}>
      <TasksDispatchContext.Provider
        value={{ addTask, changeDone, deleteTask }}
      >
        {children}
      </TasksDispatchContext.Provider>
    </TasksStateContext.Provider>
  );
};
```

次にcustom hooksを作成していきます。

```ts
export function useTasks(): { state: Task[] } {
  return {state: []};
}
```

テストコードの方も修正します。
```ts
describe("useTasks", () => {
  let renderHookResult: RenderHookResult<{state: Task[]}, null>;
+  beforeEach(() => {
+    renderHookResult = renderHook(() => useTasks(), {
+      wrapper: (props) => <TasksProvider>{props.children}</TasksProvider>,
+    });
+  });

  test("初期値は空の配列である", () => {
    const { result } = renderHookResult;
    expect(result.current.state).toEqual([]);
  });
});
```

TasksProviderでwrapすることで本来TasksProvider内でしか使えないタスクの値を取得することができるようになります。

### 実装(リファクタリング)
```ts
export function useTasks(): { state: Task[] } & TasksDispatch {
const state = useContext(TasksStateContext);
  const { addTask, changeDone, deleteTask } = useContext(TasksDispatchContext);
  return { state, addTask, changeDone, deleteTask };}
```


## UIのテスト

UIのユニットテストにおいては、Web アプリケーションがどのようにユーザーに見えるか、操作されるかに注意しテストをしていきます。

今回は"タスク作成コンポーネント"と"タスクリストコンポーネント"にコンポーネントを分けて開発していきます。

### 要件と仕様をテストに起こす

"タスク作成コンポーネント"は以下の要件と仕様で構成します。
- ユーザーから文字列の入力を受け取る入力フォームを持つ
  - 「タスクを入力してください」の文言のラベルテキストを持つ入力フォームを表示する
- 作成ボタンを持つ
  - 入力フォームの内容が空の状態の場合、作成ボタンは非活性である
  - 入力フォームに何かしら入力してある状態の場合、作成ボタンは活性である
  - 作成ボタンをクリックすると入力フォームの内容が空になる

こちらの要件でテストコードを書いていきますが、これら全てをテストコードにしてしまうと実装が大変なため、一つずつRed → Green のサイクルで書いていきます。

#### 「タスクを入力してください」の文言のラベルテキストを持つ入力フォームを表示する

```ts
describe("TaskForm", () => {
  beforeEach(() => {
    render(<TaskForm />);
  });

  test("「タスクを入力してください」の文言のラベルテキストを持つ入力フォームを表示する", () => {
    expect(screen.getByLabelText("タスクを入力してください")).toHaveValue("");
  });
});
```

```tsx
export const TaskForm: React.FC = () => {
  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input type="text" id="task" />
    </div>
  );
};
```
#### 作成ボタンを持つ

```ts
test("作成ボタンを表示する", () => {
  expect(screen.getByRole("button")).toHaveTextContent("作成");
});
```

```tsx
export const TaskForm: React.FC = () => {
  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input type="text" id="task" />
      <button>作成</button>
    </div>
  );
};
```

#### 入力フォームの内容が空の状態の場合、作成ボタンは非活性である

```ts
test("入力フォームの内容が空の状態の場合、作成ボタンは非活性である", () => {
  expect(screen.getByLabelText("タスクを入力してください")).toHaveValue("");
  expect(screen.getByRole("button")).not.toBeEnabled();
});
```

```tsx
export const TaskForm: React.FC = () => {
  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input type="text" id="task" />
      <button disabled>作成</button>
    </div>
  );
};
```

#### 入力フォームに何かしら入力してある状態の場合、作成ボタンは活性である

```ts
test("入力フォームに何か入力してある状態の場合、作成ボタンは活性である", async () => {
  await userEvent.type(
    screen.getByLabelText("タスクを入力してください"),
    "sample"
  );
  expect(screen.getByRole("button")).toBeEnabled();
});
```

```tsx
export const TaskForm: React.FC = () => {
  const [text, setText] = useState("");

  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input
        type="text"
        id="task"
        value={text}
        onChange={(ev) => setText(ev.target.value)}
      />
      <button disabled={!text}>作成</button>
    </div>
  );
};
```

#### 作成ボタンをクリックすると入力フォームの内容が空になる

```ts
test("作成ボタンをクリックすると入力フォームの内容が空になる", async () => {
  await userEvent.type(
    screen.getByLabelText("タスクを入力してください"),
    "sample"
  );
  await userEvent.click(screen.getByRole("button"));
  expect(screen.getByLabelText("タスクを入力してください")).toHaveValue("");
});
```

```tsx
export const TaskForm: React.FC = () => {
  const [text, setText] = useState("");

  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input
        type="text"
        id="task"
        value={text}
        onChange={(ev) => setText(ev.target.value)}
      />
      <button onClick={() => setText("")} disabled={!text}>
        作成
      </button>
    </div>
  );
};
```

これでこのコンポーネントは一通テストはOKです。

続いて"タスクリストコンポーネント"にうつります。

### 要件と仕様をテストに起こす
- タスクが存在しない場合、「タスクが存在しません。」という文言を表示する
- タスクの数だけ以下を表示する
  - タスク内容
  - doneボタン
  - 削除ボタン
- done状態のタスクは取り消し線を表示する

こちらも"タスク作成コンポーネント"同様、一つずつテストと実装を書いていきます。

#### タスクが存在しない場合、「タスクが存在しません。」という文言を表示する
```tsx
describe("TaskList", () => {
  test("タスクが存在しない場合、「タスクが存在しません。」という文言を表示する", () => {
    render(<TaskList />);
    expect(screen.getByText("タスクが存在しません。")).toBeInTheDocument();
  });
});
```

```tsx
export const TaskList: React.FC = () => {
  return <p>タスクが存在しません。</p>;
};
```
#### タスクの数だけ以下を表示する
  - タスク内容
  - doneボタン
  - 削除ボタン

タスクが0件の場合のテストを成り立たせたままテストを書くことになるのでひと工夫が必要そうです。  
今回は説明の都合上、コンポーネントから書いていきます。  
props引数かcustom hooksを使ってtask配列を受け取れば良さそうです。せっかくuseTasksを作ったのでそちらを使っていきます。

  ```tsx
export const TaskList: React.FC = () => {
  const { state } = useTasks();

  // タスクが0件の場合「タスクが存在しません。」を表示する
  if (!state.length) {
    return <p>タスクが存在しません。</p>;
  }
  return (
    <ul>
      {state.map((task) => (
        <li key={task.id}>
          <p data-testid={`task:${task.id}`}>{task.text}</p>
          <button>done</button>
          <button>削除</button>
        </li>
      ))}
    </ul>
  );
};
```

テストを書いていきます。useContextを用いて作ったcustom hooksを使っているので以下のようにしてuseContextのContextProviderでwrapして使います。
```tsx
test("タスクの数だけ[タスク内容], doneボタン, 削除ボタン表示する", () => {
  render(
    <TasksStateContext.Provider
      value={[
        {
          id: 1,
          text: "sample1",
          done: false,
        },
        {
          id: 2,
          text: "sample2",
          done: false,
        },
        {
          id: 3,
          text: "sample3",
          done: false,
        },
      ]}
    >
      <TaskList />
    </TasksStateContext.Provider>
  );
  expect(screen.getAllByTestId(/task:/)).toHaveLength(3);
  expect(screen.getAllByRole("button", { name: "done" })).toHaveLength(3);
  expect(screen.getAllByRole("button", { name: "削除" })).toHaveLength(3);
});
```
以前作成したTasksStateContextのvalueに初期値としてテストで使う値を代入してテストすることでコンポーネントの変化が簡単に再現できます。


#### done状態のタスクは取り消し線を表示する
同様にテストを書いていきます。

```tsx
test("done状態のタスクは取り消し線を表示する", () => {
  render(
    <TasksStateContext.Provider
      value={[{ id: 1, text: "sample", done: true }]}
    >
      <TaskList />
    </TasksStateContext.Provider>
  );
  expect(screen.getByText("sample")).toHaveStyle({
    textDecoration: "line-through",
  });
});
```

```tsx
export const TaskList: React.FC = () => {
  const { state } = useTasks();

  if (!state.length) {
    return <p>タスクが存在しません。</p>;
  }
  return (
    <ul>
      {state.map((task) => (
        <li key={task.id}>
          <p
            data-testid={`task:${task.id}`}
+            style={{ textDecoration: task.done ? "line-through" : "none" }}
          >
            {task.text}
          </p>
          <button>done</button>
          <button>削除</button>
        </li>
      ))}
    </ul>
  );
};
```

これでそれぞれのコンポーネントのテストは完了です。  
続いて、コンポーネントを結合した時のテスト・実装を行っていきます。
これらをテストします。

- タスク入力フォームにテキストを入力し、作成ボタンをクリックするとタスクリストに入力したテキストのタスクを表示する
  - 新たに作られたタスクに取り消し線が引かれていない
- doneボタンをクリックすると取り消し線を表示する
- 削除をクリックするとタスを削除する
- タスクが削除され、0件になった場合、「タスクが存在しません。」という文言を表示する

これらのテストを一つのストーリーとして、通してテストしていきます。
* 本来test()にそれぞれのテストを書いていくべきなのですが、前のテストでDOMを変更した結果を保持したまま次のテストを行う方法がわからないのでこうしてあります。ご存知の方いらっしゃいましたらコメント頂けるとありがたいです...!

```tsx
describe("TodoList", () => {
  render(
    <TasksProvider>
      <TaskForm />
      <TaskList />
    </TasksProvider>
  );

  test("タスクを作成して削除できること", async () => {
    // タスク入力フォームにテキストを入力する
    await userEvent.type(
      screen.getByLabelText("タスクを入力してください"),
      "sample"
    );

    // 作成ボタンをクリックする
    await userEvent.click(screen.getByRole("button", { name: "作成" }));

    // タスクリストに入力したテキストのタスクを表示する
    expect(screen.getByRole("listitem")).toHaveTextContent("sample");

    // 取り消し線が引かれていないこと
    expect(screen.getByText("sample")).not.toHaveStyle({
      textDecoration: "line-through",
    });

    // doneボタンをクリックすると取り消し線を表示する
    await userEvent.click(screen.getByRole("button", { name: "done" }));
    expect(screen.getByText("sample")).toHaveStyle({
      textDecoration: "line-through",
    });

    // 削除をクリックするとタスを削除する
    await userEvent.click(screen.getByRole("button", { name: "削除" }));
    expect(screen.queryByText("sample")).not.toBeInTheDocument();

    // "タスクが存在しません。"という文字列を表示する
    expect(screen.getByText("タスクが存在しません。")).toBeInTheDocument();
  });
});
```

今のままのコンポーネントだと、コンポーネントからコンポーネントへのstateの受け渡しができないのでコンポーネントにそれらの機能を作っていきます。

```tsx
export const TaskForm: React.FC = () => {
  const [text, setText] = useState("");
+  const { addTask } = useTasks();

+  const handleChange = useCallback(
+    (ev: React.ChangeEvent<HTMLInputElement>) => {
+      setText(ev.target.value);
+    },
+    []
+  );

+  const handleClick = useCallback(() => {
+    // idはひとまずDate().getTime()から取得することにする
+    const id = new Date().getTime();
+    addTask({ id, text, done: false });
+    setText("");
+  }, [addTask, text]);

  return (
    <div>
      <label htmlFor="task">タスクを入力してください</label>
      <input type="text" id="task" value={text} 
-       onChange={(ev) => setText(ev.target.value)}
+       onChange={handleChange}
      />
      <button 
-       onClick={() => setText("")}
+       onClick={handleClick}
        disabled={!text}>
        作成
      </button>
    </div>
  );
};
```

```tsx
export const TaskList: React.FC = () => {
-   const { state } = useTasks();
+   const { state, changeDone, deleteTask } = useTasks();

  if (!state.length) {
    return <p>タスクが存在しません。</p>;
  }
  return (
    <ul>
      {state.map((task) => (
        <li key={task.id}>
          <p
            data-testid={`task:${task.id}`}
            style={{ textDecoration: task.done ? "line-through" : "none" }}
          >
            {task.text}
          </p>
-         <button>done</button>
+         <button onClick={() => changeDone(task.id)}>done</button>
-         <button>削除</button>
+         <button onClick={() => deleteTask(task.id)}>削除</button>
        </li>
      ))}
    </ul>
  );
};
```

これで全てのテストが通り要件通りのものが出来上がりました。

## 最後に
プログラミングを勉強し始めReactに触れ、テスト？なにそれという状態で開発をしていましたが、改めてテストを学び、デバッグしやすくデグレしずらく、堅牢にできてそれでいて保守性も上がるというとても良いものだとわかりました。これからもテストを書いて快適な開発ライフを送っていきたいです！
