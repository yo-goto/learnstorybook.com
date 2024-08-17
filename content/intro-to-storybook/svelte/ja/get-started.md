---
title: 'Svelte 向け Storybook のチュートリアル'
tocTitle: 'はじめに'
description: '開発環境に Storybook を導入しましょう'
---

Storybook は開発時にアプリケーションと並行して動きます。Storybook を使用することで、UI コンポーネントをビジネスロジックやコンテキストから切り離して開発できるようになります。この文書は Svelte 向けです。他にも [Vue](/intro-to-storybook/vue/en/get-started)、[Angular](/intro-to-storybook/angular/en/get-started)、[React](/intro-to-storybook/react/en/get-started)、[React Native](/intro-to-storybook/react-native/en/get-started)、[Ember](/intro-to-storybook/ember/en/get-started) 向けのバージョンがあります。

![Storybook と開発中のアプリの関係](/intro-to-storybook/storybook-relationship.jpg)

## Set up Svelte Storybook

Storybook を開発プロセスに組み込むにあたり、いくつかの手順を踏む必要があります。まずは、[degit](https://github.com/Rich-Harris/degit) を使用してビルド環境をセットアップしましょう。このパッケージを利用することで、テンプレート（アプリケーションの一部をデフォルト設定で構築したもの）をダウンロードし、開発ワークフローの短縮に役立てることができます。

それでは、次のコマンドを実行してください:

```shell:clipboard=false
# Create our application:
npx degit chromaui/intro-storybook-svelte-template taskbox

cd taskbox

# Install dependencies
yarn
```

<div class="aside">
💡 このテンプレートには本バージョンのチュートリアルに必要なスタイル、アセット、最低限の設定が含まれています。
</div>

それでは、アプリケーションのさまざまな環境が問題なく動くことを次のコマンドで確認しましょう:

```shell:clipboard=false
# Start the component explorer on port 6006:
yarn storybook

# Run the frontend app proper on port 5173:
yarn dev
```

主なフロントエンド開発のモード: コンポーネント開発 (Storybook)、アプリケーション自体

<!-- Needs to be updated and the link updated to ![Main modalities](/intro-to-storybook/app-main-modalities-svelte.png) -->

![3 のモード](/intro-to-storybook/app-three-modalities-svelte.png)

作業をする対象に応じて、このモードのうち 1 つまたは複数を同時に動かしながら作業します。今は単一の UI コンポーネントを作るのに集中するため、Storybook を動かすことにしましょう。

## 変更をコミットする

この段階で、ローカルリポジトリにファイルを追加しても大丈夫です。以下のコマンドを実行して、ローカルリポジトリを初期化し、これまでに行った変更を追加してコミットしてください。

```shell
git init
```

つづいて:

```shell
git add .
```

次に以下を実行します

```shell
git commit -m "first commit"
```

最後に:

```shell
git branch -M main
```

それでは最初のコンポーネントを作り始めましょう！
