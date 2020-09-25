---
title: "［Golang］`errors.Is()` `errors.As()` 完全ガイド〜使い方と違いをしっかり調査しました〜"
date: 2020-09-26T07:55:56+09:00
draft: false
---

# はじめに

`errors.As()`を雰囲気で使っていたらハマったので、`errors.Is()`も含めて、しっかりと調査してドキュメントとコードを読んだ上でまとめてみました。

ハマったところ、ハマりそうなところを重点的にまとめてみたので、お役に立てれば幸いです。

# 何をするメソッドなのか

## 簡単に

`error`を比較して`bool`を返してくれます。
使いみちとしては、アプリケーションのエラーを外部のエラー（例：gRPCのエラー）に変換したり、ライブラリで使用されているエラーをハンドリングして、アプリケーションのエラーに変換したりするときがあると思います。

### `errors.Is()`
- 比較対象が保持している値と比較します。
	- 値が同じなら`true`を返します。
	- 値が異なるなら`false`を返します。
- `interface`など比較できない者同士だと必ず`false`になります。

### `errors.As()`
- 比較対象を型レベルで比較します。
	- 型が同じなら`true`を返します。
	- 型が異なるなら`false`を返します。
- 値は違っても`true`を返します。
- 引数`target`（第2引数）には、`nil`ではないポインタ型を渡しましょう。

## 詳しく

### `errors.Is()`のGoDocとコードを読みました

>Is reports whether any error in err's chain matches target.
The chain consists of err itself followed by the sequence of errors obtained by repeatedly calling Unwrap.
An error is considered to match a target if it is equal to that target or if it implements a method Is(error) bool such that Is(target) returns true.
An error type might provide an Is method so it can be treated as equivalent to an existing error. 
https://golang.org/pkg/errors/#Is

そのまま訳すと...

Is() は、`err`のチェーン内のエラーがターゲットにマッチするかどうかを報告します。
このチェーンは`err`自身の後に、`Unwrap`を繰り返し呼び出すことで得られる一連のエラーで構成されています。
エラーがターゲットと等しい場合、または `Is(target)` が`true`を返すような `Is(error) bool` メソッドを実装している場合、エラーはターゲットと一致しているとみなされます。
エラーの型は、既存のエラーと同等の扱いができるように、`Is()`メソッドを提供している場合があります。

**要するに、`errors.Is()`は、エラーを比較して同じ値を持っていたら`true`、持っていないなら`false`を返してくれます。**

**注目する点は...**

>エラーがターゲットと等しい場合

は、`Is()`の実装は必要無いということです。

もっというと、

**エラーの値が比較可能であれば、`Is()`の実装は必要無いです。**
**エラーの値が比較不可能であれば、`Is()`の実装は必要です。**

あとは、**Wrapしたエラーには使える無いも抑えておくべき**です！
→ Wrapしたエラーと比較するときは、`errors.As()`を使いましょう。

### エラーの値が比較可能なとき

例えば、`struct`に`Error()`を実装し`error interface`を満たして、`error`として扱っている場合のことです。
`struct`同士は比較可能なので、`Is()`の実装は必要ありません。

実際のコードを見ると、値を比較できるまで`Unwrap()`して、比較可能になった時点で、比較していることがわかります。

```go
	isComparable := reflectlite.TypeOf(target).Comparable()
	for {
		if isComparable && err == target {
			return true
		}
		if x, ok := err.(interface{ Is(error) bool }); ok && x.Is(target) {
			return true
		}
		// TODO: consider supporting target.Is(err). This would allow
		// user-definable predicates, but also may allow for coping with sloppy
		// APIs, thereby making it easier to get away with them.
		if err = Unwrap(err); err == nil {
			return false
		}
	}
```
https://github.com/golang/go/blob/master/src/errors/wrap.go#L44-L58

### エラーの値が比較 不 可能なとき
例えば、`error interface`を内包した`struct`同士を比較したときなどです。
`interface`同士の比較はできません。

```go
isComparable := reflectlite.TypeOf(target).Comparable()
```
https://github.com/golang/go/blob/master/src/errors/wrap.go#L44

ここに`false`が入るわけです。

**例**

下記のサンプルコードでは、`interface error`は比較できないので、`false`になっています。

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	is := errors.Is(originalError{err: errors.New("1")}, originalError{err: errors.New("1")})
	fmt.Printf("is = %v because err is not comparable\n", is)
}
```
https://play.golang.org/p/IN7BHbriu26

なので、`Is()`を実装して、比較する必要があります。
実装された`Is()`では、`err.Error()`の結果`string`を使って比較を行っているので、`true`が返ります。

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

// implemented!!
func (e originalError) Is(target error) bool { return e.err.Error() == target.Error() }

func main() {
	is := errors.Is(originalError{err: errors.New("1")}, originalError{err: errors.New("1")})
	fmt.Printf("is = %v because originaleError implements Is()\n", is)
}
```
https://play.golang.org/p/5p8u-D1Hr6q

### `Wrap`したエラーには使えない

なぜなら、値を比較するので`Wrap`された時点で値は比較対象とは異なるはずだからです。
先程も書きましたが、`Wrap`されたエラーと比較したいなら`errors.As()`を使いましょう！

ちなみに、エラーをラップするためには標準パッケージを使うと

```go
fmt.Errorf("failed to do something: %w", err)
```

みたいな感じで`Wrap`できます。


### `errors.As()`のGoDocとコードを読みました

>As finds the first error in err's chain that matches target, and if so, sets target to that error value and returns true. Otherwise, it returns false.
The chain consists of err itself followed by the sequence of errors obtained by repeatedly calling Unwrap.
An error matches target if the error's concrete value is assignable to the value pointed to by target, or if the error has a method As(interface{}) bool such that As(target) returns true. In the latter case, the As method is responsible for setting target.
An error type might provide an As method so it can be treated as if it were a different error type.
As panics if target is not a non-nil pointer to either a type that implements error, or to any interface type.
https://golang.org/pkg/errors/#As

そのまま訳すと...

`As()`は`err`のチェインの中で最初のエラーが target にマッチするものを見つけ、マッチしていれば target をそのエラー値に設定して`true`を返します。そうでなければ`false`を返します。
チェーンは err 自体の後に、`Unwrap`を繰り返し呼び出すことで得られる一連のエラーで構成されています。
エラーの具体的な値が target が指す値に代入可能な場合、またはエラーが `As(target)` が`true`を返すような `As(interface{}) bool` メソッドを持っている場合、エラーは target にマッチします。後者の場合は、`As()`メソッドがtargetの設定を担当します。
エラータイプが `As()`メソッドを提供している場合は、それが別のエラータイプであるかのように扱うことができます。
`As()`は、target がエラーを実装した型、または任意の`interface`への`nil`ではないポインタである場合にパニックを起こします。

**要するに、`errors.As()`は、エラーを比較して同じ型であれば`true`、異なる型なら`false`を返してくれます。**

**注目する点は...**

- `panic`になる条件を抑えること。
- 比較対象の値は同じでは無くて良くて、代入可能であればいいこと。
- `Wrap`したエラーにも使えること。

です。

### `panic`になる条件① 比較対象が`nil`もしくは、pointerではない型である

```go
	if target == nil {
		panic("errors: target cannot be nil")
	}
	val := reflectlite.ValueOf(target)
	typ := val.Type()
	if typ.Kind() != reflectlite.Ptr || val.IsNil() {
		panic("errors: target must be a non-nil pointer")
	}
```
https://github.com/golang/go/blob/master/src/errors/wrap.go#L78-L85

**実際に`panic`を起こしてみる**

比較対象が`nil`のとき

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	err := &originalError{err: errors.New("1")}
	as := errors.As(err, nil)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/UpCzRpoYPqW

実行結果

```txt
./prog.go:14:8: second argument to errors.As must be a non-nil pointer to either a type that implements error, or to any interface type
Go vet exited.

panic: errors: target cannot be nil

goroutine 1 [running]:
errors.As(0x4deb00, 0xc000010210, 0x0, 0x0, 0xc000032778)
	/usr/local/go-faketime/src/errors/wrap.go:79 +0x5f5
main.main()
	/tmp/sandbox800961272/prog.go:14 +0x9f
```

比較対象がpointerではない型のとき

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	err := &originalError{err: errors.New("1")}
	var target originalError
	as := errors.As(err, target) // 本当は &target とするべき
	fmt.Printf("as = %v\n", as)
}

```
https://play.golang.org/p/iF-43pCJf3P

実行結果

```txt
./prog.go:15:8: second argument to errors.As must be a non-nil pointer to either a type that implements error, or to any interface type
Go vet exited.

panic: errors: target must be a non-nil pointer

goroutine 1 [running]:
errors.As(0x4deae0, 0xc00010a050, 0x4ae1c0, 0xc00010a060, 0xc000068f48)
	/usr/local/go-faketime/src/errors/wrap.go:84 +0x54f
main.main()
	/tmp/sandbox214256996/prog.go:15 +0xd1
```

### `panic`になる条件② 比較対象が`error interface`を実装していない

そもそもコンパイルできないので具体例を載せることは割愛しますが、要注意です！

```go
	if e := typ.Elem(); e.Kind() != reflectlite.Interface && !e.Implements(errorType) {
		panic("errors: *target must be interface or implement error")
	}
```
https://github.com/golang/go/blob/master/src/errors/wrap.go#L86-L88

### 「`panic`になる条件」と「比較対象の値は同じでは無くて良くて、代入可能であればいいこと」を踏まえて実装してみる

2つのパターンを用意してみました。

### `enum`を使った実装

```go
type ErrorCode uint64

const (
	Zero ErrorCode = iota
	One
)

func (code ErrorCode) Error() string {
	return [...]string{
		"Error: Zero",
		"Error: One",
	}[code]
}
func main() {
	var code ErrorCode
	as := errors.As(Zero, &code)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/Zad83sF-Dxv

### `struct`を使った実装

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	var err originalError
	as := errors.As(originalError{err: errors.New("1")}, &err)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/WA-pdwXcM9W

### 注意点：`pointer`を意識してください

`errors.As()`の実装以外でもハマりがちなのが、pointerです。
下記を例にすると、`originalError` と `*originalError` は違います。
よって、`errors.As()`は`false`を返します。

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	err := originalError{err: errors.New("1")}
	var target *originalError
	as := errors.As(err, &target)
	fmt.Printf("err = %T, target = %T\n", err, target)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/V3WravAWHpb

そして、当たり前なんですが、`err`の方をpointerにすれば、`true`を返します。

```go
type originalError struct{ err error }

func (e originalError) Error() string { return e.err.Error() }

func main() {
	err := &originalError{err: errors.New("1")} // pointer
	var target *originalError                   // pointer
	as := errors.As(err, &target)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/2RHK2k7ZBJk

### `Wrap`したエラーとの比較に使えます

`As()`は値の差異は関係ないので、型があっていると`true`を返します。
なので、エラーが持っているメッセージは関係なく、型レベルで同じか確かめたいときに有効です！

```go
type ErrorCode uint64

const (
	Zero ErrorCode = iota
	One
)

func (code ErrorCode) Error() string {
	return [...]string{
		"Error: Zero",
		"Error: One",
	}[code]
}
func main() {
	wrappedError := fmt.Errorf("wrap: %w", Zero)
	var code ErrorCode
	as := errors.As(wrappedError, &code)
	fmt.Printf("as = %v\n", as)
}
```
https://play.golang.org/p/9Sdw-th7znr

# さいごに

結構詳しめに`errors.Is()` `errors.As()`について調べて疲れましたw
ただ利用頻度が高いライブラリだと思うので、しっかりと抑えて今日学んだ知識を活かしていきたいと思います。
