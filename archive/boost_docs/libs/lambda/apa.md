# A. Rationale for some of the design decisions

## <a id="sect_why_weak_arity">1. Lambda functor arity</a>

λ式におけるプレースホルダの最大のインデックスによって、結果の関数オブジェクトの引数の数が決定される。
しかし、これは単なる最少の引数の数であって、関数オブジェクトは任意の数の引数を取ることが可能である。
(これらは不要なものであり、無視される。)
二つの bind 式と次の呼出しを考えてみる。

```cpp
bind(g, _3, _3, _3)(x, y, z);
bind(g, _1, _1, _1)(x, y, z);
```

最初の行では、引数のうち `x` と `y` を無視して、次のような呼出しとなる。

```cpp
g(z, z, z)
```

一方、二行目では、引数の `y` と `z` が無視され、次のような呼出しとなる。

```cpp
g(x, x, x)
```

初期のライブラリでは、後者の結果はコンパイル時エラーとなった。
これは基本的に安全性と柔軟性のトレードオフであり、この問題はライブラリの Boost のレビューの時期に大規模に話し合われた。
*厳密な引数の数* チェックの主な利点は、プログラムのエラーを初期の段階で発見できるかもしれないということと、次のように明示的に引数を無視するλ式は簡単に記述できるからである。

```cpp
(_3, bind(g, _1, _1, _1))(x, y, z);
```

このλ式は三つの引数を取る。
カンマ演算子の左側の引数は何もしなく、カンマ演算子は右側の引数の評価結果を返す。
引数の数を厳密にチェックしたとしても、結局は `g(x, x, x)` の呼出しになる。

厳密な引数の数のチェックに反対することの主な利点は、引数を無視することが必要となるのは普通であるので、簡単に記述できるべきであるということと、実際には厳密に引数をチェックしても、対象的なλ式の場合には特に、実際上あまり安全性を得ることができないということである。
例えば、プログラマが `_1 + _2` という式を記述したかったが誤って `_1 + 2` と記述した場合に、厳密に引数の数をチェックすれば、コンパイラはエラーを発見する。
しかし、間違った式が `1 + _2` であった場合には、エラーは発見されない。
さらに、引数を緩和してチェックすれば、実装が少し単純になる。
Boost のレビューの推薦に従って、厳密な引数のチェックは却下された。

