+++
date = "2018-07-04T20:00:00+09:00"
draft = false
slug = "consul-1-2-service-mesh-translate-jp.md"
title = "【参考訳】HashiCorp Consul 1.2:Service Mesh" 
categories = ["HashiCorp"]
tags = ["Consul", "Service Mesh", "HashiCorp","translation"]
+++

# "HashiCorp Consul 1.2:Service Mesh" 概要

HashiCorp の blog に、  [Consul 1.2のリリースと新機能に関する投稿](https://www.hashicorp.com/blog/consul-1-2-service-mesh) が先日ありました。新しくベータ機能として提供がはじまった Connect 機能や今後の展開についてが参考になるかと思い、例によって参考訳として共有させていただきます。以下どうぞ。


# HashiCorp Consul 1.2: サービス・メッシュ(Service Mesh)

[HashiCorp Consul 1.2](https://www.consul.io/) のリリース発表をとてもうれしく思います。このリリースでは [Connect（コネクト）](https://www.consul.io/segmentation.html) と呼ぶ重要な新機能をサポートします。この機能は、既存の全ての Consul クラスタをサービス・メッシュ（service mesh）ソリューションへと自動的に転換します。Connect により、TLS 暗号化の自動化と、同一性をベースとした認証によって、サービスとサービスとの間の通信を安全にします。

現在の Consul は、世界で 100 万を超えるマシン上で稼働しています。Consul 1.2 にアップグレードし、Connect を有効化したら、あらゆる既存のクラスタが即座にサービスメッシュ・ソリューションになるでしょう。そしてこれは物理マシン、クラウド、コンテナ、スケジューラなどを問わず、あらゆるプラットフォーム上で動作します。

## 最新のサービスメッシュを用いたサービスネットワーク機能

組織においてマイクロサービスの導入やクラウドを基盤とした動的なインフラを導入するには、サービスメッシュ（service mesh）が必要です。近年の実行環境は、極めて動的な性質を持ちます。そのため、こちらに対応するためには、旧来のホストを土台とする（host-based）ネットワーク・セキュリティを、最新のサービスを土台とする（service-based）セキュリティに置き換える必要があります。

サービスメッシュによって提供されるのは、高い可用性と、分散環境における重要な課題３つに対する解決策です：

* [ディスカバリ（discovery）](https://www.consul.io/discovery.html)：サービスは相互に発見可能な必要
* [設定（configuration）](https://www.consul.io/configuration.html)：サービスは中央源（central source）からのランタイム設定を受け入れる必要
* [分割（segmentation）](https://www.consul.io/segmentation.html)：サービス通信は必ず認証および暗号化されている必要

今回のリリースまで、 Consul はディスカバリと設定の問題を解決するために、ディスカバリには DNS を、設定にはキーバリュー（ストア）を用いてきました。Connect が新たに解決するのは、分割（segmentation）のためです。３つの機能全てが同時に機能したら、あらゆるプラットフォーム上における完全なサービスメッシュ・ソリューションをもたらします。

## Consul Connect

Connect は Consul における主要な新機能であり、TLS 自動暗号化と同一性を基盤とした（identity-based）認証により、安全なサービス間通信を提供します。今日発表した Connect 機能は、完全に自由（free）かつオープンソースです。Connect は Consul 1.2 においては公開ベータ機能として利用できます。

使い勝手のよさを考慮して Connect は構築されました。単なる設定オプションとして扱えるだけではありません。これに加え、１つの外部ラインを追加し、 Connect を基盤とした接続を採用するだけで、既存のあらゆるアプリケーションは自動的にサービスへの登録が可能となります。証明書の切り替えは自動的かつ無停止です。Connect に必要なのはバイナリ１つだけであり、これで全ての必要なサブシステムが使えます。今後は、更に他の機能も扱います。

利便性を高めるため、Connect は Consul に対して多くの新しい機能を提供します。この投稿の後半で機能の幾つかを御紹介しますが、まずは Consul で Connect を利用した場合の主な新機能を列挙します：

* **トラフィック暗号化（Encrypted Traffic）** : 全てのトラフィックは共通の TLS を通し、 接続を確立します。これを確かなものとするため、全てのトラフィックを通信中に暗号化します。そのため、信頼性の低い環境においても、安全にサービスを展開（デプロイ）できるようになります。

* **接続認証（Connection Authorization）** : [intentions](https://www.consul.io/docs/connect/intentions.html) でサービス・アクセス・グラフを作成し、サービス通信の許可（allow）・拒否（deny）をします。ファイアウォールで IP アドレスを使うのとは違い、Connect はサービスの論理名を使います。規模に関係なくルールを指定できるため、ウェブサーバは１つでも100でも構いません。intention の設定は UI 、CLI、API や HashiCorp Terraform から行えます。

* **プロキシ・サイドカー（Proxy Sidecars）** : アプリケーションは軽量な proxy サイドカーのプロセスで、自動的にインバウンドとアウトバウンドの TLS 接続を確立します。これを、既存のアプリケーションに変更を加えることなく利用可能にします。Consul はプロキシ機能を内蔵して提供するため、Envoy のようなサードパーティによるプロキシは不要です。

* **ネイティブな統合（Native Integration）**: 性能が気になるアプリケーションは Consul Connect API でネイティブに統合できるため、最適な性能とセキュリティをプロキシせずに接続の確立と受け入れをします。

* **レイヤ４対レイヤ７（Layer 4 vs. Layer 7）**: レイヤ４では同一性が強制されます。Consul はレイヤ７機能と設定を置き換え可能なデータレイヤに委任します。サードパーティ製プロキシとも統合できますので、Consul 用のサービスディスカバリ、同一性、認証を学ぶのと同時に、Envoy のようなパスをベースとするルーティング、追跡（トレーシング）等の機能を扱えます。

* **証明書管理（Certificate Management）**: Consul は認証局（CA）プロバイダを用い、証明書の生成と配布をします。Consul は CA システムを内蔵しているため、外部の依存性はありません。また HashiCorp Vault との統合により、他の PKI システムをサポートするように拡張できます。

* **証明書のローテーション（Certificate Rotation）**: Connect は root と leaf 証明書の両方を自動的に切り替え可能です。root ローテーションは証明書を相互に署名し、ローテーション中も新しい証明書と古い証明書が共存できるため、サービスの中断はありません。また、この仕組みは新しい CA プロバイダを有効にするときも、シームレスな設定を可能とします。

* **SPIFFE を基盤とした識別（SPIFFE-based Ientities）**: Consul はサービス識別の仕様に [SPIFFE](https://spiffe.io/) を用います。そのため Connect サービスは他の SPIFFE 互換システムとの接続の確立や受け入れが可能です。

* **ネットワークとクラウドの独立（Network and Cloud Independent）**: Connect は標準の TLS over TCP/IP を用います。そのため、Connect はあらゆるネットワーク設定上で動作します。また、目指すサービスが動作しているオペレーティングシステムに到達するためには、 IP アドレスで伝えます。更に、複雑なオーバレイを用いることなく、サービスはクラウド間で相互に通信可能です。

## 自動プロキシ・サイドカー

アプリケーションでインバウンドとアウトバウンドの接続を確立するとき、プロキシ・サイドカーを使えばオリジナルのアプリケーションに対する変更は不要です。Connect を有効化するために1行の変更を加えるだけで、あらゆる登録サービスで Connect をベースとした接続を受け入れ可能になります。

```
{
  "service": {
    "name": "web",
    "port": 8080,
    "connect": { "proxy": {} }
  }
}
```

唯一の違いは「connect」で始まる行のみです。この1行の設定変更が存在するだけで、Consul はこのサービスに対するプロキシ接続を自動的に開始・管理します。プロキシするプロセスが対象サービスに相当します。動的に割り当てられるインバウンドの接続を受け入れ、TLS によって検証かつ認証された接続で、プロセスに対して標準的な TCP 接続でプロキシし返します。

上流（upstream）の依存関係では、更に数行の記述を追加したら、Connect の外にあるサービスに対するプロキシ接続のリスナーを設定できます。例えば、「web」サービスが「db」サービスと Connect を経由して通信する必要がある場合は次のようにします：

```
{
  "service": {
    "name": "web",
    "port": 8080,
    "connect": {
      "proxy": {
        "config": {
          "upstreams": [{
             "destination_name": "db",
             "local_bind_port": 9191
          }]
        }
      }
    }
  }
}
```

このプロキシ管理設定は、ローカルのみのポート 9191 のリスナーに対し、あらゆるリモートの「db」にプロキシするようにセットアップしています。「web」でこのローカルポートを使うように再設定したら、「web」と「db」間の全ての通信が暗号化かつ認証された状態で行われます。

以上の例を通しての注意点としては、元々のアプリケーション「web」は *変更しておらず、Connect を意識してないまま*  なのです。あらゆるアプリケーションが設定ファイルに対してわずか数行を追加するだけで、自動的に管理されたサイドカー・プロキシを用いる Connect をベースとした通信の受け入れ・確立が可能です。

より詳しく学ぶには、 [プロキシに関する参照ドキュメント](https://www.consul.io/docs/connect/proxies.html) を御覧ください。アプリケーションが著しく高い性能を必要とする場合は、アプリケーションは Connect と [ネイティブに統合](https://www.consul.io/docs/connect/native.html) できます。そうしますと、サービスは全てをプロキシする必要はなくなります。

## 開発が簡単な接続

セキュリティ最適化のために、サービスは Connect をベースとした接続のみを受け入れるようにすべきです。しかしながら、導入のためには、開発やテスト段階において、サービスに対する接続の変更が必要となります。 Consul が提供する `consul connect proxy` コマンドを使えば、ローカルのプロキシを動かし、 サービスが Connect を経由して通信を確立できるようにします。

例として PostgreSQL サーバが動作しているシナリオを検討しましょう。このサーバは Connect を経由した通信のみ受け入れ可能です。そして、作業者がデータベースをメンテナンスするには、コマンドラインで `psql` を使い作業を試みるでしょう。作業者は `consul connect proxy` コマンドを使えば、ローカルマシン上での `psql` を経由して接続可能になります：

```
$ consul connect proxy -service mitchellh -upstream postgresql:9191
==> Consul Connect proxy starting...
    Configuration mode: Flags
               Service: mitchellh
              Upstream: postgresql => :9191
       Public listener: Disabled
...
```

そして、他のシェルで、通常の `psql` で接続できます：

```
$ psql -h 127.0.0.1 -p 9191 -U mitchellh mydb
>
```

`-service` フラグはソースを識別するのに使います。サービスを終了する必要はありませんが、対象サービスを登録可能な有効な ACL トークンを持っている必要があり、また、Consul は送信元（source）と送信先（destination）間で通信可能に設定しておく必要があります。

ローカルの開発とリモートサービスにおけるテストが、サービスメッシュにおいて共通のワークフローにする課題がありますが、これを Consul と Connect で極めて簡単にします。

## intentions によるアクセス制御

サービス間のアクセス制御は "intentions"  （意図）で設定可能です。intention は１つの送信元（source）と送信先（destination）の間を、許可（allow）と拒否（deny）のルールで指定します。intentions は UI、CLI、API 、Terraform で作成できます。

以下の例は、 `db` サービスに対して `web` からの全てのアクセスを許可（allow）します。

```
$ consul intention create -allow web db
Created: web => db (allow)
```

「web」サービスは「db」サービス間との通信が許可されています。 intention では `-deny` で更新したら、簡単に２つのサービス間の通信を無効化できます。

intention はサービス開発と ACL ルールの管理を設定によって分離できるため、担当者は特定のサービスに対する intention のみ変更できます。これによりセキュリティと分割（segmentation）のどちらもリアルタイムに近く動的な設定変更と管理が可能となります。

## 機能について更に詳しく

Consul 1.2 では重要な新機能をサポートできましたので、私たちはうれしく思います。機能に対する栄光と大きさを鑑み、Consul 1.2 では Connect はベータであるべきと考えています。今夏は Connect に対して更に熱心に取り組み、年末に至るまでにはベータのタグを外したいです。

また、近いうちに Connect に更なる機能性を追加します。これには新しい UI 拡張や、Envoy をプロキシとしてのサポートや、Nomad や Kubernetes との統合などです。Consul 1.2 における Connect は、ほんの始まりにすぎません。

Consul 1.2 のダウンロードは https://www.consul.io/ に移動します。

次のステップに進むため、更に学ぶには以下のページが役立ちます：

* [機能のホームページ](https://www.consul.io/segmentation.html) - Consul 内の Connect 専用のホームページがこちらです。機能の概要と、これから始めるためのガイドがあります。
* [Connect 導入ステップ](https://www.consul.io/intro/getting-started/connect.html) - 導入ガイドでは Connect の導入ステップを経ながら、 Connect 概要を数分で見渡せます。
* [Connect 双方向チュートリアル](https://play.instruqt.com/hashicorp/tracks/connect) - 双方向チュートリアルでは Connect 実行のために重要なステップを見ていきます。
* [Connect リファレンス・ドキュメント（参照文章）](https://www.consul.io/docs/connect/index.html) - Connect 用のリファレンス・ドキュメントには、どのように Connect が動作し、プロキシをし、ネイティブに統合し、証明書を管理するか等の詳細があります。Connect をデプロイする前に読んでおくのを推奨します。
* [Connect セキュリティ・チェックリスト（安全確認項目）](https://www.consul.io/docs/connect/security.html) - Connect のセキュリティに対する考え方は Consul とは異なります。より安全に扱うために、このチェックリストを使ったレビューと、読み込みと、Consul の脅威モデルの理解を推奨します。
* [Connect プロダクション・ガイド](https://www.consul.io/docs/guides/connect-production.html) - プロダクション（本番向け）Connect 用の Consul クラスタの設定をするための完全ガイドです。完全に安全な設定をするために必要なステップの全てがあります。
* [Connect ホワイトボード・セッション](https://www.hashicorp.com/resources/introduction-consul-connect) - 創業者かつ共同 CTO の Armon Dadger が作成したビデオでは、ネットワーク管理、セキュリティ、性能における Connect の機能を紹介しています。

## 原文

* HashiCorp Consul 1.2: Service Mesh
  * https://www.hashicorp.com/blog/consul-1-2-service-mesh

