# Luaを用いたMOD開発

## 役立つリンク

- [Lua 5.3 リファレンスマニュアル](https://www.lua.org/manual/5.3/)
- [Sol2 ドキュメント](https://sol2.readthedocs.io/en/latest/)
- [Lua プログラミング (初版)](https://www.lua.org/pil/contents.html)

## MODの例

`data/mods/` 内に、ここで説明するLua APIを利用した、コメントが豊富に付けられたMODがいくつかあります。

- `smart_house_remotes` - ガレージドアや窓のカーテンを操作するためのリモートを追加します。
- `saveload_lua_test` - LuaのセーブまたはロードAPIをテストするためのMODです。

## ゲーム内Luaコンソール

ゲーム内Luaコンソールは、デバッグメニュー、または `Lua コンソール` ホットキー（既定では未割り当て）から利用可能です。

これは比較的シンプルですが、入力履歴の保持、Luaスクリプトからの出力およびエラーの表示、Luaスニペットの実行、および戻り値の出力が可能です。

`gdebug.set_log_capacity( num )` を実行することでコンソールログの容量を調整できます（既定は100エントリ）か、`gdebug.clear_lua_log()`を実行することでログをクリアできます。

## Luaホットリロード

MOD開発プロセスをスピードアップするため、BNはLuaホットリロード機能をサポートしています。

ファイルシステムウォッチャーはないため、ホットリロードは対応する
`Luaコードのリロード` ホットキー（既定では未割り当て）を介して手動でトリガーする必要があります。ホットリロードは、コンソールウィンドウから対応するホットキーを押すか、`gdebug.reload_lua_code()` コマンドを実行することでもトリガーできます。通常のLuaスクリプトからこのコマンドを実行すると、意図しない結果を招く可能性があるため、自己責任で使用してください。

なお、すべてのコードがホットリロード可能であるわけではありません。これについては後のセクションで説明します。

## ゲームデータのロード

世界がロードされる際、ゲームはおおよそ以下の手順で処理を実行します。

1. 世界関連の内部状態を初期化し、ワールドをアクティブとして設定します。
2. 世界のアーティファクトアイテムタイプをロードします（アーティファクトはハック的なものであり、レリック
    + Luaに有利な形で近いうちに廃止される可能性が高いです）。
3. 世界で使用されているMODのリストを取得します。
4. リストに従ってMODをロードします。
5. アバター関連の内部状態を初期化します。
6. セーブディレクトリから、実際のオーバーマップ地形データ、アバターデータ、およびリアリティバブルデータをロ
   ードします。

ここで重要となるのは、MODのロードステージです。これにはいくつかのサブステップがあります。
1. ロード関数が世界MODのリストを受け取ります。
2. 存在しないMODを破棄し、それぞれについてデバッグメッセージを出力します。
3. リストに残っているMODをチェックし、MODがLuaを必要とするにもかかわらずゲームビルドがLuaをサポートしない場
   合、エラーをスローします。
4. また、ゲームのLua APIバージョンがMODが使用しているものと異なる場合、警告を出します。
5. リスト上のLuaを使用するすべてのMODについて、そのMODの [`preload.lua`](#preloadlua)スクリプトを実行します
   （存在する場合）。
6. リストと同じ順序ですべてのMODを巡回し、各MODのフォルダからJSON定義をロードします。
7. ロードされたデータをファイナライズします（copy-fromの解決、複雑な状態を持つ一部のタイプを使用できるように
   準備）。
8. リスト上のLuaを使用するすべてのMODについて、そのMODの [`finalize.lua`](#finalizelua)スクリプトを実行しま
   す（存在する場合）。
9.  ロードされたデータの整合性をチェックします（値の検証、疑わしい値の組み合わせに関する警告など）。
10. (R) リスト上のLuaを使用するすべてのMODについて、そのMODの [`main.lua`](#mainlua) スクリプトを実行します
    （存在する場合）。

したがって、MODのLuaコードを配置できるスクリプトは、[`preload.lua`](#preloadlua)、
[`finalize.lua`](#finalizelua)、および[`main.lua`](#mainlua)の3つのみです。これら3つの違いと、それぞれの意図された使用例を以下に説明します。

必要に応じて、1つ、2つ、またはすべてのスクリプトを使用できます。

ホットリロードを実行する場合、ゲームは(R)とマークされたステップを繰り返します。つまり、作業中のコードをホットリロード可能にしたい場合は、[`main.lua`](#mainlua) に配置してください。

### `preload.lua`

このスクリプトは、イベントフックを登録し、その後ゲームのJSONロードシステムによって参照される定義（例: アイテム使用アクション）を設定することを目的としています。ここでフックを登録し、後続のステージ(例: ホットリロードの影響を受けられるように [`main.lua`](#mainlua))でそれらを定義することができます。

### `finalize.lua`

このスクリプトは、copy-fromが解決された後、MODがJSONからロードされた定義を変更できるようにすることを目的としていますが、現時点ではこれを行うAPIはありません。

TODO: ファイナライゼーションのためのAPI

### `main.lua`

このスクリプトは、MODの主要なロジックを実装することを目的としています。これには、以下のものが含まれますが、これらに限定されません。

1. MODの実行時状態
2. ゲーム開始時のMODの初期化
3. 必要に応じたMODのセーブ/ロードコード
4. [`preload.lua`](#preloadlua)で設定されたフックの実装

## Lua APIの詳細

標準のLuaだけでも多くの興味深いことができますが、統合上の制約により、潜在的なバグを防ぐためにいくつかの制限が課されています。

- パッケージ（またはLuaモジュール）のロードは、`data/lua/` ディレクトリと `data/mods/<mod_id>/` ディレクトリ
  に限定されます。
- 現在のMOD IDは `game.current_mod` 変数に格納されます。
- MODの実行時状態は、`game.mod_runtime[ game.current_mod ]` テーブルに存在する必要があります。他のMODのIDを
  知っていれば、`game.mod_runtime[ that_other_mod_id ]`と同様の方法でそれらの実行時状態にアクセスすることで、連携することも可能です。
- グローバル状態への変更は、スクリプト間で利用できません。これは、関数名や変数名間の偶発的な衝突を防ぐためで
  す。グローバル変数や関数を定義することは可能ですが、それらは自分のMODに対してのみ可視となります。

### Luaライブラリと関数

スクリプトが呼び出されると、いくつかの標準Luaライブラリがプリロードされた状態で提供されます。

| ライブラリ| 説明                                         |
| --------- | -------------------------------------------- |
| `base`    | print、assert、その他の基本関数              |
| `math`    | すべての数学関連の機能                       |
| `string`  | 文字列ライブラリ                             |
| `table`   | テーブル操作および監視関数                   |
| `package` | `require`によるモジュールのロード            |

詳細については、Luaマニュアルの `標準Luaライブラリ` セクションを参照してください。

これらの関数の一部はBNによってオーバーロードされています。詳細については [グローバルオーバーライド](#global-overrides) を参照してください。

### グローバル状態

必要なデータとゲームの実行時状態のほとんどは、グローバルテーブル`game`を介して利用可能です。これには以下のメンバーがあります。

| 変数                        | 説明                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------- |
| `game.current_mod`          | ロード中のMODのID（スクリプト実行時のみ利用可能）                                     |
| `game.active_mods`          | アクティブな世界MODのリスト（ロード順）                                               |
| `game.mod_runtime.<mod_id>` | MODの実行時データ（各MODはIDにちなんだ独自のテーブルを取得）                          |
| `game.mod_storage.<mod_id>` | ゲームのセーブ/ロード時に自動的にセーブ/ロードされるMODごとのストレージ               |
| `game.cata_internal`        | 内部ゲーム目的用です。使用しないでください。                                          |
| `game.hooks.<hook_id>`      | Luaスクリプトに公開されるフック。対応するイベントにより呼び出されます。               |
| `game.iuse.<iuse_id>`       | アイテムファクトリによって認識され、アイテム使用時に呼び出されるアイテム使用関数      |

### ゲームバインディング

ゲームは、様々な関数、定数、および型をLuaに公開しています。関数と定数は、整理の目的で「ライブラリ」に編成されています。型はグローバルに利用可能であり、メンバー関数とフィールドを持つ場合があります。

関数、定数、および型の完全なリストを確認するには、`--lua-doc`コマンドライン引数を付けてゲームを実行してください。これにより、`config`フォルダ内にドキュメントファイル`lua_doc.md` が生成されます。

#### グローバルオーバーライド

一部の関数は、ゲームとの統合を改善するためにグローバルにオーバーライドされています。

| 関数       | 説明                                                                      |
| ---------- | ------------------------------------------------------------------------- |
| print      | debug.log に`INFO LUA` として出力します (既定のLua printをオーバーライド) |
| package    | `data/lua/`、`data/mods/<mod_id>/`からロードします                        |
| dofile     | 無効化されています                                                        |
| loadfile   | 無効化されています                                                        |
| load       | 無効化されています                                                        |
| loadstring | 無効化されています                                                        |

TODO: dofileなどの代替手段

#### フック

フックのリストを確認するには、自動生成されたドキュメントファイルの`hooks_doc`セクションをチェックしてください。そこには、フックIDのリストと、それらが期待する関数シグネチャが示されています。以下のようにフックテーブルに追加することで、新しいフックを登録できます。

```lua
-- preload.luaにて
local mod = game.mod_runtime[game.current_mod]

table.insert(game.hooks.on_game_save, function(...)
  -- これは本質的に前方宣言です。
  -- フックが存在し、game_saveイベントで呼び出されるべきであることを宣言しますが、
  -- すべての可能な引数（引数がない場合でも）と、
  -- 後で宣言する関数からの戻り値を転送します。
  return mod.my_awesome_hook(...)
end)

-- main.luaにて
local mod = game.mod_runtime[game.current_mod]

mod.my_awesome_hook = function()
  -- ここで実際の作業を行います
end
```

#### Item use function

アイテム使用関数は、固有のIDを使用してアイテムファクトリにそれ自体を登録します。アイテムの起動時、それらは以下の例で説明される複数の引数を受け取ります。

```lua
-- preload.luaにて
local mod = game.mod_runtime[ game.current_mod ]
game.iuse_functions[ "SMART_HOUSE_REMOTE" ] = function(...)
  -- これは単なる前方宣言ですが、
  -- これによりJSON内でSMART_HOUSE_REMOTE iuseを使用できるようになります。
  return mod.my_awesome_iuse_function(...)
end

-- main.luaにて
local mod = game.mod_runtime[ game.current_mod ]
mod.my_awesome_iuse_function = function( who, item, pos )
  -- ここで実際の起動効果を実装します。
  -- `who` はアイテムを起動したキャラクターです
  -- `item`はアイテム自体です
  -- `pos` はアイテムの位置です（キャラクターが所持している場合はキャラクターの位置と同じです）
end
```

#### 翻訳関数

MODを他の言語に翻訳可能にするには、`locale` ライブラリにバインドされている関数を介してテキストを取得します。それらのC++の対応物に関する詳細な説明については、[翻訳 API](../explanation/lua_integration.md) を参照してください。

使用例を以下に示します。

```lua
-- シンプルな文字列。
--
-- "Experimental Lab"というテキストはスクリプトによってこのコードから抽出され、
-- 翻訳者向けに利用可能になります。
-- Luaスクリプトが実行される際、この関数は"Experimental Lab"の翻訳を検索し、
-- 翻訳された文字列、または翻訳が見つからなかった場合は元の文字列を返します。
local location_name_translated = locale.gettext( "Experimental Lab" )

-- エラー: gettextは文字列リテラルで呼び出す必要があります。
-- 「Experimental Lab」というテキストが抽出されず、
-- 翻訳者が翻訳作業時に参照できなくなります。
local location_name_original = "Experimental Lab"
local location_name_translated = locale.gettext( location_name_original )

-- エラー: 関数を別の名前でエイリアス化しないでください。
-- 「Experimental Lab」というテキストが抽出されず、
-- 翻訳者が翻訳作業時に参照できなくなります。
local gettext_alt = locale.gettext
local location_name_translated = gettext_alt( "Experimental Lab" )

-- ただし、これは問題ありません。
local gettext = locale.gettext
local location_name_translated = gettext( "Experimental Lab" )

-- 複数形が可能な文字列。
-- 多くの言語には、どれを使用するかに関する複雑なルールを持つ2つ以上の複数形があります
local item_display_name = locale.vgettext( "X-37 Prototype", "X-37 Prototypes", num_of_prototypes )

-- コンテキスト付きの文字列
local text_1 = locale.pgettext("the one made of metal", "Spring")
local text_2 = locale.pgettext("the one that makes water", "Spring")
local text_3 = locale.pgettext("time of the year", "Spring")

-- コンテキストと複数形の両方を持つ文字列。
local item_display_name = locale.vpgettext("the one made of metal", "Spring", "Springs", num_of_springs)

--[[
  一部のテキストが分かりにくく説明を必要とする場合、
  翻訳者を助けるために `~` で示される特別なコメントを配置するのが一般的です。
  このコメントは、関数呼び出しの直上にある必要があります。
]]

--~ このコメントは良好であり、翻訳者に見えるようになります。
local ok = locale.gettext("Confusing text that needs explanation.")

--~ エラー: このコメントはgettext呼び出しから遠すぎ、抽出されません!
local not_ok = locale.
                gettext("Confusing text that needs explanation.")

local not_ok = locale.gettext(
                  --~ エラー: このコメントは間違った場所にあり、抽出されません!
                  "Confusing text that needs explanation."
                )

--[[~
  エラー: 複数行のLuaコメントは翻訳者コメントとして使用できません!
  このコメントは抽出されません!
]] 
local ok = locale.gettext("Confusing text that needs explanation.")

--~ 複数行の翻訳者コメントが必要な場合は、
--~ 2つ以上の単一行コメントを使用してください。
--~ それらは連結され、1つの複数行コメントとして表示されます。
local ok = locale.gettext("Confusing text that needs explanation.")
```
