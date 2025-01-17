---
title: SML♯のREPLで速習SML
---

パターンマッチの話題に入る前に、SMLの基本的な挙動を確認しておきましょう。幸いSML♯にはREPL（対話的実行環境）があるので、これを利用して以降の解説に必要な最低限の情報をまとめます。

シェルから`smlsharp`としてSML♯を起動すると、下記のようにREPLが立ち上がります。 `# ` がSML♯のプロンプト記号です。

```console
$ smlsharp ⏎
SML# 3.4.0 (2017-08-31 19:31:44 JST) for x86_64-pc-linux-gnu with LLVM 3.7.1
# 
```

プロンプトに続けていくつか式を入力してみましょう。数値やブール値は見たままです。下記は `1` 、 `true` 、 `1 + 3` を入力したところです。

```console
# 1; ⏎
val it = 1 : int
# true; ⏎
val it = true : bool
# 1 + 3; ⏎
val it = 4 : int
```

REPLに式を入力するときは、末尾に `;` が必要です。`;`がないと、`1` を入力して改行したときに続けて `+ 2` と打って計算をしたいのか、それとも `1` を入力したいだけなのか、区別がつかないからです。

## 変数

続いて変数です。変数は `val 〈名前〉 = 〈式〉` で定義します。下記ではトップレベルの変数 `x` を定義しています。

```console
# val x = 1 * 2; ⏎
val x = 2 : int
# x; ⏎
val it = 2 : int
```

SMLの変数はイミュータブルであり、再代入はありません。 `=` は等価比較にも割り当てられているので、Cのような言語の感覚で次のように書くとブール値が返ります。

```console
# x = x; ⏎
val it = true : bool
# x = x + 1; ⏎
val it = false : bool
```

トップレベルの変数だけでなくローカルの変数もあります。ローカルの変数を定義するには `let 〈宣言〉 in 〈式〉 end` という構文を使い、 `let` の中でだけ有効な宣言をしてから `〈式〉` を評価します。下記の例では、ローカルで `x = 3` と宣言してから `x * x` を計算しています。前の例から続けて実行していると、トップレベルにも `x` が定義されている状態ですが、近いスコープの値が使われるので、この式の結果は `9` になります。

```console
# let val x = 3 in x * x end; ⏎
val it = 9 : int
```

しかし、 `let` の束縛はそのスコープ内でしか有効でないので、トップレベルに戻ると `x` は `2` に戻ります。

```console
# x; ⏎
val it = 2 : int
```

ここまでREPLの応答のスタイルについて何も触れませんでしたが、SML♯のREPLの応答が単なる `2` でなく `val it = 2 : int` という表示になっているのは、はじめて見ると不思議に思えるかもしれません。SMLの規格ではREPLの応答について何も決められていませんが、多くの処理系はこのようなスタイルになっているようです。

REPLの出力表示が示しているように、入力した式の返り値は、変数 `it` に束縛されています。したがって、以下のように、REPLで `it` を使えます。

```console
# it; ⏎
val it = 2 : int
# it * it + it; ⏎
val it = 6 : int
# it - 2; ⏎
val it = 4 : int
```

また、REPLの応答の末尾にある `: int` や `: bool` は型を表します。SMLは静的型付言語ですが、型推論が備わっているので、処理系側でコンパイル時に型を決定し、このように型を付けてくれます（決定できなかったらコンパイルエラーになります）。

## 関数

関数は `fun 〈名前〉 〈引数1〉 〈引数2〉 … 〈引数n〉 = 〈式〉` という構文で定義します。C-like syntaxしか見たことがないと、引数の指定で括弧やカンマを使わないことに少し慣れが必要かもしれません。

```console
# fun hello name = print ("Hello " ^ name ^ "!\n"); ⏎
val hello = fn : string -> unit
```

関数の型は `〈引数の型〉 -> 〈返り値の型〉` です。引数が増えると、 `〈引数1の型〉 -> 〈引数2の型〉 -> … -> 〈引数nの型〉 -> 〈返り値の型〉` のようになります。上記の例では、関数`hello`が`string`型から`unit`型への関数であると、処理系が型推論をしてくれています。`string`型は文字列です。`unit`は、SMLでは、何も意味のある値を返さないときに使われる型です。振る舞いは違いますが、役割としてはCの `void` に似ています。

関数を呼び出すときも括弧は不要です。

```console
# hello "κeen"; ⏎
Hello κeen!
val it = () : unit
```

## タプル

次はタプル（tuple、組）です。タプルを使うと、いくつかのデータ型をまとめられます。

```console
# (4, "κeen"); ⏎
val it = (4,"κeen") : int * string
```

上記の例では、`int` 型の値 `4` と `string` の値 `"κeen"` のタプルを作っています。このタプルの型は `int * string` で、各要素の型を組み合わせたものになっています。

SMLのタプルは、Cのような言語における構造体に近い存在だといえます[^tapl]。

[^tapl]: といっても、タプルでは要素に名前を付けられないので、その点でCの構造体よりも表現力は劣ります。SMLには各フィールドに名前が付く「レコード」もあり、その点ではこちらのほうがCの構造体には近いといえます。

## リスト

リスト型は、同じ型の値を複数並べたものです。

```console
# val y = [1, 2, 3]; ⏎
val y = [1, 2, 3] : int list
```

右結合の演算子 `::` で、リストの先頭に要素を追加した新しいリストを作成できます。

```console
# 0 :: y; ⏎
val it = [0, 1, 2, 3] : int list
```

補足すると、 `[〈要素〉, ..., 〈要素〉]` という構文は、実際には空のリスト `nil` とそれへの要素の追加 `::` の略記です。先ほどの `y` の定義は以下と同等です。

```console
# val y = 1 :: 2 :: 3 :: nil; ⏎
val y = [1, 2, 3] : int list
```

## 例外

最後に、制御構造のひとつとして例外があります。たとえば、ゼロ除算をすると以下のように例外が送出されます。

```console
# 1 div 0; ⏎
uncaught exception Div at (interactive):11
```

送出された例外は `handle 〈捕捉する例外〉 => 〈式〉` で捕捉できます。例外が起きなければ `handle` は使われません。

```console
# 1 div 0 handle Div => 100; ⏎
val it = 100 : int
# 1 div 1 handle Div => 100; ⏎
val it = 1 : int
```

## ファイルに書いたSML♯のコードをコンパイルするときの注意

本節ではSML♯のREPLを使ってSMLの基礎を説明しましたが、以降の例ではファイルにコードを書いてそれをコンパイルします。引き続きSML♯を利用する場合には、少しだけそのための作法が必要なので、ここでそれを補足しておきます。

まず、 `algebraic_data_types.smi` というファイルを用意し、以下のように記述してください。（ファイル名の末尾は `smi` です。 `sml` ではないので間違えなようにしてください。）

``` sml
_require "basis.smi"
```

上記では、標準ライブラリを使うことを宣言しています。この宣言を`algebraic_data_types.smi`（末尾は **`smi`**）にしたうえで、SMLのコードを`algebraic_data_types.sml`（末尾は **`sml`**）に書いていきます。

ファイル`algebraic_data_types.sml`に書いたコードは、以下のようにしてコンパイルして実行します。

```console
$ smlsharp -o algebraic_data_types algebraic_data_types.sml ⏎
$ ./algebraic_data_types ⏎
```

あるいは、REPLからファイルを読み込みたいなら `use` が使えます。

```console
# use "algebraci_data_types.sml" ⏎
```

その他、詳しくはSML♯のマニュアル[^smlsharpman]を参照してください。

[^smlsharpman]: [https://www.pllab.riec.tohoku.ac.jp/smlsharp/docs/3.4.0/ja/Ch5.S5.xhtml](https://www.pllab.riec.tohoku.ac.jp/smlsharp/docs/3.4.0/ja/Ch5.S5.xhtml)
