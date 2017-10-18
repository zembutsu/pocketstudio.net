+++
date = "2017-10-18T06:00:00+09:00"
draft = false
slug = "docker-kubernetes-translate"
title = "【参考訳】DockerCon EU 17 における Docker の Kubernetes サポート関連発表"
categories = ["Docker", "Kubernetes"]
tags = ["Docker", "Kubernetes","translation"]
+++

# 概要

2017年10月17日、コペンハーゲン（デンマーク）で開催の [DockerCon EU](https://europe-2017.dockercon.com/) 基調講演で、Docker プラットフォームと moby プロジェクトで、 Kubernetes サポートの対応を開始する発表がありました。要点を纏めますと、

* Docker は Kubernetes に対応します(moby love Kubernetes）
* Docker for Mac/Winで Kubernetes 動きます（年内にベータ提供開始予定）
* Swarm も従来通り続けます

関連して [ブログも更新](https://blog.docker.com/2017/10/kubernetes-docker-platform-and-moby-project/) されたので、ご理解の参考程度にと共有します。

## Docker プラットフォームと Moby プロジェクトに Kubernetes が加わる

今日私たちは、 Docker プラットフォームが Kubernetes との連携をサポートする発表を行いました。これにより、Docker の利用者と開発者の皆さんは、コンテナ処理のオーケストレートのため、Kubernetes と Swarm の両方を選べます。 [ベータ・アクセスのご登録](https://beta.docker.com/) と、どのように私たちが Kubernetes を使えるようにしているかを詳細を学ぶため、ブログの投稿をご覧ください。

* [Docker Enterprise Edition（英語）](https://blog.docker.com/2017/10/Docker-enterprise-edition-kubernetes)
* [Docker Community Edtion on the desktop with Docker for Mac and Windows（英語）](https://blog.docker.com/2017/10/docker-for-mac-and-windows-with-kubernetes-beta)
* [The Moby Project（英語）](https://blog.mobyproject.org/moby-and-kubernetes-bf888ab31e38)

（各ブログの翻訳は、このページの後方にあります）

Docker はアプリケーションとインフラストラクチャ（システム基盤）に跨がるプラットフォームです。Docker 上でアプリケーションを構築すると、開発者や IT 運用者は自由と柔軟さを手に入れられます。 Docker はどこでも実行できますので、エンタープライズにおけるアプリケーションのデプロイを、オンプレミス上（IBMメインフレームだけでなく、エンタープライズ Linux と Windows も含みます）でもクラウドでも可能にします。アプリケーションをコンテナ対応（containerized）しておけば、再構築、再デプロイ、移動を簡単に行えます。それだけでなく、従来のオンプレミス環境とクラウド環境のどちらでも動作するようにセットアップできます。

Docker プラットフォームは多くのコンポーネントを４つのレイヤに構成します：

* 業界標準コンテナ・ランタイムである OCI 規格を実装する containerd
* ノード群を分散システムに変換する Swarm オーケストレーション
* Docker コミュニティ・エディションは、開発者に対してコンテナアプリケーションを開発・移動するワークフローを簡単にします。ワークフローとは、アプリケーションの構成や、イメージ構築・管理などです。
* Docker エンタープライズ・エディションは、プロダクションにおけるソフトウェアのサプライ・チェーン連携を安全に管理し、コンテナを実行します。

[<img src="/images/docker-kubernetes-01.png" width="100%">](/images/docker-kubernetes-01.png)

これら４つのレイヤは、オープンソースの Moby プロジェクトの一次コンポーネントで構成されます。

Docker の設計哲学とは、常に選択と柔軟性の提供であり、既存の IT システムと Docker を統合する利用者の皆さんにとって大切です。なぜならば、既にデプロイ済みのネットワーク、ログ記録、ストレージ、負荷分散、CI/CD システムと連携するように Docker が作られているからです。Docker が依存しているのは、業界標準プロトコルや論文、ドキュメント化されたインタフェースです。そして、これら全てにより、Docker エンタープライズ・エディションはデフォルトで実用的です。また、これら標準機能により、皆さんは既存のシステムや望ましいソリューションを利用するにあたり、認証済みのサードパーティ製のものへと置き換え可能なのです。

2016年、Docker はプラットフォームに SwarmKit プロジェクトから導入した [オーケストレーションを追加](https://pocketstudio.net/2016/07/30/docker-1-12-goes-ga-translate/) しました。そしてこれまでの１年、私たちは Swarm に対する多数の建設的なフィードバックをいただきました。セットアップが簡単で、スケーラブルであり、難しい設定を行わなくても安全です、と。

その一方で、Docker プラットフォームですべてをコンテナで管理する利用者の方からもフィードバックをいただきました。コンテナのスケジューリングのために Kubernetes のような他のオーケストレータを使いたいと。あるいは、既にサービスを Kubernetes 上で動作するように設計済みであったり、Kubernetes にこそ求めている機能がある場合にです。これこそが、私たちが Docker エンタープライズ・エディションと Docker for Mac と Windows におけるオーケストレーションのオプションに、（Swarm と並んで） Kubernetes 対応を追加した理由なのです。

[<img src="/images/docker-kubernetes-02.png" width="100%">](/images/docker-kubernetes-02.png)

さらに、Docker 利用者が Kubernetes オーケストレーションにネイティブ対応した、 Docker アプリケーションのデプロイが簡単になるように、革新的なコンポーネントに取り組んでいます。たとえば、 [カスタム・リソース](https://kubernetes.io/docs/concepts/api-extension/custom-resources/) のような Kubernetes の拡張機構を使う場合や、API サーバとの連携レイヤについては、Docker が Kubernetes をサポートする将来のバージョンにより、 Docker Compose アプリケーションを Kubernetes ネイティブの Pod やサービスとしてでデプロイできるようになります。

次期バージョンの Docker プラットフォームでは、プロダクションが  Kubernetes で動くようなアプリケーションを、開発者の皆さん自身ワークステーション上で構築・テスト可能となります。そして、運用担当は Docker エンタープライズ・エディションの全ての利点も得られるのです。安全なマルチ・テナンシー、イメージの検査、役割に応じたアクセス制御などを、プロダクションにおけるアプリ実行を Kubernetes あるいは Swarm でオーケストレートできるのです。

私たちが Docker との連携に取り組んでいる  Kubernetes のバージョンは vanilla Kubernetes です。こちらは CNCF（訳者注；業界団体"クラウドネイティブ・コンピューティング・ファウンデーション"の略）が監督のもと、皆さんが扱いやすいものです。こちらからのフォークではなく、古いバージョンでもなく、包括および何らかを限定するものではありません。

Docker は昨年より Moby プロジェクトを通して、  Kubernetes の採用と貢献に取り組んできました。既にコンテナ・ランタイムとしての InfraKit 上にある containred と cri-containerd は、Kubernetes インストールにおける作成・管理とオーバレイ・ネットワーク機能用の libnetwork に対して取り組んでいます。サンプルおよび詳細については [Moby プロジェクトのブログ投稿（英語）](https://blog.mobyproject.org/moby-and-kubernetes-bf888ab31e38)  をご覧ください。

Docker と Kubernetes は系統を共有しています。いずれも同一のプログラミング言語であり、コンポーネントや貢献者やアイディアが重複しています。私たち Docker が期待しているのは、 Kubernetes サポートを私たちの製品に取り込み、また、オープンソース・プロジェクトにおいても取り組むことです。そして、私たちは Kubernetes コミュニティを待たずとも、コンテナやコンテナオーケストレーションをより協力かつ簡単に使えるようにします。

Kubernetes をサポートする Docker エンタープライズ（インフラをサポート）とコミュニティ・エディション（Mac と Windows 対応）のベータ版は、今年末に利用可能となります。 [準備が整ったら通知を受けたい場合は、サインアップしてください](https://beta.docker.com/)  。

私たちは Docker におけるオーケストレーションのオプションとして Kubernetes に対応しますが、Swarm に対して取り組み（コミット）続けますし、お客さまや、スケールするプロダクションにおける、クリティカルなアプリケーションを実行する Swarm および Docker に依存している利用者の皆さんに対応し続けます。Docker がどのように Kubernetes を統合するのか詳細を知りたい場合は DockerCon EC の "What's New in Docker" と "Gordon's Secret Session" をご確認ください。

### 原文

* Docker Platform and Moby Project add Kubernetes - Docker Blog
  * https://blog.docker.com/2017/10/kubernetes-docker-platform-and-moby-project/

## Kubernetes 対応 Docker for Mac および Windows

今日、Docker プラットフォームにおける Kubernetes サポートに対する取り組みの１つとして、Docker コミュニティ・エディションの Mac および Windows 版で Kubernetes オプションの追加を発表します。DockerCon でプレビューを実演しており（Docker ブースで足を止めてください！）、ベータプログラムを 2017 年末に準備します。ベータの準備が整ったら通知を受けるにはサインアップしてください。

Docker CE の Mac および Windows で Kubernetes をサポートすると、利用者の皆さんはソフトウエアとサービスを、開発ワークステーションにおけるテストや CI/CD を通し、オンプレミスまたはクラウド上のプロダクションに至るまで、コンテナで管理できるようになります。

Docker for Mac と Windows は Docker 開発環境の調整で最も人気のある手法であり、毎日何十万人もの開発者がコンテナ対応アプリケーションの開発、テスト、デバッグに使われています。Docker for Mac と Window が人気なのは、インストールがシンプルであり、更新は自動的に行われ、macOS や Windows とそれぞれ密接に統合しているからです。

Kubernetes コミュニティは開発ワークステーションで限定的な Kubernetes をセットアップする良い手法を [Minikube](https://github.com/kubernetes/minikube) など（こちら自身も一部は Docker for Mac と Windows の前身である、docker-machine プロジェクトを元にしています）として提供しています。これらソリューションは一般的ですが、docker build → run → test を頻繁に繰り返す設定をするにはトリッキーであり、古い Docker バージョンに依存しています。

Docker for Mac と Windows における Kubernetes サポートが見えてきたので、開発者は docker-compose と Swarm をベースとしたアプリケーションの両方を構築し、Kubernetes 上へデプロイ可能なアプリケーション設計が簡単になります。そのため、ノート PC やワークステーション上で使うには最良の選択肢となるでしょう。全てのコンテナ・タスク（biuld だけでなく run や push ）を、イメージ、ボリューム、コンテナが共通の同じ Docker インスタンス上で実行できます。そして、それらは最新かつ良い Docker プラットフォームのバージョンをベースとしているため、Kubernetes デスクトップ利用者に対して [マルチステージ・ビルド](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) のような拡張機能を扱えるようにします。

Docker に対する Kubernetes 取り組み成果の一部に、Kubernetes コンポーネントのカスタム・リソースを使う物や、API サーバ統合レイヤがあります。これらにより、Docker Compose アプリを Kubernetes ネイティブ Pot やサービスとしてシンプルにデプロイ可能となります。これらのコンポーネントは Docker EE と Docker CE for Mac と Windows でも提供していきます。

* [Kubernetes in Docker for Mac (YouTube)](https://www.youtube.com/watch?v=jWupQjdjLN0)

皆さんに Docker for Mac と Windows で Kubernetes が動いているのを早くお見せしたいです。DockerCon EU 17 の Docker ブースにお立ち寄りください。あるいは、ベータ版が利用可能になったときの通知の受け取りをお申し込みください。

### 原文
* Beta Docker for Mac and Windows with Kubernetes - Docker Blog
  * https://blog.docker.com/2017/10/docker-for-mac-and-windows-with-kubernetes-beta/