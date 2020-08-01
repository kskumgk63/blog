---
title: "［Dockerfile］RUN, CMD, ENTRYPOINT, Shell, Exec形式の違いと使いみちをまとめました"
date: 2020-08-01T17:07:40+09:00
draft: false
---

# RUN

イメージをビルドするときに使います。
ビルドされる環境（Google Cloud Build とか）で行いたい処理を記述しましょう。

例えば、アプリケーションをビルドするために必要なパッケージを取得する、アプリケーションをビルドするとかです。


```dockerfile
RUN apk update && apk add gcc g++

WORKDIR /go/src/app
COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /go/bin/app -ldflags="-s -w" .
```

# CMD, ENTRYPOINT

コンテナが起動するときに実行されます。
コンテナが動作する環境（K8s, Cloud Run とか）で行いたい処理を記述しましょう。

例えば、アプリケーションの起動です。

```dockerfile
ENTRYPOINT ./app
```

```dockerfile
CMD ./app
```

## どっちがいいの？

`ENTRYPOINT` → 実行可能なDockerイメージをビルドする場合、常に実行されるコマンドが必要な場合
`CMD` → Dockerコンテナの実行時にコマンドラインから上書きされる可能性のあるデフォルトの引数を追加する必要がある場合

# Shell, Exec

`RUN` `CMD` `ENTRYPOINT` は、いずれもコマンドを実行するために使用されます。
ただし、実行するコマンドを指定する方法は2通りあります。

## Shell

こんな書き方をします。

```dockerfile
RUN yarn run build
```

コマンドは新しいシェルプロセスの中で実行され、デフォルトでは Linux では /bin/sh -c、Windows では cmd /S /C となっています。
Shell形式は、変数置換のようなShell処理機能を使用したり、複数のコマンドを連結したりすることを可能にするために存在します。

## Exec

こんな書き方をします。

```dockerfile
RUN ["yarn", "run", "build"]
```

コマンドは新しいシェルプロセス内では実行されません。

## どっちがいいの？

複数のコマンドを連結しない限り、基本的に Exec を使いましょう。

# 参考資料

- [Shell versus exec forms](https://www.oreilly.com/library/view/building-enterprise-javascript/9781788477321/d5fcf845-e238-460f-9a10-041d44fb3d50.xhtml)
- [Dockerfile: ENTRYPOINT vs CMD](https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/)
