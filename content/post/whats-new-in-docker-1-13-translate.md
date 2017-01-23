+++
date = "2017-01-24T06:32:00+09:00"
draft = false
slug = "whats-new-in-docker-1-13-translate.md"
title = "【参考訳】Docker 1.13 の紹介"
categories = ["Docker"]
tags = ["Docker", "translation"]
+++

# 概要

2017年1月19日(木)に Docker 1.13 のメジャー・バージョンアップが公開されました。新機能ポイントを整理すると、以下の通りです。

* swarm モードで Compose ファイルのサポート（新しい `docker stack deploy` コマンド ）、Compose v3 フォーマット追加
* docker コマンドラインの後方互換性に対応
* `docker system df` の容量表示と、 `docker system prune` による不要なデータ削除コマンド
* CLI の再編成（今のコマンドは維持するものの、 `docker container xxx` や `docker image history` の利用が可能に）
* `docker service logs` でサービス全てのログを取得、Prometheus スタイルのエンドポイント
* 構築時に `--squash` オプションを指定すると１つのレイヤを作成（オーバヘッドとなる可能性あり）、圧縮オプションのサポート
* Docker for AWS / Azure はベータが取れ、プロダクション対応

Docker 社の blog にリリースノート相当の更新情報が掲載されていました。

* Introducing Docker 1.13 - Docker Blog
 * https://blog.docker.com/2017/01/whats-new-in-docker-1-13/

例によって、自分用の翻訳ですが、参考情報としてここで共有いたします。

## Docker 1.13 の紹介

今日、私たちは多くの新機能・改良・修正を施した Docker 1.13 をリリースしています。新年の決意を抱く Docker の利用者にとっては、コンテナ・アプリの更なる構築と改善の手助けとなるでしょう。Docker 1.13 は[Docker 1.12 で導入した swarm モード](https://docs.docker.com/engine/swarm/)を改良し、多くの修正を行っています。それでは Docker 1.13 の機能ハイライトを見ていきましょう。

## compose ファイルを使い、swarm モードのサービスをデプロイ

Docker 1.13 は   Compose ファイルにする「docker stack deploy」コマンドを追加しました。このコマンドは「docker-compose.yml」ファイルを直接使ってサービスをデプロイ可能です。この機能の強化にあたっては、[swarm サービス API をより柔軟で有用にするため](https://github.com/docker/docker/issues/25303)に、大きな努力が払われました。

次のような利点があります：

* 各サービスごとに希望するインスタンス数（訳者注：動作するサービス数）を指定
* ポリシーのローリング・アップデート（訳者注：サービスを動かしながらイメージを更新）
* サービスに対する constraint（訳者注：サービス実行時の制約・条件。どのホストやどの環境で実行するかなど）

複数のホスト上に複数のサービス・スタックをデプロイするのも、簡単になりました。

```
docker stack deploy --compose-file=docker-compose.yml my_stack
```

## CLI 後方互換性の改良

Docker CLI を更新する時、「Error response from daemon: client is newer than server」というエラーが出る問題に怯えていました。ですが、古い Docker Engine を使い続けたい場合には、どうしたらよいのでしょうか。

1.13 からの[新しい CLI は古いデーモンとも通信可能になります](https://github.com/docker/docker/pull/27745)。そして、新しいクライアントは、古いデーモンでサポートされていない機能を使おうとした場合に、適切なエラーを返せる機能を追加しました。これにより、異なるバージョンの Docker を同じマシン上にインストールする時の管理を簡単にし、相互運用性を大幅に改善します。

## クリーンアップ・コマンド

Docker 1.13 は２つの便利なコマンドを導入しています。Docker がどれだけディスク容量を使用しているのか調べるものと、使っていないデータを削除するために役立つものです。

* `docker system df` は unix ツールの df 同様に、使用中の容量を表示
* `docker system purne` は使っていない全てのデータを削除

prune（訳者注：プルーン、「不要なモノを取り除く」や「削る」の意味）は特定のデータ種類を指定した削除も可能です。たとえば、 `docker volume prune` は未使用のボリュームだけ削除します。

## CLI の再開発（restructured）

この２年以上にわたり、Docker は多くの機能を追加してきました。そして、現在の Docker CLI には 40 以上ものコマンド（執筆時点で）があります。コマンドには `build` や `run` のように頻繁に使われるものもあれば、 `pause` や `history` のように余り知られていないものもあります。トップレベルに多くのコマンドがあるため、ヘルプページを混乱させ、タブ補完を難しくしています。

Docker 1.13 では論理的な目的の下に全てのコマンドがおさまるようにコマンドを再編成しました。たとえばコンテナの `list` （一覧）と `start` （起動）は新しいサブコマンド `docker container` で利用可能となり、 `history` は `docker image` サブコマンドになります。

```
docker container list
docker container start
docker image history
```

これら Docker CLI 構文のクリーンアップ（見直し）により、ヘルプ文書の改善や Docker をより簡単に使えるようになります。古いコマンド構文はサポートし続けますが、皆さんが新しい構文の採用を推奨します。

## 監視の改良

サービスのデバッグをより簡単にするため、`docker service logs` は実験的な新しい強力なコマンドとなります。従来はホストやコンテナで問題を調査するためには、各コンテナやサービスのログをそれぞれ取得する必要がありました。 `docker service logs` を使えば対象のサービスを動かしている全てのコンテナのログを取得し、コンソール上に表示します。

また、Docker 1.13 ではコンテナ、イメージ、その他デーモン状態といった基本的なメトリック（訳者注：システムのリソース利用状況など）を持つ  [Prometheus スタイルのエンドポイント](https://github.com/docker/docker/pull/25820) を追加しました。

## 構築の改良

`docker build` に新しい実験的な `--squash` （スカッシュ；「押し込む」「潰す」の意味）フラグを追加しました。スカッシュを指定すると、Docker は構築時に作成した全てのファイルシステム・レイヤを取得し、それを１つの新しいレイヤに押し込み（スカッシュし）ます。これにより最小のコンテナ・イメージ作成に関わる手間を簡単にします。一方、イメージが移り変わる場合は（スカッシュされたレイヤはイメージ間で共有されなくなるため）、極めて大きなオーバヘッドとなる可能性があります。Docker は以後の構築を速くし続けるために、個々のレイヤのキャッシュを続けます。

また、1.13 では[構築時の内容物（コンテクスト；build context）の圧縮をサポート](https://github.com/docker/docker/pull/25837) しました。使うには CLI からデーモンに対して `--compress` フラグを使います。これにより、リモートのデーモンに大量のデータを送る必要がある場合は構築スピードが向上します。

## Docker for AWS と Azure がベータ版の段階を終了

Docker for AWS と Azure はパブリック・ベータを終了し、プロダクション対応（実際に利用可能）となりました。皆さんからのフィードバックを取り込み、Docker for AWS と Azure の対応には半年を費やしました。そして、テストや個々の問題に取り組んでいただいた全てのユーザの皆さんに大きな感謝を申しあげます。また、更なる更新や拡張を調整したバージョンを数ヶ月のうちに提供します。

## Docker 1.13 を始めましょう

Docker for Mac と Windows のベータ・安定版の両ユーザは、自動更新の通知を受け取っているでしょう（実際の所、ベータ・チャンネルのユーザには、半月前に Docker 1.13 を提供していました）。Docker が初めてでしたら、Docker for [Mac](https://docs.docker.com/docker-for-mac/) か [Windows](https://docs.docker.com/docker-for-windows/) で始めましょう。

[Docker for AWS](https://docs.docker.com/docker-for-aws/) と [Docker for Azure](https://docs.docker.com/docker-for-azure/) は、ドキュメントを確認するか、ボタンを押して始めましょう。

* [AWS: Launch Stack](https://img.scoop.it/QBKfxUSF43XxVEiQeZW6TLnTzqrqzN7Y9aBZTaXoQ8Q=)
* [Azure: Deploy to Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fdownload.docker.com%2Fazure%2Fstable%2FDocker.tmpl)

Linux に Docker をインストールしたい場合は、[インストールの詳細手順（英語）](https://docs.docker.com/engine/installation/)をご覧ください。

* What's new in Docker 1.13 - YouTube
  * https://www.youtube.com/watch?v=y_RiG_9jEJ0

## 原文

* What's new in Docker 1.13 - YouTube
 * https://www.youtube.com/watch?v=y_RiG_9jEJ0
