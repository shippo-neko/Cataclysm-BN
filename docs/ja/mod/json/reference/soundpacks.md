# サウンドパック

サウンドパックは `data/sound` ディレクトリにインストールできます。これは、少なくとも `soundpack.txt`というファイルを含むサブディレクトリの必要があります。また、任意の数の JSON ファイルを含めることができ、任意の数のsound_effectまたは playlist を追加できます。

## soundpack.txt のフォーマット

`soundpack.txt` ファイルのフォーマットには、NAME と VIEW の 2 つの値が必要です。NAME は固有値の必要があります。VIEW の値はオプションメニューに表示されます。`#` で始まるすべての行はコメント行です。

```
#Basic provided soundpack
#Name of the soundpack
NAME: basic
#Viewing name of the soundpack
VIEW: Basic
```

## JSON フォーマット

### サウンドエフェクト

サウンドエフェクトは、以下のようなフォーマットを含めることができます。

```json
[
  {
    "type": "sound_effect",
    "id": "menu_move",
    "volume": 100,
    "files": [
      "nenadsimic_menu_selection_click.wav"
    ]
  },
  {
    "type": "sound_effect",
    "id": "fire_gun",
    "volume": 90,
    "variant": "bio_laser_gun",
    "files": [
      "guns/energy_generic/weapon_fire_laser.ogg"
    ]
  }
]
```

多様性の追加: 特定の`id`の`variant` に対して複数の `files` が定義されている場合、その`variant` が再生される際にファイルがランダムに選択されます。

volume キーは0から100の範囲で指定できます。

Cataclysm には、サウンドに影響を与える、ユーザー制御可能な独自の音量設定(0 から 128 の範囲、既定値は 100) があります。これは、既定値の音量設定は、Cataclysm が再生するすべてのサウンドが最大音量の約 78%で再生されることを意味しています。外部のオーディオエディタでサウンドを調整している場合、Cataclysm の既定の音量設定では、エディタで聞こえるよりも音量が小さくなることを想定しておいてください。

### SFXのプリロード

プリロード用のサウンドエフェクトを、以下のようなフォーマットに含めることが出来ます。

```json
[
  {
    "type": "sound_effect_preload",
    "preload": [
      { "id": "fire_gun", "variant": "all" },
      { "id": "environment", "variant": "daytime" },
      { "id": "environment" }
    ]
  }
]
```

`"variant": "all"` は特別に扱われ、指定IDのすべてのバリアントをロードします。

> [!WARNING]
>
> `"variant": "all"` はアルゴリズムの最適化がなされていないため、ゲームのロード時間が長くなります。

`"variant"` が省略された場合、`"default"` になります。

### プレイリスト

プレイリストは、以下のようなフォーマットを含めることができます。

```json
[
  {
    "type": "playlist",
    "playlists": [
      {
        "id": "title",
        "shuffle": false,
        "files": [
          {
            "file": "Dark_Days_Ahead_demo_2.wav",
            "volume": 100
          },
          {
            "file": "cataclysmthemeREV6.wav",
            "volume": 90
          }
        ]
      }
    ]
  }
]
```

各サウンドエフェクトは、IDとバリアントによって識別されます。 JSON ファイル内に存在しないバリアントでサウンドエフェクトが再生された場合、"default"バリアントが存在すれば、かわりに再生されます。サウンドエフェクトのファイル名は、サウンドパックディレクトリからの相対パスです。したがって、ファイル名が "sfx.wav" に設定され、サウンドパックが `data/sound/mypack` にある場合、ファイルは `data/sound/mypack/sfx.wav` に配置されなければなりません。

## サウンドエフェクトIDリスト

サウンドエフェクトIDとバリアントの完全なリストを以下に示します。
リスト内の各行は、以下のようなフォーマットを持っています。

`id variant1|variant2`

idにはサウンドエフェクトのIDを記述し、その後に | で区切られたバリアントのリストが続きます。
バリアントが省略された場合、"default" が設定されます。バリアントがリテラルな文字列ではなく
変数を表す場合、`<` と `>`で囲まれます。例として、`<furniture>` は任意の有効な家具IDのプレースホルダーです。

    # ドアの開閉

- `open_door default|<furniture>|<terrain>`
- `close_door default|<furniture>|<terrain>`

  # 破壊の試みと結果、いくつかの特別なものと家具/地形固有のものがあります。
- `bash default`
- `smash wall|door|door_boarded|glass|swing|web|paper_torn|metal`
- `smash_success hit_vehicle|smash_glass_contents|smash_cloth|<furniture>|<terrain>`
- `smash_fail default|<furniture>|<terrain>`

  # 近接戦闘音
- `melee_swing default|small_bash|small_cutting|small_stabbing|big_bash|big_cutting|big_stabbing`
- `melee_hit_flesh default|small_bash|small_cutting|small_stabbing|big_bash|big_cutting|big_stabbing|<weapon>`
- `melee_hit_metal default|small_bash|small_cutting|small_stabbing|big_bash|big_cutting|big_stabbing!<weapon>`
- `melee_hit <weapon>` # 素手攻撃には武器ID「null」を使用してください。

  # 銃器/遠隔武器音
- `fire_gun <weapon>|brass_eject|empty`
- `fire_gun_distant <weapon>`
- `reload <weapon>`
- `bullet_hit hit_flesh|hit_wall|hit_metal|hit_glass|hit_water`

  # 環境音 SFX、ここでは分かりやすさのためにセクションに分けられています。
- `environment thunder_near|thunder_far`
- `environment daytime|nighttime`
- `environment indoors|indoors_rain|underground`
- `environment <weather_type>` # 例:
  `WEATHER_DRIZZLE|WEATHER_RAINY|WEATHER_THUNDER|WEATHER_FLURRIES|WEATHER_SNOW|WEATHER_SNOWSTORM`
- `environment alarm|church_bells|police_siren`
- `environment deafness_shock|deafness_tone_start|deafness_tone_light|deafness_tone_medium|deafness_tone_heavy`

  # その他環境音
- `footstep default|light|clumsy|bionic`
- `explosion default|small|huge`

  # 周囲の危険度状況音楽、多数のゾンビを見たときの環境音です。
- `danger_low`
- `danger_medium`
- `danger_high`
- `danger_extreme`

  # チェーンソーパック
- `chainsaw_cord     chainsaw_on`
- `chainsaw_start    chainsaw_on`
- `chainsaw_start    chainsaw_on`
- `chainsaw_stop     chainsaw_on`
- `chainsaw_idle     chainsaw_on`
- `melee_swing_start chainsaw_on`
- `melee_swing_end   chainsaw_on`
- `melee_swing       chainsaw_on`
- `melee_hit_flesh   chainsaw_on`
- `melee_hit_metal   chainsaw_on`
- `weapon_theme      chainsaw`

  # モンスターの死亡と噛みつき攻撃
- `mon_death zombie_death|zombie_gibbed`
- `mon_bite bite_miss|bite_hit`

- `melee_attack monster_melee_hit`

- `player_laugh laugh_f|laugh_m`

  # プレイヤーの移動 SFX
  重要: `plmove <terrain>` は、既定の `plmove|walk_<what>` (`|barefoot`を除く)
  よりも優先されます。例:`plmove|t_grass_long` が定義されている場合、全ての草地の既定音です。
  `plmove|walk_grass` よりも優先して再生されます。
- `plmove <terrain>|<vehicle_part>`
- `plmove walk_grass|walk_dirt|walk_metal|walk_water|walk_tarmac|walk_barefoot|clear_obstacle`

  # 疲労
- `plmove fatigue_m_low|fatigue_m_med|fatigue_m_high|fatigue_f_low|fatigue_f_med|fatigue_f_high`

  # プレイヤーの負傷音
- `deal_damage hurt_f|hurt_m`

  # プレイヤーの死亡とゲーム終了音
- `clean_up_at_end game_over|death_m|death_f`

  # 様々なバイオニック音
- `bionic elec_discharge|elec_crackle_low|elec_crackle_med|elec_crackle_high|elec_blast|elec_blast_muffled|acid_discharge|pixelated`
- `bionic bio_resonator|bio_hydraulics|`

  # 様々なツール/トラップの使用音 (関連する一部の地形/備品を含む)
- `tool alarm_clock|jackhammer|pickaxe|oxytorch|hacksaw|axe|shovel|crowbar|boltcutters|compactor|gaspump|noise_emitter|repair_kit|camera_shutter|handcuffs`
- `tool geiger_low|geiger_medium|geiger_high`
- `trap bubble_wrap|bear_trap|snare|teleport|dissector|glass_caltrop|glass`

  # 様々な活動音
- `activity burrow`

  # 楽器、`_bad` は演奏に失敗したときに使用されます
- `musical_instrument <instrument>`
- `musical_instrument_bad <instrument>`

  # 様々な叫び声と悲鳴
- `shout default|scream|scream_tortured|roar|squeak|shriek|wail|howl`

  # スピーチ、現在はアイテム ID またはモンスター ID にリンクされているか、特別な `NPC` または `NPC_loud`です。
  # TODO:speech.json の完全な音声化。
- `speech <item_id>` # 例: talking_doll, creepy_doll, Granade,
- `speech <monster_id>` # 例: eyebot, minitank, mi-go, 多くのロボット
- `speech NPC_m|NPC_f|NPC_m_loud|NPC_f_loud` # NPCのための特殊なもの。
- `speech robot` # 機械、ロボットのための特殊音声。

  # ラジオの雑音
- `radio static|inaudible_chatter`

  # 様々な発生源のハミング音
- `humming electric|machinery`

  # (燃えている) 火に関連する音
- `fire ignition`

  # 車両音 - エンジンとその他の動作中の部品
  # 注釈: 特定のオプションが定義されていない場合、既定設定が実行されます。
- `engine_start <vehicle_part>` # 注釈: 特定のエンジンの始動音 (任意の
  engine/motor/steam_engine/paddle/oar/sail/etc.のID)。
- `engine_start combustion|electric|muscle|wind` # 既定のエンジン始動音グループ。
- `engine_stop <vehicle_part>` # 注釈: 特定のエンジンの停止音 (任意の
  engine/motor/steam_engine/paddle/oar/sail/etc. のID)。
- `engine_stop combustion|electric|muscle|wind` # 既定のエンジン停止音グループ。

  # 注釈: エンジン内部音は、車両速度に応じてピッチが動的にシフトされます。
  # 専用チャンネルを持つ環境ループ音です。
- `engine_working_internal <vehicle_part>` # 注釈: 車両内部で聞こえるエンジンの動作音。
- `engine_working_internal combustion|electric|muscle|wind` # 既定のエンジン動作 (内部) グループ。

  # 注釈: エンジン外部音の音量とパン（左右定位）は、車両からの距離と角度に応じて動的にシフトされます。
  # 特定の距離で聞こえる音量は、エンジンの `noise_factor` とエンジンへの負荷 (`vehicle::noise_and_smoke()`を参照)にリンクしています。
  # 専用チャンネルを持つ環境ループ音です。
  # これはシングルチャンネルソリューションです (TODO:聞こえるすべての車両に対するマルチチャンネル化)。 最も大きな音の車両が選択されます。
  # ここではピッチシフトはありません (必要性が生じた場合に導入される可能性があります)。
- `engine_working_external <vehicle_part>` # 注釈: 車両外部で聞こえるエンジンの動作音。
- `engine_working_external combustion|electric|muscle|wind` # 既定のエンジン動作(外部)グループ。

  # 注釈: gear_up/gear_down はピッチ操作によって自動的に行われます。
  # ギアシフトは最大安全速度に依存し、 前進 6速、ギア 0 = ニュートラル、
  # ギア -1 = リバースという仮定に基づいて機能します。
- `vehicle gear_shift`

- `vehicle engine_backfire|engine_bangs_start|fault_immobiliser_beep|engine_single_click_fail|engine_multi_click_fail|engine_stutter_fail|engine_clanking_fail`
- `vehicle horn_loud|horn_medium|horn_low|rear_beeper|chimes|car_alarm`
- `vehicle reaper|scoop|scoop_thump`

- `vehicle_open <vehicle_part>` # 注釈: ドア、トランク、ハッチなどのID。
- `vehicle_close <vehicle part>`

  # その他雑多な音
- `misc flashbang|flash|shockwave|earthquake|stairs_movement|stones_grinding|bomb_ticking|lit_fuse|cow_bell|bell|timber`
- `misc rc_car_hits_obstacle|rc_car_drives`
- `misc default|whistle|airhorn|horn_bicycle|servomotor`
- `misc beep|ding|`
- `misc rattling|spitting|coughing|heartbeat|puff|inhale|exhale|insect_wings|snake_hiss` # 主に生物的なノイズ。
