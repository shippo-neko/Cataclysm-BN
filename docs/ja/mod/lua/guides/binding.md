# 新しい型のバインディング

### 内部をバインドせずにドキュメントジェネレータに新しい型を追加する

C++型がドキュメントジェネレータに登録されていない場合、その型は
`<cppval: **gibberish** >`（判読不能な値）として表示されます。この問題を軽減するには、`catalua_luna_doc.h` に `LUNA_VAL( your_type, "YourType" )` を追加します。これにより、ジェネレーターは引数型として `YourType` という文字列を使用するようになります。

### 新しい型をLuaにバインドする

まず、バインディングシステムに新しい型を登録する必要があります。これは、ドキュメントジェネレータがその型を認識し、実行時にその型を含む任意のLuaテーブルをJSONからデシリアライズできるようにするなど、多くの理由から不可欠です。この登録を行わないと、`Type must implement luna_traits<T>`というコンパイルエラーが発生します。

1. `catala_luna_doc.h`で、バインドしたい型を宣言します。 binding an imaginary
   例えば、架空の`horde`型 (`struct`)をバインドする場合、ファイルの冒頭付近に次の一行を追加します。

   ```cpp
   struct horde;
   ```
   複雑なテンプレート型は、関連するヘッダーを実際にインクルードする必要があるかもしれませんが、これはコンパイル時間に大きな影響を与えるため、可能な限り避けてください。

2. 同じファイルで、その型をドキュメントジェネレータに登録します。`horde`の例を
   続けると、次のように記述します。

   ```cpp
   LUNA_VAL( horde, "Horde" );
   ```
   C++の型は様々な命名スタイルを使用しますが、Lua側ではすべて`CamelCase`にする必要があります。

これで、実際の詳細に進むことができます。バインディングは `catalua_bindings*.cpp`
ファイル群に実装されています。 これらはコンパイルの高速化とナビゲーションの容易化のために複数の `.cpp` ファイルに分散されているため、既存の `catalua_bindings*.cpp` ファイルのいずれかに配置するか、独自の同様なファイルを作成しても構いません。また、同じ理由で関数にも分散されています。
`horde` 型を登録し、新しいファイルと新しい関数に配置してみましょう。

1. `catalua_bindings.h` に新しい関数の宣言を追加します:
   ```cpp
   void reg_horde( sol::state &lua );
   ```
2. `catalua_bindings.cpp` の `reg_all_bindings` 内でこの関数を呼び出します:
   ```cpp
   reg_horde( lua );
   ```
3. 新しいファイル `catalua_bindings_horde.cpp` を作成し、以下の内容を記述します:
   ```cpp
   #include "catalua_bindings.h"

   #include "horde.h" // あなたの型が定義されているヘッダーに置き換えてください

   void cata::detail::reg_horde( sol::state &lua )
   {
       sol::usertype<horde> ut =
           luna::new_usertype<horde>(
               lua,
               luna::no_bases,
               luna::constructors <
                   // 実際のコンストラクタをここで定義します
                   horde(),
                   horde( const point & ),
                   horde( int, int )
                   > ()
               );
       
       // 必要なすべてのメンバを登録します
       luna::set( ut, "pos", &horde::pos );
       luna::set( ut, "size", &horde::size );

       // 必要なすべてのメソッドを登録します
       luna::set_fx( ut, "move_to", &horde::move_to );
       luna::set_fx( ut, "update", &horde::update );
       luna::set_fx( ut, "get_printable_name", &horde::get_printable_name );

       // セーブ/ロードの境界を越えてhordeを保持できるように
       // シリアライズ関数またはデシリアライズ関数をを追加します
       reg_serde_functions( ut );

       // 算術演算子、to_string演算子など、さらに多くのものを追加します
   }
   ```
4. これで完了です。あなたの型はLua内で`Horde`という名前で参照可能になり、バインドされたメソッドとメンバを使
   用できます。

### 新しい型をLuaにバインドする（Neovimの正規表現を使用）

クラス/構造体をLuaに手動でバインドするのは非常に面倒な作業になる可能性があるため、クラスをバインドするもう一つの方法は、そのヘッダーファイルを変換することです。
[前述の](#binding-new-type-to-lua)の2番目のセクションのステップ3については、Neovimの組み込み正規表現とC++マクロを使用して、クラスを自動的にバインドすることが可能です。

1. クラス定義のコピーを作成します。
2. 両方を適用します: `%s@class \([^{]\)\+\n*{@private:@` `%s@struct \([^{]\)\+\n*{@public:@`
3. 冒頭のコンストラクタ/不要なメソッドを手動で削除します。
4. すべての `private`/`protected`メソッドを削除します: `%s@\(private:\|protected:\)\_.\{-}\(public:\|};\)@\2`
5. クラス定義の最後にある `};` を削除します。
6. `public` ラベルを削除します: `%s@ *public:\n@`
7. コメントを削除します: `%s@\( *\/\*\_.\{-}\*\/\n\{,1\}\)\|\( *\/\/\_.\{-}\(\n\)\)@\3@g`
8. ベースインデントがゼロになるまでインデントを解除します。
9. メソッド定義を宣言に変換します: `%s@ *{\(}\|\_.\{-}\n^}\)@;`
10. ほとんどのメソッド宣言を一行にまとめます: `%s@\((\|,\)\n *@\1@g`
11. デフォルト値を削除します: `%s@ *= *\_.\{-}\( )\|;\|,\)@\1@g`
12. `overriden`/`static` メソッド/メンバ、および `using`を削除します:
    `%s@.*\(override\|static\|using\).*\n@@g`
13. `template`を削除します: `%s@^template<.*>\n.*\n@@g`
14. `virtual` タグを削除します: `%s@^virtual *@`
15. すべての行がセミコロンで終わっているか確認します: `%s@\([^;]\)\n@\0@gn`
16. 関数の数を数えます: `%s@\(.*(.*).*\)@@nc`
17. 最初に見つかった関数を最後に移動します: `%s@\(.*(.*).*\)\n\(\n*\)\(\_.*\)@\3\1\2`
18. ステップ15のカウントから1を引いた回数だけステップ16を繰り返します。Neovimでは、マッチ数から1を引いた数を入力し、'@'、次に ':', を入力します (例:'217@:'は最後のコマンドを217回繰り返します)。
19. 改行をクリーンアップします: `%s@\n\{3,}@\r\r`
20. メソッドをマクロでラップします: `%s@\(.*\) \+\([^ ]\+\)\((.*\);@SET_FX_T( \2, \1\3 );`
21. メンバをマクロでラップします。最初に影響を与える行を選択するようにしてください:
    `s@.\{-}\([^ ]\+\);@SET_MEMB( \1 );`
22. 以前に複数行だったメソッド宣言を再び複数行に展開します:
    `%s@\(,\)\([^ ]\)@\1\r        \2@g`

これで残りの作業は、このテキストの塊を取得し、Luaバインディングで使用することです。hordeの例を続けると、これらのマクロを使用したコードは次のようになります:

```cpp
#include "catalua_bindings.h"
#include "catalua_bindings_utils.h"

#include "horde.h" // あなたの型が定義されているヘッダーに置き換えてください

void cata::detail::reg_horde( sol::state &lua )
{
    #define UT_TYPE horde
    sol::usertype<UT_TYPE> ut =
    luna::new_usertype<UT_TYPE>(
        lua,
        luna::no_bases,
        luna::constructors <
            // 実際のコンストラクタをここで定義します
            UT_TYPE(),
            UT_TYPE( const point & ),
            UT_TYPE( int, int )
            > ()
       );

    // 必要なすべてのメンバを登録します
    SET_MEMB( pos );
    SET_MEMB( size );

    // 必要なすべてのメソッドを登録します
    SET_FX_T( move_to, ... ); // ...の代わりに、メソッドの型宣言が入ります。
    SET_FX_T( update, ... );
    SET_FX_T( get_printable_name, ... );

    // セーブ/ロードの境界を越えてhordeを保持できるように
    // シリアライズ関数またはデシリアライズ関数をを追加します
    reg_serde_functions( ut );

    // 算術演算子、to_string演算子など、さらに多くのものを追加します
    // ...
    #undef UT_TYPE // #define UT_TYPE horde
}
```

このLuaへのバインディング方法は、テンプレートメソッドのバインディングを欠いており、壊れている可能性があります（コンパイラエラー、リンカーフリーズなど）。したがって、これらのバインディングはデフォルトで壊れていると見なし、わずかな修正/手動での追加のみが必要であると仮定するのが最善です。

### 新しいenumをLuaにバインドする

enumsのバインディングは、型のバインディングと似ています。ここで架空の `horde_type` 列挙型をバインドしてみましょう。

1. enum が明示的に定義されたコンテナ (`: type` それが定義されているヘッダー内の `enum name` の後の`: type`
   部分)を持っていない場合、最初にコンテナを指定する必要があります。例:

   ```diff
     // hordes.h
   - enum class horde_type {
   + enum class horde_type : int {
       animals,
       robots,
       zombies
     }
   ```
2. `catalua_luna_doc.h`に宣言を追加します:
   ```cpp
   enum horde_type : int;
   ```
3. `catalua_luna_doc.h`で以下を使用して登録します:
   ```cpp
   LUNA_ENUM( horde_type, "HordeType" )
   ```
4. このenum が `std::string`との自動変換を実装していることを確認します。詳細については
   `enum_conversions.h` を参照してください。一部のenumはすでに実装していますが、ほとんどはそうではありません。通常、これはヘッダーファイル内であなたのenum`T` に対して
   `enum_traits<T>` を特殊化し、次に `.cpp`ファイルでenumからstringへの変換を行う
   `io::enum_to_string<T>` を定義するだけの問題です。一部の `enum` には `enum_traits<T>`に必要な「最後の」値がありません。その場合は、それを追加する必要があります:

   ```diff
     enum class horde_type : int {
       animals,
       robots,
   -   zombies
   +   zombies,
   +   num_horde_types
     }
   ```

これは、「単調な」enums、つまり0から始まり値をスキップしないenumにのみ機能することに注意してください。上記の例では、`animals` は暗黙の値`0`を持ち`robot`は暗黙の値`1`を持ち、
`zombies` は暗黙の値$2$を持つため、正しく期待される暗黙の値`3`を持つ `num_horde_types`を簡単に追加できます。
5. `catalua_bindings.cpp` の `reg_enums` 関数内でenumフィールドをバインドします:

```cpp
reg_enum<horde_type>( lua );
```

これはステップ4の自動変換を使用するため、JSONとLuaの間で同等の名前を持つことになります。

### 新しい `string_id<T>` または `int_id<T>` をLuaにバインドする

これらをバインドすることは、型`T`自体をバインドすることとは別に行うことができます。

1. まだ行っていない場合は、型 `T` をドキュメントジェネレータに登録します (
   [関連ドキュメント](#adding-new-type-to-the-doc-generator-without-binding-internals)を参照)。
2. ステップ1の `LUNA_VAL` を `LUNA_ID` に置き換えます。
3. 型 `T` が演算子 `<` と `==`を実装していることを確認します。これらは通常、手動で実装するのが簡単であり、
   `catalua_type_operators.h` にあるマクロ `LUA_TYPE_OPS`を使用して半自動的に行うことができます。
4. 型 `T` がnullの `string_id`を持っていることを確認します。存在しない場合は、
   `string_id_null_ids.cpp`に追加します。`T`がクラスとして定義されている場合はマクロ`MAKE_CLASS_NULL_ID` を、それ以外の場合はマクロ `MAKE_STRUCT_NULL_ID` を使用してください。
5. 型 `T` の `string_id` が `obj()` と `is_valid()` メソッドを実装していることを確認します。これらのメソッ
   ドはケースバイケースで実装されます。他の `string_id`を例として確認することが推奨されます。
6. `catalua_bindings_ids.cpp`に、型 `T`が定義されているヘッダーを追加します:
   ```cpp
   #include "your_type_definition.h"
   ```
7. `reg_game_ids` 関数で、以下のように登録します:
   ```cpp
   reg_id<T, true>( lua );
   ```

この `true` は、`string_id<T>` のみをバインドし、`int_id<T>` を気にしない（または実装できない）場合は`false` に置き換えることができます。

この段階で、リンカエラーが発生する可能性があります。例えば、`is_valid()` や `NULL_ID()` メソッドが、様々な理由ですべての文字列IDまたは整数IDに対して実装されていないことに関するエラーです。この場合、これらのメソッドを手動で定義する必要があります。詳細については、`string_id` および `int_id` に関する関連ドキュメントを参照してください。

これで完了です。これで、型 `T` はLuaで `Raw` の接尾辞、`string_id<T>` は
`Id` の接尾辞、 `int_id<T>` は `IntId` の接尾辞を付けて表示されます。例として、
`LUNA_ID( horde, "Horde" )`の場合、次のようになります:

- `horde` -> `HordeRaw`
- `string_id<horde>` -> `HordeId`
- `int_id<horde>` -> `HordeIntId`
  これら3つの間のすべての型変換は、システムによって自動的に実装されます。型`T`の実際のフィールドとメソッドは、通常と同じ方法でLuaにバインドできます。
