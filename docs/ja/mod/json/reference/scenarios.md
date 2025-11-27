# シナリオ

シナリオは、`type` メンバーが `scenario`に設定された JSON オブジェクトとして指定されます。

```json
{
    "type": "scenario",
    "id": "schools_out",
    ...
}
```

id メンバーは、シナリオの固有IDの必要があります。

以下のプロパティがサポートされています（特に断りのない限り、必須です）。

## `description`

(文字列)

ゲーム内でのシナリオの説明です。

## `name`

(文字列、またはメンバーとして "male" と "female" を持つオブジェクト)

ゲーム内でのシナリオ名です。性別を問わない単一の文字列、または性別固有の名前を持つオブジェクト形式で指定できます。

例:

```json
"name": {
    "male": "Runaway groom",
    "female": "Runaway bride"
}
```

## `points`

(整数)

シナリオのポイントコストです。正の値はポイントを消費し、負の値はポイントを付与します。

## `items`

(省略可能、メンバー "both"、"male"、"female"を持つオブジェクト)

プレイヤーがこのシナリオを選択したときに開始時に所持するアイテムを指定します。キャラクターの性別に基づいて異なるアイテムを指定できます。

各アイテムリストは、アイテム ID の配列である必要があります。ID は複数回出現してもよく、その場合はアイテムが指定された回数分作成されます。

例:

```json
"items": {
    "both": [
        "pants",
        "rock",
        "rock"
    ],
    "male": [ "briefs" ],
    "female": [ "panties" ]
}
```

これはプレイヤーに、 パンツ(pants), 石 (rocks) が２つ、男性用下着としてブリーフ(briefs)、女性用下着としてパンティー (panties)を与えます。

MOD は、既存のシナリオのリストを "add:both" / "add:male" / "add:female" および
"remove:both" / "remove:male" / "remove:female"を介して変更できます。

MOD の変更例:

```json
{
  "type": "scenario",
  "id": "schools_out",
  "edit-mode": "modify",
  "items": {
    "remove:both": ["rock"],
    "add:female": ["2x4"]
  }
}
```

## `surround_groups`

(省略可能、グループIDと密度の数値を含む配列)

シナリオに対する `SUR_START` フラグを置き換えるもので、シナリオの開始地点の周囲に出現するモンスターグループを指定します。

```json
"surround_groups": [ [ "GROUP_BLACK_ROAD", 70.0 ] ],
```

文字列は、周囲のエリアに配置されるモンスターグループのIDを定義し、数値は出現密度です。 70.0 は、元の `SUR_START` の動作値を再現します。

## `flags`

(省略可能、文字列の配列)

フラグのリストです。 (TODO: これらのフラグについてはここに文書化する必要があります。)

Mods can modify this via "add:flags" and "remove:flags".

## `cbms`

(省略可能, 文字列の配列)

キャラクターに移植される CBMのIDの一覧です。

MOD は、"add:CBMs" および "remove:CBMs" を介してこれを変更できます。

## `traits", "forced_traits", "forbidden_traits`

(省略可能、文字列の配列)

特性IDの一覧です。 "forbidden_traits"に含まれる特性は禁止され、キャラクター作成時に選択できません。 "forced_traits" に含まれる特性は、キャラクターに自動的に追加されます。
"traits" に含まれる特性は、初期特性でなくても、選択可能になります。

MOD は、"add:traits" / "add:forced_traits" / "add:forbidden_traits" および
"remove:traits" / "remove:forced_traits" / "remove:forbidden_traits"を介してこれを変更できます。

## `allowed_locs`

(省略可能、文字列の配列)

このシナリオを使用する際に選択できる開始位置 ID の一覧です (start_locations.jsonを参照)。

## `start_name`

(文字列)

開始地点として表示される名前です。シナリオが複数の開始地点を許可しているが、シナリオ説明として一覧表示できない場合に役立ちます。例として、シナリオの開始地点として荒野のどこかであった場合、森(forest)と野原(fields)が含まれる可能性があります。 しかし"start_name" は単に荒野となる場合があります。

## `professions`

(省略可能、文字列の配列)

このシナリオで開始したときに選択できる専門職 (profession)の一覧です。最初の項目が初期設定の専門職となります。設定されていない場合は、すべての専門職が選択可能になります。

## `map_special`

(省略可能、文字列)

開始位置にマップスペシャルを追加します。可能なスペシャルについては
[json_flags](json_flags.md) を参照してください。

## `missions`

(省略可能、文字列の配列)

ゲーム開始時のプレイヤーに割り当てられるミッションIDの一覧です。ORIGIN_GAME_START を持つミッションのみが許可されます。複数のミッションが割り当てられた場合、一覧内の最後尾のミッションが有効なミッションとなります。
