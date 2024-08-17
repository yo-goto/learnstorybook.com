---
title: '単純なコンポーネントを作る'
tocTitle: '単純なコンポーネント'
description: '単純なコンポーネントを切り離して作りましょう'
---

それでは、[コンポーネント駆動開発](https://www.componentdriven.org/) (CDD) の手法にのっとって UI を作ってみましょう。コンポーネント駆動開発とは、UI を最初にコンポーネントから作り始めて、最後に画面を作り上げる「ボトムアップ」の開発プロセスです。CDD を用いれば UI を作る際に直面する複雑性を軽減することができます。

## Task (タスク)

![Task コンポーネントの 3 つの状態](/intro-to-storybook/task-states-learnstorybook.png)

`Task` は今回作るアプリケーションのコアとなるコンポーネントです。タスクはその状態によって見た目が微妙に異なります。タスクにはチェックされた (または未チェックの) チェックボックスと、タスクについての説明と、リストの上部に固定したり解除したりするためのピン留めボタンがあります。これをまとめると、以下のプロパティが必要となります:

- `title` – タスクを説明する文字列
- `state` - タスクがどのリストに存在するか。またチェックされているかどうか。

`Task` の作成を始めるにあたり、事前に上記のそれぞれのタスクに応じたテスト用の状態を作成します。次いで、Storybook で、モックデータを使用し、コンポーネントを切り離して作ります。コンポーネントのそれぞれの状態について"ビジュアルテスト"を行いながら進めます。

## セットアップする

まずは、タスクのコンポーネントと、対応するストーリーファイル `src/components/Task.svelte` と `src/components/Task.stories.js` を作成しましょう。

`Task` の基本的な実装から始めます。`Task` は上述したプロパティと、タスクに対して実行できる 2 つの (リスト間を移動させる) アクションを引数として取ります:

```html:title=src/components/Task.svelte
<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  /** Event handler for the Pin Task */
  function PinTask() {
    dispatch('onPinTask', {
      id: task.id,
    });
  }

  /** Event handler for the Archive Task */
  function ArchiveTask() {
    dispatch('onArchiveTask', {
      id: task.id,
    });
  }

  /** Composition of the task */
  export let task = {
    id: '',
    title: '',
    state: '',
  };
</script>

<div class="list-item">
  <label for="title" aria-label={task.title}>
    <input type="text" value={task.title} name="title" readonly />
  </label>
</div>
```

上のコードは Todo アプリケーションの HTML を基にした `Task` の簡単なマークアップです。

下のコードは `Task` に対する 3 つのテスト用の状態をストーリーファイルに書いています:

```js:title=src/components/Task.stories.js
import Task from './Task.svelte';

import { action } from '@storybook/addon-actions';

export const actionsData = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

export default {
  component: Task,
  title: 'Task',
  tags: ['autodocs'],
  //👇 Our exports that end in "Data" are not stories.
  excludeStories: /.*Data$/,
  render: (args) => ({
    Component: Task,
    props: args,
    on: {
      ...actionsData,
    },
  }),
};

export const Default = {
  args: {
    task: {
      id: "1",
      title: "Test Task",
      state: "TASK_INBOX",
    },
  },
};

export const Pinned = {
  args: {
    task: {
      ...Default.args.task,
      state: "TASK_PINNED",
    },
  },
};

export const Archived = {
  args: {
    task: {
      ...Default.args.task,
      state: "TASK_ARCHIVED",
    },
  },
};
```

<div class="aside">

💡 [**Actions**](https://storybook.js.org/docs/essentials/actions)は切り離された環境で UI コンポーネントを開発する際の動作確認に役立ちます。アプリケーションの実行中には状態や関数を参照出来ないことがよくあります。`action()` はそのスタブとして使用できます。

</div>

Storybook には基本となる 2 つの階層があります。コンポーネントとその子供となるストーリーです。各ストーリーはコンポーネントに連なるものだと考えてください。コンポーネントには必要なだけストーリーを記述することができます。

- **コンポーネント**
  - ストーリー
  - ストーリー
  - ストーリー

Storybook にコンポーネントを認識させるには、以下の内容を含む `default export` を記述します:

- `component` -- コンポーネント自体
- `title` -- Storybook のサイドバーにあるコンポーネントを参照する方法
- `excludeStories` -- ストーリーから要求されるがStorybookではレンダリングされない情報
- `tags` -- コンポーネント用のドキュメントを自動生成する
- `render` -- ストーリーのレンダリングを追加で制御するための関数

To define our stories, we'll use Component Story Format 3 (also known as [CSF3](https://storybook.js.org/docs/api/csf) ) to build out each of our test cases. This format is designed to build out each of our test cases in a concise way. By exporting an object containing each component state, we can define our tests more intuitively and author and reuse stories more efficiently.

ストーリーを定義するには、テスト用の状態ごとにストーリーを生成する([CSF3](https://storybook.js.org/docs/api/csf)とも知られる)コンポーネントストーリーフォーマット3で使用します。このフォーマットは


Arguments or [`args`](https://storybook.js.org/docs/writing-stories/args) for short, allow us to live-edit our components with the controls addon without restarting Storybook. Once an [`args`](https://storybook.js.org/docs/writing-stories/args) value changes, so does the component.

`action()` allows us to create a callback that appears in the **actions** panel of the Storybook UI when clicked. So when we build a pin button, we’ll be able to determine if a button click is successful in the UI.

As we need to pass the same set of actions to all permutations of our component, it is convenient to bundle them up into a single `actionsData` variable and pass them into our story definition each time. Another nice thing about bundling the `actionsData` that a component needs is that you can `export` them and use them in stories for components that reuse this component, as we'll see later.

When creating a story, we use a base `task` arg to build out the shape of the task the component expects. Typically modeled from what the actual data looks like. Again, `export`-ing this shape will enable us to reuse it in later stories, as we'll see.

## 設定する

作成したストーリーを認識させ、アプリケーションの(`src/index.css` に位置する) CSS ファイルを使用できるようにするため、Storybook の設定をいくつか変更する必要があります。

まず、設定ファイル (`.storybook/main.js`) を以下のように変更してください:

```diff:title=.storybook/main.js
/** @type { import('@storybook/svelte-vite').StorybookConfig } */
const config = {
- stories: [
-   '../src/**/*.stories.mdx',
-   '../src/**/*.stories.@(js|jsx|ts|tsx)'
- ],
+ stories: ['../src/components/**/*.stories.js'],
  staticDirs: ['../public'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
  ],
  framework: {
    name: '@storybook/svelte-vite',
    options: {},
  },
};
export default config;
```

上記の変更が完了したら、`.storybook` フォルダー内の `preview.js` を、以下のように変更してください:

```diff:title=.storybook/preview.js
+ import '../src/index.css';

//👇 Configures Storybook to log the actions( onArchiveTask and onPinTask ) in the UI.
/** @type { import('@storybook/svelte').Preview } */
const preview = {
  actions: { argTypesRegex: "^on.*" },
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
  },
};

export default preview;
```

[`parameters`](https://storybook.js.org/docs/react/writing-stories/parameters) は Storybook の機能やアドオンの振る舞いをコントロールするのに使用します。この例では、`actions` (呼び出しのモック) がどのように扱われるかを設定しています。

`acitons` を使用することで、クリックした時などに Storybook の **Actions** パネルにその情報を表示するコールバックを作成できます。これにより、ピン留めボタンを作成するとき、ボタンがクリックされたことがテスト用の UI 上で確認できます。

Storybook のサーバーを再起動すると、タスクの 3 つの状態のテストケースが生成されているはずです:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/inprogress-task-states-7-0.mp4"
    type="video/mp4"
  />
</video>

## 状態を作り出す

ここまでで、Storybook のセットアップが完了し、スタイルをインポートし、テストケースを作りました。早速、デザインに合わせてコンポーネントの HTML を実装していきましょう。

今のところコンポーネントは簡素な状態です。まずはデザインを実現するために最低限必要なコードを書いてみましょう:

```html:title=src/components/Task.svelte
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  /** Event handler for the Pin Task */
  function PinTask() {
    dispatch('onPinTask', { id: task.id });
  }

  /** Event handler for the Archive Task */
  function ArchiveTask() {
    dispatch('onArchiveTask', { id: task.id });
  }

  /** Composition of the task */
  export let task = {
    id: '',
    title: '',
    state: ''
  };

  /* Reactive declaration (computed prop in other frameworks) */
  $: isChecked = task.state === "TASK_ARCHIVED";
</script>

<div class="list-item {task.state}">
  <label
    for={`checked-${task.id}`}
    class="checkbox"
    aria-label={`archiveTask-${task.id}`}
  >
    <input
      type="checkbox"
      checked={isChecked}
      disabled
      name={`checked-${task.id}`}
      id={`archiveTask-${task.id}`}
    />
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <span
      class="checkbox-custom"
      role="button"
      on:click={ArchiveTask}
      tabindex="-1"
      aria-label={`archiveTask-${task.id}`}
    />
  </label>
  <label for={`title-${task.id}`} aria-label={task.title} class="title">
    <input
      type="text"
      value={task.title}
      readonly
      name="title"
      id={`title-${task.id}`}
      placeholder="Input title"
    />
  </label>
  {#if task.state !== 'TASK_ARCHIVED'}
    <button
      class="pin-button"
      on:click|preventDefault={PinTask}
      id={`pinTask-${task.id}`}
      aria-label={`pinTask-${task.id}`}
    >
      <span class="icon-star" />
    </button>
  {/if}
</div>
```

先ほど追加したマークアップとインポートした CSS により以下のような UI ができます:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/inprogress-task-states-7-0.mp4"
    type="video/mp4"
  />
</video>

## コンポーネントのビルド

これでサーバーを起動したり、フロントエンドアプリケーションを起動したりすることなく、コンポーネントを作りあげることができました。次の章では、Taskbox の残りのコンポーネントを、同じように少しずつ作成していきます。

見た通り、コンポーネントだけを切り離して作り始めるのは早くて簡単です。あらゆる状態を掘り下げてテストできるので、高品質で、バグが少なく、洗練された UI を作ることができることでしょう。

## アクセシビリティの問題の検知

アクセシビリティテストとは、[WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) のルールと他の業界で認めれたベストプラクティスに基づく経験則に対して、自動化ツールを用いることでレンダリングされた DOM を監視することを指します。これは視覚障害、聴覚障害、認知障害などの障害をお持ちの方を含む、できるだけ多くのユーザーがアプリケーションを利用できるように、明らかなアクセシビリティの違反を検知するために QA の第一線として機能します。

Storybook には公式の[アクセシビリティアドオン](https://storybook.js.org/addons/@storybook/addon-a11y)があります。これは、Deque の [axe-core](https://github.com/dequelabs/axe-core) を使っており、[WCAG の問題の 57%](https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues/) に対応しています。

それでは、どのように動かすのかみてみましょう! 以下のコマンドでアドオンをインストールします:

```shell
yarn add --dev @storybook/addon-a11y
```

アドオンを利用可能にするために、Storybook の設定ファイル(`.storybook/main.js`)を以下のように設定します:

```diff:title=.storybook/main.js
/** @type { import('@storybook/svelte-vite').StorybookConfig } */
const config = {
  stories: ['../src/components/**/*.stories.js'],
  staticDirs: ['../public'],
  addons: [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
+   '@storybook/addon-a11y',
  ],
  framework: {
    name: "@storybook/svelte-vite",
    options: {},
  },
};
export default config;
```

最後に、Storybookを再起動して、UI内で有効化した新しいアドオンを確認してみましょう。

![Task accessibility issue in Storybook](/intro-to-storybook/finished-task-states-accessibility-issue-7-0.png)

ストーリーを一通り見てみると、このアドオンが一つのアクセシビリティの問題を検知したことがわかります。[**「Elements must have sufficient color contrast」**](https://dequeuniversity.com/rules/axe/4.4/color-contrast?application=axeAPI)というエラーメッセージは本質的にタイトルと背景に十分なコントラストがないことを指しています。そのため、アプリケーションの CSS (`src/index.css` にある)の中で、テキストカラーをより暗いグレーに修正する必要があります。

```diff:title=src/index.css
.list-item.TASK_ARCHIVED input[type="text"] {
- color: #a0aec0;
+ color: #4a5568;
  text-decoration: line-through;
}
```

以上です！これで、UI のアクセシビリティ向上の最初のステップが完了です。アプリケーションをさらに複雑にしても、他の全てのコンポーネントに対してこのプロセスを繰り返すことができ、追加のツールやテスト環境を準備する必要はありません。

<div class="aside">
💡 Git へのコミットを忘れずに行ってください！
</div>
