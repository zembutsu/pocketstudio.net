+++
date = "2016-07-02T17:00:00+09:00"
draft = false
slug = "nomad-4-0-translate"
title = "【参考訳】Nomad 0.4"
categories = ["Nomad"]
tags = ["HashiCorp", "Nomad","translation"]
+++

## 概要

6月28日に、[HashiCorp](https://www.hashicorp.com) から [Nomad 0.4](https://www.nomadproject.io/) バージョン 0.4 がリリースされました。新しい ``nomad plan`` コマンドのサポート、リソース調査機能、Consul との一部機能統合が行われています。HashiCorp の blog に[リリースに関する解説記事](https://www.hashicorp.com/blog/nomad-0-4.html)がありましたので、例によって訳してみました。参考程度にどうぞ。

## Nomad 0.4

私たちは [Nomad 0.4](https://www.nomadproject.io/) をリリースしました。Nomad は分散型の高い可用性を持つクラスタ・マネージャとスケジューラであり、マイクロサービスとバッチ作業の両方に対応する設計です。

Nomad 0.4 に備わっている新機能が焦点を当てているのは、ツール運用面での改良です。主な機能は次の項目です。

* Nomad Plan
* ライブ・リソース使用量（Live Resource Utilization）
* クラスタリングを簡単に（Simpler Clustering）

## Nomad Plan

nomad plan を実行して表示されるのは、ジョブに対する変更についてと、Nomad がジョブをクラスタ上に割り当て可能かどうかです。これにより、システムに対してどのような変更を行うかと、ジョブが適切に割り当てられるかを確認できるようにします。

Terraform とは異なり、nomad plan は割り当ての成功を保証しません。nomad plan は変更するリソースを予約（reserve）しません。確認するのは、その時点で割り当てが成功するかです。この情報を元に、作業者は対象となる場所でジョブを実行するかどうか、詳細な情報を得た上で決断できます。

Nomad は宣言型システム（declarative system）です。「何」を実行するか宣言したら、Nomad は「どのように」実行するか決めます。どのサーバ上でジョブを実行するか、いつ実行するかなど、Nomad に対して詳細を伝える必要はありません。これがクラスタ・マネージャにとって重要な所です。たとえば、Nomad はリソースを効率的に使えるようにし、障害時には自動的にワークロードの移行などをします。

一方、マイナス面として、ジョブを発行後の挙動については分かりません。作業者であれば、何点か気に掛けることがあるでしょう。ジョブの実行に十分なリソースがあるだろうか？　ジョブを更新するには？　既存のジョブのダウンタイムやローリング・アップデートは？　などです。

``nomad plan`` は作業を確かなものにするため、その時点において Nomad が何をするか表示します。次の plan は、ジョブを更新する例です：

```
$ nomad plan example.nomad
+/- Job: "example"
+/- Task Group: "cache" (3 create/destroy update)
  +/- Task: "redis" (forces create/destroy update)
    +/- Config {
        args[0]: "--port ${NOMAD_PORT_db}"
      + args[1]: "--loglevel verbose"
        command: "redis-server"
    }

Scheduler dry-run:
- All tasks successfully allocated.
- Rolling update, next evaluation will be in 10s.

Job Modify Index: 7
```

出力はジョブに対する変更を表示します（この例では、コマンドに新しい引数を追加します）。ここで出力されるのは、ジョブの作成/破棄、更新といった処理（タスク）で、どのように変わるかです。

あとは、下の方に「scheduler dry-run」（スケジューラ・ドライ・ラン）と表示があります。ここではジョブが割り当て可能であるのと、ローリング・アップデートは 10 秒間隔のポリシーでデプロイできそうです。

新しいジョブや既存のジョブに対し、nomad plan が使えます。クラスタの状態には変更を加えませんので、安全に実行できます。[nomad plan の詳細はドキュメント](https://www.nomadproject.io/docs/commands/plan.html)をご覧ください。

## ライブ・リソース使用量（Live Resource Utilization）

Nomad はタスクとノードに割り当てられた実際のリソースを報告できるようになりました。

Nomad 上のすべてのジョブは、どれだけのリソースが必要かの宣言が必須です。リソースとは、CPU、メモリ、ネットワーク等です。通常、処理の要求時点で、どれだけのリソースが必要かを知るのは大変です。これまで、実際に必要なリソース使用量を決めるのは困難でした。Nomad 0.4 からは、ジョブ、タスク、ノードのリソース使用量を簡単に調べられます。

次の例はタスクのリソース使用量を表示します。

```
$ nomad alloc-status abcd1234
Task: "www"
CPU     Memory MB  Disk MB  IOPS  Addresses
100/250   212/256     300      0
```

Nomad 0.3 までは、CPU とメモリはジョブで指定した要求値を簡単に表示するだけでした。Nomad 0.4 からは、実際に現在割り当て中の値を表示します。

ノードの状態では、より詳細な情報を表示します。

```
$ nomad node-status -stats abcd1234
...

Detailed CPU Stats
CPU    = cpu0
User   = 1.03%
System = 0.00%
Idle   = 98.97%

CPU    = cpu1
User   = 1.00%
System = 2.00%
Idle   = 93.00%

Detailed Memory Stats
Total     = 2.1 GB
Available = 1.9 GB
Used      = 227 MB
Free      = 1.4 GB

Detailed Disk Stats
Device         = /dev/mapper/ubuntu--1404--vbox--vg-root
MountPoint     = /
Size           = 41 GB
Used           = 3.4 GB
Available      = 36 GB
Used Percent   = 8.14%
Inodes Percent = 4.94%
```

これらの情報は常に更新されるため、ジョブの調整作業を行いやすくし、適切な挙動をしていないアプリケーションの発見などに使えます。

## クラスタリングを簡単に（Simpler Clustering）

Nomad 0.4 は、クラスタの作成と操作を簡単にするため、２つの重要な変更を行いました。 [Consul](https://www.consul.io/) を使えば、クラスタの作成は自動的です。Nomad サーバとクライアントは、Consul のサービスとヘルスチェックに自動登録されます。Consul デプロイとの統合は、Nomad サーバが自動的にリージョン間を統合するのを意味します！

クラスタの安定性だけでなく、サーバの更新もシンプルになります。Nomad サーバには、サーバとクライアント間をハートビート（heartbeats）を通す一式を備えました。ハートビートを約 30 秒ごとに行い、Nomad サーバがリージョン上で現在把握しているノード情報を提供します。これにより、Nomad サーバはクライアント側の設定を変更しなくても、イミュータブル（訳者注：システムの状態を変えずに）に更新を行えるようにします。

これまで Nomad サーバを更新するには、新しいサーバを準備し、Raft 複製が発生する前に古いものを廃止していました。そして、クライアントは新しいサーバを参照するするよう、設定ファイルを書き換える必要があったのです。この更新手順は厄介であり、間違いやすい傾向がありました。

Nomad 0.4 のサーバ、はクライアントに対して利用可能なサーバ一覧を通知します。つまり、クライアント側の設定を変えなくても、Nomad サーバの入れ替えを可能とします。

## まとめ

まだ Nomad は非常に若いプロジェクトです。しかし、導入と成長、そして、Nomad がプロダクションのワークロードを実行する様子はエキサイティングです。Nomad 0.3 は [100 万コンテナのスケーラビリティ](https://www.hashicorp.com/c1m.html) をもたらしました。Nomad 0.4 で集中するべく選んだ機能は、操作における信頼性の改善でした。

さらに前進するため、私たちは複数の重要な機能を計画しています。ネイティブな Vault 統合、さらなる Consul 統合などです。正確なロードマップを近々リリースする予定です。

[Nomad](https://www.nomadproject.io/) のサイトに移動し、詳細を学びましょう。

## 原文

* Nomad 0.4 - HashiCorp
  * https://www.hashicorp.com/blog/nomad-0-4.html

