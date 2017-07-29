+++
date = "2017-07-29T12:50:00+09:00"
draft = false
slug = "hashicorp-nomad-0-6-translate.md"
title = "【参考訳】HashiCorp Nomad 0.6"
categories = ["HashiCorp"]
tags = ["Nomad", "HashiCorp","translation"]
+++

# "Nomad 6.0" 概要

HashiCorp の blog に、  [Nomad 0.6 のリリースに関するブログ投稿](https://www.hashicorp.com/blog/hashicorp-nomad-0-6/) がありました。Nomad は分散環境におけるジョブ管理に特化しているツールですが、最近の更新では Terraform 的な思想も受けている、あるいは継承しているように思えます。Terraform はインフラが主であるのに対し、Nomad はジョブやサービス視点でという違いはあります。

例によって詳細情報として参考訳を作成しました。以下どうぞ。


[<img src="/images/blog-nomad.svg" width="100%">](/images/blog-nomad.svg)


## HashiCorp Nomad 0.6

[HashiCorp Nomad 0.6](https://www.nomadproject.io/)  のリリース発表を、私たちは 嬉しく思います。Nomad は分散（distributed）、スケーラブル（scalable）かつ高可用性（highly available）クラスタ・マネージャであり、マイクロサービスとバッチ・ワークロード（batch workloads）の双方に向けて設計されたスケジューラです。

Nomad 0.6 は多くの改良やバグ修正に加え、ジョブ管理と設定の改良に集中した新機能を含みます。

* ジョブのデプロイ
* ジョブの履歴と、古いバージョンへの修復（revert）機能
* 動的な環境変数
* HashiCorp Consul でコンテナ IP アドレスの自動通知（automatic advertisement）

また、発表にあたりまして、現在は Apache Spark バージョンを含む  Nomad エコシステムに感謝申しあげます。[Apache Spark バージョン](https://github.com/hashicorp/nomad-spark) は Nomad を Spark クラスタ・マネージャとスケジューラとして、ネイティブに統合したものです。詳細については私たちのブログ投稿 [Nomad で Apache Spark を動かす](https://www.hashicorp.com/blog/running-apache-spark-on-nomad/) をご覧ください。

## ジョブのデプロイ（job deployments）

Nomad 0.6 はアプリケーションのバージョン間における移行において、rolling（ローリング）、canary（カナリア）、blue/green upgrades（ブルーグリーン・アップグレード）によって、安全に行う仕組みを導入しました。また、この新機能により、デプロイが失敗した時は安定したバージョンに自動修復（auto-revert）できるようになりました。ジョブの更新をするには、更新ストラテジ（update strategies；更新方針）のうち１つを指定します。また、ジョブの詳細として、 `update` 区間で注釈を追加することもできます。以下の例は Nomad でジョブを更新する例です。時間と条件を２つ割り当ててており、ロールアウト（ジョブの展開）をする前に、少なくとも 30 秒は正常（healthy）である条件を指定しています。

```
group "api-server" {
  # api サーバを 10 インスタンス起動
  count = 10

  update {
    # 同時に２つを並列更新
    max_parallel = 2

    # 新規割り当てにあたり、更新のためのブロック解除（unblock）をする前に
    # 少なくとも 30 秒は正常（healthy）であることを確実に処理
    min_healthy_time = "30s"

    # デプロイが失敗したとみなす時間を5分と割り当てる
    healthy_deadline = "5m"

    # 新しい割り当てが正常ではない（unhealthy）であれば、直近の最新版に自動修復（auto-revert）
    auto_revert = true
  }

  # API サーバの実行に Docker を使う
  driver = "docker"
  config {
    image "api-server:0.1"
  }

  ...
  }
```

ジョブの更新とは、見方によっては、新しい割り当て（allocation）の作成が必要と言えるため、Nomad はグループの更新ストラテジ（update strategy）を適用します。先の例では、イメージのグループ定義を `api-server:0.1` から `api-server:0.2` に変更し、再送信（resubmitted）しています。すると、 Nomad はローリング・アップデートを行います。ローリング・アップデートとは、新しい２つの `api-server:0.2` の割り当て（allocation）にあたり、１つが 30 秒は正常（healthy）になるまで待った後、新しく割り当てるものです。このヘルスチェックによる割り当てに失敗しますと、Nomad はデプロイが失敗したと印を付け（マークし）ます。 `auto_revert`  （自動修復）がセットされていれば、全てのジョブが正常に割り当てられるまで、直近のジョブにロールバックします。

全てのタスクが実行中（running）の状態になり、登録した [service checks](https://www.nomadproject.io/docs/job-specification/service.html) （サービス・チェック）が全て通過（passing）したとき、全ての割り当て状態が正常（healthy）になります。正常な指標（health metrics）に至らない時のため、 Nomad はカナリア・アップグレード（canary upgrades）をサポートしています。カナリア変更における更新ポリシーでは、次のように注釈を追加できます。

```
update {
  # デプロイにあたって、２つのカナリアに対する変更テストを行い、
  # 処理が進行したら、ローリング・アップデートを恵贈
  canary = 2

  # 先ほどと同様の更新フィールド
  ...
}
```

説明を続けます。更新ストラテジ通りにイメージの更新処理ができれば、 Nomad は `api-server:0.1` コンテナの実行を 10 インスタンス維持したまま、 `api-server:0.2` を割り当てて実行するカナリア２つを作成します。作業者がカナリアの状態が正常であると確認したら、作業者は次のコマンドを使い、カナリアを昇格（プロモート；promote）できます。

```
$ nomad job promote api-server
```

カナリアの昇格後は、 Nomad は割り当てられている古いイメージからの置き換え、すなわちローリング・アップデートの準備に入ります。カナリア・カウント（canary count）を任意のカウントに至るようにしておけば、ブルーグリーン・デプロイメントも行えます。このようにして、作業者はバージョンの更新のために昇進またはロールバックを行うため、ブルーとグリーンの環境をクラスタ上で完全に実現できます（訳者注；更新前の環境と更新後の２つ環境を、１つの環境上で並行稼働でき、必要があれば切り替えや巻き戻しも Nomad を通して簡単に行えるようになりました）。

update で囲まれた区間の記述に関する詳細は、 [job 操作ガイド](https://www.nomadproject.io/docs/operating-a-job/update-strategies/index.html) または [update stanza ドキュメント](https://www.nomadproject.io/docs/job-specification/update.html)  をご覧ください。

## ジョブ履歴と復旧（job history and reverting）

現在の Nomad 0.6 は複数バージョンのジョブを追跡するため、作業者は直近の変更状態を調査できます。作業者が新しいバージョンのジョブを送信（submit）したら、Nomad は自動的に新しい `Version` フィールドを増やし、次のようなバージョン情報を追加します。

```
# -p フラグで違うバージョンのジョブを表示
$ nomad job history -p example
Version     = 2
Stable      = true
Submit Date = 07/25/17 00:08:57 UTC
Diff        =
+/- Job: "example"
+/- Task Group: "cache"
  +/- Task: "redis"
    +/- Resources {
          CPU:      "500"
          DiskMB:   "0"
          IOPS:     "0"
      +/- MemoryMB: "256" => "512"
        }

Version     = 1
Stable      = true
Submit Date = 07/25/17 00:08:45 UTC
Diff        =
+/- Job: "example"
+/- Task Group: "cache"
  +/- Task: "redis"
    +/- Config {
      +/- image:           "redis:3.2" => "redis:4.0.1"
          port_map[0][db]: "6379"
        }

Version     = 0
Stable      = true
Submit Date = 07/25/17 00:08:33 UTC
```

さらに、 Nomad はジョブのバージョン間の修復機能（reverting）をサポートしています。これにより、ジョブの挙動が正しくなければ、作業者は迅速に復旧できます。

```
$ nomad job revert example 1
==> Monitoring evaluation "98dd3a0a"
    Evaluation triggered by job "example"
    Evaluation within deployment: "810d5c19"
    Allocation "399ad719" created: node "24dc095f", group "cache"
    Evaluation status changed: "pending" -> "complete"
==> Evaluation "98dd3a0a" finished with status "complete"
```

より詳細については [history](https://www.nomadproject.io/docs/commands/job/history.html) と [revert](https://www.nomadproject.io/docs/commands/job/revert.html) コマンドのドキュメントをご覧ください。

## 動的な環境変数（Dynamic environment variables）

Nomad 0.5 から新しいテンプレート・ブロック（template block）を導入しました。これは設定ファイルに様々な値を取り込むのに便利です。たとえば、 Consul データ、Vault シークレットからのデータを取り出したものや、Nomad タスク内で処理する一般的な設定などです。この機能は非常に強力ですが、全てのアプリケーションが設定ファイルを利用できるわけではありません。

Nomad 0.6 はこの機能を拡張し、 `template` ブロックの機能で `env` パラメータを導入できるようになりました。こちらを設定すると、 Nomad はテンプレートやパーサの値に動的な環境変数を割り当てるだけでなく、開始したタスクに対しても同様に処理します。

以下の例は Consul と Vault 両方のデータを環境変数に割り当てます。また、テンプレートはジョブファイルの外に分けて保存できるので、別々に保存します。

```
task "example" {
  # ...
  template {
    data = <<END
  LOG_LEVEL={{key "service/geo-api/log-verbosity"}}
  API_KEY={{with secret "secret/geo-api-key"}}{{.Data.key}}{{end}}
    END

    destination   = "secrets/config"
    env = true
  }
  # ...
}
```

タスクの環境においては、次のように環境変数を保持します。

```
LOG_LEVEL=DEBUG
API_KEY=12345678-1234-1234-1234-1234-123456789abc
```

これにより、設定ファイルを用いながら [twelve-factor](https://12factor.net/) 風の環境変数を扱えるようになり、しかも全ての機能が Nomad テンプレとの機能や構文と近いままなのです。

より詳しい情報は [template stanza ドキュメント](https://www.nomadproject.io/docs/job-specification/template.html) をご覧ください。

## Consul でコンテナの IP アドレスの自動通知（Automatic advertisement of container IP addresses with Consul）

Nomad 0.6 は Consul との統合を拡張しました。オーバレイ・ネットワークと Docker ドライバを使うユーザが対象です。これは、ホストネットワークではなくネットワーク・プラグインによって割り当てられる、到達可能な IP アドレスを自動的に通知（Advertise / 広報）します。この通知機能のためにはConsul とは別に、コンテナ自身を  [Registrator](https://github.com/gliderlabs/registrator) のようなツールをつかって登録する必要があります。

より詳しい情報は [Docker ドライバ・ドキュメント](https://www.nomadproject.io/docs/drivers/docker.html) をご覧ください。

## その他のバグ修正と改良（Other bug fixes and improvements）

以上の新機能だけでなく、多くのバグ修正や改良が施されています。変更点の一覧および詳細は [v0.6.0 changelog](https://github.com/hashicorp/nomad/blob/v0.6.0/CHANGELOG.md) をご参照ください。また、 ]更新ガイド](https://www.nomadproject.io/docs/upgrade/index.html) もご覧ください。

## Nomad 0.6 のウェビナー

最近、 Armon Dadger （HashiCorp の共同創業者・共同 CTO）と Caius Howcroft（Citadel の計算・データインフラ部長）が、組織におけるあらゆるワークロードの実行にあたり、 あらゆるインフラで柔軟性を確保しつつ、簡単に使え、安定して、性能を出す。そのために Nomad をどのように使うのかを議論しました。動画では次の内容を扱っています：

* Armon Dadgar による Nomad ディープ・ダイブ
* 最新の Nomad 機能を紹介するライブデモ
* Caius Howcroft からは、Citadel は Nomad によって、複数のパブリック・クラウドを高いスループットで横断してバッチ解析ができるように

ウェビナーでは Nomad 0.6 の新機能の概要と、Citadel がどのように Nomad の機能を活用しているかを扱っています。

https://www.youtube.com/watch?v=faNBpxOo8FQ

Nomad の更に詳しい情報や、使い始めるにあたっては、 https://www.nomadproject.io/ をご覧ください。

## 原文

* HashiCorp Nomad 0.6 | HashiCorp
  * https://www.hashicorp.com/blog/hashicorp-nomad-0-6/

