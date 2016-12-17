+++
date = "2016-11-10T22:20:00+09:00"
draft = false
slug = "introducing-operators-translate-jp"
title = "【翻訳】Operator の紹介：運用の知見をソフトウェアに入れる"
categories = ["CoreOS"]
tags = ["CoreOS","Kubernetes","Operator","translation"]
+++

## 概要

CoreOS の Blog に "[Introducing Operators: Putting Operational Knowledge into Software](https://coreos.com/blog/introducing-operators.html)" という記事が掲載されました。先日の KubeCon でも Keynote で取り上げられていた、 Operator という Kubernetes 上で指定した状態を維持するためのツールについての概要です。

「運用知識をソフトウェアに」と興味深いタイトルに釣られ、ついつい読んでしまいました。記事を書かれた Brandon Philips さん（ [@BandonPhilips](https://twitter.com/BrandonPhilips) ）に翻訳を許諾いただきましたので、ここで公開します。翻訳に間違いがありましたら、どうぞお気軽に @zembutsu までご指摘ください。

## Operator の紹介：運用の知見をソフトウェアに入れる

サイト信頼性エンジニア（SRE; Site Reliability Engineer）とは、ソフトウェアを書いてアプリケーションを運用（operate）する人です。SRE はエンジニアであり、開発者でもあり、特定のアプリケーション領域に特化したソフトウェアをどのように開発するかを知っています。結果として、アプリケーション運用範囲の知識を、ソフトウェアの要素にプログラムとして入れ込むことになります。

私たちのチームは Kubernetes コミュニティの企画や実装に対し、今日に至るまでずっと取り組んでいます。実装が目指すのは、 Kubernetes 上の複雑なアプリケーションを、先の概念にもとづき確実に作成・設定・管理することです。

私たちはこの種のソフトウェアを Operator （オペレータ）と呼びます。Operator とはアプリケーションに特化したコントローラです。Kubernetes の利用者に代わり、複雑でステートフルなアプリケーションを作成・設定・管理するために Kubernetes API を活用します。kubernetes 上にリソースとコントローラの概念を構築します。それだけでなく、特定の範囲（domain）やアプリケーション固有の知識を一般的なタスクに自動化します。

## ステートレスは簡単、ステートフルは難しい

ウェブ・アプリケーションやモバイル・バックエンド、API サービスの管理やスケールにあたり、Kubernetes では細かな設定をしなくてもすぐ使えるため、比較的簡単です。なぜでしょうか。理由は、これらアプリケーションは一般的にステートレスだからです。追加の知識がなくても、Deployments のような基本的な Kubernetes API によってスケールしたり障害から復帰したりできるのです。

データベース、キャッシュ、監視システムのように、ステートフルなアプリケーションの管理に取り組むのは大変です。これらのシステムでは、正確なスケール、更新（upgrade）、再設定といったアプリケーション領域の知識が必要です。これは単なる知識ではなく、データの欠損（loss）や利用できなくなるのを回避するものです。私たちはこのようなアプリケーション固有の運用知識をソフトウェア内にエンコード（変換）することで、高性能な Kubernetes の抽象化を活用し、アプリケーションを正確に実行・管理できるようにしたいのです。

Operator（オペレータ）はこの領域の知識を変換（encode）します。そして Kubernetes API の拡張により、[サードパーティ・リソース（third party resources）](http://kubernetes.io/docs/user-guide/thirdpartyresources/) メカニズムに対して、利用者によるアプリケーションの作成・設定・管理を可能とします。Kubernetes の内部リソースのように、Operator はアプリケーションの単一インスタンスを管理するのではなく、クラスタを横断する複数のインスタンスを管理します。

[<img src="/images/operator-fig1.png" width="100%">](/images/operator-fig1.png)

Operator の概念がコードをして動くのを実演（デモンストレーション）するために、今日、２つの実装例をオープンソース・プロジェクトとして公開します。

1. [etcd Operator](https://coreos.com/blog/introducing-the-etcd-operator.html) は etcd クラスタの作成・設定・管理をします。etcd は信頼性のある分散キーバリュー・ストアです。CoreOS では分散システムにおいて最も重要なデータを維持するために導入しています。また、Kubernetes 自身の主要な設定データの保存先としても使います。

2. [Prometheus Operator](https://coreos.com/blog/the-prometheus-operator.html) は Prometheus 監視インスタンスを作成・設定・管理します。Prometheus は高性能な監視とメトリック（訳者注：監視データの数値化）とアラート（通知用）ツールです。そして、クラウド・ネイティブ・コンピューティング・ファウンデーション（CNCF: Cloud Native Computing Foundation）のプロジェクトとして、CoreOS チームが支援しています。

## どのように Operator が構築するのか？

Operator は Kubernetes の中心となる２つの概念、リソース（resources）とコントローラ（controllers）に基づき構築します。たとえば内部の [ReplicaSet](http://kubernetes.io/docs/user-guide/replicasets/) リソースの場合、ユーザは実行したいポッドの数を設定します。次に、コントローラは ReplicaSet リソースで設定した数を Kubernetes 内で 常に維持するよう、ポッドを作成、あるいは実行中のポッドの削除といった動作をします。これらの動作は [Services](http://kubernetes.io/docs/user-guide/services/)、[Deployments]http://kubernetes.io/docs/user-guide/deployments/() 、[Daemon Sets](http://kubernetes.io/docs/admin/daemons/) でも同様な挙動であり、 Kubernetes におけるコントローラとリソースの多くの基本となるものです。

[<img src="/images/operator-fig2.png" width="100%">](/images/operator-fig2.png)
例1a：１つのポッドが動作中で、ユーザは希望のポッド数を３に設定

[<img src="/images/operator-fig3.png" width="100%">](/images/operator-fig3.png)
例1b：数秒後、Kubernetes 内のコントローラは、ユーザが要求した数と一致するようポッドを作成


Operator は Kubernetes のリソースとコントローラの概念を基盤として、その上に知識や設定を積み上げます。そのため、 Operator は一般的なアプリケーション・タスクを実行できるようになります。たとえば、etcd クラスタを手動でスケール（拡大）しようとする場合、利用者は数ステップを踏みます。具体的には、新しい etcd メンバ用の DNS 名を作成し、新しい etcd インスタンスを起動します。それから、etcd 管理ツール（ `etcdctl member add` ）を使い、既存のクラスタに新しいメンバを追加するのを伝えます。この手順ではなく、etcd Operator があれば、利用者は etcd クラスタのサイズのフィールドを１から単に増やすだけで済むのです。
 
[<img src="/images/operator-fig4.png" width="100%">](/images/operator-fig4.png)
 例2：ユーザの kubectl をトリガにバックアップ
 
複雑な管理タスクの例としては、Operator でアプリケーションを安全な状態のまま更新や、離れた場所にあるストレージに設定ファイルのバックアップ、ネイティブな Kubernetes API を通したサービス・ディスカバリ、アプリケーションの TLS 認証設定やサービスディスカバリなどに利用できるでしょう。

## Operator はどのように作成するのか？

Operator は性質上、アプリケーションごとに固有のものです。アプリケーション運用領域に関するすべての知識を、適切なリソース設定と制御のループにエンコード（変換）するのは大変です。ここでは Operator で構築するにあたり、私たちが発見した一般的なパターンを紹介します。これがあらゆるアプリケーションで重要となるものと考えています。

1. Operator のインストールには、１つだけデプロイします。たとえば `kubectl create -f https://coreos.com/operators/etcd/latest/deployment.yaml` を実行するのみであり、他にインストール作業はありません。

2. Operator は Kubernetes に新しいサードパーティ・タイプを追加するでしょう。利用者はこのタイプを使い、新しいアプリケーションを作成するでしょう。

3. 度重なるテストやコードに対する深い理解をする時に、 Operator は Kubernetes 内部のサービスやレプリカ・セットといったプリミティブのように扱います。

4. Operator はユーザが作成したリソースの後方互換性と古いバージョンを理解します。

5. Operator が停止または削除されても、Operator はアプリケーション・インスタンスが影響なく実行し続けられるよう設計しています。

6. Operator は希望のバージョンを宣言し、希望のバージョンを元にしたアプリケーション更新のオーケストレートを提供するでしょう。ソフトウェアの更新を行っても、操作時のバグやセキュリティの問題を引き起こさないため、Operator を使えば確実に処理する手助けとなるでしょう。

7. Operator は「Chaos Monkey」テスト・スイートによる試験を実施されることで、Pod や設定やネットワークに関する潜在的な障害をシミュレートするでしょう。

## Operator の今後

CoreOS が提供する etcd Operator と Prometheus Operator は、今日時点での高性能な Kubernetes プラットフォームの例です。昨年より、私たちは幅広い Kubernetes コミュニティと行動を共にしています。焦点を当てているのは 、Kubernetes の安定・安全・簡単な管理・簡単なインストールです。

これでようやく Kubernetes の基本を構築できました。次に注力するのは、この上にシステムを構築することです。具体的には、新しい能力を Kubernetes に拡張するためのソフトウェアです。私たちが思い描く未来とは、利用者が自身の Kubernetes クラスタ上に Postgres Operator 、Cassandra Operator 、Redis Operator をインストールし、今日のステートレスなウェブ・アプリケーションを簡単にデプロイするかのように、これらプログラムのインスタンスをスケーラブルに操作できるようにすることです。

より詳しく学ぶには、GitHub リポジトリに飛び込むか、私たちの[コミュニティ](https://coreos.com/community/)・チャンネルでの議論、あるいは、11 月 8 日に開催の [KuberCon](https://tectonic.com/blog/kubecon-preview.html?_ga=1.29798396.230424496.1475473202) の CoreOS チームの登壇にお越しください。私のキーノートは [11 月 8 日(火) の太平洋標準時午後 5:25](https://cnkc16.sched.org/event/8g4I) です。そこで Operator とその他の Kubernetes トピックを話します。

## FAQ

**Q: StatefulSets （以前は PetSets ）との違いは？**

A: StatefulSets は Kubernetes 用アプリケーションをサポートするよう設計されています。アプリケーションはクラスタ上で IP やストレージのような「ステートフルなリソース」を必要とします。よりステートフルなデプロイ・モデルを必要とするアプリケーションは、障害・バックアップ・再設定のためのアラートや処理のための運用自動化（Operator automation）が必要です。そのため、これらデプロイ特性を必要とするアプリケーションの運用担当者は、Replica Setsや Deployments を活用する代わりに、StatefulSets を使えます。

**Q: Puppet や Chef のような設定管理との違いは？**

A: コンテナと Kubernetes では、運用担当者が実現できるのに大きな違いがあります。２つの技術は新しいソフトウェアのデプロイ、分散環境における設定の調整、複数のホスト上のシステム状態を一貫性に保つ確認と、Kubernetes API の利用を簡単にします。運用担当者は、アプリケーション利用者が使いやすくなるよう、これらプリミティブを一つにつなぎ合わせます。つまり、これは設定ではありません。今現在のアプリケーションすべての状態を示すのです。

**Q: Helem との違いは？**

A: Helem は複数の Kubernetes リソースを１つのパッケージにまとめるツールです。概念は複数のアプリケーションを一緒にまとめるものです。Operator はアプリケーション管理で補助的に使えるでしょう。たとえば、traefik はロードバランサであり、etcd をバックエンド・データベースとして使います。皆さんは Helm Chart で treafik のデプロイと etcd クラスタ・インスタンスを一緒にデプロイできます。そして etcd クラスタは etcd Operator を使ってデプロイや管理することもできます。

**Q: Kubernetes にとっては何が新しいのですか？ どのような意味があるのですか？**

A: etcd や Prometheus や今後出てくる複雑なアプリケーションを簡単にデプロイしたい新しいユーザ以外は、これまでと殆ど変わらないでしょう。Kubernetes 上で実行するための推奨手法は、まだ [minikube](https://github.com/kubernetes/minikube) であり、[kubectl run](http://kubernetes.io/docs/user-guide/kubectl/kubectl_run/) なのです。Prometheus Operator を使って監視するアプリケーションのデプロイには `kuberctl run` を使うでしょう。

**Q: etcd Operator と Prometheus Operator のコードは今日から使えますか？**

A: はい！ GitHub の https://github.com/coreos/etcd-operator と https://github.com/coreos/prometheus-operator をご覧ください。

**Q: 他の Operator を予定していますか？**

A: はい、近いうちに。また、コミュニティを通して新しい Operator が作られるのを歓迎します。皆さんの Operator で何か出来るようになれば、ご連絡ください。

**Q: Operator はクラスタを安全にするために役立ちますか？**

A: いいえ。ソフトウェアの更新は、一般的に操作時のミスやセキュリティ問題があり、Operator ができるのは利用者に対し、より確実に更新処理を行うことです。

**Q: Operator はディザスタ・リカバリに使えますか？**

A: Operator が簡単にするのは、アプリケーション状態の定期的なバックアップと、バックアップから以前の状態を戻すことです。私たちが目指す機能としては、ユーザが Operator さえ使えば、バックアップから新しいインスタンスのデプロイを簡単に行えるようにすることです。

## 原文

* Introducing Operators: Putting Operational Knowledge into Software
 * https://coreos.com/blog/introducing-operators.html
