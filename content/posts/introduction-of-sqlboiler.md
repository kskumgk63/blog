---
title: "［超便利］Go ORMライブラリのSQL Boilerを使った感想"
date: 2020-06-14T16:13:50+09:00
draft: false
categories: ["programming language"]
tag: ["golang"]
author: "Keisuke Umegaki"
---

# はじめに

Go ORM ライブラリ [SQL Boiler](https://github.com/volatiletech/sqlboiler)を見つけて、コードを自動生成してくれたり、`gorm` `xorm` よりもパフォーマンスが優れているということを聞いたので使ってみました。

具体的な使い方は、サンプルアプリケーションを作ったので、そのソースコードを見てください。

https://github.com/kskumgk63/sqlboiler-example

もしくは、こちらのブログが非常に参考になりましたので参照ください。

https://note.crohaco.net/2020/golang-sqlboiler/

# SQL Boiler の説明

[SQL Boiler](https://github.com/volatiletech/sqlboiler) とは、データベーススキーマに合わせたGo ORMを生成してくれるライブラリです。
「データベーススキーマに合わせた」とある通り「データベース優先」のORMなので、まず初めにデータベーススキーマを作成して、それから`sqlboiler`コマンドでコードを生成します。
コード優先である`gorm` `xorm` とは、大きく違いますね。

パフォーマンスは、他の有名ORMライブラリに比べて圧倒的に良いです。
https://github.com/volatiletech/sqlboiler#benchmarks

# SQL Boiler のコンセプト

> we set out with these goals:
> Work with existing databases: Don't be the tool to define the schema, that's better left to other tools.
> - ActiveRecord-like productivity: Eliminate all sql boilerplate, have relationships as a first-class concept.
> - Go-like feel: Work with normal structs, call functions, no hyper-magical struct tags, small interfaces.
> - Go-like performance: Benchmark and optimize the hot-paths, perform like hand-rolled sql.DB code.

README を下記に翻訳しました。

> 私たちは以下の目標を掲げました。
> 既存のデータベースを利用する。スキーマを定義するツールではなく、他のツールに任せた方が良い。
> - ActiveRecordのような生産性。SQLの陳腐化を排除し、リレーションシップをファーストクラスのコンセプトとする。
> - Goのような感じ。通常の構造体で動作し、関数を呼び出し、超巨大な構造体タグを使用せず、小さなインターフェイスで動作します。
> - Goライクなパフォーマンス。ベンチマークとホットパスの最適化を行い、手書きのsql.DBコードのようなパフォーマンスを実現します。

# 使ってみた感想

下記のデータベーステーブルを作成して、`articles`を一覧表示したり、新しく`article`を作成する簡単なアプリケーションを作成してみました。

```sql
CREATE TABLE articles (
    id BIGINT PRIMARY KEY,
    title character varying(200) NOT NULL,
    content TEXT NOT NULL,
    created_at timestamp NOT NULL,
    updated_at timestamp NOT NULL,
    deleted_at timestamp NULL
);
```

ソースコードはこちら。
https://github.com/kskumgk63/sqlboiler-example

感想は、めっちゃ便利だったので今すぐに本番採用したいレベルです。今まで地道にデータベーステーブルに合う`struct`を書いていた時間がもったいないとすら思いました笑
一応、注意点としては、いろんな人のブログとかを見ていると複雑なクエリを書こうとすると生のSQLを書く必要が有るようです。

# 最後に

`SQL Boiler`を使ってサンプルアプリケーションを作ってみて非常に効率よく開発できたと感じました。みなさんも触ってみてください！
