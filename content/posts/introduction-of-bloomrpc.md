---
title: "［実装不要で超手軽］gRPCサーバーの動作確認ができるGUI BloomRPC の紹介"
date: 2020-06-13T18:17:26+09:00
draft: false
categories: ["development"]
author: "Keisuke Umegaki"
---

# はじめに

本記事は、gRPCのサーバーを実装していた筆者が動作確認めんどくさいなー、gRPCクライアント実装しなくてもサーバーの動作確認ができるツール無いかなーと探していたら見つけたGUI `BloomRPC` の紹介です。

# BloomRPC の説明

https://github.com/uw-labs/bloomrpc

先述通り、gRPCのクライアントの実装をしなくてもgRPCサーバーの動作確認が行える便利なGUIです。APIのテストで有名な[Postman](https://www.postman.com/)やFacebookが開発したWeb API規格[GraphQL](https://graphql.org/)の動作確認に使われる[GraphQL Playground](https://github.com/prisma-labs/graphql-playground) に影響を受けているようです。

## 特徴

`README`を翻訳しただけです。

- ネイティブ`gRPC`呼び出し
- `UnaryCall`と`Server Side Streaming`のサポート
- クライアント側と双方向ストリーミング
- 自動入力認識
- マルチタブの操作
- メタデータのサポート
- 永続的なワークスペース
- キャンセルのリクエスト

## インストール方法

mac だと `homebrew` でインストールできます。

```
brew cask install bloomrpc
```

また `homebrew` が無い環境だと、クローンして`yarn install`して`npm run`すれば起動します。
詳しくは、[こちら](https://github.com/uw-labs/bloomrpc#build-from-source)を参照してください。

## 使い方

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F275587%2F32c00547-24de-8ce5-325a-c957b27ce9f4.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&s=05a58d8d6e3b47489394f189a5f96304)

1. gRPCサーバーを起動して希望のポート番号で待ち受ける。
2. BloomRPCを起動する。
3. GUIの画面左上にある＋ボタンから`.proto`ファイルをインポートする。
4. 1で起動したgRPCサーバーのアドレスを入力。（例）`http://localhost:3000` `127.0.0.1:8080`
5. 4でインポートした`.proto`ファイルにあるリクエストを選択すると画面左側にクエリが出力される。
6. 希望のパラメータを入力して画面中央の▶ボタンをクリックする。
7. 期待通りのレスポンスか、画面右側を確認する。

GIFのほうがイメージがわくと思うので、[こちら](https://github.com/uw-labs/bloomrpc#preview)を見てください。

# 最後に

今まで、gRPCのクライアントをわざわざ動作確認のためにサンプルのアプリケーションを実装していた僕には革命的に便利でした。
gRPCサーバーの動作確認がめんどくさいなと思っている方がいたらぜひ試してください！
