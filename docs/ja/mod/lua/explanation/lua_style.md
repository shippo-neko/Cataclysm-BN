# Lua スタイルガイド

Luaエコシステムはプロジェクトへの新しい追加であるため、現在はフォーマットのガイドラインのみを定めています。

## フォーマット

Luaファイルは、[dprint-plugin-stylua](https://github.com/RubixDev/dprint-plugin-stylua)をデフォルト設定で用いてフォーマットします。

Luaファイルをフォーマットするには、以下を実行します:

```sh
deno task dprint fmt
```

### VSCodeでのLuaファイルのフォーマット

1. [dprint vscode 拡張機能](https://marketplace.visualstudio.com/items?itemName=dprint.dprint)をインストールします。
2. `.vscode/settings.json`に以下の行を追加します。

```json
{
  "[lua]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "dprint.dprint"
  }
}
```

これで、Luaファイルを保存する際に自動的にフォーマットが適用されます。
