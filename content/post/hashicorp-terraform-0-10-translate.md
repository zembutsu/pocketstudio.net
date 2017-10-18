+++
date = "2017-08-03T21:14:00+09:00"
draft = false
slug = "hashicorp-terraform-0-10-translate.md"
title = "【参考訳】HashiCorp Terraform 0.10"
categories = ["HashiCorp"]
tags = ["Terraform", "HashiCorp","translation"]
+++

# "HashiCorp Terraform 0.10" 概要

HashiCorp の blog に、  [Terraform 0.10 のリリースに関するブログ投稿](https://www.hashicorp.com/blog/hashicorp-terraform-0-10/) がありました。今回はコアとプロバイダの分離という大きな変更を伴っています。その背景の理解が、今後の Terraform を利用する上で欠かせないと感じ、共通認識を確保したく、例によって参考訳を作成しました。以下どうぞ。


[<img src="/images/blog-terraform.svg" width="100%">](/images/blog-terraform.svg)


## HashiCorp Terraform 0.10

[HashiCorp Terraform 0.10](https://www.terraform.io/) のリリー発表を嬉しく思います。Terraform はインフラ（infrastructure、訳者注；システム基盤）を安全かつ効率的に構築、比較、起動するツールです。今回のリリースには多くの新機能と改良が含まれています。

* [ダウンロード](https://www.terraform.io/downloads.html)

前回の Terraform メジャー・リリース以降、11 のマイナー・リリースを行い、６つの新しいプロバイダ、24 のデータソース、60 以上の新しいリソースを追加しました！　それだけでなく、 Terraform プロジェクトは 1,100 名を超える貢献者からのコントリビューションを受けました。

Terraform 0.10 では、Terraform に多くの重要な新機能が加わります。重要なのはこちらです：

* Terraform コアとプロバイダを分割
* 数々のプロバイダを改良
* Stete environments はワークスペース（Workspaces）に

## Terraform コア（Core）と Terraform プロバイダを分割

terraform 0.10 では、プロジェクトを２つの論理コンポーネントに分割しました。それが Terraform コアと Terraform プロバイダです。Terraform コアは GItHub 上オリジナルの [hashicorp/terraform](https://github.com/hashicorp/terraform) に存続します。一方のプロバイダは新しい [GitHub 上の Terraform Providers organization](https://github.com/terraform-providers/) に移動しました。また、Terraform プロバイダのバイナリも Terraform コアからは独立してリリースされます。全てのリリースは [releases.hashicorp.com](https://releases.hashicorp.com/) からダウンロードできます。

GitHub リポジトリと配布用バイナリの分離によって目指すのは、コミュニティのメンバーが所有・貢献することによる Terraform 改良の加速です。

今後のプロバイダ・プラグインは Terraform のバイナリと共に配布されません。そのかわり、個々に配布されることになり、 `terraform init` コマンドにより、必要に応じて取得およびインストールが行われます。この新しい手法により、利用者は個々のプロバイダごとにアップグレード可能になります。また、プロバイダの調整やパッチをあて、プロバイダの削除を個別に行えます。

Terraform 本体のバージョンとプロバイダ・プラグインが切り離されましたので、個別にバージョン制限を行うことにより、上流側による破壊的な変更（breaking changes）から利用者を守ります。これは `git init` のようなもので、 `terraform init` が日々の Terraform ワークフローにおいて重要な部分になります。

分割に関する詳細な情報については、以前の投稿 [Upcoming Provider Changes in Terraform 0.10](https://www.hashicorp.com/blog/upcoming-provider-changes-in-terraform-0-10/) をご覧ください。

## プロバイダの分割

各プロバイダのソースが現在置かれている場所は、 [Terraform Providers GitHub organization](https://github.com/terraform-providers) にある個々のリポジトリ内です。 Terraform コアから各プロバイダが分割されたため、各プロバイダ自身がリリースの調整、バージョン指定、ドキュメントを持てるようになりました。

今後のプロバイダ・プラグインは Terraform のバイナリと共に配布されません。そのかわり、個々に配布されることになり、`terraform init` コマンドにより、必要に応じて取得およびインストールが行われます。新しいプロジェクトのスタート時に、あるいはプロバイダの追加・削除といった設定に関する変更時は、 `terraform init` を実行しますと Terraform は設定を読み込み、必要なバイナリを取得します。

[Terraform Providers GitHub organization](https://github.com/terraform-providers)  にある、あらゆるプロバイダを Terraform は自動取得します。各プロバイダのバイナリは、 HashiCop によって事前にビルドされ、 [releases.hashicorp.com](https://releases.hashicorp.com/)  上に置かれています。

## プロバイダの制約

プロバイダとコアの分割により一部で重要なのは、今後のプロバイダはそれぞれがバージョン番号を餅、コアや他のプロバイダとは別に進むことです。Terraform は動的に利用可能なプロバイダを必要に応じて取得し、設定ファイル上でバージョン指定があれば従います。たとえば、AWS と Fastly の２つのプロバイダを使う設定を考えましょう。

```
provider aws {
  version = "~> v0.1.3"
  region  = "us-west-2"
}

provider fastly {
  api_key = "exampleapikey"
}
```

こちらには２つのプロバイダ・ブロックを宣言し、AWS プロバイダは少なくとも v0.1.3 であるべきと指定しています。Fastly プロバイダの宣言ではバージョン指定を省きましたので、Terraform は最新版を取得します。そのため、 `terraform init` を実行することで Terraform は設定ファイルを読み込み、それから、動的に必要なバイナリを取得するのが想像できるでしょう。

## プロバイダの改良

Terraform プロバイダは、新しいリソース、新しいデータソース、バグ修正など、数々の改良や追加機能を導入しました。さらに進めているのは、Terraform コアの Changelog には Terraform プロバイダの変更に関する情報を記録しません。各プロバイダは既に GitHub 上に独立したリポジトリをそれぞれ持っており、自身の課題追跡（issue tracker）や変更履歴（Changelog）を持っています。例：

* [Terraform AWS Provider Changelog](https://github.com/terraform-providers/terraform-provider-aws/blob/master/CHANGELOG.md)
* [Terrform Google Compute Changelog](https://github.com/terraform-providers/terraform-provider-google/blob/master/CHANGELOG.md)

[Terraform Provider GitHub organization リポジトリ一覧](https://github.com/terraform-providers) で、すべてのプロバイダと対応している変更履歴を参照できます。

また、Terraform エコシステムを将来的に拡張するため、 [Terraform プロバイダ開発プログラム](https://www.terraform.io/guides/terraform-provider-development-program.html) も開始しました。これが目指すのは、自分たちのインフラをサポートする Terraform プロバイダを構築したい提供者（ベンダ）と利用者（ユーザ）のためです。Terraform プロバイダ開発プログラムは大部分が自己解決型であり、情報源へのリンク、明確なステップの定義、チェックポイントがあります。

## State Environemnts はワークスペース（workspaces）へ

Terraform 0.9 は ["State Environments"](https://www.hashicorp.com/blog/terraform-0-9/#environments) を導入しました。これは state （状態）ファイルに名前空間を使うことで、１つのフォルダ内で複数の明確なインフラ・リソースを管理する設定が可能にしました。初リリース以降、コミュニティからは用語が混乱を招くとの指摘を受けました。 Terraform 0.10 からは「State Environments」の用語を置き換え「Workspace」の概念を導入しました。それに伴い、 `terraform env` コマンド系列は `terraform workspace` へと名前が変わります。 `env` サブコマンドは後方互換性のためにサポートされ続けますが、将来的なリリースにおいて削除予定です。これらのコマンドを使う自動処理やスクリプトの書き換えを推奨します。

Terraform ワークスペースに関する詳しい情報は [ドキュメントのページ](https://www.terraform.io/docs/state/workspaces.html) をご覧ください。

## アップグレード

コアとプロバイダが分離するため、私たちが作成したガイド [Upgrading to Terraform 0.10](https://www.terraform.io/upgrade-guides/0-10.html) を注意深くお読みください。Terraform 0.10 のコアには後方互換性がありませんが、反対や変更があれば、可能な限り早く取り込むでしょう。0.9 にアップグレードする前に、アップグレード・ガイドを確認し、考慮点の全てをご検討ください。

これらの変更に加え、プロバイダやリソースに対する数々の変更が、皆さんのご利用方法によっては影響を引き起こす可能性があります。ほぼ全てのリリースにおいて、既存のプロバイダやリソースには何らかの変更があります。より詳しい情報については、適切な変更履歴（changelog）をご覧ください。

## まとめ

Terraform はリリースの度に成長と成熟を重ねています。Terraform をコードとしてのインフラ（infrastructure as code）を管理する業界有数のツールにすべく尽力いただいている、Terraform のコミュニティのメンバーすべてに感謝を申しあげます。

Terraform コアについては、私たちはこれまで通りの基準でリリースを続けます。しかし、プロバイダを分割したことで、リリースは以前のペースよりは遅くなるでしょう。今後の Terraform プロバイダは、必要性に応じて個々にリリース・スケジュールを調整できるようになりました。

あとは [Terraform をダウンロード](https://www.terraform.io/downloads.html) して、試してみましょう！

## 原文

* HashiCorp Terraform 0.10 | HashiCorp
https://www.hashicorp.com/blog/hashicorp-terraform-0-10/  * 
