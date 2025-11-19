# 変更履歴のガイドライン

PR(プルリクエスト)のタイトルは、変更履歴をより簡単に生成できるように、 [Convensional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従ってください。
形式は以下のいずれかです:

```
<Type>: <PR subject>
<Type>(<Scope>, <Scope>, ...): <PR subject>
```

例えば、PRのタイトルは以下のようになります:

```
feat: add new mutation
feat(port): port mutation description from DDA
```

PRのタイトルは、プレイヤーが一目で理解しやすいようにすべきです。命令形かつ説明的なタイトルand 命令形かつ説明的なタイトル(`<verb> <noun>`) を使用することが推奨されます:

例えば以下のとおりです。

```diff
- feat: rebalancing some rifles
+ feat: nerf jam chance of m16 and m4
```

変更前: バフされたのかナーフされたのか、どのライフルが変更されたのか、PR(プルリクエスト)の完全な説明を読まないと分かりづらい。
変更後: タイトル自体から何が正確に変更されたのか容易に理解できる。

## タイプ

タイプはPRタイトルの最初の単語です。行われている変更の種類を指定します。
迷ったときは、新しい機能には`feat`を、バグ修正には`fix`を使用してください。
以下に、頻繁に使用されるいくつかのカテゴリを示します:

### `feat`: 機能

新しい機能、追加、またはバランス調整。

### `fix`: バグ修正

バグを修正するもの、またはゲームをより安定させるもの。

### `refactor`: インフラ整備

動作を変更せずに開発を容易にするもの。例えば、以下のとおりです。

- `C++` のリファクタリングとオーバーホール
- `Json` の再編成
- `docs/`, `.github/` およびリポジトリの変更
- その他の開発ツール

### `build`: ビルド

ビルドプロセスを改善する:

- より堅牢にする
- より使いやすくする
- コンパイル時間を短縮する

### その他

- `docs`: ドキュメントの変更
- `style`: コードスタイルの変更（空白、フォーマットなど）、通常はJSONフォーマットの修正
- `perf`: パフォーマンスの改善
- `test`: 不足しているテストの追加または既存のテストの修正
- `ci`: CI（継続的インテグレーション）プロセスの変更
- `chore`: 上記のどのカテゴリにも当てはまらないその他の変更
- `revert`: 以前のコミットの取り消し

## スコープ

1. スコープをさらに絞り込むために、カテゴリの後に括弧内でそれらを使用します。
2. スコープの数に制限はありません。
3. それらは任意ですが、推奨されます。
4. これらは単なるガイドラインであり、ルールではありません。ご自身のPRに最適なものを自由に選択してください！

### `<None>`: 一般的な機能

例として:

プレイヤーに関連する変更点:

- プレイヤーが何か新しいことを実行できる (例：変異、スキル)
- プレイヤーに何か新しいことが起こり得る (例：新しい病気)

新しいコンテンツとして:

- 新しいモンスター
- 新しいマップエリア
- 新しいアイテム
- 新しい乗り物
- 新しい仕掛け

PRタイトルの例:

```
feat: strength training activity
feat: mutation system overhaul
feat: semi-plausible smokeless gunpowder recipe
feat(port): game store
```

### `lua`: Lua APIへの変更

Lua APIへの変更。例えば、以下のものを含みます:

- [新しいバインディングの追加](../mod/lua/guides/binding.md)
- Luaのドキュメント／API生成の改善
- [ハードコードされたC++機能をLuaへ移行](https://github.com/cataclysmbnteam/Cataclysm-BN/pull/6901)

PRタイトルの例:

```
feat(lua): add dialogue bindings
```

### `UI`: インターフェース

UI/UX（ユーザーインターフェース／ユーザーエクスペリエンス）の変更。
例えば、以下のものを含みます:

- マウスサポートの追加
- メニューの追加／調整
- ショートカットの変更
- ワークフローの合理化
- プレイ体験の改善

PRタイトルの例:

```
feat(UI): More info about items dropped with bags
feat(UI): overhaul encumbrance UI
```

### `i18n`: 国際化

翻訳およびその他の言語のサポートを改善します。

```
fix(UI,i18n): recipe names not translated unless learned
```

### `mods` および `mods/<MOD_ID>`: Mods

- Mod内に含まれる変更
- Mod内で可能なことを拡張するもの

PRタイトルの例::

```
feat(mods/Magical_Nights): add missing owlbear pelts recipe
fix(mods): No Hope doesn't make the world freezing
```

### `balance`: バランス調整

ゲームバランスへの変更

PRタイトルの例:

```
feat(balance): Give moose pelts instead of hides
```

### `port`: DDAまたはその他のフォークからの移植

PRタイトルの例:

```
feat(port): game shop
```
