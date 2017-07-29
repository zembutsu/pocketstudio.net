+++
date = "2017-07-29T13:20:00+09:00"
draft = false
slug = "running-apache-spark-on-nomad-translate.md"
title = "【参考訳】HashiCorp Nomad で Apache Spark を動かす"
categories = ["HashiCorp"]
tags = ["Nomad", "HashiCorp","translation"]
+++

# Apache Spark 版 Nomad のリリース発表

HashiCorp の blog で Spark のクラスタマネージャ・スケジューラとして Nomad がネイティブに対応したという [記事](https://www.hashicorp.com/blog/running-apache-spark-on-nomad/) がでていました。こちらも参考程度にどうぞ。

[<img src="/images/blog-nomad.svg" width="100%">](/images/blog-nomad.svg)


## HashiCorp Nomad 上で Apache Spark を動かす

Apache Spark は人気のデータ処理エンジン／フレームワークであり、第三者によるスケジューラを使えるような設計がされています。スケジューラは利用可能ではありましたが、複雑さのレベルが高かったため、Spark ユーザの多くにとって不快だったでしょう。この溝を埋めるため、私たちは [HashiCorp Nomad](https://www.nomadproject.io/) エコシステムを発表します。ここに含まれる [Apache Spark バージョン（a version of Apache Spark）](https://github.com/hashicorp/nomad-spark) は、Spark クラスタのマネージャとスケジューラとして Nomad をネイティブに統合します。

## なぜ Nomad で Spark を？

Nomad の設計（Google の Borg と Omega から影響を受けた）は、解析アプリケーションの実行をより適切に行えるようにするための機能セットを有効化することです。特に関係があるのは [バッチ・ワークロード](https://www.nomadproject.io/docs/runtime/schedulers.html) のネイティブなサポート、並列化、 [高スループットのスケジューリング](https://www.hashicorp.com/c1m/) （Nomad のスケジューラ内部の詳細は、 [こちら](https://www.nomadproject.io/docs/internals/scheduling.html) をご覧ください）です。また、 Nomad はセットアップや利用が簡単であり、Spark 利用者の学習曲線と作業負担を軽減できうるでしょう。使いやすい主な機能は、次の通りです。

* バイナリ１つをデプロイするだけであり、外部の依存性が無い
* シンプルかつ双方向のデータ・モデル
* [宣言型のジョブ定義（declarative job specification）](https://www.nomadproject.io/docs/job-specification/index.html)
* 高可用性のサポートと [マルチ・データセンタ統合](https://www.nomadproject.io/guides/cluster/federation.html) を簡単にする

また、Nomad は [HashiCorp Consul](https://www.hashicorp.com/products/consul/) や [HashiCorp Vault](https://www.hashicorp.com/products/vault/) の [サービス・ディスカバリ](https://www.nomadproject.io/docs/service-discovery/index.html) 、 [ランタイム設定](https://www.nomadproject.io/docs/job-specification/template.html) 、 [シークレット管理](https://www.nomadproject.io/docs/vault-integration/index.html) とシームレスに統合します。

## どのように動作するのか

Nomad 上で実行すると、Spark エクゼキュータはアプリケーション用のタスクを実行します。そして（オプションで）、アプリケーション・ドライバ自身が Nomad ジョブ内で Nomad タスクを実行します。

利用者は Spark アプリケーションをこれまで通り実行（submit）できます。次の例は `spark-submit`  コマンドで SparkPi サンプル・アプリケーションを Nomad のクラスタ・モードで実行します。

```
$ spark-submit --class org.apache.spark.examples.SparkPi \
    --master nomad \
    --deploy-mode cluster \
    --conf spark.nomad.sparkDistribution=http://example.com/spark.tgz \
    http://example.com/spark-examples.jar 100
```

利用者は Nomad ジョブをカスタマイズできます。ジョブによって（上の例にあるような）設定プロパティの記述を明示して Spark を作成したり、開始時点でカスタム・テンプレートも用いられます。

```
job "template" {
  meta {
    "foo" = "bar"
  }

  group "executor-group-name" {
    task "executor-task-name" {
      meta {
        "spark.nomad.role" = "executor"
      }

      env {
        "BAZ" = "something"
      }
    }
  }
}
```

ジョブ・テンプレート（job templates）にてはメタデータや条件（constraints）の追加、環境変数の追加が可能です。ほかにも付随タスクや Consul と Vault を統合した利用もできます。

また、Nomad/Spark 統合は粒度の高い [リソース割り当て](https://www.nomadproject.io/guides/spark/resource.html) 、 [HDFS](https://www.nomadproject.io/guides/spark/hdfs.html) 、 アプリケーション出力の [継続的監視](https://www.nomadproject.io/guides/spark/monitoring.html) をサポートしています。

## はじめましょう

使いはじめるには、私たちの公式 [Apache Spark 統合ガイド](https://www.nomadproject.io/guides/spark/spark.html) が役立つでしょう。あるいは Nomad の [Terraform 設定例](https://github.com/hashicorp/nomad/tree/master/terraform) や拡張  [Spark クイックスタート](https://github.com/hashicorp/nomad/tree/master/terraform/examples/spark) によって AWS 上で統合のテストが行えます。Nomad 拡張ビルドは、現時点で Spark [2.1.0](https://s3.amazonaws.com/nomad-spark/spark-2.1.0-bin-nomad.tgz) と [2.1.1](https://s3.amazonaws.com/nomad-spark/spark-2.1.1-bin-nomad.tgz)  に対応しています。

## 原文

* Running Apache Spark on Nomad | HashiCorp
  * https://www.hashicorp.com/blog/running-apache-spark-on-nomad/

