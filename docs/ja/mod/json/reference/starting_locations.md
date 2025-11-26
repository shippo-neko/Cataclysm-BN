# 開始位置

開始位置は、JSONファイル内の "type" に "start_location"を指定することで設定できます。

```json
{
    "type": "start_location",
    "id": "field",
    "name": "An empty field",
    "target": "field",
    ...
}
```

"id" は、開始位置の固有IDを指定する必要があります。

以下の要素がサポートされています（いずれも特に断りのない限り、必須です）：

## `name`

(文字列)

ゲーム内での、この開始位置の表示名です。

## `target`

(文字列)

開始位置となるオーバーマップ地形タイプのIDです (overmap_terrain.jsonを参照) 。ゲームは、その地形属性を持つ場所の中からランダムな地点を選択します。

## `flags`

(省略可能, 文字配列)

任意のフラグを指定します。MODは "add:flags" や "remove:flags"を介してフラグを変更できます。TODO: これらについては文書化が必要です。
