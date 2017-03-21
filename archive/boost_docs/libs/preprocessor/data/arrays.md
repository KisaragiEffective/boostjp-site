# Arrays

*配列*は二要素の*タプル*からなるデータ構造である。
その第一要素は*配列*の要素数であり、第二要素は*配列*の要素となる別の*タプル*である。
例えば、

```cpp
(3, (a, b, c))
```

は `a`、`b`、`c` の三要素からなる*配列*である。

*配列*の最も重要な長所は、それが自分自身のサイズを保持していることである。
このおかげで、要素へのアクセスにはサイズを必要としない。
必要なことは、そのインデックスに要素が存在することだけだ。

この構造により、**マクロ引数は可変長となることができ(?)**、ユーザーが自力でサイズ変化を追尾することをせずとも、データのサイズを変更できる。

*配列*の要素は `BOOST_PP_ARRAY_ELEM`、サイズは `BOOST_PP_ARRAY_SIZE` により展開され、さらに*配列*は `BOOST_PP_ARRAY_DATA` により、より基本的なデータ構造である*タプル*に変換できる。

## Primitives

- [`BOOST_PP_ARRAY_DATA`](../ref/array_data.md)
- [`BOOST_PP_ARRAY_ELEM`](../ref/array_elem.md)
- [`BOOST_PP_ARRAY_SIZE`](../ref/array_size.md)
