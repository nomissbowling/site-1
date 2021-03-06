# Regex++, Regular Expression Syntax

*Copyright (c) 1998-2001*

*Dr John Maddock*

*Permission to use, copy, modify, distribute and sell this software and its documentation for any purpose is hereby granted without fee, provided that the above copyright notice appear in all copies and that both that copyright notice and this permission notice appear in supporting documentation.
Dr John Maddock makes no representations about the suitability of this software for any purpose.
It is provided "as is" without express or implied warranty.*

---

### <a id="syntax">Regular expression syntax</a>

この章では、このライブラリで使われる正規表現文法について述べる。
これは、プログラマーズガイドであって、プログラムの中でユーザに与えられる実際の文法は正規表現の翻訳の間に使われるフラグに依存する。

*Literals*

以下のものを除く全ての文字はリテラルである: ".", "|", "\*", "?", "+", "(", ")", "{", "}", "[", "]", "\^", "\$", "\\" 。
これらの文字列は、"\\" に続いたときはリテラルである。
リテラルはそれ自身にマッチするか、または、 `traits_type::translate()` の結果にマッチする文字である。
`traits_type` は `reg_expression` クラスの特性テンプレートパラメータである。

*ワイルドカード*

ドット文字 "." はあらゆる1文字にマッチする。
例外: `match_not_dot_null` がマッチアルゴリズムに渡されたときは、ドットはヌル文字にはマッチしない。
`match_not_dot_newline` がマッチアルゴリズムに渡されたときは、ドットは改行文字にはマッチしない。

*繰り返し*

繰り返しは任意の回数繰り返される正規表現である。
"\*" が続く正規表現はゼロを含む何回の繰り返しも可能である。
"+" が続く正規表現は1回以上の何回の繰り返しも可能である。
もし正規表現が `regbase::bk_plus_qm` フラグ付きで翻訳されるなら、 "+" は通常の文字であり、 "\\+" が1回以上の繰り返しを表す。
"?" が続く正規表現は0回か1回の繰り返しである。
もし正規表現が `regbase::bk_plus_qm` フラグ付きで翻訳されるなら、 "?" は通常の文字であり、 "\\?" が0回か1回の繰り返しを示す。
繰り返しの最小回数と最大回数を明示する必要があるなら、範囲指定子 "{}" が使われる。
"a{2}" は、ちょうど2回だけ繰り返される文字 "a" である。
"a{2,4}" は2回以上4回以下繰り返される文字 "a" であり、 "a{2,}" は2回以上上限なく繰り返される文字 "a" である。
{}の中に空白があってはいけないこと、最小回数と最大回数の指定に上限がないことに注意せよ。
正規表現が `regbase::bk_braces` フラグ付で翻訳されるとき、 "{" と "}" は通常の文字であり、代わりに "\\{" と "\\}" が範囲指定に使われる。
全ての繰り返し表現は、可能な限り短く、前方の子表現を参照する。
例えば、1文字、文字集合、 "()" で囲まれた子表現を参照する。

例:

"ba\*" は "b", "ba", "baaa" などの全てにマッチする。

"ba+" は "ba" や "baaaa" にマッチするが、 例えば "b" にはマッチしない。

"ba?" は "b" と "ba" にマッチする。

"ba{2,4}" は "baa" と  "baaa" と "baaaa" にマッチする。

*貪欲でない繰り返し*

「拡張」正規表現構文が(デフォルトで)使われるときはいつでも、繰り返しのあとに '?' を付け足すことで、貪欲でない繰り返しが可能である。
貪欲でない繰り返しとは、 可能な限り *もっとも短い* 文字列にマッチするものである。

例えば、一組の html タグのマッチには次のようにすることができる:

`<\s*tagname[^>]*>(.*?)<\s*/tagname\s*>`:

この場合、 `\$1` はタグに挟まれたテキストを保持する。
これが、可能な限り最も短い文字列である。

*丸括弧*

丸括弧はふたつの目的で使われる。
一つは子表現をまとめることであり、もう一つはマッチを生成したものを印付けることである。
例えば、正規表現 "(ab)\*" は文字列 "ababab" の全てにマッチする。
マッチアルゴリズム [`regex_match`](template_class_ref.md#query_match) と [`regex_search`](template_class_ref.md#reg_search) はそれぞれ、何がマッチを引き起こしたかを報告する、 [`match_results`](template_class_ref.md#reg_match) のインスタンスを得る。
これらの関数から抜けるとき、 [`match_results`](template_class_ref.md#reg_match) 正規表現全体が何とマッチしたか、そしてそれぞれの子表現が何とマッチしたかについての情報を保持している。
上の例では、 `match_results[1]` はマッチした文字列の最後の "ab" を示す一組のイテレータを保持している。
子表現がヌル文字列にマッチすることも許されている。
もし子表現がどの部分ともマッチしなければ、例えばそれが、マッチしなかった選択肢の一部なら、その子表現に対して返される両方のイテレータは、入力文字列の終端を指していて、 その子表現に対する `matched` パラメータは `false` である。
子表現は1で始まり左から右に数えられる。
子表現 0 は正規表現全体である。

*印付けをしない丸括弧*

丸括弧で子表現をグループ化したいが、丸括弧に子表現の印を作って欲しくないようなときは、印付けをしない丸括弧 (?:expression) を使うことができる。
例えば、 次の正規表現は子表現を作らない: (訳注: 子表現を作らないいうよりは、子表現に印付けをしないということ)

`"(?:abc)*"`

*前方先読み宣言*

これにはふたつの形式がある。
一つは肯定的前方先読み宣言であり、もう一つは否定的前方先読み宣言である:

"(?=abc)" はゼロ文字列に正規表現 "abc" が続くときのみ、そのゼロ文字列にマッチする。

"(?!abc)" はゼロ文字列に正規表現 "abc" 続かないときのみ、そのゼロ文字列にマッチする。

*選択*

選択は、正規表現がある子表現か、別の子表現のいずれかにマッチするときに起こる。
それぞれの選択肢は "|" か、 `regbase::bk_vbar` フラグが設定されているなら "\\|" または `regbase::newline_alt` フラグが設定されていれば改行文字によって区切られる。
それぞれの選択肢は先行する可能な限り大きな子表現である。
これは繰り返し演算子とは逆の動作である。

例:

"a(b|c)" は "ab" または "ac" にマッチする。

"abc|def" は "abc" または "def" にマッチする。

*集合*

集合は、集合の要素であるあらゆる1文字とマッチすることができる文字集合である。
文字集合は "[" と "]" で囲まれていて、リテラル、文字範囲、文字クラス、照合要素、等価クラスを含むことができる。
"\^" で始まる文字集合の宣言は、それに続く要素以外を含む。

例:

文字リテラル:

"[abc]" は "a" , "b" , "c" のいずれかとマッチする。

"[\^abc]" は "a", "b", or "c" 以外のあらゆる文字にマッチする。


文字範囲:

"[a-z]" は "a" から "z" の範囲にあるあらゆる文字にマッチする。 

"[\^A-Z]" は "A" から "Z" の範囲にある文字以外の、あらゆる文字にマッチする。

文字範囲は、ロケールに強く依存することに注意せよ: これは、その範囲の両端の間に並んでいる、あらゆる文字にマッチする。
デフォルトの "C" ロケールが有効なときは、文字範囲は ASCII に基づいて振舞うだけである。
しかし、例えばライブラリが、 Win32 ロケールモデルでコンパイルされていれば、 "[a-z]" は ASCII 文字 a-z 、そして 'A'、'B'などにもマッチするが、 'z'のすぐあとに並んでいる 'Z'にはマッチしない。
このロケール特有の振る舞いは翻訳するときに `regbase::nocollate` フラグを設定することで、不可能にすることができる。
これは `regbase::normal` を使ったときのデフォルトの振る舞いであり、文字範囲が ASCII 文字コードに基づいて並んでいることを強制する。
同様に、もし POSIX C API 関数をつかうなら、 `REG_NOCOLLATE` を設定することでロケールに依存する並びを無効にすることができる。

文字クラスは、文字集合の中で、構文 "[:classname:]" を使うことで表現される。
例えば、 "[[:space:]]" は全ての空白文字の集合である。
文字クラスは `regbase::char_classes` フラグが設定されているときのみ利用できる。
利用できる文字クラスは:

- alnum
	- 全ての数字とアルファベット
- alpha
	- a-z と A-Z の全てのアルファベット。
		ロケールによっては他の文字も含まれるかもしれない。
- blank
	- スペースかタブの、あらゆる空白文字
- cntrl
	- 全ての制御文字
- digit
	- 0-9 の全ての数字
- graph
	- 空白以外の印刷可能文字
- lower
	- a-z の全ての小文字。
		ロケールによっては他の文字も含まれるかもしれない。
- print
	- 全ての印刷可能な文字
- punct
	- 全ての句読点
- space
	- 全ての空白文字
- upper
	- A-Z の全ての大文字。
		ロケールによっては他の文字も含まれるかもしれない。
- xdigit
	- 全ての16進文字。
		0-9, a-f, A-F。
- word
	- 全ての単語形成文字。
		つまり、全てのアルファベットとアンダースコア。
- unicode
	- 文字コードが 255 より大きい全ての文字。
		これはワイド文字特性クラスだけに適用される。

文字クラスの代わりに使うことができる略記がいくつかある。
これらは `regbase::escape_in_lists` フラグが設定されていときに使うことができる。

\\w in place of [:word:]

\\s in place of [:space:]

\\d in place of [:digit:]

\\l in place of [:lower:]

\\u in place of [:upper:] 

照合要素は文字集合宣言の中での [.tagname.] という一般的な形式をとる。
*tagname* は一文字か、照合要素の名前である。
たとえば、 [[.a.]] は [a] と等価であり、 [[.comma.]] は [,] と等価である。
ライブラリは全ての標準 POSIX の照合要素名に加えて、以下の連字サポートしている: "ae", "ch", "ll", "ss", "nj", "dz", "lj", それぞれ小文字、大文字、タイトルケース版がある。
マルチ文字照合要素は結局、一文字より多くにマッチする集合である。
例えば、 [[.ae.]] は2文字にマッチするが、 [\^[.ae.]] は一文字にしかマッチしないことに注意せよ。

等価クラスは文字集合宣言の中で [=tagname=] という一般的な形式をとる。
*tagname* は一文字か、照合要素の名前である。
これは、照合要素 [.tagname.] と同じ第一等価クラスの要素である、あらゆる文字とマッチする。
等価クラスは照合順序が同じ文字集合である。
第一等価クラスは、第一のソートキーがすべて同じである文字集合である(例えば文字列は典型的に文字によって並べられ、続いてアクセント、そして大文字/小文字によって並べられる。
この時、第一のソートキーは文字に関係し、第二のそれはアクセントに、第三のそれは大文字/小文字である)。
もし *tagname* に対応する等価クラスがなければ、[=tagname=] は実際には [.tagname.] と同じである。
不幸にも、Win32 環境を除いては、文字に対する第一のソートキーをロケールに依存しないで得る方法はない。
他のOSでは、ライブラリは (`strxfrm` から得た)全部のソートキーから、第一のソートキーを「推測する」ので、等価クラスはおそらく、 Win32 以外の OS では最も不確実だと考えられる。

文字集合の中にリテラル "-" を含むためには、それを 開き括弧 "[" 、 "[\^" 、範囲指定の端、または照合要素に続く最初の文字にしなければならない。
`regbase::escape_in_lists` フラグが設定されているなら、 "[\\-]" のようにエスケープ文字に続けてもよい。
リテラル "[" 、 "]" と "\^" を文字集合に含めるには、範囲指定の端、照合要素、またはもし `regbase::escape_in_lists` フラグが設定されているなら、エスケープ文字に続けてもよい。

*ラインアンカ*

アンカは 行頭及び行末で null 文字に一致するものである: "\^" は行頭の null 文字に一致し、 "\$" は行末の null 文字に一致する。

*後方参照*

後方参照は、既に一致した、先行する子表現への参照である。
これは、一致した子表現に対する参照であり、表現そのものに対する参照ではない。
後方参照はエスケープ文字 "\\" に数字 "1" から "9" を続けることで出来る。
"\\1" は最初の子表現に、 "\\2" は2番目の正規表現に、といった具合である。
例えば正規表現 "(.\*)\\1" は中間点について繰り返されるあらゆる文字列に一致する(訳注:文字列の前半と後半が同じであるということ)。
例えば、 "abcabc" や "xyzxyz" である。
どんな一致も起こらない子表現への後方参照は、 null 文字列に一致する: これが他のいくつかの正規表現の一致とは異なることに注意せよ。
後方参照は正規表現がフラグ `regbase::bk_refs` を設定されてコンパイルされた時のみ利用できる。

*コードによる文字*

これは他のライブラリでは利用できないアルゴリズムへの拡張である。
コードはエスケープ文字と、それに続く数字 "0" と、更にそれに、8進数の文字コードを続けることで成り立つ。
例えば、 "\\023" はその8進文字コードが 23 である文字をあらわす。
曖昧になるような場合、正規表現を区切るのに丸括弧を利用すること: "\\0103" は文字コードが 103 の文字をあらわし、 "(\\010)3" は文字コードが 10 の文字と、それに続く "3" をあらわす。
16進コードで文字を一致させるには、 \\x に 16 進の文字列を続ける。
この文字列は {} の中に閉じ込めることも可能である。
例えば、 \\xf0 や \\x{aff} など。
後者はユニコード文字である。

*単語演算子*

次の演算子は GNU 正規表現ライブラリとの互換性のために提供されている。

"\\w" は "word" 文字クラスの要素であるあらゆる1文字に一致する。
これは表現 "[[:word:]]" と同じである。

"\\W" は "word" 文字クラスの要素ではないあらゆる1文字に一致する。
これは表現 "[\^[:word:]]" と同じである。

"\\<" は単語の先頭の null 文字列に一致する。

"\\>" は単語の終端の null 文字列に一致する。

"\\b" は単語の先頭及び終端の null 文字列に一致する。

"\\B" は単語の中の null 文字列に一致する。

一致判定アルゴリズムに渡されるシーケンスの先頭は、フラグ `match_not_bow` が設定されていない限り、単語の先頭の可能性があると考えられる。
一致判定アルゴリズムに渡されるシーケンスの終端は、フラグ `match_not_eow` が設定されていない限り、単語の終端の可能性があると考えられる。

*バッファ演算子*

次の演算子は GNU 正規表現ライブラリ、及び Perl 正規表現との互換性のために提供されている:

"\\\`" はバッファの先頭に一致する。

"\\A" はバッファの先頭に一致する。

"\\'" はバッファの終端に一致する。

"\\z" はバッファの終端に一致する。

"\\Z" はバッファの終端に一致する。
または可能であれば、バッファの終端が続く1つ以上の改行文字に一致する。

フラグ `match_not_bob` 及び `match_not_eob` が設定されていない限り、バッファは一致判定アルゴリズムに渡されるシーケンス全体で出来ていると考えられる。

*エスケープ演算子*

エスケープ文字 "\\" は多くの一致を持つ。

集合宣言の中ではエスケープ文字は、フラグ `regbase::escape_in_lists` が設定されていない限り通常の文字である。
この場合、エスケープに続くどんな文字も、その通常の意味に関わらずリテラル文字である。

エスケープ演算子は例えば、後方参照や単語演算子などの演算子を導入する。

エスケープ演算子は、それに続く文字を通常の文字にすることもある。
例えば、"\\\*" は繰り返し演算子ではなく、リテラル "\*" をあらわす。

*単一文字エスケープシーケンス*

次のエスケープシーケンスは単一文字の略記である:

| エスケープシーケンス | 文字コード | 意味 |
|----------------------|------------|------|
| \\a                  | 0x07       | ベル文字 |
| \\f                  | 0x0C       | フォームフィード(FF)文字 |
| \\n                  | 0x0A       | 改行文字 |
| \\r                  | 0x0D       | 復帰(キャリッジリターン) |
| \\t                  | 0x09       | タブ文字 |
| \\v                  | 0x0B       | 垂直タブ |
| \\e                  | 0x1B       | ASCII エスケープ文字 |
| \\0dd                | 0dd        | 8進文字コード。 *dd* は1文字以上の8進文字 |
| \\xXX                | 0xXX       | 16進文字コード。 XX は1文字以上の16進文字 |
| \\x{XX}              | 0xXX       | 16進文字コード。 XX は1文字以上の16進文字。 ユニコード文字でもよい。 |
| \\cZ                 | z-@        | ASCII エスケープシーケンス control-Z。 Z は '@' の文字コードに等しいか、それより大きければどんな ASCII 文字でもよい。 |

*様々なエスケープシーケンス*

次のものの多くは perl との互換性のために提供されている。
しかし、 \\l \\L \\u \\U の意味はいくらか異なることに注意せよ。

|     |                            |
|-----|----------------------------|
| \\w | [[:word:]] と等価          |
| \\W | [\^[:word:]] と等価        |
| \\s | [[:space:]] と等価         |
| \\S | [\^[:space:]] と等価       |
| \\d | [[:digit:]] と等価         |
| \\D | [\^[:digit:]] と等価       |
| \\l | [[:lower:]] と等価         |
| \\L | [\^[:lower:]] と等価       |
| \\u | [[:upper:]] と等価         |
| \\U | [\^[:upper:]] と等価       |
| \\C | あらゆる1文字。 '.' と等価 |
| \\X | あらゆるユニコード複合文字シーケンスに一致する。 例えば、 "a\\x 0301" (鋭アクセントをもつ文字) |
| \\Q | 引用符開始演算子。 これに続く全ては 引用符終了演算子 \\E が発見されるまでリテラル文字として扱われる。 |
| \\E | 引用符終了演算子。 \\Q で始まったシーケンスを終了させる。 |

*何が一致するのか?*

正規表現ライブラリは、可能な限り最初の一致文字列と一致する。
もし与えられた場所で始まる一つ以上の文字列が一致可能なら、フラグ `match_any` が設定されていない限り、可能な限り長い文字列に一致する。
そして出会った最初の一致が返される。
`match_any` フラグを設定することで、一致を発見するのにかかる時間を削減することが出来る。
しかしこれは、ユーザが何が一致するかに関心がないときのみ役に立つ。
例えば、検索と置換の操作には適していない。
同じ場所で始まる複数の可能な一致があるような場合、そしてそれら全てが同じ長さである場合、選ばれる一致は最も長い最初の子表現を持つものである。
ふたつ目以上の一致でもそれが同じなら、2番目の子表現が試され、3番目・・・と続いていく。

---

*Copyright* [*Dr John Maddock*](mailto:John_Maddock@compuserve.com) *1998-2001 all rights reserved.*

---

*Japanese Translation Copyright (C) 2003 [Kohske Takahashi](mailto:k_takahashi@cppll.jp)*

オリジナルの、及びこの著作権表示が全ての複製の中に現れる限り、この文書の複製、利用、変更、販売そして配布を認める。
このドキュメントは「あるがまま」に提供されており、いかなる明示的、暗黙的保証も行わない。
また、いかなる目的に対しても、その利用が適していることを関知しない。

