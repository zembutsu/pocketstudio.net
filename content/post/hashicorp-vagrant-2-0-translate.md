+++
date = "2017-09-08T18:17:00+09:00"
draft = false
slug = "hashicorp-vagrant-2-0-translate"
title = "【参考訳】HashiCorp Vagrant 2.0"
categories = ["HashiCorp"]
tags = ["Vagrant", "HashiCorp","translation"]
+++

# 概要

HashiCorp の blog に「 [HashiCorp Vagrant 2.0](https://www.hashicorp.com/blog/hashicorp-vagrant-2-0/) 」というブログが投稿されました。例によって翻訳しましたので、参考程度にどうぞ。概要は、Vagrant 1.0 のリリースから今に至るまで様々な改良を施し、昨日安定版としての 2.0 リリースに至ったという内容です。

## HashiCorp Vagrant 2.0

[HashiCorp Vagrant 2.0](https://www.vagrantup.com/) を発表します。Vagrant は開発環境の構築および配布用ツールです。

* [すぐにダウンロード](https://www.vagrantup.com/downloads.html)

開発環境のプロビジョニング（訳者注；システム環境の環境構築を自動的に行う動作）において、 Vagrant 2.0 はVirtualBox、VMware、Hyper-V、Docker、AWS、GCP などをサポートしています。Windows や macOS 上で仮想化できるだけでなく、他の多くの新しいオペレーティングシステムにも対応しました。Vagrant 2.0 は [Vagrant Cloud](https://app.vagrantup.com/) と連携し、box を検索・利用できます。Vagrant 1.0 から今日に至るまでは長い道のりでした。サポートしていたのは VirtualBox のみでした。また、リリース以降のコミュニティは著しく成長しました。

Vagrant 2.0 は [Vagrrant のウェブサイト](https://www.vagrantup.com/) からすぐにダウンロードできます。直近の Vagrant リリースに関する変更点の一覧は [Changelog](https://github.com/mitchellh/vagrant/blob/v2.0.0/CHANGELOG.md) からご覧いただけます。

## HashiCorp Vagrant

Vagrant は 2009 年から開発が始まり、開発環境とインフラストラクチャ（訳者注；OSを実行可能なシステムおよびネットワーク基盤）の自動デプロイに対して、瞬く間に頼りになるツールとなりました。プロダクション（本番環境）の鏡となる開発インフラの構築を、１つのワークフローで作成することが Vagrant の目的です。

20013 年に安定版（stable）として Vagrant 1.0 がリリースされました。Vagrant 1.0 はプロバイダ（訳者注；Vagrantにおける用語で、インフラを操作する API ドライバのこと）としてサポートしたのは VirtualBox のみでした。また、Linux ゲスト OS としてサポートしていたのは一部のみです。それに、サポートしていたのは単純な起動（up）／停止（destroy）のワークフローのみでした。Vagrant 1.0 以降、私たちは VMware と Docker といった複数のプロバイダに対するサポート追加や、Windows と macOS のようなゲストの追加、そしてスナップショットを含む複雑なワークフローを追加しました。これらの主な変更は何百もの改良とバグ修正によるものです。

Vagrant 登場以前の開発環境は、ほとんどが手作業での構築、エラー解決であり、時間を浪費しました。また、インフラの自動デプロイには、実際のマシンを作成・破棄するように、極めて長いフィードバック・サイクル（訳者注；試行錯誤の繰り返し）がありました。Vagrant は、これら両者のプロセスを１つのコマンドで行えるように変えたのです。

HashiCorp Product suite は、あらゆるアプリケーションをあらゆるインフラ上にプロビジョンし（訳者注；自動的なインストールや設定）、安全であり、接続し、実行できるものであり、Vagrant はその一部です。Vagrant で開発環境のプロビジョンが可能ですし、HashiCorp Packer で Vagrant イメージを構築できます。そして、HashiCorp Terraform でインフラを構築し、Vault でシークレット（訳者注；APIやSSH鍵などの機密データ・情報）管理を扱い、Nomad でワークロードのスケジュール（訳者注；どのノードまたはホスト上で、どのようなジョブまたはプロセスを実行するか決めること）をし、Consul でインフラを連結します。

## コミュニティとチーム

私たちは Vagrant 2.0 を作り上げたコミュニティと Vagrant コアチームに感謝を申しあげます！ HashiCorp Vagrant は過去７年で 750 人以上の貢献者がいらっしゃいます。何年もの間、貢献者の皆さんが機能を追加し、バグを修正し、Vagrant を前へと進めました。

Vagrant 互換性の領域が広がるにつれ、プロジェクトの維持にはコミュニティによる改善が不可欠でした。私たちのコミュニティには、あらゆる種類のプロバイダ、あらゆるオペレーティングシステム、あらゆるプロビジョナー等の専門家に溢れています。これらコミュニティのメンバーが協力し、多様性を持つツールとして作り上げたのが Vagrant なのです。

HashiCorp の Chirs Roberts が 2.0 に向けての変更を牽引しました。2016 年にプロジェクトの責任者となり、バージョン 2.0 に至るために不可欠である、重要な安定性に関する課題に取り組みました。Chris は Brian Cain および Justin Campbell と連携し、３人が一丸となって毎日 Vagrant の改善に取り組んでいます。

## 原文

* HashiCorp Vagrant 2.0
  * https://www.hashicorp.com/blog/hashicorp-vagrant-2-0/

