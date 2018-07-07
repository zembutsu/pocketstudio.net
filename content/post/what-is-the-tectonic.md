+++
date = "2017-12-06T12:12:00+09:00"
draft = false
slug = "what-it-the-tectonic"
title = "Tectonic は Kubernetes の構築・管理基盤であるー概要の章ー"
categories = ["Tectonic", "Kubernetes"]
tags = ["Tectonic", "CoreOS","Kubernetes"]
+++

本投稿は、 [さくらインターネットアドベントカレンダー2017](https://qiita.com/advent-calendar/2017/sakura) の、５日目の投稿です。都合により一日遅れましたが、今回は Tectonic（テクトニック）についてご紹介します。なお、前日４日目の投稿は blp1526 さんによる「[さくらのクラウドのサーバの tig ライクな TUI ツール](http://blp1526.hatenablog.com/entry/2017/12/04/003000) 」です。６日目の投稿は、hitsumabushiさんによる「 [さくらのクラウドでN百台を管理するためにterraformとansibleを使っている話](https://qiita.com/hitsumabushi/items/e89b763dd4fc41e15136) 」です。

## はじめに

Tectonic は Kubernetes だけでなく、クラスタを管理するときに役立つ様々な機能をサポートしているツールです。本投稿では Kubernetes の是非を論ずるのではなく、眼前の Kubernetes クラスタを管理する必要が出る場合、どのような選択肢が生まれ得るのかという実地検証の報告が主体です。あくまでも、一利用者の視点で情報を共有いたします。


もともとの投稿では「さくらのクラウド」上で Tectonic が動きますよ！という手順のご紹介を考えていました。結論から申しあげると、PoC（技術検証）レベルにおいては、さくらのクラウド上でもベアメタル・セットアップの手順で動いています（ただし、サポート対象外の模様）。

しかしながら、単にセットアップ手順をご紹介するよりも、Tectonic とは何かあらためて共有させて頂いた上で、後日のエントリにて改めてTectonic環境構築のために必要な Matchbox、Terraform、Ignition の解説、そして、評価環境構築のための手順を皆さまに公開することを予定しています。

また、本エントリは、 [先日の発表](https://www.slideshare.net/zembutsu/terraform-on-bare-metal-with-tectonic) の補足の意味も兼ねています。

[<img src="/images/tectonic-02.png" width="100%">](/images/tectonic-01.png)

## そもそも Tectonic とは？

Tectonic ( https://coreos.com/tectonic ) は、Container Linux などでお馴染み CoreOS 社が提供しているエンタープライズ対応の Kubernetes です。さらに、実運用で必要な Prometheus の監視設定や、Grafana ダッシュボードの監視設定なども自動的に行えます。どちらかというと Tectonic は Kubernetes のラッパー（wrapper）であり、Kubernetes だけでは足りないエンタープライズ向け機能を、広く知られているオープンソースを使いながら、導入を促している点です。



Tectonic はサイトの記述によりますと「CoreOS の自動化技術を用い、プライベートやパブリック・クラウド間での可搬性（ポータビリティ）を保てるようにしたツール」とあります。サポート対象のプラットフォームは、AWS、Azure のようなクラウド・プロバイダだけではなく、ベアメタル・マシンも含まれています。そして、単に Kubernetes を動かすだけでなく、Tectonicを通して etcd クラスタの管理や、ノード障害時に向けた HA 的な機能も備えられています（ただし、現時点では Tectonic の Provisioner ホスト＝Matchbox 設置サーバの冗長化はサポートされていませんが、将来的には計画されています）。

つまり、プラットフォームがどこであろうとも Kubernetes が動作する環境を構築し、その上でコンテナ対応（クラウド・ネイティブ）アプリケーションを実行できるようにします。提供されている機能は、以下の通りです。

[<img src="/images/tectonic-04.png" width="100%">](/images/tectonic-04.png)

しかし、これだけでは Tectonic の機能が分かりづらいので、全貌を図に整理しました。

[<img src="/images/tectonic-01.png" width="100%">](/images/tectonic-01.png)

上図にあるように、物理またはクラウド上のサーバで Kubernetes クラスタを構築します。また、CoreOS 社のプロダクトではありますが、主要なコードやモジュール（Matchbox, Ignition 等）は GitHub 上で公開されており、必要があれば CoreOS 社の商用サポートも受けられます。

## ノード管理の諸問題を解決してくれる

Tectonic のアーキテクチャは Rancher をはじめとして、既存のアプローチと一見すると似ているかもしれません。どちらも物理または仮想サーバ上でクラスタのをノード管理し、その上でコンテナを関する、あるいは、 Kubernetes クラスタの管理を GUI を通して行えるようにしています。

しかし方向性は明確に異なります（優劣ではありません）。

[<img src="/images/tectonic-03.png" width="100%">](/images/tectonic-03.png)

Tectonic は、あくまでも Kubernetes クラスタの大規模な自動運用であったり、セキュリティや監査など、大規模組織で求められる機能をまとめています。方向性としては、オープンソースで提供されている Kubernetes と、Kubernetes の運用・管理ツールとして広く知られている Prometheus および Grafana を始めから「何もせずに」使えるようにします。つまり、Kubernetes クラスタを運用するベストプラクティスが詰まっているのが Tectonic とも言えるでしょう。

一方、Rancher は何か動かしたいアプリケーションがあり、環境構築や管理に手間をかけなくない場合。あるいは、素早くサービス提供を進めたくて結果的にコンテナを効率的に使っていきたい場合に、手軽に Kubernetes クラスタを導入・管理・運用できる機能や、商用サポートを提供するものです。

おそらく、入口としての導入がしやすいのは Rancher に軍配が上がるでしょう。一方で、数百台規模以上のノードを効率的に管理する場合は、Tectonic のほうが必要な機能群が備わっているようにも見えます。また、Tectonic は既存のツールの組み合わせでもありますので、要素要素で Kubernetes ないし CNCF 関連エコシステムのツールやプロダクトとも連携しやすいとも言えるでしょう。

## Tectonic は、ノード管理の煩雑さをなくしてくれる

実運用を考慮すると、どのようなツールであれ、クラスタを構成するノード（としてのサーバやインスタンス）の管理は頭の悩ましい問題です。たとえば、初期の環境構築であったり、ノード側の OS のセキュリティ対策です。

Tectonic はこの問題を解決するため、Matchbox というプロビジョニング用のサーバ機能と Terraform を連携し、自動的に CoreOS Container Linux のインストールと Kubernetes の自動プロビジョニングを行う仕組みを提供しています。

通常であれば、ノードごとに OS をゼロから構築し、 Kubernetes 用の環境をセットアップし、クラスタのワーカーとしての登録など諸作業が必要です。Tectonic であれば、必要なのは Terraform 用の変数設定ファイルに MAC アドレスの情報を記入し、terraform plan, apply を実行し、電源を入れるだけ。

[<img src="/images/tectonic-06.png" width="100%">](/images/tectonic-06.png)

## 概要の章まとめ

[<img src="/images/tectonic-05.png" width="100%">](/images/tectonic-05.png)

次回は、Tectonic を構成する要素のなかで、ノードのプロビジョニング（自動環境構築）に欠かせない Matchbox、Ignition、Terraform それぞれの役割詳細をお伝えします。

