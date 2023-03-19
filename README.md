# [Turborepo](https://turbo.build/repo) まとめ
- モノレポ管理ツール
    - 他には[nx](https://nx.dev/getting-started/intro)、[moon](https://moonrepo.dev/moon)などがある

- JSに閉じられているためbackendも併せて管理したい場合は選びにくいか

- 重要なのは`package.json`の`name`
    - ディレクトリ名と`package.json`の`name`は一致させておくのがベストか。

- ルートの`package.json`の以下部分で各workspaceを認識できるように設定されている
```
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
```

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