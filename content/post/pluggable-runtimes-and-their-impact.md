+++
date = "2017-10-19T07:38:00+09:00"
draft = false
slug = "pluggable-runtimes-and-their-impact-translate"
title = "【参考訳】入れ替え可能なランタイム：containerdとCRI-Oと、Kubernetesユーザに与える影響"
categories = ["Docker", "Kubernetes"]
tags = ["Docker", "Kubernetes","translation","Techtonic"]
+++

ここ数日の動きに関連して、 [CoreOS の blog 投稿](https://coreos.com/blog/pluggable-oci-runtimes-and-their-impact-on-kubernetes-users) で状況がよくまとめられていると思われた投稿がありました。参考情報として翻訳・共有いたします。内容間違い等ございましたら、ご指摘ください。

## 入れ替え可能なランタイム：containerdとCRI-Oと、Kubernetesユーザに与える影響

Pluggability（プラガビリティ ※1）とは Kubernetes のサクセス・ストーリー（成功談）の一部です。そして、Kubernetes ユーザ体験（user experience）を変えることなく、ストレージ、ネットワーク、スケジューラといった多くのレイヤを置換可能かつ改良可能なのを、コミュニティの皆さんにお約束するものです。今年の初め、 Kubernetes 上でのコンテナ実行をプラガブルにする手法として、Kubernetes プロジェクトはコンテナ・ランタイム・インターフェース（CRI; Container Runtime Interface）と呼ぶ API を作成しました。今週、Kubernetes CRI API を実装する２つのプロジェクトが成熟に向かいつつあります。 [CRI-O](http://cri-o.io/) はバージョン 1.0.0 に到達し、 [containerd](https://containerd.io/) は 1.0.0-beta.2 ですが間もなくベータを外せる模様です。

（※1 訳者注；Pluggabilityとは、入れ替え可能な機能、の意味。システムなどが Pluggable、つまりプラグを差し替えるように、入れ替え可能なもの。ここではそのままにしました）

これらのリリースはプロダクション（本番環境）利用に向けての初期の通過点（マイルストーン）であり、プロジェクトに関わるチームの皆さんにお祝いを言わせてください。また、プロジェクト初期から今日に至るまでのほとんどの努力は、多くの人々にとって初めて知られることになるかもしれません。そこで、私たちは これらが Kubernetes エコシステムの中で使われるように至った経緯と背景をご紹介します。

もしも、皆さんが Kubernetes を日常でお使いであれば、ここで読むのを止めても構わないでしょう。CRI と Docker Engine、containerd、CRI-O のような様々な実装が目指すのは、ユーザに対する透明性と Kubernetes API から離れるのを、これまでの経験を変えずに実現するためです。これらの大部分は内部のシステム設計に関わる部分ですが、これらの 99% が同じ機能の実装です。似たような例としては、皆さんのノート PC 上における GNU sed 対 BSD sed です。そして、皆さんが CoreOS Tectonic （訳者注；CoreOS社が開発・サポートしている Kubernetes をベースとしたコンテナ管理プラットフォーム）ユーザであれば、デプロイと運用のために、私たちのフルスタックの自動運用と、クラウドおよびオンプレミスの柔軟なインストーラーにより、Kubernetes 内部の設定調整に時間をかけずに行える、ベストな選択肢であると自信をお持ちでしょう。

もしまた Kubernetes や CoreOS Tectonic を試していなければ、 Linux/Windows/macOS に対応した [Tectonic Sandbox](https://coreos.com/tectonic/sandbox?utm_source=blog&utm_medium=referral)  をお試しいただくか、 [クラウド上で自由に](https://coreos.com/tectonic/docs/latest/account/?utm_source=blog&utm_medium=referral)  お試しください。

## コンテナ用語

Kubernetes コミュニティに対する各プロジェクトを扱う前に、まずは用語を簡単に復習しましょう。

* **コンテナ・イメージ（Container images）** とは、アプリケーションに対する依存関係全てを、１つの名前のアセット（asset；もの、存在）にパッケージしたものです。これにより、多くのマシン間にコピーできます。これはサーバ上におけるモバイルアプリの考え方と同じです。つまり、アプリの中に全てが入っており、簡単にインストールできます。
* **コンテナ・ランタイム（Container runtimes）** とは、Kubernetes エコシステムにおいて Kubernetes システムの内部コンポーネントです。このシステムは、コンテナ・イメージにパッケージ化されたアプリケーションをダウンロードし、実行します。コンテナ・ランタイムの例には containerd、CRI-O、Docker Engine、frakti、rkt があります。
* **オープン・コンテナ・イニシアティブ（OCI）ランタイム仕様（Runtime Specification）** とは、コンテナ・ランタイムのスキーマ（仕様）を明確に定義したものです。具体的には userid、bind mount、カーネル名前空間（kernel namespaces）、コントロール・グループ（cgroups）であり、これらはコンテナ・イメージの実行時に作成されます。この仕様はコンテナ・ランタイムの相互運用性を保証するために重要ですが、インフラ管理者が日々で扱う領域ではありません。

ここ数年、コンテナは広範囲かつ急速に広まり、開発者とインフラ管理者らに有名になりました。そして、皆さんが必要としたのは長期にわたるフォーマット（仕様）の安定性と、内部ツールの互換性です。私たちは 2014年に提供を始めた [appc](https://coreos.com/rkt/docs/latest/app-container.html?utm_source=blog&utm_medium=referral) から、コンテナ相互運用に関して業界との対話に努めてきました。以降、コンテナ業界の他社とオープン・コンテナ・イニシアティブ（OCI）を組織し、２年の取り組みの後、最近 [イメージおよびランタイム仕様 v1.0](https://coreos.com/blog/open-container-initiative-specifications-are-10) の発表に至りました。

containerd が生まれたのは、この努力の一部です。Docker Engine のコードを上書きした containerd は、スタンドアロン（訳者注；単体で動作する、の意味）なコンテナ・ランタイム・コンポーネントです。これは OCI 仕様に則った実装です。そして、OCI が開発したコンポーネントである [CRI-O](http://cri-o.io/) プロジェクトも同様であり、その目標とするのは Kubernetes のためのコンテナ・ランタイム・インターフェースの実装をゼロから行うことでした。そして今、 CRI-O と containerd のどちらもコンテナ・イメージをダウンロード可能であり、OCI ランタイム仕様を用いるコンテナをセットアップし、実行可能になりました。

### コンテナ・ランタイムと Kubernetes での動作について

2016年後半にリリースされた Kubernetes 1.5 は、コンテナ・ランタイム・インターフェースを Kubernetes に実装した初めてのバージョンです。抽象化レイヤの基盤は、 Kubernetes プロジェクト SIG Node と、Google や CoreOS やその他の皆さんとの協力作業によるものです。そして、 Kubernetes と他のコンテナ・ランタイム間における API インターフェースを、コンテナ・ランタイム・インターフェース（CRI）として定義しました。２つの要素がゴールです。テストとしてこのインターフェースをモックとして使える API を導入すること。それと、 Docker Engine や CRI-O、rkt、containerd のようなコンテナ・ランタイムを Kubernetes において置き換え可能とすることです。

より詳細な情報については、CRI を Kubernetes で利用可能にするプロジェクトの [コンテナ・ランタイム・インターフェースのブログ投稿](http://blog.kubernetes.io/2016/12/container-runtime-interface-cri-in-kubernetes.html) をご覧ください。

### コンテナ・ランタイム・インターフェースの影響

containerd と CRI-O のようなプロジェクトの開発は興味深く、情報を追う価値がありました。これまでのところ、Kubernetes コミュニティに対する CRI の主な恩恵は、より優れたコードの体系化と、Kubelet 自身が扱う範囲のコード改善でした。得られたコードをもとに、従来よりも高い品質と全面的なテストの両方を得ました。

ほとんど全てのデプロイにおいて、まだしばらくは、私たちは Kubernetes コミュニティが Docker Engine を使い続けると予想しています。なぜならば、新しいコンテナ・ランタイムの導入により、既存の機能性を壊す可能性が潜在しており、既存の Kubernetes ユーザに対して新しい価値を直ちにもたらせないからです。

CRI の働きにより、Kubernetes 内部のコンテナ・ランタイムの入れ替えが簡単になりました。そして、エンド・トウ・エンドのテストも通過しています。これまでの３年、Kubernetes と Docker Engine 間にわたる多くの統合により、依存関係が作り出されました。想定しているのは、ディスクに対するログの保存場所、Linux コンテナ内部、コンテナ・イメージの保管、そして、その他の細かな相互関係です。

依存性に対する取り組みには、抽象化の調整と改良が必要であり、開発には時間を費やすでしょう。そして、containerd、CRI-O、あるいは他の CRI 実装がありますが、プロダクションのほとんどの場面で利用するにあたり、理論的に何がベストなものとしてあり続けるのか、Kubernetes コミュニティで何を変えて行くのかの議論が続くでしょう。

## コンテナ・ランタイムの更なる選択肢？ 問題ありません

皆さんが Kubernetes ユーザであれば、CoreOS が優れたコンテナ基盤へと案内・導入する手助けを続けますので、ご安心ください。これまで通り、一貫して行います。エコシステムに参加する新しい方には、CoreOS はエコシステムにおける成熟したコンテナ・ランタイムとして、Kubernetes 環境をプロダクションで使うには、Docker Engine と比較して安定性があると CoreOS は評価されています。コンテナ・ランタイムの代替は Kubernetes 利用者に対して大きな意味のある改良ですが、私たちは上位プロジェクトの決定を支持し、私たちはプロジェクトに対して CoreOS Tectonic プラットフォームの選択肢をサポートし続けます。

さらに、Tectonic を利用する皆さんに対して。Kubernetes プラットフォームで避けられない内部変更があったとしても、導入およびアップグレードを独自に自動的に処理できるよう、Tectonic プラットフォーム全体が設計されています。Kubernetes エコシステムでは、将来の Kubernetes リリースに向けて、どのコンテナ・ランタイムを選択するのか評価が続きます。ですが、 皆さんに対しては、柔軟性を発揮するのにベストであり、最も安全な選択肢であるべく Tectonic の準備が整っているのを申しあげます。

エンタープライズへの準備が整った Kubernetes を皆さんの現在の環境で動かすために、Tectonic が最も簡単なのは何故なのか、皆さん自身で確かめたい場合は、 [Tectonic をダウンロード](https://coreos.com/tectonic/l) できますし、10ノードまでのクラスタをデプロイするまでは無料でご利用いただけます。あるいは、皆さんのノート PC 上で Tectonic を触ってみたい場合は、 [Tectonic Sandbox](https://coreos.com/tectonic/sandbox) が独自の機能や実験用環境として使えるでしょう。こちらは皆さんのマシン上で動作するものであり、ハードウェアの追加やクラウド・アカウントは不要です。

### 原文

* Pluggable runtimes: containerd, CRI-O and their impact on Kubernetes users | CoreOS
  * https://coreos.com/blog/pluggable-oci-runtimes-and-their-impact-on-kubernetes-users

