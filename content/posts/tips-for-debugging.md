---
title: "バグの原因は何か？はやくバグを潰す観点"
date: 2020-05-04T16:21:36+09:00
draft: false

categories: ["development"]
author: "Keisuke Umegaki"
---

# 本記事の目的

完成したとおもったコードが期待通りに動かず実装していた時間よりデバッグをしている時間の方が長くかかることがあります。
大抵の場合、デバッグに夢中になるあまり手当たり次第にバグの原因を探しており、気づく頃には時間が溶けていて、消費した時間を憂いてさらに焦って失敗します。
そのような悲しくて辛いことを繰り返さないようにすることが本記事の目的です。

![](https://4.bp.blogspot.com/-N7u7lxBmZs0/VCOJY5xt7XI/AAAAAAAAmwk/wswFYwQbzfU/s400/isogashii_woman.png)


バグの一番の原因は、「冷静さの欠如」です。デバッグに時間がかかって「焦り」「イライラ」を感じたら一旦手を置いてください。
あなたが悩んでいるそのバグは実は簡単に潰すことができます！あなたが苦しんでいるのは冷静さを失っているからです！さあ冷静さを取り戻すために次の質問に回答しましょう！

### 実装箇所のコードが達成したい目的は何ですか？
焦るあまりに元々のコードが達成したかった目的を見失っていることがあります。修正を加えていって当初のコードの目的を見失うとデバッグどころか1から再実装なんてことにもなりかねません。
もう一度、あなたが実現したい機能は何だったのか言葉にしてみましょう。達成しなければならないものが思っているより簡単だったことを思い出すかもしれませんよ。

筆者は、そもそも目的を見失わないように開発する手段としてテスト駆動開発をおすすめします。

簡単にテスト駆動開発を紹介すると、コードを実装する前にユニットテスト（UT）を書いてコードに期待する動作を明確にし、テストが通るように実装する開発手法です。
コードの期待動作をUTで明確にすることで実装の間違いはコンピュータが素早く正確に伝えてくれるのでコードの目的からずれることを防ぎます。
また、一通り実装が終わりリファクタリングをする場合もUTが通ることを守れば自信を持ってリファクタリングができます。
さらには、UTを実装することが大変であればそもそも関数の定義が悪いことにも気づくことができます。

詳細は、こちらの記事がおすすめです。
- [テスト駆動開発（TDD）とは？TDDの進め方をステップ毎に解説！](https://www.valtes.co.jp/qbookplus/1069)
- [テスト駆動開発って何だろう](https://dev.classmethod.jp/articles/what-tdd/)

また[テスト駆動開発](https://www.amazon.co.jp/%E3%83%86%E3%82%B9%E3%83%88%E9%A7%86%E5%8B%95%E9%96%8B%E7%99%BA-%EF%BC%AB%EF%BD%85%EF%BD%8E%EF%BD%94%EF%BC%A2%EF%BD%85%EF%BD%83%EF%BD%8B-ebook/dp/B077D2L69C/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&dchild=1&keywords=%E3%83%86%E3%82%B9%E3%83%88%E9%A7%86%E5%8B%95%E9%96%8B%E7%99%BA&qid=1588245450&s=digital-text&sr=1-1)はamazonで買えるので一読をおすすめします。

### エラーメッセージの意味を日本語で他人に伝えられますか？
エラーメッセージは大抵の場合、英語で出力されます。その英語のメッセージを正しく理解しないことが原因で沼にはまることがあります。
筆者はどうしようもなくなり、助けを求めようとしてメンバーにエラーメッセージの意味を説明しようとし始めると「あ！」と気づいて、すぐに解決できたことがあります。

カタカナを使わずにエラーメッセージを日本語に翻訳してみましょう。なにかに気づくかもしれません。

例えば、"Unauthorized" とエラーメッセージ中にあったとしましょう。
カタカナにすると「アンオウソライズド」です。聞き馴染みのある音なのでなんとなく意味はわかると思います。
しかし、デバッグ中になんとなくは危険です！

```
"Unauthorized" - without authority or permission
```


日本語にすると「権限が無い」という意味になります。「権限が無い」と「認証されていない」を勘違いしていませんでしたか？
2つの意味は大きく違います。正しく日本語でエラーメッセージを理解することで、あっけなくデバッグできることも多々あります。

おすすめ記事：[よくわかる認証と認可](https://www.merriam-webster.com/dictionary/unauthorized)

### 動作環境に問題がないでしょうか？
ローカル環境で実装したコードを開発環境、ステージング環境で動作を確認しようとしたらなぜかコードが動かない。訳のわからないエラーが出ている。この現象は、マイクロサービス構成を取っていると起こりやすいかもしれません。

あなたが実装したアプリケーションだけ最新のコードがデプロイされており、連携するアプリケーションのデプロイは1週間前で止まっている、開発途中でデプロイされたアプリケーションと連携している、かもしれません。
依存するアプリケーションのコードをすべて最新にしてデプロイしなおしてみましょう。

また外部のアプリケーションと連携する機能を試験したいときにローカル環境では動作しないということもあります。開発環境かステージング環境へデプロイしてみると、動くやないかい！ということもあるかもしれません。

つまり、あなたが実装したコードは悪くなかったということです。安心しましたね。

### デバッグ用のログは読みやすいですか？
デバッグのために動作結果を出力させることも多いと思いますが、それらは読みやすいものでしょうか？
面倒くさいからと言って読みにくいログを必死に読んでいませんか？
読みづらいログを必死に読んでも焦ってイライラしている精神状態では正しくログを読み取れないと思います。さらに焦ってイライラして時間を溶かす前にログを読みやすく整形しましょう。

改行させたり、特定の文字をハイライトさせたり、不要な文字を削除したり、読みやすくする工夫をしましょう。

テキストエディタを使って新しくファイルを作ってログを貼り付けて、見やすく、検索しやすくしています。

### いつから動作しなくなりましたか？
実装途中で動かなくなった場合、どの時点のコードなら動いていたのか認識することは非常に重要です。
`git`を有効活用して動作するコードと現在の差分を確かめましょう。なぜ動かないか気づけるかもしれません。

### ライブラリのコードを疑っていませんか？
ライブラリのコードが間違っていることもありますが、大抵の場合、あなたの使い方が間違っています。
あなたが疑っているコードは大抵の場合、動作は保証されていますので、使い方が間違っていることがほとんどです。
ライブラリのリポジトリのサンプルコードが掲載されていることも多いです。正しくライブラリを使うためにドキュメントを読みましょう。

あなたの使い方が絶対に間違っていない自信があるならプルリクエストを出しましょう。喜ばれるはずです。

### バージョンは正しいでしょうか？
公式ドキュメント通りに実装しているのになぜか動かないな、なんてことがあったらバージョンがあっているか確かめましょう。
必死になって試行錯誤していた時間が無駄だったことに気づくかもしれません。

ドキュメントに推奨バージョン等の記載が無くとも使っているプログラミング言語、ライブラリ、フレームワーク、ツールのバージョンをアップ、ダウンさせることは試す価値があります。

# 最後に

本記事は筆者がデバッグ時に役立つ観点をまとめたものになります。誰かのお役に立てれば嬉しいです。
また新しい観点を発見すれば更新していきたいです！