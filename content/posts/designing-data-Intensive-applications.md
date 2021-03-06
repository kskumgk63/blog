---
title: "データ指向アプリケーションデザインをまとめた"
date: 2020-04-12T21:44:11+09:00
draft: true

categories: ["application design"]
author: "Keisuke Umegaki"
---

# データ指向アプリケーションデザインをまとめた

明日からの行動が変わりそうかどうかという視点で各章の重要な点を書き出し備忘録として使います。

# 第1章：信頼性、スケーラビリティ、メンテンス性に優れたアプリケーション

今日の多くのアプリケーションは演算指向ではなく、**データ指向**なので、データの量や複雑さ、そしてデータの変化する速度の方が大きな問題です。

数多くのアプリケーションのデータ処理やストレージに対する要求が幅広くなっており、単一のツールではそれらの要求を満たせないことが多いです。
なので、処理を複数のタスクに分散し、下記にあるように機能毎に最も適したツールで効率よく処理できるようにした上で、それらをアプリケーションのコードで結びつけます。

#### データベース
データを永続化して、そのアプリケーション自身、もしくはその他のアプリケーションが再度データを見つけられるようにする機能。

#### キャシュ
処理用の多い操作の結果をメモし、読み取り速度を高める機能。

#### 検索インデックス
データをキーワードで検索、もしくは様々な方法でフィルタリングできる機能。

#### ストリーム処理
他のプロセスへメッセージを送り、非同期に処理をする機能。

#### バッチ処理
蓄積された大量のデータを定期的に処理する機能。

## データシステムに関する考察

今やエンジニアはアプリケーション開発者であるだけでなく、よりレベルの高い要求に答えるため複数のツールを束ねるためデータシステムの設計者でもあります。
そのデータシステムの設計では数多くの難題が発生しますが、本書では３つの課題に焦点を当てます。

1. 信頼性
   システムは障害が発生したとしても正しく動作するべきです。
2. スケーラビリティ
   システムの成長（データ量、トラフィック量、複雑さ）に対して無理のない方法で対応可能であるべきです。
3. メンテンス性
   時間が経つにつれてシステムには多くの様々な人々が後から参入してきたとしても生産性を上げれるように関われるべきです。

では、3つについて詳しく見ていきましょう。

## 1. 信頼性

信頼性とは、「何か問題が発生したとしても正しく動作し続けること」です。その誤った動作を引き起こす原因を表すものが次の単語です。

**フォールト**：仕様を満たしておらず問題を起こしうるものです。

**障害**：システムが全体として必要なサービスの提供を止めてしまった場合を指します。障害をゼロにすることは不可能と言われています。

### ハードウェアの障害

ハードウェアの寿命は10年から50年と言われており、10000台のディスクを持つストレージクラスタでは、1日に1回はディスクが壊れることを予想すべきです。

なので、ハードウェアの障害が発生してもダウンタイムが発生しないシステム設計にすべきです。

### ソフトウェアのエラー

ハードウェアの障害はそれぞれ独立しているので、1つのマシンが壊れたからと言ってもう1台も連鎖的に壊れることは考えにくいです。
しかし、ソフトウェアのエラーは、ノード間で相関関係を持っている（独立していない）ことから大規模なシステム障害に繋がりやすいです。
さらにこういったソフトウェア障害を引き起こすようなバグは、ある条件が重なって初めて見つかるので、長い間潜伏していることもよくあります。

解決策としては、徹底したテスト、プロセスの分離、プロセスのクラッシュと再起動の許容、計測、モニタリングなどがあります。

### ヒューマンエラー

**「最大限に努力しても人間には信頼性が無いことが知られています。**

大規模なサービス障害として最も多いのは人間の設定ミスです。（ハードウェアが原因になるケースは全体の10~25%に過ぎないです）
対策としては次が挙げられます。

#### エラーの可能性が最小限になるようにシステムを設計する
うまく抽象化されたAPI、管理インターフェイスは「正しいこと」を行いやすく、「間違ったこと」を行いにくくします。

#### 人間が最も間違いやすそうな部分を分離する
完全な機能をもったサンドボックス環境をプロダクション環境とは別に用意して実際のユーザーに影響すること無く開発を行えるようにすべきです。

#### ユニットテスト、システム全体の結合テスト、マニュアルテストまで徹底して行う
それらを自動化することで確実に品質を上げることができます。

#### 障害発生時のインパクトを最小化しましょう
設定変更のロールバックを即座にできるようにしたり、新しいコードのロールアウトは徐々に行ったりすることで、予想外のバグがあっても影響されるユーザーを一部に留めることができます。

#### 詳細で明確なモニタリングをしましょう

## 2. スケーラビリティ
スケーラビリティに関する議論をするには正しく課題を理解する必要があります。次に説明するように負荷とパフォーマンスを正しく表現しないと適切な議論は行なえません。

### 負荷の表現
システムの負荷を簡潔に表現できなければ成長に対する問いに関して議論ができるようになります。
最適な負荷のパラメータはシステムによって異なりますが、一般的に以下があります。

- 毎秒のリクエスト数
- DBの読み書き比率
- 同時アクティブユーザー数
- キャッシュのヒット率

### パフォーマンスの表現
負荷の表現ができるようになると、負荷が増大したときに起こることを調査することができるようになります。
調査方法としては2つあります。

1. リソースを増やさずに負荷をかけたときのパフォーマンスはどう変化するか
2. どのくらいリソースを増やせば現状のパフォーマンスを維持できるか

#### レスポンスタイムとレイテンシの違い

レスポンスタイム：クライアント絡みた値で、リクエストの処理そのものに費やされた時間
レイテンシ：リクエスが処理を待っている期間

#### レスポンスタイムの測り方
レスポンスタイムは、毎回異なり振り幅が大きいので、**パーセンタイル | percentile**がパフォーマンスを計測するのに有効です。

ユーザーの典型的な待ち時間を測定したい場合：レスポンスタイムのリストを作成、ソートし、その中央値を見ます。

外れ値がどれくらい悪いか知りたい場合：95, 99, 99.9パーセントとその他を比べます。大きなパーセンタイルのレスポンス値は**テイルレテンシ**と呼ばれます。

Amazonでは、99.9パーセンタイルを使っています。1000リクエスト中の1つにしか影響が無い数字ではありますが、これを測定することで、大量に購入しているユーザーのレスポンスが遅いことが明確となりました。
大量に商品を購入するユーザーは、会社にとって利益が大きいので、そのユーザー体験を良くすることはビジネス的に価値があります。またAmazonはレスポンスタイムが100ミリ秒下がると売上が1%下がることを観測しています。
また、1秒の遅延が顧客満足度を16%も下げることを報告している会社もあります。

### 負荷への対処アプローチ
負荷を表現することができ、パフォーマンスを正しく測定できるようになるとスケーラビリティに関して議論をすることができます。

アプローチに関しては大きく2つの方法で議論されることが多いです。

#### スケールアップ
垂直なスケーリング。マシンスペックを強力にします。
単一のマシン上で動作するシステムの方がシンプルではありますが、ハイエンドなマシンは極めて高価なので、
極端に集中的なワークロードには適さないことがありますが、小さな仮想マシンを大量に使うよりも少ない数の強力なマシンを使ったほうがコストを下げられることもあります。

#### スケールアウト
水平なスケーリング。複数の比較的小さなマシンに負荷を分散します。
こちらは**シェアードナッシングアーキテクチャ**とも呼ばれます。
これを実現するには、サービスを細かく独立させる必要があります。

## 3. メンテナンス性
レガシーシステムのメンテンスは多くの人にとって嫌な仕事です。メンテンスの際の苦痛を最小化するには、3つの設計原則を守る必要があります。

1. 運用性：システムをスムーズに動作させ続けるために、運用チームが扱いやすいようにしましょう。
2. 単純性：新しいエンジニアがシステムを理解しやすいようにしましょう。システムから複雑性を取り除きましょう。
3. 進化性（拡張性）：要求の変化にともなって生じる予想外のユースケースに対応できるようにしましょう。

### 1. 運用性

優れた運用チームが追う責任
- システムの健全性をモニタリングして、状態が悪くなれば素早く改善する
- システムの障害、パフォーマンスの低下の問題原因を早期発見する
- システム同士の相互作用に注目し、問題が起こる前に回避する
- デプロイメント、設定管理などのための優れた習慣やツールを確立する
- 運用で予想外のことがおこらないようにプロセスをチギし、プロダクション環境の安定を保つ

### 2. 単純性
小さなプロジェクトはとてもシンプルでありますが、大きく成長するにつれてコードは複雑になり、巨大な泥の玉と表現されるまでになることが多々あります。

複雑さのためにメンテンスが困難になると予算とスケジュールが超過し、変更の際に新たなバグが混入するリスクも高くなります。
逆に言うと複雑さを取り除くことでメンテンス性は大きく改善されるので、単純性の追求をするべきです。

**多くの場合、複雑であることの原因は、ソフトウェアが解決しようとしている（ユーザーからみた）問題ではなく、実装からのみ生じています。**

単純さを追求するのに最も優れた手段の1つが抽象化です。
実装の詳細を隠蔽することで幅広く再利用することが可能になります。抽象化されたコンポーネントの品質の向上はシステム全体をも向上させます。

### 3. 進化性（拡張性）
システムに対する要求は常に変化します。開発者は新しいことを学び以前は予想しなかったユースケースが発生したり、ビジネスの優先順位は入れ替わり、ユーザーからは新機能を求められ、依存していた外部の古いプラットフォームは新しいものに変わり、
システムの成長に伴ってアーキテクチャを変更しなければならないことはよく発生します。

この変化の激しい状況に対応する策として考えられたのがアジャイル開発です。対象とするスコープを小さく保ち再利用可能で単純な機能を積み上げていくことで状況の変化にすぐに対応可能な状態にしておきます。

# 第2章：データモデルとクエリ言語

## リレーショナルモデル
SQLは1970年に発案されたリレーショナルデータモデルに基づいており、データはリレーション（SQLではテーブルにあたる部分）として構成されます。
それぞれのリレーションは順序なしのタプル（SQLにおける行）の集合です。
リレーショナルデータベースは非常に汎用性が高く、ビジネスデータ処理を超え、幅広くユースケースに対応できることが知られています。

### オブジェクトとリレーショナルのミスマッチ
多くのアプリケーション開発はオブジェクト指向言語で行われているため、リレーショナルデータベースに保存する前にオブジェクトからテーブル、行、列に変換するレイヤーが必要になってしまいます。
このアプリケーションモデルとデータベースモデルとの断絶を**インピーダンスミスマッチ**と呼びます。

それを解決するのがJSONで保存する方法です。ほぼそれ自体で完結しているドキュメントである履歴書のようなデータ構造に適しています。
もしリレーショナルデータベースで履歴書のデータを取得しようとすると複数のクエリを実行し、従属しているテーブルを加味して結合する必要があります。しかし、JSONで保存されていれば、関連情報は一箇所に集中しているので一度のクエリを実行するだけで良いです。

## NoSQL（ドキュメントデータベース）の誕生
2010年代に入ってリレーショナルモデルの支配を終了させるために誕生したのがNoSQLです。NoSQLを採用する理由には次があります。

- 巨大なデータセットや優れた書き込みのスループットを含む、リレーショナルデータベース以上のスケーラビリティが求められるようになったから。
- 商用のデータベース製品よりもフリーでオープンなソフトウェアが好まれるようになったから。
- リレーショナルモデルではうまくサポートされない特殊なクエリ操作が可能だから。
- リレーショナルなスキーマの制約に対するフラストレーションと、もっと動的で表現力に富むデータモデルに対する欲求があるから。

## リレーショナルデータベースとドキュメントデータベースの比較

### アプリケーションコードをシンプルにするデータモデルは？
アプリケーションのデータがドキュメントのような構造を持っている、一対多の関係からなるツリー構造を持っているなら、ドキュメントモデルを採用するのはいい考えです。
リレーショナルデータベースであれば、ドキュメントのような構造を複数のテーブルに分割して表現するので、スキーマが複雑になることが多いです。

しかし、ドキュメントモデルには制約があります。例えば、ドキュメント内にネストされたアイテムは直接参照できず、「ユーザー251のポジションのリストの2番目のアイテム」といった指定が必要になります。
つまり、ドキュメントデータベースは、結合のサポートが貧弱なのです。アプリケーションが多対多の関係を求めるならドキュメントモデルの魅力は薄れます。

### ドキュメントモデルにおけるスキーマの柔軟性
多くのドキュメントデータベース、そしてリレーショナルデータベースにおけるJSONサポートはドキュメント内のデータに対してスキーマを強制しません。
これは暗黙的にスキーマがあると想定しているので、データベースがそれを強制しないということです。これを**スキーマオンリード**と呼びます。
これはプログラミング言語の動的型チェックに似ています。

リレーショナルデータベースにおける伝統的なアプローチであり、スキーマは明示され、データベースは書き込まれるすべてのデータそのスキーマに従っていることを保証することを**スキーマオンライト**と呼びます。

これら2つのアプローチの違いが特に目立つのは、アプリケーションのフォーマットを変更するときです。例えば、フルネームで保存していたフィールドを姓と名に分けたいという要望が出てきたとします。
ドキュメントデータベースでは、新しいフィールドを使って、それ以降のデータを書き始め、古いフィールドを持つドキュメントを読んだ場合は、アプリケーションに変換の処理をもたせます。

一方で、リレーショナルデータベースでは、マイグレーショを実行することになります。

## グラフ型のデータモデル

多対多の関係を表すデータをアプリケーションで扱うときは、頂点（ノード、エンティティと呼ばれます）と辺（エッジや関係、弧と呼ばれる）の2つのオブジェクトから構成されるデータベースが最適です。
例えば、次のようなデータを表すときに有効です。

- ソーシャルグラフ：頂点が人、辺が知人関係を表します
- Webグラフ：頂点がWebページ、辺が他のWebページへのHTMLリンクを示す
- 道路や鉄道のネットワーク：頂点が接続点を辺が接続点を結ぶ道路や鉄道の路線を示す

# 第3章：ストレージと抽出
データベースによるストレージの扱いとデータの取り出しの最下層を学び、自分のアプリケーションに適したエンジンを選択できなければなりません。
最適なエンジンを選ぶためにストレージエンジンの違いを知っておく必要があります。
ストレージエンジンはトランザクション処理（OLTP）に最適化されたものと、分析（OLAP）に最適化されたものの2つに分類できました。
これらのユースケースには、アクセスパターンに大きな違いがあります。

### トランザクション処理（OLTP）
通常のユーザーが利用し、莫大なリクエストを受け付けることを考えなければなりません。
ディスクのシークがボトルネックになることが多いです。

### 分析処理（OLAP）
エンドユーザーではなく主にビジネスアナリストが使用するものなので、クエリの数はOLTPシステムに比べて少量ですが、クエリ1つ1つの負荷は大きく、短時間に数百万レコードをスキャンすることが求められます。
ディスクの帯域がボトルネックになることが多いです。

# 第4章：エンコーディングと進化

学んだことは特になし。

アプリケーションのアーキテクチャやアプリケーションのデプロイの選択肢にも影響するデータ構造をネットワーク上やディスク上のバイト列に変換する方法を学びます。

データフォーマットやスキーマが変更されるとしばしばアプリケーションのコードにも対応する変更が必要になります。（例えば、新しいフィールドにレコードを追加したら、アプリケーションのコードはそのフィールドの読み書きしはじめなければなりません）
しかし、大規模なアプリケーションでは多くの場合、コードの変更を即座に行うことはできません。
それに対応するデプロイメント方法が、ローリングアップグレードです。新しいバージョンのデプロイを一度に数ノードずつ行い、新しいバージョンに動作しているかをチェックしながら、徐々にすべてのノードに対して作業していくというやり方です。
こうすることで、サービスのダウンタイムなしに新しいバージョンをデプロイできるので頻繁なリリースがしやすくなり進化性が高まります。

# 第5章：レプリケーション

レプリケーションとは、ネットワークで接続された複数のマシンにい同じデータのコピーを保持しておくことを指します。

レプリケーションを行う理由は次です。
- レイテンシを下げるため、データを地理的にユーザーの近くで保持しておく。
- 可用性を高めるために一部に障害があってもシステムを動作し続けられるようにする。
- スループットを高めるため読み取り専用のクエリを処理するマシン数をスケールアウトする。

## リーダーとフォロワー
データベースのコピーを保存する各ノードは、レプリカと呼ばれます。複数のレプリカが存在する場合、すべてのデータがレプリカに行き渡っていることを保証するにはどうすればいいでしょうか？
データベースへのすべての書き込みは、すべてレプリカで処理されてなければなりません。それができない場合は、レプリカが持つデータに差異が生じているかもしれません。

これの最も一般的な解決策は、**リーダーベースレプリケーション**と呼ばれるものです。

動作は以下の通りです。

1. レプリカは1つのリーダー（マスターあるいはプライマリと呼ばれます）に指定されます。DBに書き込みをしたい場合、クライアントはリクエストをリーダーに送信します。リーダーはローカルストレージに書き込みます。
2. 他のレプリカはフォロワー（リードレプリカ、スレーブ、セカンダリ、ホットスタンバイ）と呼ばれます。リーダーはデータをローカルストレージに書くと、その変更をレプリケーションログ、あるいは変更ストリームの一部としてすべてのフォロワーに送信します。
   各フォロワーは、リーダーからログを受け取り、その内容に従ってリーダー上で処理されたのと同じ順序ですべて書き込みを行います。
3. クライアントがDBから読み取りをしたい場合には、リーダーあるいはいずれかのフォロワーにクエリを送ることができます。ただし書き込みを受け付けられるのはリーダーのみです。

こういったレプリケーションの動作は、PostgresSQL、MySQL、MongoDBなど主要なデータベースに組み込まれています。

## フェイルオーバー
リーダーが障害を起こした場合、フォロワーのいずれかをリーダーに昇格させる必要があります。
クライアントは新しいリーダーに書き込み先を変える必要があり、他のフォロワーは新しいリーダーからデータの変更を受信しなければなりません。このプロセスをフェイルオーバーと呼びます。

自動的なフェイルオーバーは以下のプロセスからなります。

1. リーダーに障害が起きたことの確認。（例）一定時間応答がなければ障害が発生していると考える。
2. 新しいリーダーの選出。次のリーダーになるのは通常、データ変更に一番最近まで追従していたレプリカです。
3. 新しいリーダーを使用するためのシステムの再設定。クライアントの書き込み先を変える。さらに元のリーダーにも変わったことを伝える。復活したときにフォロワーになったことを知らないでリーダーの挙動をすることがある。

## マルチリーダーレプリケーション
シングルリーダーの欠点である書き込み先が単一になるという欠点を補うのがこのアーキテクチャです。
しかし、得られるメリットが少なく、複雑性が増すので単一のデータセンター内でマルチリーダー構成を使う意味は殆ど有りません。

### シングルリーダーとマルチリーダーの動作比較

#### パフォーマンス
シングルリーダーはすべての書き込みが単一のリーダーへインターネットを通じてリクエストされるので大きなレイテンシが発生することがあります。
対してマルチリーダーは分散してリクエストを受け付けて、非同期で結果整合性を保つので、ユーザーから見るとパフォーマンスは向上するかもしれないです。

#### データんセンターの障害
マルチリーダーは独立して別々のデータセンターで稼働するのでシングルリーダーより障害耐性があるのは間違いないです。

## マルチリーダー構成が適しているユースケース

### オフラインでもアプリケーションが動作する必要があるとき
モバイルアプリなどオフラインでも書き込みのリクエストを受け付ける必要がある場合は、デバイスがリーダーのように振る舞い、次にオンラインになったときに非同期でデータの整合性を保つ必要があります。

### 共同編集可能な場合
オンラインで共同編集が可能な場合、デバイスに即座に反映され、サーバー及びドキュメントを編集している他のユーザーに非同期にレプリケーションされます。
編集の衝突がないことを保証したいのならドキュメントを一定の単位でロックしなければなりません。

## 書き込みの衝突回避
マルチリーダー構成の最大の問題は書き込みの衝突です。
書き込みの衝突を扱う最もシンプルな解決策は、衝突を回避することです。あるレコードに対する書き込みが同じリーダーに送られることをアプリケーションが保証できるなら衝突が生じることは有りません。

## データの一貫性
最後の書き込みを勝者とするケースが多いです。データの損失が大きくなる可能性があります。

## リーダーレスレプリケーション
Amazon Dynamo DB が採用しているアーキテクチャであり、リーダーという概念を捨ててすべてが書き込みを受け付けるというものです。Dynamoアーキテクチャと呼びます。

このアーキテクチャでは、書き込み、読み取りのリクエストをすべてのノードに対して並列に送信します。
書き込みが2/3しかされなかったとしても1/3は無視します。最終的には全てに同期されるのですが、読み取りのタイミング次第では、それぞれが違うデータをレスポンスするときがあります。
そのときは、バージョンを見ることで一番新しいものをレスポンスすることができ、さらに古いバージョンを持つものには新しいものを適応させる動きができます。

また**反エントロピー処理**といってレプリカ間のデータの差異を探して整合性を保つバックグラウンド処理を行うこともあります。

運用上、常に最新のデータをレスポンスできているかをモニタリングする必要があります。

# 第6章：パーティショニング
パーティションではそれぞれがデータの断片（各レコード、行、あるいはドキュメントが厳密に1つのパーティションに属するものと定義されます。

データをパーティショニングする主な理由は**スケーラビリティ**です。それぞれのパーティションはシェアードナッシングクラスタの別々のノードに配置できます。
したがって大規模なデータセットの数多くのディスクに分散配置でき、クエリの負荷を大量のプロセッサに分散させることができます。またノードは独立にパーティションに対してクエリを実行できるので、スループットをスケールさせられます。
パーティショニングはそれぞれのパーティションが複数のノードに保存されることからレプリケーションと組み合わされます。それぞれのレコードが厳密にパーティションに属しているとはいえ、耐障害性をもたせるために複数のノードに保存されるかもしれないということです。
つまり、それぞれのノードは、あるパーティションのリーダーであるとおもにフォロワーでもあります。

パーティショニングの目標は、データとクエリの負荷をノード間で均等に分散させることです。
理論上は、10個のノードがあれば1つの負荷は1/10になるはずですが、集中してデータやクエリを受け取ってしまうノードが発生することがあります。
この状態を`スキュー skew`と呼ばれます。
スキューであるとパーティショニングの効果は大きく損なわれます。すべての負荷が1つのノードに集中してしまうことをホットスポットと呼びます。

ノードをランダムに割り振ることでスキューを避けますが、あるアイテムを読み取ろうとするとどこのノードにあるか調べるために並列にクエリを実行しなければなりません。
そのためパフォーマンスを上げるためにキーバリュー型にすることで常にレコードにはプライマリキーにアクセスすることになります。
さらにパフォーマンスをあげるためにパーティションごとのキーの範囲をソートするなどして調整します。そうすることで、検索時間を更に減らすことができます。

そのキーバリュー型のキーを作成するときに用いられるのがコンシステントハッシュ法と呼ばれるものです。このアルゴリズムを使うことで非常に入力された文字列が似ていても全く違うハッシュ値を生成することができます。

## セカンダリインデックス
上記で紹介したキーバリューストアは、プライマリキーだけであることを前提としていましたが、セカンダリインデックスを用いることでプライマリキーだけでなく特定の属性値で絞る事ができます。
しかし、実装がかなり複雑になります。

（例）中古車販売のWebサイト
車のデータはID 0 ~ 499, 500 ~ 999 といった具合にパーティショニングされています。
セカンダリインデックスに `color` を指定することで、さらに `red` など色の指定で高速に検索ができることになりました。

セカンダリインデックスは読み取りが負荷になってしまうことがあるデメリットがあります。実装が複雑なだけでなく、例でいうところの `red` の指定があると結局すべてのノードに対して並列にクエリを実行し結果を結合する必要が発生します。

## パーティションのリバランシング
時間が立つにつれてデータベースの中では変化が発生します。

- クエリのスループットが増大し、CPUを追加して負荷に対処する
- データベースのサイズが大きくなるので保存のためにディスクやRAMを追加する
- マシンに障害が発生し、他のマシンがそのマシンが受け持っていた処理を肩代わりすることになる

上記の変化が生じた場合、あるノードから別のノードへのデータやリクエストの移動が必要になります。負荷をクラスタ内のあるノードから別のノードへ移行するプロセスをリバランシングと呼びます。

- リバランシング終了後、負荷（データストレージ、読み書きリクエスト）はクラスタ内のノード間で公平に分配されていなければならない
- リバランシングが行われている間、データベースは読み書きを続けなければならない
- ノード間を移動させるデータは必要最小限にとどめ、リバランシングが高速にお行われ、ネットワークやディスクIOの負荷が最小になるようにする

### リバランスの戦略

#### パーティション数の固定

#### 動的なパーティショニング

#### ノード数に比例するパーティショニング
