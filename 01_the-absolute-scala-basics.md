# The absolute scala basics
以下の要約.

https://www.udemy.com/course/rock-the-jvm-scala-for-beginners/

### 所感

- Exercise はないと思っていた. が、がっつりあった
  - ２時間弱の内容を見るのに６時間かかってしまった
- Auto caption は割とミスがある
- 毎回、最後の Takeaways で覚えて帰ることをまとめてくれている
- 予習すべき内容
  - [素数判定](https://ja.wikipedia.org/wiki/%E7%B4%A0%E6%95%B0%E5%88%A4%E5%AE%9A#%E6%A7%98%E3%80%85%E3%81%AA%E5%88%A4%E5%AE%9A%E6%B3%95)
  - 本講座では簡易版として sqrt の代わりに２で割っている
- Announcements のページは割と更新されている
- Daniel は関連ライブラリについても講座を開いている
  - [cats](https://typelevel.org/cats/)
  - [scalaz](https://github.com/scalaz/scalaz)
  - [lagom](https://www.lagomframework.com/)
  - Akka

直訳だと若干わかりにくい用語：

- inference : 推論
- imperative : 命令的
- auxiliary: 補助的
- illusory: 錯覚を起こす、幻影の
- incidence: 発生

### Getting Started

- 損はさせへんで、みたいなことを言っている
- IntelliJ をインストールする
- 使用する jdk は 1.8
- 自分は Windows 上で実施

### Values, Variables and Types

- val と var の違いについて
- val 使おう
- object Brrr extends App と書くと, そのスコープ内が main として扱われる
  - 厳密にはそういう実装ではないと思うが..

```scala
object Brrr extends App {
  // main code
}
```

### Expressins

- Compute の記述方法が C や Java とは違うよ
  - --> java: instructin, scala: expression
- while は副作用 (side-effects) を伴うので使うべきではない
  - side-effects の例：println, re-assigning
- var でなく val を使うと re-assigning できないので side-effects を伴いにくいコードになる
- Code block を変数へ代入できる
  - 1 statement の条件付き代入などでかけない場合に利用する
  - 最後に評価した式が変数の値になる(関数型言語だとよくある記述方法？)
  - 最後にカッコを書かずとも、変数には block を評価した値が代入される
- 所感
  - 条件付き代入は Java にもあるけど読みづらいから好きではない
  - 改行の位置がフォーマッタで固定にできるなら多少読みやすくなるかもしれない
  - Takeaways で if は expression と言っているがようわからん（他の言語でも expression なのでは.. 意味論での話？）

### Functions

- この辺は読み書きして覚えよう、という感じ. 詰まるところはあまりない.
- return 文は書かない
  - 最後に評価する式が function type になる

```scala
object Functions extends App {
  def aFunction(arg:String, b:Int):String = {
    "" //:String
  }
}
```

- 最後にカッコを書かなくても評価される
  - なんか気持ち悪いが..

```scala
println(aFunction())
println(aFunction) // almost same with above
```

- Code block 内でも def を使った関数定義が書ける
- 所感
  - Exercise では fibonacci と素数判定の問題を解く
  - 時間はかかるかもしれないがやっておいたほうが良い

### Type Inference

- Compiler が推論できる型は省略しても良い
- Compiler が推論できない型（例えば自身を再帰呼び出ししている関数の型など）を省略すると compiler error となる
- 所感
  - 正直不慣れなライブラリなどで型省略を多用されるとコードが読みづらいので型は書いておいてほしい

### Recursion

- Tail recursive
  - 関数内の最後の statement が「自身の呼び出しで終わっている」かどうかでコンパイラの挙動が変化する
  - `Tail call optimization (TCO)` と呼ばれているらしい
  - 自身の呼び出しを「含む」 expression ではダメ
- Tail recursive となっているかどうかは IntelliJ なら [GUI で見ることができる](https://blog.jetbrains.com/scala/2012/05/24/embrace-recursion/)

![](https://blog.jetbrains.com/wp-content/uploads/2012/05/scala-1.png)

![](https://blog.jetbrains.com/wp-content/uploads/2012/05/scala-2.png)

- 経過状態を保存する accumulator を引数へ導入することで tail recursive に変換しやすくなる
  - --> accumulator より result 変数の方がわかりやすいかもしれない
  - --> 再帰でリストを使って結果を持ち回しするときによく使う
- def の前に`@tailrec` annotation をつけることで tail recursive を矯正することができる

```scala
@tailrec
def fibo(nn:Int, cc:Int, fib1:Int, fib2:Int): Int = {
  if (cc <= 2) 1
  else if (nn == cc) fib1 + fib2
  else fibo(nn, cc+1, fib1 + fib2, fib1)
}
```

- 所感
  - 繰り返しを初期値から始めた方が書きやすい？
  - こちらの exercise も時間はかかるかもしれないがやっておいたほうが良い

Tail call optimization の導入状況：

- [Kotlin](https://medium.com/coding-blocks/tail-call-optimization-in-jvm-with-kotlin-ebdf90b34ec9): あり
- [Golang](https://github.com/golang/go/issues/22624): なし
  - 議論はされたが凍結された
- [Ruby](https://harfangk.github.io/2017/01/01/how-to-enable-tail-call-recursion-in-ruby.html): あり
- [Python](https://github.com/baruchel/tco): なし
  - モジュールはある
- [Rust](https://users.rust-lang.org/t/when-will-rust-have-tco-tce/20790): あり

### Call-by-Name and Call-by-Value

- default の挙動は call by value

- call by name
  - ByName 引数として渡す expression が、関数内で遅延評価されるようになる
  - ByName 引数として渡す expression は、呼び出し時には評価されない
  - 例えば call function の引数が ByName 引数で call(arguments()) と書いた場合に, arguments() の部分は関数内部で実際に使われるまで評価されない
  - なんかきもい. でも Akka の example code に出てくる

```scala
def callee(x: => Long): Unit = {
  // callee task
  println(x) // x would be evaluated at this timing
}

def arguments(): Long = {
  System.nanoTime()
}

callee(arguments()) // not yet evaluated
```

### Default and Named Arguments

- default argument
  - accumulator みたいなのは隠ぺいすべきでは..

```scala
def callee(x: Int = 1, y: String = "gagaga", z: Long = 2f): String = {
  x + " " + y + " " + z
}

callee(y = "GOGOGO") // 1 as x, 2f as z
```

### Smart Operations on Strings

- append, prepend operator は必要なのか..？
- interpolators
