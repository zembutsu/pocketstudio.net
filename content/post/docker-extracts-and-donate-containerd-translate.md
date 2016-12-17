+++
date = "2016-12-17T11:36:00+09:00"
draft = false
slug = "docker-extracts-and-donete-containerd-translate"
title = "【参考訳】Docker の containerd 寄贈に関する発表"
categories = ["Docker"]
tags = ["Docker","containerd","translation"]
+++

## 概要

Docker が containerd を独立したファウンデーションに寄贈するという[発表](https://www.docker.com/docker-news-and-press/docker-extracts-and-donates-containerd-its-core-container-runtime-accelerate)を 2016年12月14日に行いました。例によって、自分の整理用の翻訳ですが、公開します。

各発表に対する日本語訳は正式なものではありません。私が趣味で翻訳しているものです。訳注、様々な意図を持つ言葉に対しては、（ ）内で原文と訳注を補足しています。ただし、翻訳の専門家ではないため、表現に拙い箇所があるかもしれません。その場合は遠慮なくご指摘いただければと思います。

原文は https://www.docker.com/docker-news-and-press/docker-extracts-and-donates-containerd-its-core-container-runtime-accelerate です。

以下翻訳。

## Docker Extracts and Donate containerd, its Core Container Runtime, to Accelerate Innovation Across the Container Ecosystem

Docker はコンテナ・エコシステム全体の革新を促進するため、自身のコア・コンテナ・ランタイムである containerd を取り出して寄贈。

Alibaba Cloud、AWS、Google、IBM、Microsoft は Docker と協調し、コンテナ・ランタイムのオープンソース・プロジェクト に対する支援（resources；人的・金銭的）と貢献を表明

サンフランシスコ―2016年12月14日―本日、[Docker](http://www.docker.com/) は、containerd（コンテナディー；Con-tay-ner-D）のスピンアウト（独立）を発表しました。Docker Engine は業界トップのコンテナ・プラットフォームであり、containerd とは Docker Engine の中心となる構成要素（コア・コンポーネント）です。そして、この containerd を新しいコミュニティ・プロジェクトに寄贈します。これまでの Docker Engine は利用者（エンドユーザ）向けのコンテナ・プラットフォーム一式として、Docker API 、Docker コマンドとサービス、containerd で構成されていました。これらの構成要素は業界向けにオープンであり、安定し、Docker を使わない製品やコンテナ・ソリューションを作るための基盤となるよう拡張可能です。有数のクラウド提供事業者である Alibaba Cloud、Amazon Web Services [AWS]、Google、IBM、Microsoft は、プロジェクトに対するメンテナとコントリビュータの提供を表明しました。

containerd の機能に含まれるのはコンテナ・イメージの転送手法、コンテナの実行と管理、ローレベルのローカル・ストレージとネットワーク・インターフェースであり、Linux と Windows のどちらも扱います。containerd はオープン・コンテナ・イニシアティブ [OCI] のランタイムを十分に活用します。たとえばイメージのフォーマット指定や OCI リファレンスの実装 []runC であり、OCI 認証が決まれば追従しています。皆さんは今日から containerd プロジェクトに貢献できるようになります。サードパーティのメンテナによる強力なサポートのもと、共同作業（collaboration）や貢献はオープンに行われます。

Docker の創業者・CTO・最高プロダクト責任者（Chief Product Officer）である Solomon Hykes は「この成果とは、何ヶ月にも及ぶ綿密な共同作業と、Docker コミュニティの今後の方向性を示すリーダー達らの情報提供によるものです。」と述べました。「containerd は新しい革新の段階に解き放たれ（unlock；アンロック）、エコシステム全体の成長をもたらすでしょう。言い換えれば、Docker の開発者と利用者のすべての利益となります。Docker は常に利用者の問題解決を優先して取り組んでいます。それゆえ、これまで課題として取り込んだ配下のプロジェクトを独立（スピンアウト）します。containerd プロジェクトが業界のリーダー達からサポートを得られることになり、私達は大きな刺激を受けています。そして、リーダー達の支援は、この共同プロジェクトの成長に対して原動力となるでしょう。」

Docker による containerd への寄贈は、配下の主要なオープンソース・プロジェクトをコミュニティに対して提供してきた従来の流れに沿っています。この取り組みは 2014 年に [libcontainer](https://github.com/docker/libcontainer) を社がオープンソース化した時から始まりました。それから２年が経ち、Docker は同じような流れでコミュニティに対して [libnetwork](https://github.com/docker/libcontainer) 、[notary\(https://github.com/docker/notary) 、[runC](https://github.com/opencontainers/runc) [OCI に寄贈]、[HyperKit](https://github.com/docker/hyperkit)、[VPNkit](https://github.com/docker/vpnkit)、[Datakit](https://github.com/docker/datakit)、(swarmkit)[https://github.com/docker/swarmkit]、[Infrakit](https://github.com/docker/infrakit) を提供しています。containerd が特権を持つのは限定された機能の範囲に対してです。明確なゴールは Docker に安定して使われ続けることと、すべてのコンテナ・システムと優れたオーケストレータに対して拡張可能であることです。これらの結果、システム全体が共有する構成要素の配下にある「つまらない」（boring）基盤でも、Docker とコンテナのエコシステムによって、利用者が必要とする革新をもたらすでしょう。プロジェクトでは、新機能に対する品質強化のために、コミュニティが定義しているリリース手順を今後も続けます。そして、これらの活動は Docker のブランドとは切り離されます。

RedMonk の業界アナリスト Fintan Ryan は「技術組織はますますコンテナに対して注目しています。迅速なアプリケーション開発のためと、複数の環境に横断したデプロイを将来の成長のためにスケール（拡大・縮小）できるようするため。それと、エコシステム全体の互換性が重要な考慮点なのです。」と述べました。「コンテナのエコシステムに対する containerd の寄贈とは、幅広いコミュニティが構築に使う、標準化されたコア・コンポーネント（中心となる構成要素）を Docker が提供することです。コアが標準化されることにより、開発者は安定性を確保するための技術選択を、あらゆる基盤の互換性を保ちながら再検討できるようになるのです。」

### 利用とリソース

containerd リポジトリ https://github.com/docker/containerd/ は今日からオープンです。コメントや貢献をお待ちしています。リポジトリ内には詳細な情報があります。

* ロードマップ：https://github.com/docker/containerd/blob/master/ROADMAP.md
* コンポーネントの機能対応表（スコープ）：https://github.com/docker/containerd#scope
* アーキテクチャの文章：https://github.com/docker/containerd/blob/master/design/architecture.md
* ドラフト版 API ：https://github.com/docker/containerd/tree/master/api/

containerd は 2017年の第1四半期（Q1）までに独立した財団（ファウンデーション）に寄贈します。財団はガバナンスの監督、商標の保持、商標の取り扱いを行います。

（Supporting Quotes、 支持に対する引用は省略しました）

## 原文

* Docker Extracts and Donates containerd, its Core Container Runtime, to Accelerate Innovation Across the Container Ecosystem | Docker
 * https://www.docker.com/docker-news-and-press/docker-extracts-and-donates-containerd-its-core-container-runtime-accelerate

## 関連

* More details about containerd, Docker’s core container runtime component - Docker Blog
  * https://blog.docker.com/2016/12/containerd-core-runtime-component/
  
  