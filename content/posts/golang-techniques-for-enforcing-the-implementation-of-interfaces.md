---
title: "[Golang] 1行加えるだけで interface を実装するテクニック"
date: 2020-04-09T20:33:43+09:00
draft: false
categories: ["golang"]
---

# interfaceの実装を強制する簡単なテクニックを紹介

Goの`interface`を使って抽象的にオブジェクトを扱いたいときにメソッドの実装漏れってないでしょうか？

抽象化したい構造体が多かったり、`interface`のメソッド定義を変更した場合に簡単に発生しますよね。
今回は、それを起こさないために、Goコンパイラに`interface`の実装漏れを伝える簡単なテクニックを共有します。

# 導入

`Doer`という`Do()`を実装するだけで使える`interface`を定義しました。
下記のコードの`Person`は`Do()`を実装しているので、`Doer`として扱うことができます。

https://play.golang.org/p/W9Gd3gZKY2z

```go
type Doer interface {
    Do()    
}

type Person struct {
    Name string    
}

func (p Person) Do() {
    fmt.Printf("My name is %s. I am Doer!!!!\n", p.Name)
}

func main() {
    p := Person{Name: "John"}
    p.Do()
}
```


```
実行結果
My name is John. I am Doer!!!!
```

ここまで問題は無いと思います。
それでは、`Doer`にメソッド`Greet()`を追加してみましょう。
今まで`Do()`を実装するだけで、`Doer`として扱うことができていたのに、`Greet()`も実装する必要が発生しました。

なので、`Do()`しか実装していない構造体`Person`は、`Doer`として扱うことができませんが、プログラムは正しいです。

```go
type Doer interface {
    Do()    
    Greet() // 追加！！！
}

type Person struct {
    Name string    
}

func (p Person) Do() {
    fmt.Printf("My name is %s. I am Doer!!!!\n", p.Name)
}

func main() {
    p := Person{Name: "John"}
    p.Do()
}
```

このように1ファイルに収まるシンプルさなら実装漏れにすぐに気づくはずですが、
`interface`を実装したい構造体がたくさんあったら...ファイルが分けられていたら...パッケージが分けられていたら...
なかなか気づくのは難しいと思います。

それを探すのってかなり面倒ではありませんか....？

# テクニックを紹介

`var _ Doer = Person{}` というコードを追加するだけです。
`_` は省略を表しているので、メモリも無駄になりません。

これを加えるだけで、コンパイルエラーが発生し、さらにエラーが発生しているファイル名、行列数まで教えてくれるのですぐに見つけることができます。

https://play.golang.org/p/XvFkqetQ-Df

```go
type Doer interface {
	Do()
	Greet()
}

var _ Doer = Person{} // たったこれだけ！

type Person struct {
	Name string
}

func (p Person) Do() {
	fmt.Printf("My name is %s. I am Doer!!!!\n", p.Name)
}

func main() {
	p := Person{Name: "John"}
	p.Do()
}
```

```
./prog.go:5:5: cannot use composite literal (type Person) as type Doer in assignment:
	Person does not implement Doer (missing Greet method)
```

# 最後に

たった１行追加するだけで`interface`の実装漏れをふせぐことができるこのテクニックを是非使ってみてください！！

実はこれ`Uber`がおすすめしているテクニックでもあります。

https://github.com/uber-go/guide/blob/master/style.md#verify-interface-compliance

他にもGoのテクニックや、`Uber`で推奨されているスタイルがまとまっているので気になる方はチェックしてみてください。

コードレビューをする際にめちゃくちゃ使えます！
「Uberで使われているんですよ〜」といえば、納得できそうではないでしょうか？笑
