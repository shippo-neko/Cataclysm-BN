# ドキュメントの更新

ドキュメントサイトを更新するには、以下の作業が必要です:

- [markdownを短時間で学ぶ](https://learnxinyminutes.com/docs/markdown/)
- ドキュメントサイトのソースコードはGitHubでホストされているため、[GitHubアカウントを作成する](https://github.com/join)

## ブラウザ経由

![edit page](img/edit.webp)

1. ページ下部にある `Edit page` ボタンをクリックします。

![alt text](img/github-edit.webp)

2. ドキュメントサイトのGitHubページにリダイレクトされます。ここで変更を編集し、プレビューできます。

> [!NOTE]
>
> - `CONTRIBUTING.md` の `.md` は Markdown ファイルを表します。
> - `docs.mdx` の`.mdx` は [MarkDown eXtended](https://mdxjs.com) を表します。
>   - これは、JavaScript および [jsx コンポーネント][jsx] のサポートを含む Markdown のスーパーセットです。
>   - やや複雑ですが、インタラクティブなコンポーネントを使用できます。

[jsx]: https://www.typescriptlang.org/docs/handbook/jsx.html

![propose changes window](https://github.com/scarf005/Cataclysm-BN/assets/54838975/d4a06795-1680-4706-a84c-072346bff109)

3. 右上隅にある `Commit changes...` ボタンをクリックして、 [変更をコミットします。](https://github.com/git-guides/git-commit) 以下の点を確認してください。

- 短く分かりやすい`コミットメッセージ`を記述する。
- `Create a new branch for this commit and start a pull request` のチェックボックスにチェックを入れる。

![comparing changes page](https://github.com/scarf005/Cataclysm-BN/assets/54838975/3551797e-847b-45fe-8869-8b0b15bfb948)

4. `Comparing changes` ページにリダイレクトされます。 `Create pull request` ボタンをクリックして、 [プルリクエストを作成します](./contributing.md#pull-request-notes)。

![open a pull request page](https://github.com/scarf005/Cataclysm-BN/assets/54838975/2a987c19-b165-43c2-a5a2-639f22202926)

5. `Create pull request` ボタンをクリックして、[PRを開きます](./contributing.md#pull-request-notes)。 小規模な変更の場合は、PR の本文を空欄にして構いません。

## ローカル開発

> [!NOTE]
>
> このセクションでは、[git](https://git-scm.com) および[javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)に関するある程度の知識があることを前提としています。もちろん、作業を進めながら学ぶことも可能です。

ドキュメントサイトをローカルで実行するには、以下の作業が必要です:

- フォーマット設定と自動ドキュメント生成のために [deno](https://deno.com) をインストールする

### 開発サーバーのセットアップ

```sh
(Cataclysm-BN) $ deno task docs serve

# または、すでに docs ディレクトリ内にいる場合
(Cataclysm-BN/docs) $ deno task serve
```

ドキュメントサイトに `http://localhost:3000`でアクセスできるようになります。ドキュメントに変更を加えると、開発サーバーは自動的にリロードされます。

### ページの自動生成

Lua および CLI のドキュメントは、ソースコードから自動的に生成されます。これらを生成するには、プロジェクトのルートに移動して、以下を実行してください:

```sh
(Cataclysm-BN) $ deno task docs:gen
```

## ライセンス

- Markdown ファイル (`.md` および `.mdx` ファイルを含むがこれらに限定されない)への貢献を行うことにより、貢献をゲームと同じライセンスである [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/)の下でライセンスすることに同意するものとします。

- ドキュメンテーションページのソースコード (`.ts` ファイルを含むがこれらに限定されない)への貢献を行うことにより、貢献を [AGPL 3.0](https://www.gnu.org/licenses/agpl-3.0.en.html)の下でライセンスすることに同意するものとします。
