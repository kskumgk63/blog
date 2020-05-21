---
title: "Githubアカウント切り替える方法（鍵作成〜アカウント切り替え）"
date: 2020-04-09T22:14:07+09:00
draft: false
categories: ["github"]
---

# はじめに

早速質問ですが、Githubのアカウントを会社用とプライベートなアカウントを持っている方は多いと思うのですが、
どうやってアカウントの切り替えをしていますか？

今回は、鍵の作成・登録から仕事用とプライベート用のアカウントの切り替え方法を可能な限り丁寧にお届けしたいと思います。

# 記事の構成

#### ① ssh用の公開鍵と秘密鍵の作成

Githubとssh通信するために必要な認証情報を作成します。
秘密鍵は絶対に人と共有してはいけません。

Githubのリポジトリと通信してプルやプッシュとかやりとりするために必要です。

#### ② 公開鍵と秘密鍵をgithubへ登録

##### 鍵のイメージ

銀行口座に例えると、

**公開鍵は口座番号です。お金を振り込んでもらうために公開します。**
**秘密鍵はATMを操作するときに使用する暗証番号です。自分1人しか知りません。**

#### ③ 鍵とGithubアカウントを紐付ける

#### ④ アカウントを切り替えて git clone する

#### ⑤ ssh-agent に鍵を登録する（パスワードの入力を省略する）

# ① sshの公開鍵と秘密鍵の作成

まずはsshを管理しているディレクトリへ移動します。

```bash
cd ~/.ssh
```

会社用、プライベート用の
公開鍵と秘密鍵を早速作成します。

`ssh-genkey` というコマンドを使用します。

```
ssh-keygen [オプション] -t 鍵タイプ [-N 新しいパスフレーズ] [-C コメント] [-f 鍵ファイル]
ssh-keygen -p [-P 古いパスフレーズ] [-N 新しいパスフレーズ] [-f 鍵ファイル]
ssh-keygen -i [-f 鍵ファイル]
ssh-keygen -l [-f 鍵ファイル]
```

下記を実行するとパスワードが求められます。
この鍵を使用してsshをするとき（`git push`）とかにこのパスワードを使用します。
また `-f`に指定した名前で２つファイルが作成されています。
`hoge` `hoge.pub` こんな感じで。

なので、 `-C` 会社、プライベート用のメールアドレス `-f` ユーザー名 とかが管理しやすいかもですね。

```
ssh-keygen -t rsa -b 4096 -C your_email@example.com -f file_name
```

ちなみにこれでも鍵は作成できます。

```
ssh-keygen -t rsa
```

すると `id_rsa` `id_rsa.pub` という鍵ができますが、後で何の鍵かは忘れるし、パスワードを設定しないためセキュアではないのでオプションつけて作成することをおすすめします。

詳しくは、この記事が非常に参考になります。

[お前らのSSH Keysの作り方は間違っている](https://qiita.com/suthio/items/2760e4cff0e185fe2db9)


# ② 公開鍵と秘密鍵をgithubへ登録

https://github.com/settings/ssh

Githubのアカウントを持っていることが前提で、ここから鍵を登録できます。

<img width="1397" alt="screenshot 2020-01-11 13.53.47.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/275587/633e08e8-a3f6-5012-1700-ee8ef91953a7.png">


<img width="1397" alt="screenshot 2020-01-11 13.58.12.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/275587/4ed6c1dc-2364-4526-ef8e-e2d095557e93.png">

Title に後から見てもなんの為にSSH鍵を登録したかわかる名前を付けます。
Keyには公開鍵を貼り付けます。

まず公開鍵をコピーします。

`pbcopy` はクリップボードにコピーしてくれます。
言い換えると選択して、`⌘ c` するのと同じです。

そして `ssh-keygen` して作成したファイル名を選択してください。

**会社、プライベートの鍵を混同して Github に登録しないように気をつけてください。**

```
pbcopy < ~/.ssh/file_name.pub
```

# ③ 鍵とGithubアカウントを紐付ける

① で作成した鍵とGithubのアカウントを紐付けます。

先程と同じで鍵を管理しているディレクトリへ移動します。

```
cd ~/.ssh
```

`config` というファイルを作成してここへ認証情報を書いていきます。

vim がわからないという方は VS Code とか使いやすいテキストエディタで編集してください。

```
vim config
```

```
#------------------------------------
# 仕事
#------------------------------------
Host github.com
  HostName github.com
  User git
  Port 22
  HostName github.com
  IdentityFile ~/.ssh/file_name
  TCPKeepAlive yes
  IdentitiesOnly yes
#------------------------------------
# プライベート
#------------------------------------
Host github-private
  HostName github.com
  User git
  Port 22
  HostName github.com
  IdentityFile ~/.ssh/file_name
  TCPKeepAlive yes
  IdentitiesOnly yes

```

僕はこんな感じで管理してます。
`Host` で指定した文字列を `git clone` の中に含めることで `IdentityFile ` で指定した鍵を使ってくれます。

`#仕事` 用のアカウントで `clone` したいときは、

```
              __________
git clone git@github.com:username/repository.git
```

`#プライベート`のアカウントで `clone` したいときはこうです！

```
              _______________
git clone git@github-private:username/repository.git
```

`git clone` で ssh 通信できているかテストするのは微妙なので、 `ssh` コマンドを使用しましょう。
`-T` で指定するのは接続したいアカウントの `Host` を指定しましょう。

`#仕事`

```
ssh -T github.com
```

下記のメッセージがレスポンスされたらOKです。

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

`#プライベート`

```
ssh -T github-private
```

下記のメッセージがレスポンスされたらOKです。

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

# ④ アカウントを切り替えて git clone する

① ② ③ で作成した認証を使ってアカウントを切り替えてみたいと思います。

僕の場合は、デフォルトを仕事用のGithubアカウントにしているので普通に仕事用のアカウントでクローンしてしまいます。

```
git clone git@github.com:hoge/hoge.git
```

このままだとプライベート用のアカウントが招待されたリポジトリはクローンできないし、 commit も仕事用のアカウントでされてしまいます。

なので、 ① ② ③ で作成した認証情報を使ってクローンします。

```
git clone git@github-private:hoge/hoge.git
```

パスワードを入力すればクローンされると思います。

でもそれだけだとまだアカウントは変わっていません。

リポジトリ内の `.git/config` を書き換える必要があります。

まずはリポジトリ内の認証情報を確認。

```
git config --local -l
```

この結果の `user.name` `user.email` がプライベート用ではない、もしくは空なら書き換える必要があります。

```
git config --local user.name hoge
git config --local user.email hoge@gmail.com
```

これでプライベート用のアカウントからリポジトリの操作ができます。

# ⑤ ssh-agent に鍵を登録する（パスワードの入力を省略する）

① ② ③ で作成した鍵を使ってアカウントの切り替えができるようになりましたが、面倒なのがパスワードを毎回入力することです。そこで登場するのが `ssh-agent` です。

### ssh-agent とは

簡単に説明すると、公開鍵認証方式による認証を行っているSSHサーバへ接続する際、秘密鍵に設定されているパスフレーズの入力を代わりにやってくれるアプリケーションです。（秘密鍵とパスフレーズはメモリ上にキャッシュされます）

[Githubの公式ヘルプページ](https://help.github.com/ja/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)にも ① ② ③ で作成した ssh鍵を ssh-agentに登録する方法が掲載されています。

###  手順

バックグラウンドでssh-agentを開始

```
$ eval "$(ssh-agent -s)"
> Agent pid 59566
```

③ で編集した `~/.ssh/config` を編集

```
Host github.com
  HostName github.com
  User git
  Port 22
  HostName github.com
  IdentityFile ~/.ssh/file_name
  TCPKeepAlive yes
  IdentitiesOnly yes
# --- 下記2行を追加 ------
  AddKeysToAgent yes
  UseKeychain yes
```

SSH 秘密鍵を ssh-agent に追加して、パスフレーズをキーチェーンに保存

`hogehoge` の部分は ① で作成した鍵ファイル名になります。

```
ssh-add -K ~/.ssh/hogehoge
```

これで `ssh-agent` に追加することができました！！
リモートサーバーとの通信でパスワードが求められることがなくなるはずです。

# 最後に

今回、鍵の作成からアカウントを切り替えるまでの手順を紹介しました。
ちなみに僕は ④ の操作を bashスクリプト を使って 1コマンドで済むようにしました。

そのスクリプトは個人用で汎用的ではないので公開できませんが、皆さんも書いてみてください。

作業効率が全く違いますよ！！
