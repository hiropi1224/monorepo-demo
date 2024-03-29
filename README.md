# [Turborepo](https://turbo.build/repo) まとめ
- モノレポ管理ツール
    - 他には[nx](https://nx.dev/getting-started/intro)、[moon](https://moonrepo.dev/moon)などがある

  - JSに閉じられているためbackendも併せて管理したい場合は選びにくいか

- 重要なのは`package.json`の`name`
    - ディレクトリ名と`package.json`の`name`は一致させておくのがベストか

- ルートの`package.json`の以下部分で各workspaceを認識できるように設定されている
```
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
```

- appsからpackagesのファイルを使うためには`package.json`の`dependencies`に追加する
```
  "dependencies": {
    "next": "^13.1.1",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "ui": "*"
  },
```
※tsconfigも同様
```
  "devDependencies": {
    "@babel/core": "^7.0.0",
    "eslint": "7.32.0",
    "eslint-config-custom": "*",
    "tsconfig": "*",
    ...以下略
```

- packages/uiの`package.json`の`main`、`types`でエントリーポイントを指定している
```
{
  "name": "ui",
  "version": "0.0.0",
  "main": "./index.tsx", // 読み込まれた時のエントリーポイントを指定
  "types": "./index.tsx", // 読み込まれた時のエントリーポイントを指定
  "license": "MIT",
  ...以下略
```
index.tsxでButtonがexportされているためappsで使うことができる
```
import * as React from "react";
export * from "./Button";
```

- next.js環境の`tsconfig.json`の設定は`packages/tsconfig`の`nextjs.json`を参照することで統一できる

- `extends`についても作成したjsonファイルを参照することで依存させることができる
```
  "extends": "./base.json",
```

- [ESLintについて](https://turbo.build/repo/docs/getting-started/create-new#understanding-eslint-config-custom)

.eslintrc.js
```
module.exports = {
  // This tells ESLint to load the config from the workspace `eslint-config-custom`
  extends: ["custom"],
};
```
ルートに`.eslintrc.js`を置くことで各プロジェクトに`.eslintrc.js`を持たない場合はルートの`.eslintrc.js`を参照することになる
  - packages/uiはカレントディレクトリに`.eslintrc.js`を持たないため上のディレクトリに探しに行き、ルートにある`.eslintrc.js`を参照する
  - カレントディレクトリに`.eslintrc.js`を作成してルールを追加することで、特定のworkspaceのみルールを適用することもできる

- `turbo lint`コマンド実行でlintを走らせる
  - 1度走らせたものはキャッシュされ、変更がないものはチェックされない  
  [Using the cache](https://turbo.build/repo/docs/getting-started/create-new#using-the-cache)
  - `package.json`にlintの設定があるものが実行されている

- `turbo.json`のbuild設定について  
outputsに指定されたものはturboタスク終了後、キャッシュに保存される
```
{
  "pipeline": {
    "build": {
      "outputs": [".next/**", "!.next/cache/**"]
    }
  }
}
```
  - ビルド後にディレクトリ内の`.next`を削除して再度ビルドするとキャッシュから復元される

- [Configuring Cache Inputs](https://turbo.build/repo/docs/core-concepts/caching#configuring-cache-inputs)  
`turbo.json`内で`inputs`を指定することで特定のファイルの変更を検知してキャッシュとみなすかどうかを管理できる

- [リモートキャッシング](https://turbo.build/repo/docs/core-concepts/remote-caching)  
Vercelなどのプロバイダーと連携することで、Turborepo はリモート キャッシュ (タスクの結果を保存するクラウド サーバー) と安全に通信できる
  - `turbo login`、`turbo link`とすると、キャッシュ アーティファクトがローカルとリモート キャッシュに保存される
  - CI/CD、デプロイなど、キャッシュがあればリソース削減につながる

- [workspaceのフィルタリング](https://turbo.build/repo/docs/core-concepts/monorepos/filtering)  
`--filter`コマンドを使うことで、タスクを実行するworkspaceを選択できる
```
turbo build --filter=my-pkg --filter=my-app
```
```
turbo run build --filter=admin-*
```
- [Configuring Workspaces](https://turbo.build/repo/docs/core-concepts/monorepos/configuring-workspaces)  
ルートで定義されたタスクの構成をオーバーライドするには`turbo.json`で指定していたが、任意のworkspaceに`turbo.json`を作成し、最上位の`extends`キーを使用して設定できる
```
{
  "extends": ["//"],
  "pipeline": {
    "build": {
      // custom configuration for the build task in this workspace
    },
    // new tasks only available in this workspace
    "special-task": {},
  }
}
```

- [Comparison to Workspace-specific tasks](https://turbo.build/repo/docs/core-concepts/monorepos/configuring-workspaces#comparison-to-workspace-specific-tasks)  
Workspace Configurations はroot のworkspace#task構文とよく似ているが重要な違いが1つある
  - workspace#taskでは設定が完全に上書きされるため、重複するものでも複製する必要がある
```
  {
  "pipeline": {
    "build": {
      "outputMode": "hash-only",
      "inputs": ["src/**"],
      "outputs": [".next/**", "!.next/cache/**"],
    },
    "my-sveltekit-app#build": {
      "outputMode": "hash-only", // must duplicate this
      "inputs": ["src/**"], // must duplicate this
      "outputs": [".svelte-kit/**"]
    }
  }
}
```
  - Workspace Configurationsでは`outputMode`、`inputs`は継承されるため複製する必要はない


## [公式ドキュメント](https://turbo.build/repo/docs)
### What is a Monorepo?
```
モノレポとは、1つのコードベースに多くの異なるアプリやパッケージを集めたものです。
ポリレポというのは、複数のコードベースを別々に公開し、バージョン管理する方法です。
```

## Quickstart
以下コマンドを実行してモノレポを作成

```
npx create-turbo@latest
```

![picture 1](images/c742895f9e265cab2891e6b9788a8b6c4a3d00e6073f7adeb9998953033c8c36.png)  

- docs,web,eslint-config-custom,tsconfig,uiそれぞれが独立したworkspace（polyrepoだと5つのリポジトリがあるようなイメージ）
![picture 2](images/78ef58196828d27fc0622cada400f923e7604a62b9c016c159ccfbd697c4072b.png)  

- apps
    - ユーザー側が見るもの（デプロイする環境）
- packages
    - 開発用

packagesは相互に依存することはある

## [Summary](https://turbo.build/repo/docs/getting-started/create-new#summary)

ワークスペース間の依存関係マッピング
```
web - depends on ui, tsconfig and eslint-config-custom
docs - depends on ui, tsconfig and eslint-config-custom
ui - depends on tsconfig and eslint-config-custom
tsconfig - no dependencies
eslint-config-custom - no dependencies
```

## [Monorepo Handbook](https://turbo.build/repo/docs/handbook)

### How do monorepos work?

### [Package Install](https://turbo.build/repo/docs/handbook/package-installation)
- packageをインストールする場合は以下コマンドを利用することでそれぞれのworkspaceにインストールすることができる。

npmの場合
```
npm install <package> --workspace=<workspace>
```

yarnの場合
```
yarn workspace <workspace> add <package>
```