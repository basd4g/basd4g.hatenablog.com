---
title: Nuxt.js(v2.9.2)とTypeScriptの開発環境を作る。
date: 2019/09/26 19:30
tags: Nuxt.js TypeScript
---

Nuxt.jsとTypeScriptで開発環境を作るときのまとめ。(2019/9/5時点)

お急ぎの方は、
記事中の作業を行ったものを[nuxt.ts-template](https://github.com/basd4g/nuxt.ts-template)としてGitHubのリポジトリに上げたので、cloneないしForkして使ってほしい。

## 目指すもの

以下のものが問題なく使える環境。

- Nuxt.js v2.9.2
- TypeScript
- ESLint
- nuxt-property-decorator

## nuxt-property-decorator (vue-property-decorator)

Nuxt.jsとTypeScriptを組み合わせるときは、nuxt-property-decorator(vue-property-decorator)の利用が推奨されている。[^1]

もともとのNuxt.jsとは書き方が変わる。参考文献にいくつか載せてあるので、もともとの書き方と比較しながら慣れると良さそう。[^2]

今回は最後にインストールするので、一つ前の[ESLint](#3-eslint)までで止まればこれを使わない環境も構築できる。

## 手順

大まかには次の流れで進む。
いくつかハマりどころがあったので、私の環境でうまく行った手順を残しておく。

(特にESLintに文句を言われることが多く、最初いきあたりばったりで進めていたら沢山のパッケージをインストールしてしまった。
この記事は、その後不要なものがあまり入らないようにやり直した内容の記録である。)

1. yarn create nuxt-app
2. TypeScriptを入れる
3. vue-property-decoratorを入れる
4. ESLintを入れる

### 1. create nuxt-app

Nuxt.jsで環境構築するときにお決まりのcreate-nuxt-app。
yarnだと、 `$ yarn create nuxt-app hogehoge` とする。

```sh
$ yarn create nuxt-app nuxt.ts-template
# yarn create v1.17.3
# create-nuxt-app v2.10.1

# 今回は次のように設定した
# Project name, Project description, Author name は適宜
# package manager : Yarn
# UI framework : None
# custom server framework : None (Recommended)
# Nuxt.js modules :
# test framework : None
# rendering mode : Universal (SSR)
# development tools :

$ cd nuxt.ts-template/
# 以下全てカレントディレクトリは移動しない。編集するファイル名もnuxt.ts-template/をカレントディレクトリとして表記する。

$ yarn dev
# 起動できることを確認。ブラウザからNuxt.jsのロゴが見えればオーケー。(以下の起動確認も同様。)

```

### 2. Install TypeScript

まず、TypeScriptに関するパッケージをインストールする。

```sh
$ yarn add -D @nuxt/typescript-build
$ yarn add @nuxt/typescript-runtime @nuxt/types
$ mv nuxt.config.js nuxt.config.ts
$ touch tsconfig.json
```

TypeScriptに合わせて以下の設定ファイルを2つ、ソースを3つ書き換える。

- `nuxt.config.ts`
- `Logo.vue`
- `index.vue`
- `package.json`
- `tsconfig.json` (新たに作成)

`./package.json`

```diff
...
   "scripts": {
-    "dev": "nuxt",
-    "build": "nuxt build",
-    "start": "nuxt start",
-    "generate": "nuxt generate"
+    "dev": "nuxt-ts",
+    "build": "nuxt-ts build",
+    "start": "nuxt-ts start",
+    "generate": "nuxt-ts generate"
   },
...
```

`./nuxt.config.ts`

```diff
+import {Configuration} from '@nuxt/types'

-export default {
+const nuxtConfig: Configuration = {
...
  buildModules: [
+      '@nuxt/typescript-build'
  ],
...
}
+module.exports = nuxtConfig
```

`./tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "lib": [
      "esnext",
      "esnext.asynciterable",
      "dom"
    ],
    "esModuleInterop": true,
    "allowJs": true,
    "sourceMap": true,
    "strict": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "~/*": [
        "./*"
      ],
      "@/*": [
        "./*"
      ]
    },
    "types": [
      "@types/node",
      "@nuxt/types"
    ]
  },
  "exclude": [
    "node_modules"
  ]
}
```

`./components/Logo.vue`

```diff
...
</template>
+
+<script lang="ts">
+import Vue from 'vue'
+export default Vue.extend({  
+})
+</script>
+
<style>
...
```

`./pages/index.vue`

```diff
-<script>
+<script lang="ts">
```

```sh
$ yarn dev
# TypeScriptで動くことを確認。
```

### 3. ESlint

ESLintと関連パッケージをインストールする。(今回はcreate nuxt-appでESLintをインストールしていないのでESLint自体もここでインストールする。)

```sh
$ yarn add eslint-config eslint
$ yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser @nuxtjs/eslint-config-typescript
$ touch .eslintrc.js
```

ESLintの設定を以下の3ファイルに記述する。

- `package.json`
- `nuxt.config.ts`
- `.eslintrc.js`

`package.json`

```diff
...
  "scripts": {
    "dev": "nuxt-ts",
    "build": "nuxt-ts build",
    "start": "nuxt-ts start",
-    "generate": "nuxt-ts generate"
+    "generate": "nuxt-ts generate",
+    "lint": "eslint --ext .ts,.js,.vue --ignore-path .gitignore .",
+    "lint:fix": "eslint --ext .ts,.js,.vue --ignore-path .gitignore . --fix"
  },
...
```

`nuxt.config.ts`

```diff
...
  build: {
    /*
    ** You can extend webpack config here
    */
+    /*
    extend (config, ctx) {
    }
+   */
...
```

`.eslintrc.js`

```js
module.exports = {
  plugins: ['@typescript-eslint'],
  parserOptions: {
    parser: '@typescript-eslint/parser'
  },
  extends: [
    '@nuxtjs/eslint-config-typescript'
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error'
  }
}
```

```sh
$ yarn lint:fix
$ yarn dev
# Lintが動いていることと、変わらず起動できることを確認。
```

### 4.nuxt-property-decorator

nuxt-property-decoratorをインストールする。(vue-property-decoratorでも良いかもしれない。)

```sh
$ yarn add nuxt-property-decorator
```

`index.vue`をデコレータを使った書き方に修正する。  
またそのままだとコンパイルエラーを出すので、TypeScriptのオプションにデコレータの使用を宣言する。(`tsconfig.json`を編集)

`./pages/index.vue`

```diff
...
<script lang="ts">
+import { Component, Vue } from 'vue-property-decorator'
import Logo from '~/components/Logo.vue'

-export default {
+@Component({
  components: {
    Logo
  }
-}
+})
+export default class Index extends Vue {
+}
</script>
...
```

`./tsconfig.json`

```diff
...
    "compilerOptions": {
...
      "types": [
        "@types/node",
        "@nuxt/types"
-      ]
+      ],
+      "experimentalDecorators": true
    },
...
```

```sh
$ yarn dev
# デコレータを使った表記での起動を確認。
```

お疲れさまでした。

## 参考

重要参考文献

1. [Qiita Nuxt.js 2.9でのTypeScript対応](https://qiita.com/iwata@github/items/a94c6d116a3e84911628)
1. [Nuxt TypeScript Setup](typescript.nuxtjs.org/guide/setup.html#installation)

その他の参考文献

1. [Nuxt.js インストール](https://ja.nuxtjs.org/guide/installation/) ... yarn create nuxt-app について
1. [Nuxt.js TypeScriptサポート](https://ja.nuxtjs.org/guide/typescript/) ... Nuxt.jsにTypeScript関連パッケージをインストールする。Nuxt.js v2.8までの記述のため、このページの内容のとおりではv2.9.2ではうまく行かない
1. [GitHub eslint-config](https://github.com/nuxt/eslint-config) ... eslint-configのセットアップ
1. [デコレータ | TypeScript 日本語ハンドブック | js STUDIO](https://js.studio-kingdom.com/typescript/handbook/decorators) ... `experimentalDecorators`について
1. [TypeScriptではじめるVueコンポーネント（vue-class-component）](https://qiita.com/hako1912/items/8d9968d07748d20825f8)... vue-property-decorator(nuxt-property-decorator)を使った`@Component`デコレータの書き方
1. [Nuxt.js+ExpressのプロジェクトをTypeScript化する](https://crieit.net/posts/Nuxt-js-Express-TypeScript) ... nuxt-property-decoratorを使った`@Component`デコレータの書き方
1. [はじめてのvue-property-decorator](https://qiita.com/simochee/items/e5b77af4aa36bd0f32e5) ... vue-property-decorator(nuxt-property-decorator)を使ったデコレータの書き方

[^1]: その他の参考文献2を参照
[^2]: [GitHub vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)
