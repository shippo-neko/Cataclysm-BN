# Lua スクリプティング クックブック

ここでは、Lua APIに慣れ親しみ、その使い方を学ぶのに役立つコードスニペットをいくつか紹介します。 これらの例をテストするには、バックティック `` ` `` キーを押してゲーム内のLuaコンソールを開き、コードを貼り付けてください。

## アイテム

### インベントリ内の装備品のリストを取得する

```lua
local you = gapi.get_avatar()
local items = you:all_items(false)

for _, item in pairs(items) do
  local status = ""
  if you:is_wielding(item) then
    status = "wielded: "
  elseif you:is_wearing(item) then
    status = "worn: "
  end
  print(status .. item:tname(1, false, 0))
end
```

<details>
<summary>Example output</summary>

```
wielded: smartphone
worn: bra
worn: panties
worn: pair of socks
worn: jeans
worn: long-sleeved shirt
worn: pair of sneakers
worn: messenger bag
worn: wrist watch
pocket knife
matchbook
clean water (plastic bottle)
clean water
```

</details>

## モンスター

### プレイヤーの近くに犬をスポーンさせる

```lua
local avatar = gapi.get_avatar()
local coords = avatar:get_pos_ms()
local dog_mtype = MtypeId.new("mon_dog_bcollie")
local doggy = gapi.place_monster_around(dog_mtype, coords, 5)
if doggy == nil then
    gdebug.log_info("Could not spawn doggy :(")
else
    gdebug.log_info(string.format("Spawned Doggy at %s", doggy:get_pos_ms()))
end
```

## 戦闘

### 戦闘テクニックが使用されたときにその詳細を出力する

まず、関数を定義します。

```lua
on_creature_performed_technique = function(params)
  local char = params.char
  local technique = params.technique
  local target = params.target
  local damage_instance = params.damage_instance
  local move_cost = params.move_cost
  gdebug.log_info(
          string.format(
                  "%s performed %s on %s (DI: %s , MC: %s)",
                  char:get_name(),
                  technique.name,
                  target:get_name(),
                  damage_instance:total_damage(),
                  move_cost
          )
  )
end
```

次に、このフックを一度だけ関数に接続します。

```lua
table.insert(game.hooks.on_creature_performed_technique, function(...) return on_creature_performed_technique(...) end)
```

<details>
<summary>Example output</summary>

```
Ramiro Waters performed Power Hit on zombie (DI: 27.96 , MC: 58)
```

</details>
