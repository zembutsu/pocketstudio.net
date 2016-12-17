+++
date = "2016-06-21T04:16:00+09:00"
draft = false
slug = "docker-1-12-build-in-orchestration-translate"
title = "【参考訳】Docker 1.12: ついにオーケストレーションを組み込み！"
categories = ["Docker"]
tags = ["Docker", "Swarm","translation"]
+++

# 概要

DockerCon 2016 のキーノートで、新しい Docker 1.12 の機能に関する説明がありました。また、その発表に合わせるように [blog への投稿 "Docker 1.12: Now with Built-in Orchestration!" ](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/) もありました。例によって日本語訳を作成しましたので、内容把握の参考程度にどうぞ。

図版は省略しました。原文をおたどりください。なお、自分の整理用に以前のバージョンの比較用画像を作りましたので、こちらも参考程度にどうぞ。

[<img src="/images/docker-engine-112-swarm-mode.png" width="100%">](/images/docker-engine-112-swarm-mode.png)


## Docker 1.12: ついにオーケストレーションを組み込み！

　３年前、Docker は難解な Linux カーネル技術をコンテナ化（containerization）と呼ぶシンプルで誰もが利用しやすいものにしました。今日、私たちはコンテナのオーケストレーションも同じ様にします。

　単一ホスト上にコンテナを個々にデプロイしている状態から、複雑な複数のコンテナを多くのマシンにデプロイするスタイルに移行するために、コンテナ・オーケストレーション（container orchestration）が求められています。そのためにはインフラから独立した分散プラットフォームを必要とします。つまりアプリケーションのライフタイム（存続期間）全てにわたってオンラインを維持し、ハードウェア障害やソフトウェア更新時も生存し続けるようにします。今日のオーケストレーションは、３年前のコンテナ化と同じステージです。ここには２つの選択肢があります。１つは、あなたが技術の専門部隊と共に、複雑なアドホック・システムを作り上げること。そうでなければ、多くの専門家がいる会社を信頼するかです。専門家は、あなたの代わりに全てのハードウェア、サービス、サポート、ソフトウェアを購入し、全てを管理するのです。この状態を表す言葉があります。ロックイン（lock-in）と呼ばれます。

　Docker ユーザは他のオプションを受け入れ可能かどうか、私たちとずっと共有してきました。皆さんが必要なのは、オーケストレーションを誰もが使えるプラットフォームであり、ロックインがないものです。コンテナ・オーケストレーションは実行が簡単であり、よりポータブルであり、安全、弾力的（resilient）、そしてより速いことです。たとえ、プラットフォームに組み込まれていたとしてもです。

　Docker 1.12 から Docker Engine のコアに機能を追加し始めました。機能とは、マルチ・ホストとマルチ・コンテナのオーケストレーションを簡単にします。私たちは新しい API オブジェクトを追加しました。サービス（Service）とノード（Node）です。これは swarm と呼ばれる Docker Engine のグループ上で、アプリケーションのデプロイと管理に Docker API を使います。Docker 1.12 があれば、Docker をオーケストレートする最も良い方法こそが Docker なのです！

＜図＞

　Docker 1.2 は４つの原則をベースにしています。

* **シンプルでありながら強力（Simple Yet Powerful）**  - 最近の分散アプリケーションにおいて、オーケストレーションは中核部分です。そして、その中心にあたるものは、私たちが Docker Engine のコアへシームレスに組み込みました。私たちの手法はオーケストレーションでも私たちのコンテナに対する哲学を適用することです。哲学とは、セットアップ不要であり、シンプルな概念を学ぶのは最小限であり、そして「とにかく動く」（it just work）というユーザ経験です。

* **弾力性（Resilient）** - マシンは常に落ちます。最近のシステムではこれらの障害を予測すべきであり、恒常的に存在し続け、アプリケーションのダウンタイム（停止時間）なしに適用できることです。それが単一障害点（single-point-of-failure）をゼロにする設計が必要な理由です。

* **安全（Secure）**  - デフォルトで安全であるべきです。強力なセキュリティの防壁のためには、証明書の生成（certificate generation）、そのために PKI の理解が必要だったのですが、不要になるべきです。しかし、高度なユーザであれば、これまで通り全ての証明書への署名と発行を制御・監査し続けるべきでしょう。

* **オプション機能と後方互換性**  - 100 万人ものユーザがいるため、Docker Engine の後方互換性維持は必須です。新しい全ての機能はオプションであり、使わなければオーバヘッド（メモリや CPU に対する）を被る心配はありません。Docker Engine のオーケストレーションは私たちのプラットフォームの内蔵電池（batteries included）ですが、交換可能にしています。そのため、ユーザはサードパーティ製のオーケストレータを Docker Engine 上で使い続けられます。

　それでは Docker 1.12 で動く新しい機能を見ていきましょう。

## １つの分散構築ブロックで Swarm を作る

　始めるためには swarm （訳者注：「群れ」の意味で、Docker Engine のクラスタを指す言葉）を作成します。swarm は Engine を自己組織化（self-organizing）でき、自己修復グループ（self-healing group）します。ノードの立ち上げ（bootstrap）は次のようにシンプルです。

```
docker swarm init
```

　このフードの下では [Raft](https://raft.github.io/raft.pdf) コンセンサス・グループ（合意グループ）のノードを１つ作成しています。この１つめのノードはマネージャ（manager）の役割（role）を持っています。つまり、コマンドを受け付け、タスクをスケジュールします。swarm にさらにノードを追加したら、デフォルトではワーカ（worker）になります。ワーカとはマネージャが命令したコンテナを単純に実行します。オプションで、他にもマネージャ・ノードを追加可能です。マネージャ・ノードは Raft コンセンサス・グループの一部になり得ます。私たちは最適化した Raft ストアを使います。スケジューリング性能を高速にするため、メモリを直接読み込み処理します。

## サービスの作成とスケール

　docker run でコンテナを１つ実行しておけば、 Engine の swarm （クラスタ）上で docker service を使いプロセスの複製、分散、負荷分散を始められます。

```
docker service create --name frontend --replicas 5 -p 80:80/tcp nginx:latest
```

　このコマンドは nginx コンテナの期待状態（desired state）が５つであると宣言します。swarm 上のノードのポート 80 へのアクセスは、内部ロード・バランサ・サービスを通して、いずれかのコンテナに到達します。この動作の仕組みは [Linux IPVS](http://www.linuxvirtualserver.org/software/ipvs.html) を使ったカーネル内のレイヤ４マルチ・プロトコル・ロードバランサです。これは Linux カーネルに 15 年以上も前からあるものです。IPVS ルーティング・パケットはカーネル内であり、swarm のルーティングは高性能なコンテナ検出負荷分散（container-aware load-balancing）をもたらします。

　サービスの作成時、オプションで複製サービス（replicated services）かグローバル・サービス（global services）を作成できます。複製サービスとは、利用可能なホスト上に広くコンテナを多く展開する意味です。対照的にグローバル・サービスとは、sawrm 上の各ホストに同じコンテナを１つずつスケジュールします。

　それでは Docker が提供する弾力性について考えて見ましょう。Swarm モードは engine を自己組織化（self organizing）と自己修復（self healing）ができるようにします。つまり、engine は自分が定義した通りにアプリケーションに注意を払います。そのためには、継続的なチェックや、engine がどこかに行った時は調整を担います。たとえば、nginx インスタンスを実行中のマシンの電源コードを抜いても、別のノードで新しいコンテナが復活します。ネットワーク・スイッチから swarm クラスタ上の半分のマシンを引き抜いても、他のノードが引き継ぎ、残ったノードでコンテナを再分配します。更新時には、既存のサービスを変更するため、柔軟に再デプロイ（re-deploy）できます。swarm 上のコンテナはローリング（rolling）もしくは並列に更新できます。

　100 インスタンスにスケールアップしたいですか？　次のように簡単です。

```
docker service scale frontend=100
```

　典型的な２ティア（web+db）アプリケーションは次のように作成できます。

```
docker network create -d overlay mynet
docker service create --name frontend --replicas 5 -p 80:80/tcp \
    --network mynet mywebapp
docker service create --name redis --network mynet redis:latest
```

　これはこのアプリケーションの基本アーキテクチャです。

＜図＞

## セキュリティ

　Docker 1.12 の中心にある原則は、設定しなくてもデフォルトで安全であり、Docker プラットフォームをすぐに使えるように作ることです。管理者が度々直面する主な課題の１つは、プロダクションで実行中のアプリケーションを安全にデプロイすることです。Docker 1.12 では管理者がデモ用クラスタを確実にセットアップできるようにします。つまり、安全なプロダクションのクラスタをセットアップできるでしょう。

　セキュリティは後から締め直せるようなものではありません。それゆえ、 Docker 1.12 では難しい設定を行わなくても、 swarm 上の全てのノードが相互に TLS 認証が可能であり、認証し、暗号化通信を可能とするのです。

　１つめのマネージャを起動したら、Docker Engine は新しい認証機関（CA; Certificate Authority）を作成し、自分用の初期証明書セットを作成します。この初期ステップ以降は、swarm に参加する各ノードは、ランダムに生成した ID で自動的に新しい証明書を作成し、swarm （マネージャかワーカ）の各ロールに適用します。これらの証明書は swarm 上にノードが参加している間中、ノードを安全に暗号化して認識するために使います。そしてまた、マネージャが安全にタスクやその他の更新を伝えるためにも使います。

＜図＞

　TLS 採用の最も大きな障壁の１つは、公開鍵基盤（PKI; Public Key Infrastructure）の作成・設定・管理に必要なのが大変な点です。Docker 1.12 では全てが安全にセットアップされているだけでなく、設定済みです。そして、もう１つ TLS 証明書を扱う上で最もつらい証明書の更新も自動化しました。

　フードの下では、swarm に参加する各ノードは定期的に証明書を確実に更新します。そのため、潜在的な流出や、設定の妥協はもうありません。証明証の入れ替え（rotate）設定は、ユーザによって変更できます。最小間隔は30分です。

　自分の認証機関（Certificated Authority）を使いたい場合は、私たちは外部 CA モード（external-CA mode）もサポートしています。これは swarm にノードがリモート URL から参加する時、マネージャはノードの証明書署名要求（CSR; Certificate Signing Request）を単純に信頼しようとします。

## Bundle（バンドル）

　Docker 1.12 は [Distributed Application Bundle](https://github.com/docker/docker/blob/master/experimental/docker-stacks-and-bundles.md) という新しいファイル形式を導入しました（ experimental ビルドのみ）。Bundle （荷物のあつまり、束の意味）はフルスタック・アプリケーション上のサービスに集中した新しい抽象化です。

　Docker Bundle ファイルは宣言型のサービス・セットの仕様書です。仕様には次の命令があります。

* 何のイメージのリビジョンを実行するか
* 何の名前のネットワークを作成するか
* どのようにこれらサービスのコンテナを、どのネットワーク上で実行するか

　Bundle ファイルは完全にポータブルであり、ソフトウェア・デリバリのパイプラインにおいて、アーティファクト（訳者注：最終成果物の意味）の完全なデプロイをもたらします。これは、複数コンテナを Docker アプリとして記述した全てを移動し、バージョン化できるからです。

　Bundle ファイルの仕様はシンプルかつオープンです。必要であれば自分が欲しいバンドルを作成できます。使い始めるには、Docker Compose は bundle ファイルの作成に実験的に対応しました。そして Docker 1.12 の swarm モードを有効化したら、bundle ファイルをデプロイできます。

　Bundle は開発者のノート PC から CI を通したプロダクションに、複数サービスのアプリケーションを効率的に移動する仕組みです。実験的機能であり、私たちはコミュニティからのフィードバックをお待ちしています。

## Dokcer 1.12 のフードの下

　フードの下を見ますと、Docker 1.12 では他にも多くの面白い技術を使っています。ノード間通信には [gRPC](http://www.grpc.io/) を使っており、これにより HTTP/2 の多重接続やヘッダ圧縮のような利点を得られます。私たちのデータ構造は [protobufs](https://developers.google.com/protocol-buffers/) の効果的な移植です、感謝します。

　Docker 1.12 のリソースについては、他もご覧ください。

* [Docker 1.12 ・ Docker for Mac or Windowsのダウンロード](https://www.docker.com/getdocker)
* ネイティブな [クラウドの選択を試す：AWS or Azure](https://blog.docker.com/2016/06/azure-aws-beta/)
* チュートリアルを確認：https://docs.docker.com/engine/swarm/swarm-tutorial/tutorial-setup/
* swarm について：https://docs.docker.com/engine/swarm/
* bundle を試す：https://github.com/docker/docker/blob/master/experimental/docker-stacks.md

## 原文

* Docker 1.12: Now with Built-in Orchestration! | Docker Blog
  * https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/

