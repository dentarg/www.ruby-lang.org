---
layout: news_post
title: "Ruby 2.7.0-preview3 リリース"
author: "naruse"
translator:
date: 2019-11-23 12:00:00 +0000
lang: ja
---

Ruby 2.7シリーズのプレビュー版である、Ruby 2.7.0-preview3をリリースします。

プレビュー版は、年末の正式リリースに向け、新たな機能を試し、フィードバックを集めるために提供されています。
Ruby 2.7.0-preview3では、多くの新しい機能やパフォーマンスの改善が含まれています。 その一部を以下に紹介します。

## Compaction GC

断片化したメモリをデフラグするCompaction GCが導入されました。

一部のマルチスレッドなRubyプログラムを長期間動かし、マーク&スイープ型GCを何度も実行していると、メモリが断片化してメモリ使用量の増大や性能の劣化を招くことが知られています。

Ruby 2.7では`GC.compact` というメソッドを導入し、ヒープをコンパクションすることが出来るようになります。ヒープ内の生存しているオブジェクトを他のページに移動し、不要なページを解放できるようになるとともに、ヒープをCoW (Copy on Write) フレンドリーにすることが出来ます。 [[Feature #15626]](https://bugs.ruby-lang.org/issues/15626)

## Pattern Matching [Experimental]

関数型言語で広く使われているパターンマッチという機能が実験的に導入されました。
渡されたオブジェクトの構造がパターンと一致するかどうかを調べ、一致した場合にその値を変数に代入するといったことができるようになります。 [[Feature #14912]](https://bugs.ruby-lang.org/issues/14912)

{% highlight ruby %}
case JSON.parse('{...}', symbolize_names: true)
in {name: "Alice", children: [{name: "Bob", age: age}]}
  p age
  ...
end
{% endhighlight %}

詳細については [Pattern matching - New feature in Ruby 2.7](https://speakerdeck.com/k_tsj/pattern-matching-new-feature-in-ruby-2-dot-7) を参照してください。

## REPL improvement

Ruby に添付されている REPL (Read-Eval-Print-Loop) である `irb` で、複数行編集がサポートされました。これは `reline` という `readline` 互換のピュア Ruby 実装によるものです。
また、rdoc 連携も提供されるようになっています。`irb` 内で、クラス、モジュール、メソッドのリファレンスをその場で確認できるようになりました。 [[Feature #14683]](https://bugs.ruby-lang.org/issues/14683), [[Feature #14787]](https://bugs.ruby-lang.org/issues/14787), [[Feature #14918]](https://bugs.ruby-lang.org/issues/14918)
これに加え、`binding.irb`で表示される周辺のコードや、コアクラスのオブジェクトのinspect結果に色がつくようになっています。

<video autoplay="autoplay" controls="controls" muted="muted" width="576" height="259">
  <source src="https://cache.ruby-lang.org/pub/media/irb_improved_with_key_take2.mp4" type="video/mp4">
</video>

## キーワード引数を通常の引数から分離

キーワード引数とpositionalな引数（ふつうの引数）の間の自動変換が非推奨となりました。この変換はRuby 3で除去される予定です。[[Feature #14183]](https://bugs.ruby-lang.org/issues/14183)

* メソッド呼び出しにおいて最後の引数としてハッシュオブジェクトを渡し、他にキーワード引数を渡さず、かつ、呼ばれたメソッドがキーワード引数を受け取るとき、警告が表示されます。キーワード引数として扱いたい場合は、明示的にdouble splat演算子（`**`）を足すことで警告を回避できます。このように書けばRuby 3でも同じ意味で動きます。

  {% highlight ruby %}
  def foo(key: 42); end; foo({key: 42})   # warned
  def foo(**kw);    end; foo({key: 42})   # warned
  def foo(key: 42); end; foo(**{key: 42}) # OK
  def foo(**kw);    end; foo(**{key: 42}) # OK
  {% endhighlight %}

* キーワード引数を受け取るメソッドにキーワード引数を渡すが、必須引数が不足している場合に、キーワード引数は最後の必須引数として解釈され、警告が表示されます。警告を回避するには、キーワードではなく明示的にハッシュとして渡してください。このように書けばRuby 3でも同じ意味で動きます。

  {% highlight ruby %}
  def foo(h, **kw); end; foo(key: 42)      # warned
  def foo(h, key: 42); end; foo(key: 42)   # warned
  def foo(h, **kw); end; foo({key: 42})    # OK
  def foo(h, key: 42); end; foo({key: 42}) # OK
  {% endhighlight %}

* メソッドがキーワード引数を受け取るがdouble splat引数は受け取らず、かつ、メソッド呼び出しでSymbolと非Symbolの混ざったハッシュを渡す（もしくはハッシュをdouble splatでキーワードとして渡す）場合、ハッシュは分割され、警告が表示されます。Ruby 3でもハッシュの分割を続けたい場合は、呼び出し側で明示的に分けるようにしてください。

  {% highlight ruby %}
  def foo(h={}, key: 42); end; foo("key" => 43, key: 42)   # warned
  def foo(h={}, key: 42); end; foo({"key" => 43, key: 42}) # warned
  def foo(h={}, key: 42); end; foo({"key" => 43}, key: 42) # OK
  {% endhighlight %}

* メソッドがキーワード引数を受け取らず、呼び出し側でキーワード引数を渡した場合、ハッシュの引数としてみなされる挙動は変わらず、警告も表示されません。Ruby 3でもこのコードは動き続ける予定です。

  {% highlight ruby %}
  def foo(opt={});  end; foo( key: 42 )   # OK
  {% endhighlight %}

* メソッドが任意のキーワードを受け取る場合、非Symbolがキーワード引数のキーとして許容されるようになります。[[Feature #14183]](https://bugs.ruby-lang.org/issues/14183)

  {% highlight ruby %}
  def foo(**kw); p kw; end; foo("str" => 1) #=> {"str"=>1}
  {% endhighlight %}

* メソッド定義で<code>**nil</code>と書くことで、このメソッドがキーワードを受け取らないことを明示できるようになりました。このようなメソッドをキーワード引数付きで呼び出すとArgumentErrorになります。[[Feature #14183]](https://bugs.ruby-lang.org/issues/14183)

  {% highlight ruby %}
  def foo(h, **nil); end; foo(key: 1)       # ArgumentError
  def foo(h, **nil); end; foo(**{key: 1})   # ArgumentError
  def foo(h, **nil); end; foo("str" => 1)   # ArgumentError
  def foo(h, **nil); end; foo({key: 1})     # OK
  def foo(h, **nil); end; foo({"str" => 1}) # OK
  {% endhighlight %}

* キーワード引数を受け取らないメソッドに対して空のハッシュをdouble splatで渡すとき、空のハッシュが渡る挙動はなくなりました。ただし、必須引数が不足する場合は空のハッシュが渡され、警告が表示されます。ハッシュの引数として渡したい場合はdouble splatをつけないようにしてください。

  {% highlight ruby %}
  h = {}; def foo(*a) a end; foo(**h) # []
  h = {}; def foo(a) a end; foo(**h)  # {} and warning
  h = {}; def foo(*a) a end; foo(h)   # [{}]
  h = {}; def foo(a) a end; foo(h)    # {}
  {% endhighlight %}

注意：キーワード引数の非互換に関する警告が大量すぎるという詩的があります。現在、廃止に関する警告をデフォルトでオフにする ([#16345](https://bugs.ruby-lang.org/issues/16345)) か、一度警告した箇所を二度は警告しない ([#16289](https://bugs.ruby-lang.org/issues/16289)) という2つの解決方法が議論されています。最終決定はなされていませんが、正式リリースまでには修正される予定です。

## 主要な新機能

* メソッド参照演算子 <code>.:</code> が以前のプレビューリリースで試験的に導入されていましたが、削除されました。[[Feature #12125]](https://bugs.ruby-lang.org/issues/12125), [[Feature #13581]]( https://bugs.ruby-lang.org/issues/13581), [[Feature #16275]](https://bugs.ruby-lang.org/issues/16275)

* デフォルトのブロックの仮引数として番号指定パラメータが試験的に導入されました。[[Feature #4475]](https://bugs.ruby-lang.org/issues/4475)

* 開始値省略範囲式が試験的に導入されました。これは終了値省略範囲式ほど有用ではないと思われますが、しかし DSL のような目的には役立つかもしれません。 [[Feature #14799]](https://bugs.ruby-lang.org/issues/14799)

  {% highlight ruby %}
  ary[..3]  # identical to ary[0..3]
  rel.where(sales: ..100)
  {% endhighlight %}

* `Enumerable#tally` が追加されました。各要素の出現回数を数えます。

  {% highlight ruby %}
  ["a", "b", "c", "b"].tally
  #=> {"a"=>1, "b"=>2, "c"=>1}
  {% endhighlight %}

* レシーバを`self`としてprivateメソッドを呼び出すことが許容されるようになりました。 [[Feature #11297]](https://bugs.ruby-lang.org/issues/11297) [[Feature #16123]](https://bugs.ruby-lang.org/issues/16123)

  {% highlight ruby %}
  def foo
  end
  private :foo
  self.foo
  {% endhighlight %}

* `Enumerator::Lazy#eager` が追加されました。lazyなEnumeratorを非lazyなEnumeratorに変換します。

  {% highlight ruby %}
  a = %w(foo bar baz)
  e = a.lazy.map {|x| x.upcase }.map {|x| x + "!" }.eager
  p e.class               #=> Enumerator
  p e.map {|x| x + "?" }  #=> ["FOO!?", "BAR!?", "BAZ!?"]
  {% endhighlight %}

## パフォーマンスの改善

* JIT [Experimental]

  * 最適化の際の仮定が無効とされた場合、JIT 化されていたコードがより最適化度が低いコードに再コンパイルされるようになりました。

  * あるメソッドが「純粋」であると判定された場合、メソッドのインライン化が行われるようになりました。この最適化はまだ実験的であり、また多数のメソッドが今はまだ「純粋」と判定されないままです。

  * `--jit-min-calls` オプションのデフォルト値が 5 から 10,000 に変更されました。

  * `--jit-max-cache` オプションのデフォルト値が 1,000 から 100 に変更されました。

* ~~`Symbol#to_s`, `Module#name`, `true.to_s`, `false.to_s` `nil.to_s` は常にfrozenな文字列を返すようになりました。返された文字列は常に同じオブジェクトとなります。 [Experimental]  [[Feature #16150]](https://bugs.ruby-lang.org/issues/16150)~~ 撤回されました

* `CGI.escapeHTML` のパフォーマンスが改善されました。 [GH-2226](https://github.com/ruby/ruby/pull/2226)

* Monitor 及び MonitorMixin のパフォーマンスが改善されました [[Feature #16255]](https://bugs.ruby-lang.org/issues/16255)

## その他の注目すべき 2.6 からの変更点

* いくつかの標準ライブラリがアップデートされました。
  * Bundler 2.1.0.pre.3 ([History](https://github.com/bundler/bundler/blob/2-1-stable/CHANGELOG.md#210pre3-november-8-2019))
  * RubyGems 3.1.0.pre.3 ([History](https://github.com/rubygems/rubygems/blob/3.1/History.txt))
  * CSV 3.1.2 ([NEWS](https://github.com/ruby/csv/blob/v3.1.2/NEWS.md))
  * Racc 1.4.15
  * REXML 3.2.3 ([NEWS](https://github.com/ruby/rexml/blob/v3.2.3/NEWS.md))
  * RSS 0.2.8 ([NEWS](https://github.com/ruby/rss/blob/v0.2.8/NEWS.md))
  * StringScanner 1.0.3

* 以下のライブラリが新たに default gems になりました
  * 以下のライブラリは rubygems.org からアップデート可能です。
    * benchmarck
    * cgi
    * delegate
    * getoptlong
    * net-pop
    * net-smtp
    * open3
    * pstore
    * singleton
  * 以下のライブラリはまだ rubygems.org から取得できません。
    * monitor
    * observer
    * timeout
    * tracer
    * uri
    * yaml

* ブロックを渡すメソッド呼び出し中の、ブロックを伴わない `Proc.new` と `proc` が警告されるようになりました。

* ブロックを渡すメソッド呼び出し中の、ブロックを伴わない `lambda` はエラーとなるようになりました。

* Unicode および Emoji のバージョンが 11.0.0 から 12.0.0 になりました。[[Feature #15321]](https://bugs.ruby-lang.org/issues/15321)

* Unicode のバージョンが 12.1.0 となり、新元号「令和」を表す合字 U+32FF がサポートされました。[[Feature #15195]](https://bugs.ruby-lang.org/issues/15195)

* `Date.jisx0301`, `Date#jisx0301`, および `Date.parse` で新元号に対応しました。[[Feature #15742]](https://bugs.ruby-lang.org/issues/15742)

* Ruby のビルドに C99 に対応したコンパイラが必要になりました。[[Misc #15347]](https://bugs.ruby-lang.org/issues/15347)
  * 本件についての詳細: <https://bugs.ruby-lang.org/projects/ruby-trunk/wiki/C99>

その他詳細については、[NEWS](https://github.com/ruby/ruby/blob/v2_7_0_preview3/NEWS) ファイルまたは[コミットログ](https://github.com/ruby/ruby/compare/v2_6_0...v2_7_0_preview3)を参照してください。

なお、こうした変更により、Ruby 2.6.0 以降では [3895 個のファイルに変更が加えられ、213426 行の追加と 96934 行の削除が行われました](https://github.com/ruby/ruby/compare/v2_6_0...v2_7_0_preview3) !

Ruby 2.7 で楽しいプログラミングを！

## Download

* <https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.0-preview3.tar.bz2>

      SIZE:   14628168 bytes
      SHA1:   849a96041202ad3beea82e1069694740d907d583
      SHA256: 09de8d3a014904142b7bc111195ba8094f6eeb59eb529f7bb0512d8b0fc06f5d
      SHA512: 661eaff1830546cea9e36f1f5a3cb591d792a17ad18d2c00bd097d53757bf321f0d192ddb3c8fdc75c28a0ecbef587e433ad1b345cdeeb4aa6a4744f7e828fbb
* <https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.0-preview3.tar.gz>

      SIZE:   16715144 bytes
      SHA1:   a90300e13b4c8778c798e60b7d782f9808cfe492
      SHA256: 54300275a17a0367bab91ad54b51faf3d45955041f0bf69a4612fe7b0179f4f2
      SHA512: 9083525de830a4b2e8685681dfaa3bce14cc1ed7360f5540211f6aa97967e316983554596adf442dda52041245e31ec42bffd1982d28ad5e5c4c230be3fa7d62
* <https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.0-preview3.tar.xz>

      SIZE:   11926312 bytes
      SHA1:   b40a6c8c11a0ba7c0d9f1548358da7c662673852
      SHA256: aaf3d78e2d7b3d42c668f9f619a016507771552e207f7729b3feba19b13c591a
      SHA512: 2a38ac56d63522bf998eaca755fa395ce490b4eaaa065b3345e9f4b57f6a1407b8089735b3dcdf6af9b761a9efe963f716f755867955e69c52e2ab5f0680e974
* <https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.0-preview3.zip>

      SIZE:   20689945 bytes
      SHA1:   0de6f8922f7be5b4341ad66e69f309473ba7b905
      SHA256: 3b227bf242de646f7479ec145bc9445e5fc6a24648a3912bfb8ac02b7effcdb7
      SHA512: a44bca1cd93e916ec137d65a0e844f1ea84f4d64409f8e2522c9bbcc16f43a61f3bdb5ff8821de62349678da344056b774762d6e625df9ee36fa66d08e244dce

## Ruby とは

Rubyはまつもとゆきひろ (Matz) によって1993年に開発が始められ、今もオープンソースソフトウェアとして開発が続けられています。Rubyは様々なプラットフォームで動き、世界中で、特にWebアプリケーション開発のために使われています。
