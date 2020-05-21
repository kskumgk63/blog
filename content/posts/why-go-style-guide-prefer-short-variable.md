---
title: "Goの変数名はなぜ短いのか"
date: 2020-05-18T22:55:41+09:00
draft: false

categories: ["programming language"]
tag: ["golang"]
author: "Keisuke Umegaki"
---

Goの変数名は短い方が好まれます。例えば、`member`だったら`m`、`user`だったら`u`にするといった具合です。僕は、「エディタが保管してくれるし、変数名は長い方が具体的でわかりやすいからいいのでは？」とずっと疑問に思っていました。
公式も変数名を短くしろとは書いているけど、その理由まではわからず放置していましたが、「Goに入ればGoに従え」という言葉もありますので、今回ついに短い変数名が推奨される理由を調査してみました。

公式：[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)というGoのコードをレビューするポイントをまとめたWikiのことです。


下記は、Wikiに変数名について記載されている部分の翻訳と原文です。

---

Goの変数名は長いものより短いものにするべきです。特にスコープが限定されたローカル変数に当てはまります。`lineCount`よりも`c`が、`i` は `sliceIndex` よりも優先されます。

基本的なルール: 名前が使用される宣言から離れているほど、その名前はより説明的でなければなりません。メソッドのレシーバの場合は、1文字か2文字で十分です。ループインデックスやリーダのような一般的な変数は、1文字`i`, `r`で構いません。より珍しいものやグローバル変数は、より説明的な名前を必要とします。

> Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer c to lineCount. Prefer i to sliceIndex.
The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (i, r). More unusual things and global variables need more descriptive names.

https://github.com/golang/go/wiki/CodeReviewComments#variable-names


# 調査結果

調査結果としては、**「変数名が短いとコードの内容（振る舞い）を理解しやすいから」** ということが理由みたいです。
ただし注意点としては、なんでもかんでも短ければいいというわけでは無いことです。[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)にもある通り、「変数名が宣言と使用箇所が離れている場合」「グローバル変数」は説明的でなければなりません。

以下の悪い例と良い例を見てもらえれば、短い変数名の良さを感じてもらえると思います。

**悪い例**

```go
func removeElement(nums []int, val int) int {
	numbers := make([]int, 0, len(nums))
	for index := 0; index < len(nums); index++ {
		if nums[index] != val {
			numbers = append(numbers, nums[index])
		}
	}
	return len(n)
}
```

**良い例**

確かに長い変数名と比べると短いほうが読みやすい。

```go
func removeElement(nums []int, v int) int {
	n := make([]int, 0, len(nums))
	for i := 0; i < len(nums); i++ {
		if nums[i] != v {
			n = append(n, nums[i])
		}
	}
	return len(n)
}
```

# 根拠

GoogleでGoを開発しているAndrew Gerrandさんの以下の記述を見つけました。
彼は[Rob Pike](https://ja.wikipedia.org/wiki/%E3%83%AD%E3%83%96%E3%83%BB%E3%83%91%E3%82%A4%E3%82%AF)ともGopherfestで[対談](https://www.meetup.com/ja-JP/golangsf/events/220935959/)するような凄腕エンジニアなので、彼が言うなら間違いないでしょう。（投げやり）

（訳）短くすべきです。長い名前はコードの振る舞いを曖昧にします。

>Local variables
Keep them short; long names obscure what the code does.

https://talks.golang.org/2014/names.slide#6

# 最後に

今回は、「Goの変数が短い理由」について調べてみました。個人的には、振る舞いがわかりやすくなることに加えて、変数名が何だか辿れなくなるようなスコープのでかい関数を避けることにも繋がるというのも大きな理由なのかなと思いました。
