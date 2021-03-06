# Tuple Library : design decisions rationale

## 名前空間について

その昔、タプルは独立した名前空間にあるべきか、直接 `boost` 名前空間にあるべきかについて、議論がなされた。
一般的な原則として、専門的なライブラリ(*graph* や *python* のような)は独立した入れ子の名前空間に入るべきである一方、ユーティリティ的なライブラリは直接 `boost` 名前空間に入るべきであった。

タプルはそのどちらとも決めがたい。
tupleテンプレートは明らかに汎用的なユーティリティであるが、ライブラリは他にとても多くの名前を、ただtupleテンプレートのために導入する。

タプルは最初は入れ子の名前空間のもとにあった。
議論の結果、タプルの定義は `boost` の直下に移された。
さらに継続した議論の結果、入れ子の名前空間が再び導入された。
最終的な(真剣にそう願う)解答は、今や、全ての定義を名前空間 `::boost::tuples` に置き、最もよく使われる名前を、`::boost` 名前空間にも同様に置こうというものである。

これはusing宣言によって実現された(Dave Abrahamsの提案による):

```cpp
namespace boost {
  namespace tuples {
      ...
    // 全てのライブラリ コード
      ...
  }
  using tuples::tuple;
  using tuples::make_tuple;
  using tuples::tie;
  using tuples::get;
}
```

この配置によって、コンストラクタによる、または `make_tuple` あるいは `tie` 関数によるタプル生成には、名前空間限定子が不要になる。
さらに、タプルを操作する全ての関数はKoenig-lookupで検出される。
唯一の例外は、常に明示的なテンプレート引数とともに呼び出されるため、Koenig-lookupが適用されない `get<N>` 関数である。
そこで、getはusing宣言によって `::boost` 名前空間に持ち上げられた。
こうして、アプリケーション プログラマのためのインタフェイスだけが、実際に名前空間 `::boost` のもとにあることになった。

その他の名前、ライブラリ作者のためのインタフェイス(consリスト、consリストを操作するメタ関数、...)は入れ子の名前空間 `::boost::tuples` に残された。
名前`ignore`、`set_open`、`set_close` および `set_delimiter` はアプリケーション プログラマのインタフェイスに含まれるものと考えられるが、`boost` 名前空間には入っていないことに注意されたい。
その理由は、ありがちな命名なので名前が衝突する危険があるためである。
それに、たぶん、これらはそう頻繁には使われないだろう。

#### 名前空間に魅せられた者たちへ

入れ子の名前空間名 *tuples* は多少の議論を惹き起こした。
最も自然な命名である'tuple'を使わなかった理論的根拠は、tupleテンプレートと同一の名前を持つことを避けたからである。
しかし、boost librariesでは、名前空間名は一般に複数形ではない。
最初は、名前空間とクラスに同じ名前を使っても、深刻なトラブルは報告されなかったので、'tuples'を'tuple'に変えようかとも考えた。
しかし結局トラブルが見つかった。
gccとedgコンパイラが、名前空間名とクラス名が同一の場合、using宣言を拒絶したのである:

```cpp
namespace boost {
  namespace tuple {
    ... tie(...);
    class tuple;
    ...
  }
  using tuple::tie; // ok
  using tuple::tuple; // error
    ...
}
```

とはいえ、グローバル名前空間では同様のusing宣言はokらしい:

```cpp
using boost::tuple::tuple; // ok;
```

## consリストの終端マーク(nil, null_type, ...)

タプルは内部的には `cons` リストとして表現されている:

```cpp
tuple<int, int>
```

は

```cpp
cons<int, cons<int, null_type> >
```

を継承している。

`null_type` はリストの終端を示すマークである。
最初の案では `nil` であったが、この名前はMacOSで使われており、問題を起こすことが予想されたので、代わりに `null_type` が選ばれた。
他には *null_t* および *unit* (SMLにおける空タプル型)も考慮の対象となった。

`null_type` は空タプルの内部表現である: `tuple<>` は `null_type` から派生している。

## 要素のインデックス

要素のインデックスを0から始めるか1からにするかは、徹底的のさらに上をゆく議論がなされ、次の所見が得られた:

- 0-ベースのインデックスは'C++らしい'。
	配列などで前例がある。
- 1-ベースの'name like'インデックスもまた前例がある。
	例えば `bind1st`, `bind2nd`, `pair::first` など。

`get<N>(a)` または `a.get<N>()` ( `a` はタプルで、`N` はインデックスである)のシンタックスでタプル要素にアクセスするのは、前者の種類に属すると考えられる。
そこで、最初の要素のインデックスは0とされた。

1-ベースの'name like'インデックスを、`_1st`, `_2nd`, `_3rd`, ...のような定数を使って提供しようという提案もなされた。
定数に適切な型を選べば、こんな代替構文も可能になるだろう:

```cpp
a.get<0>() == a.get(_1st) == a[_1st] == a(_1st);
```

だが、0-ベース以外のインデックスを提供するのはやめることにした。
次の理由による:

- 0-ベースのインデックスは、全員を満足させないだろうが、一度決定されれば、2種類のインデックスを持った場合より混乱は少ないだろう(配列のためにこのような定数を望むものがいるだろうか?)。
- 他の種類のインデックスを追加しても、ライブラリの利用者に、本当に新しい何か(新機能とか)を提供することにはならない。
- C++の変数と定数の命名規則では、短くて適切なインデックス用の定数(`_1st`, ...のような)を定義する余地は少ない。
	それらは、bindingやlambdaライブラリに譲り、より良い用途に使ってもらおう。
- `a[_1st]` (または `a(_1st)` )構文は魅力的で、もう少しで我々にインデックス用の定数を追加させるところだった。
	しかし、0-ペースの添字はC++に深く深く根ざしている。
	我々は混乱を恐れた。
- こんな定数、追加しようと思えばいつでもできる。

## タプルの比較

比較演算子は辞書的順序を実装する。
他の順序付け、主にdominance (*a &lt; b iff for each i a(i) &lt; b(i)*)も考慮された。
我々は、辞書的順序は、最も自然な数学的順序付けではないが、日々のプログラミングで最も頻繁に必要とされると信じている。

## ストリーム化

タプルのストリーム マニピュレータに設定された文字は、`ios_base::xalloc` によって確保された `long` 型のオブジェクト用の記憶域に格納される。
`long`とストリームの文字型との間の変換には `static_cast` が使われる。
従って、longと相互に変換できない文字型を持つストリームでは、コンパイルに失敗する。

この問題はいつか再考されよう。
可能な解決が二つある:

- 通常の `char` 型のみをタプルの区切り文字として認め、ストリームの文字型との間の変換に `widen` と `narrow` を使う。
	これは常にコンパイルされるだろうが、一部のマニピュレータ呼び出しで、(数個のデフォルトの文字が)期待したのと違う字になるかもしれない。
- ストリームの文字型を格納するのに十分な領域を確保する。
	この意味するところは、区切り文字を保持するメモリを別に確保して、それへのポインタを `ios_base::xalloc` で確保した空間に入れるということである。
	誰かやってくんない?

---

(c) Copyright Jaakko Jarvi 2001.

Japanese Translation Copyright (C) 2003 Yoshinori Tagawa.
オリジナルの、及びこの著作権表示が全ての複製の中に現れる限り、この文書の 複製、利用、変更、販売そして配布を認める。
このドキュメントは「あるがまま」 に提供されており、いかなる明示的、暗黙的保証も行わない。
また、 いかなる目的に対しても、その利用が適していることを関知しない。

