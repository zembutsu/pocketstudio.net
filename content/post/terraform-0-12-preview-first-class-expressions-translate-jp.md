+++
date = "2018-07-08T15:00:00+09:00"
draft = false
slug = "terraform-0-12-preview-first-class-expressions-translate-jp.md"
title = "【参考訳】HashiCorp Terraform 0.12 プレビュー：式の洗練" 
categories = ["HashiCorp"]
tags = ["Terraform", "HashiCorp","translation"]
+++

HashiCorp の blog に、  [Terraform 0.12 プレビューに関する投稿](terraform-0-12-preview-first-class-expressions) が先日（2018年7月5日 原題： _HashiCorp Terraform 0.12 Preview: First-Class Expressions_ ）にありました。例によって参考訳として共有させていただきます。以下どうぞ。

## HashiCorp Terraofrm 0.12 Preview: First-Class Expressions

<!--
This is the second post of the series highlighting new features in Terraform 0.12.
-->
（Terraform 0.12 の主要な新機能に関する、連載２回めの投稿です）

<!--
As part of the lead up to the release of Terraform 0.12 later this summer, we are publishing a blog post each week highlighting a new feature. The post this week is on first-class expressions.
-->
今夏後半にある [Terraform 0.12 リリース](https://pocketstudio.net/2018/07/07/cterraform-0-1-2-preview-translate-jp.md/) の下準備となるよう、新機能に焦点を当てたブログ投稿を毎週行います。今週の投稿は優れた式（First-Class Expressions）です。

<!--
First-Class Expressions
-->
## 式の洗練（First-Class Expresisons）

<!--
Terraform uses expressions to describe the relationship between different resources and to parameterize configuration. Expressions are most often used within resource attribute values.
-->
Terraform が使用する式は、は異なるリソースとパラメータ化した設定の関係性を記述します。式が頻繁に使われるのは、リソース属性値の中です。

<!--
Before Terraform 0.12, the expression language was only available in interpolation sequences within strings, like "${var.foo}".
-->
Terraform 0.12 より前、式の言語を扱うには、文字列で `"${var.foo}"` のように補完する必要がありました。

```
# Configuration for Terraform 0.11 以前の設定ファイル

variable "ami" {
}
variable "instance_type" {
}
variable "vpc_security_group_ids" {
  type = "list"
}

resource "aws_instance" "example" {
  ami           = "${var.ami}"
  instance_type = "${var.instance_type}"

  vpc_security_group_ids = "${var.vpc_security_group_ids}"
}
```

<!--
In 0.12, the expression language is integrated into the main configuration language so expressions can be used directly in contexts where dynamic expressions are permitted:
-->
0.12 では、式の言語がメインの設定言語に統合されるため、動的な式を構文中で直接書けるようになりました。


```
# Terraform 0.12 用の設定ファイル

variable "ami" {
}
variable "instance_type" {
}
variable "vpc_security_group_ids" {
  type = "list"
}

resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.instance_type

  vpc_security_group_ids = var.vpc_security_group_ids
}
```

<!--
In versions of Terraform prior to 0.12, the HCL parser supports list and map syntax via [...] and {...} sequences, but it is not possible to use this syntax in conjunction with expressions. This is an artifact of the old HCL implementation having distinct phases for structure and interpolation. To work around this limitation, Terraform provides the list and map functions for building lists inside interpolation expressions:
-->
Terraform バージョン 0.12 に先立ち、HCL パーサ（parser）は `[...]` と `{...}` 配列による リストとマップ構文をサポートしましたが、式の中ではこの構文を利用できませんでした。古い HCL 実装このようになったのは、構造上および補完によるものが明らかでした。この制限に対応すべく、Terraform は文字列補完のリスト内で `list` と `map` 機能を使えるようにします。

```
# Terraform 0.11 以前の設定ファイル

resource "aws_instance" "example" {
  # …

  # 以下のリスト構造は静的なので動作します
  vpc_security_group_ids = ["${var.security_group_1}", "${var.security_group_2}"]

  # 以下は動作しません。 [...] 構文は補完言語で解釈されないからです
  vpc_security_group_ids = "${var.security_group_id != "" ? [var.security_group_id] : []}"

  # そのかわり、list() 関数の使用が必要です
  Instead, it's necessary to use the list() function
  vpc_security_group_ids = "${var.security_group_id != "" ? list(var.security_group_id) : list()}"
}
```

<!--
In 0.12, HCL incorporates both structure and expressions into a single language, so the last expression above can be expressed more intuitively and concisely:
-->
0.12 では、HCL 補完は構造と式が１つの言語となったため、先ほどの補完は、直感的かつ完結な表現となります。

```
# Terraform 0.12 用の設定ファイル

resource "aws_instance" "example" {
  # …

  vpc_security_group_ids = var.security_group_id != "" ? [var.security_group_id] : []
}
```

<!--
This is also true of maps, allowing the {...} syntax to be used anywhere in an expression. In 0.12, HCL also no longer accepts the following counter-intuitive configuration:
-->
また、これは maps も同様であり、式のどこでも `[...]`構文が利用できます。0.12 では、直感的ではない設定は受け入れられません。

```
# Terraform 0.11 以前の設定ファイル

output "weird" {
  value = {
    foo = "foo"
  }
  value = {
    bar = "bar"
  }
}
```
<!--
Due to its attempts to flatten nested block structures down to JSON’s information model, HCL would previously interpret the above as value = [{foo = “foo”},{bar = “bar”}]. While this is a logical consequence of how repeated blocks of the same type behave, it has often caused confusion for users. There are many other tricky behaviors of this sort, which cause the resulting data structure to not conform to the shape given in configuration.
-->
JSON の情報モデルでは、入れ子ブロックが平らな構造になる必要があったため、以前の HCL は先の記述を `value = [{foo = “foo”},{bar = “bar”}]` と解釈しました。これはいくつかのタイプのブロックが繰り返すという、論理的な結果になりましたが、利用者にとってはしばし混乱を引き起こしました。他にもこのようにトリッキーな記法が多くあり、結果として設定ファイルにおけるデータ構造は、形状を一定のものに保てませんでした。

<!--
In 0.12, HCL resolves this by making an explicit distinction between attributes and blocks. The above “weird” output now produces an error in HCL, because the attribute value may be defined only once. It is valid to have more than one instance of the same block type:
-->
0.12 では、HCL はこの解決のために、属性とブロックを明確に区別します。上記にある「weird」（妙な）記述は、HCL ではエラーとして扱われます。これは属性の `value` を一度しか定義できないからです。適切にするには同じブロック型でまとめます。

```
# Terraform 0.12 用の設定ファイル

resource "aws_security_group" "example" {
  # …

  # "ingress" には等号記号（イコール）がないため、ブロックとして解釈されます
  ingress {
    # ..
  }

  ingress {
    # ..
  }
}
```

<!--
First-class expressions will be released in Terraform 0.12, coming later this summer. To learn more about how to upgrade to Terraform 0.12, read the upgrade instructions which will be continuously updated as we get closer to releasing Terraform 0.12.
-->
式の洗練（First-class expressions）は今夏後半に提供する Terraform 0.12 でリリース予定です。Terraform 0.12 にアップグレードするには、 [アップグレード手順（英語）](https://www.terraform.io/upgrade-guides/0-12.html)をお読みください。こちらのページは、Terraform 0.12 が近づくまでに継続的に更新予定です。

## 原文

* HashiCorp Terraform 0.12 Preview: First-Class Expressions
  * https://www.hashicorp.com/blog/terraform-0-12-preview-first-class-expressions

