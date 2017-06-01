+++
date = "2017-06-01T17:46:00+09:00"
draft = false
slug = "vagrant-cloud-migration0announcement-translate.md"
title = "【参考訳】Vagrant Cloud へ移行のお知らせ（6月27日）"
categories = ["HashiCorp"]
tags = ["Vagrant", "HashiCorp","translation"]
+++

# 概要

HashiCorp の blog に Vagrant イメージの保管サービス Atlas から、Vagrant に関連する機能を[6月27日に新 Vagrant Cloud に移行する](https://www.hashicorp.com/blog/vagrant-cloud-migration-announcement/)という発表がありました。

ポイントは以下の通りです。

* 公開 box のみの Vagrant 利用者は、移行後も同じ作業フローおよび Vagrantfile を利用できる
* Atlas アカウントを Vagrant Cloud で使うには、有効化が必要（使わないままなら10月に削除）
* Atlas 認証トークンは利用できなくなるので、事前に作成＆移行設定が必要
* 6月27日（日本時間28日午前7時）に30分以内のメンテナンスによる停止見込み

例によって詳細情報として参考訳を作成しました。以下どうぞ。

## Vagrant Cloud 移行のお知らせ

6月27日、HashiCorp の Atlas から  [Vagrant に関する機能](https://www.vagrantup.com/docs/vagrant-cloud/) を、自身のHashiCorp Vagrant クラウドへ移行します。

移行する機能は、以下の通りです：

- Vagrant Box の作成機能（Vagrant Box Creation）により、誰でも利用できるパブリックまたはプライベートな box の公開
- Vagrant Box のバージョン管理（Vagrant Box Versioning）により、box の更新と、利用者に変更の通知
- Vagrant Box カタログ（Vagrant Box Catalog）による公開 Vagrant box の検索と発見

将来的に Vagrant Cloud の開発を独立させることで、現状の機能改善や、Vagrant 関連の新サービスを定期的に提供できるようになります。

もしも Vagrant を公開 box のダウンロードと実行のみに利用しているのであれば、これまでと何も変わりません。すべての box 名、バージョン、URL は同じままであり（あるいはリダイレクトします）、皆さんの作業フローや Vagrantfile に対する変更は不要です。

既存の Vagrant box やアカウントの移行に関する詳細を知りたい場合は、以下を読み進めてください。

## 背景

Vagrant Cloud は HashiCorp 初のホステッド・プロダクト（訳者注：データ等を預かる製品・サービス）であり、初めてのオープンソース・プロダクトである Vagrant 用に向けて、Vagrant box の検索、管理、共有機能を提供しています。これまでは Atlas として、そして Terraform と Packer 機能に特化した Terraform Enterprise として発展してきました。

HashiCorp のプロダクト一式（suite of product）に統合されたプラットフォームは、たとえば Terraform Enterprise、Consul Enterprise、Vault Enterprise、Vagrant Cloud、いずれは Nomad Enterprise があります。これらのプロダクトから、 Atlas は離れつつあります。今ある Atlas の全機能は、プロダクトごとに分けられています。現在、 atlas.hashicorp.com ドメインに存在している機能は Terraform Enterprise と Vagrant Cloud 機能のみです。そして、近いうちに Terraform Enterprise に関するドメインは移行します。移行が完了したら、Atlas ブランドを削除します。

## Vagrant Box、ユーザ、organization、team

すべての Vagrant box は、6月27日に新しい Vagrant Cloud へ移行します。さらに、すべてのユーザと organization を複製します。既存の box に対するコラボレータやチーム制限（ACL）は、新しいシステム上でも同一です。

既存の box 名（hashicorp/precise64）と URL は現状のまま利用できます。あるいは、適切な場所へ恒久的にリダイレクトします。皆さんがパブリック Vagrant box しか使っていないのであれば、お使いの Vagrantfile や作業手順を変える必要はありません。プライベート Vagrant box の利用者は、新しい認証（詳細は後述）の作成と、移行完了後に Vagrant Cloud アカウントの有効化（アクティベート）が必要です。

Vagrant Cloud のユーザと organization のうち、Vagrant Cloud への移行した後でもログインしていないか Vagrant box を公開していなければ、使用していないものと判断します。使われていないアカウントは 2017年10月1日に削除します。

## Vagrant Cloud アカウントの有効化

Atlas アカウントで Vagrant Cloud を使い始めたければ、まず Vagrant Cloud アカウントの有効化が必要です。そのためには Atlas へのログインとパスワード認証が必要です（設定している場合は、二要素認証も）。Vagrant Cloud ログイン画面上にも、同様のリンクや手順を表示します。

Vagrant Cloud アカウントが有効化されたら、Vagrant Cloud 用の新しいパスワードと、オプションで二要素認証も設定できます。これまでの Atlas アカウント、パスワード、二要素認証の設定を Atlas で変更せずに使い続けられません。

Vagrant Cloud の新しいユーザは、アカウントをいつでも無料で作成できます。

## 認証トークン

もしも現在 Atlas の Vagrant 機能を使うために認証トークンを使っているのであれば、新しい Vagrant Cloud トークンを6月27日よりも前に作成する必要があります。既存のトークンの確認と新しいトークンの作成には、[アカウント設定にあるトークンのページ](https://atlas.hashicorp.com/settings/tokens) をご覧ください。

新しいトークンの作成時には "Migrate to Vagrant Cloud"（Vagrant Cloud へ移行）のチェックボックスを有効にします。

トークンの一覧から、との認証トークンを Vagrant Cloud に移行するかを確認できます。

（対象の）認証トークンは6月27日に Vagrant Cloud へ移行します。また、この時点で Terraform Enterprise からも削除され、Terraform や Packer 操作を行えなくなります。6月27日までに Atlas でトークンを作成していない場合は、Vagrant Cloud に移行後、トークンの作成が必要になります。

## Packer と Terraform Enterprise

Terraform Enterprise (Atlas) における Vagrant box の作成にあたり、 Packer には２つのポスト・プロセッサ（post-processor）、つまり [atlas](https://www.packer.io/docs/post-processors/atlas.html) と [vagrant-cloud](https://www.packer.io/docs/post-processors/vagrant-cloud.html) があります。atlas ポスト・プロセッサでは、6月27日以降 Vagrant box を作成できません。現在 Packer で Vagrant box の送信を行っている場合は、vagrant-cloud ポスト・プロセッサを使用中かどうかご確認ください。具体的には [vagrantup.com の Vagrant Cloud 移行ドキュメント（英語）](https://www.vagrantup.com/docs/vagrant-cloud/vagrant-cloud-migration.html) をご覧ください。

## Vagrant Share

Vagrant Share 機能は廃止となります。そのかわりに Vagrant は ngrok とのネイティブな統合をサポートします。Vagrant Share の利用者は6月27日までに [ngrok-powered Vagraht 共有ドライバ](https://www.vagrantup.com/docs/share/ngrok.html) への切替を行うべきです。こちらは Vagrant の次期バージョンからデフォルトになります。

## 停止時間

Atlas/Terraform Enterprise 内の Vagrant サービスは 6月27日の午後6時（EDT;米国東部夏時間）/午後3時（米国太平洋夏時間/ [タイムゾーンの確認](https://time.is/0600PM_27_June_2017_in_ET?Vagrant_Cloud_Migration) ）から（※日本時間6月28日(水) 午前7時～）移行完了まで少々の停止が発生します。かかる時間は30分以内を見込んでいます。移行状況に関する最新情報は [status.hashicorp.com](https://status.hashicorp.com/) をご覧ください。

## 詳細な情報

直近の移行に関する詳しい情報は、 [vagrantup.com の Vagrant Cloud ドキュメント](https://www.vagrantup.com/docs/vagrant-cloud/vagrant-cloud-migration.html) をご覧ください。Vagrant Cloud や移行に関するご質問がございましたら、 [support@hashicorp.com](mailto:support@hashicorp.com) までメールをお送りください。

## 原文

* Vagrant Cloud Migration Announcement | HashiCorp
  * https://www.hashicorp.com/blog/vagrant-cloud-migration-announcement/
