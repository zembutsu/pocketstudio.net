+++
date = "2018-07-08T12:00:00+09:00"
draft = false
slug = "cterraform-0-1-2-preview-translate-jp.md"
title = "【参考訳】HashiCorp Terraform 0.12 プレビュー" 
categories = ["HashiCorp"]
tags = ["Terraform", "HashiCorp","translation"]
+++

HashiCorp の blog に、  [Terraform 0.12 プレビューに関する投稿](https://www.hashicorp.com/blog/terraform-0-1-2-preview) が先日（2018年6月28日 原題： _HashiCorp Terraform 0.12 Preview_ ）にありました。例によって参考訳として共有させていただきます。以下どうぞ。

## HashiCorp Terraofrm 0.12 Preview

<!--
This is the introductory post of the series highlighting new features in Terraform 0.12.
-->
（Terraform 0.12 の主要な新機能に関する、連載投稿です）

<!--
Terraform 0.12 focuses on major Terraform language improvements and will be released later this summer. We are communicating about Terraform 0.12 prior to release to highlight the upcoming improvements and so the community can provide early feedback.
-->
Terraform 0.12 は広範囲にわたる Terraform 言語改良に焦点をあてており、今夏後半にリリース予定です。Terraform 0.12 のリリースに先立って、次回の主要な改良を伴うリリースについてお知らせしますので、コミュニティは早々にフィードバックできます。

<!--
The improvements in HCL include for loops, conditional expression improvements, nullable arguments, an exact 1:1 mapping with JSON, and much more. Unfortunately, introducing these changes resulted in multiple breaking changes. We've analyzed a large corpus of publicly available Terraform configurations and believe these breaking changes will only impact a small percentage of users. We're publishing the upgrade guide and these blog posts early as a way to communicate with the community on these upcoming changes.
-->
HCL の改良範囲には `for` ループ、条件式（conditional expression）の改善、null 許容型引数（nullable arguments）、JSON に対する正確な 1:1 対応（exact 1:1 mapping）などです。申し訳ありませんが、これら変更の結果、複数の破壊的な変更を伴います。私たちは一般公開され利用可能となっている膨大な Terraform 設定ファイルを解析しました。そして、これら破壊的な変更による影響があるのは、少ない数パーセントの利用者のみと信じています。来たるべき変更に対してコミュニティと対話できるように、更新ガイドを公開し、一連のブログを早々に投稿します。

<!--
Improving HCL
-->
## HCL 改良

<!--
HCL, the underlying configuration language for Terraform (as well as most of our tools), has gone mostly unchanged in the 4 years since it was originally released. After years of production usage in a wide range of organizational settings, Terraform users have found many areas of the language that could use improvement. Attempting to make up for limitations in HCL over the years has yielded a number of clever but unintuitive workarounds.
-->
HCL とは、Terraform （だけでなく私たちのツールの大部分）の基礎をなす設定言語です。HCL は最初のリリースから4年間、ほとんどの変更がありませんでした。幅広い組織の設定用途として、本番環境（プロダクション）で使い始めた後、Terraform 利用者からすると多くの改良すべき点が目に付いてきました。HCL による制約の中、何年も使い続けるために多くの工夫をこなしますが、これは直感的ではない次善の策でした。

<!--
About a year ago, we began an effort of developing a major iteration to HCL to address this feedback holistically by building a more robust, feature-rich language to simplify the process of writing, analyzing, and processing Terraform configurations. Earlier this year, we began integrating this new iteration of HCL into what will become Terraform 0.12.
-->
私たちはこれら HCL へのフィードバックに対して総体的に取り組むべく、１年前から広範囲にわたる開発項目で繰り返し取り組んできました。より堅牢に構築し、機能豊富な言語で Terraform の設定を記述し、解析し、処理する流れを簡単にするためです。私たちは今年の早々から、HCL に対する新しい機能を Terraform 0.12 で統合するように進めてきました。

<!--
We're introducing over a dozen new improvements in Terraform 0.12 that focus on improving the core language. In addition to the immediate benefits that make it into the Terraform 0.12 changelog, the new HCL engine gives us much more flexibility to introduce new features in the future.
-->
Terraform 0.12 では言語の中核に対する改良に焦点をあて、非常に多くの新機能を導入します。Terraform 0.12 の changelog（訳者注：変更履歴を記録したファイル）に書かれるに至る直接的な利点に加えて、新しい HCL エンジンは今後の新機能をより柔軟に取り入れられるようになります。

<!--
Notable Improvements
-->
## 著しい改良

<!--
The list below shows the notable improvements in HCL and Terraform that will be present in Terraform 0.12. This list contains a basic description of each and follow-on blog posts will cover each in detail.
-->
以下は Terraform 0.12 から登場する HCL と Terraform に対する著しい改良の一覧です。一覧にある各機能の詳細については、別途それぞれを解説するブログ投稿を後日します。

<!--
    First-class expressions. Prior to 0.12, expressions had to be wrapped in interpolation sequences with double quotes, such as "${var.foo}". With 0.12, expressions are a native part of the language and can be used directly. Example: ami = var.ami[1]
        For expressions. A for expression is available for iterating and filtering lists and map values. This expression always can be used anywhere a list or map is expected.
-->
* **優れた条件式（First-class expressions）** - 0.12 までの条件式では、 `"${var.foo}"` のようにダブル・クォーテーション・マーク（二重引用符）で囲む必要がありました。0.12 では、条件式は言語に取り込まれた状態（native part）であり、 `ami = var.ami[1]` のように直接利用できるようになります。
  * **for 文（for epressions）** - 繰り返し、リストのフィルタ、値の割り当て（map）のために `for` 文が使えます。リストやマップが予想される場所では、どこでもこの表記を利用できます。

<!--
    Dynamic blocks. Child blocks such as rule in aws_security_group can now be dynamically generated based on lists/maps and support iteration.
-->
* **動的ブロック（Dynamic blocks）** - `aws_security_group` にある `rule` のような子ブロックは、リストやマップとサポートしている反復型に基づき、動的に生成できます。

<!--
    Generalized "Splat" expressions. The special resource.*.field syntax used to only work for resources with count set. This is now a generalized operator that works for any list value.
-->
* **汎用「範囲指定」表現（Generalized "Splat" expressions）** - 特別な `resource.*.field` 記法が使えたのは `count` セットのリソースのみでした。今後は汎用的な記法として、あらゆるリストの値で利用できます。

<!--
    Conditional improvements. The conditional operator ... ? ... : ... now supports any value type and lazily evaluates results, as those familiar with this operator in other languages would expect.
-->
* **条件文の改良（Contitional improvements）** - 条件式 `... ? ... : ...` をあらゆる値の型でサポートし、結果を適切に評価ます。他の言語で見うけられるのと同じように機能します。

<!--
    Nullable arguments. The special value null can now be assigned to any field to represent the absence of a value. This causes Terraform to omit the field from upstream API calls, which is important in some cases for triggering certain default behaviors.
-->
* **null 許容型引数（Nullable arguments）** - 値がない場所を示すために、特別な値 `null` をあらゆるフィールドで割り当て可能です。これにより、Terraform は一次側（upstream）API コールからのフィールドを省略するため、処理によっては明確なデフォルト挙動の指定が重要になります。

<!--
    Rich types in module inputs and outputs. Terraform has supported basic lists and maps as inputs/outputs since Terraform 0.7, but elements were limited to only simple values. Terraform 0.12 allows arbitrarily complex lists and maps for any inputs and outputs, including with modules.
-->
* **inputs と outputs モジュールで豊富な型を扱える（Rich types in module inputs and outputs）** - Terraform は input/outputs における基本的なリストとマップを Terraform 0.7 で導入しましたが、要素として使えるのは１つの値のみでした。Terraform 0.12 からは、あらゆる inputs とoutputs で（modules も含む）任意に複雑なリストとマップが利用できます。

<!--
    Template syntax. Within string values, a new template syntax can be used for looping without complex nested interpolation. Example: %{ for instance in aws_instance.example ~}server ${instance.id}%{ endfor }.
-->
* **テンプレート構文（Template syntax）** - 文字列の値では、ループ用途として新しいテンプレート構文を利用できます。複雑な入れ子記法を使う必要はありあせん。例： `%{ for instance in aws_instance.example ~}server ${instance.id}%{ endfor }`

<!--
    Reliable JSON syntax. Terraform 0.12 HCL configuration has an exact 1:1 mapping to and from JSON.
-->
* **正確な JSON 構文（Reliable JSON syntax）** - Terraform 0.12 の HCL 設定ファイルは、JSON に対して正確に 1:1 となる対応をします。

<!--
    References as first-class values. References to resources and modules for fields such as depends_on used to be arbitrary strings. In Terraform 0.12, the resource identifier can be used exactly such as aws_kms_grant.example (no quotes!). This improves the validation and error messages we can provide. Similarly, a resource reference can be returned from a module as an output or accepted as a parameter.
-->
* **特別な値としての参照（References as first-class values）** - リソースとモジュールに対する `depends_on` のような参照（リファレンス）で、任意の文字列を使えます。Terraform 0.12 では、リソース識別子（resoruce identifier）を `aws_kms_grant.example` のように（引用符を使わずに！）そのまま記述できます。これにより、私たちが提供する確認とエラーメッセージを改善できるようになります。同様に、リソースに対するリファレンスは、モジュールからの出力またはパラメータとして戻れるようになります。

<!--
Over the coming weeks we will publish blog posts going into more detail on each of the above improvements, plus a few more. Each post will include example snippets so you can more easily understand what has changed as well as any important information for upgrading to Terraform 0.12 related to that specific change.
-->
来週以降、私たちはブログの投稿を通し、先ほど記述した各改良に対するより詳しい情報や付加情報を公開してきます。各投稿には記述例も含みますので、何が変わるのかだけでなく、Terraform 0.12 のアップグレードに関連する重要な情報を、より簡単に理解できるでしょう。

<!--
Breaking Changes
-->
## 破壊的変更

<!--
Introducing these changes has resulted in a number of breaking changes. Most users will be able to upgrade to Terraform 0.12 with no configuration changes. However, a small group of users will need to modify their configurations in some way.
-->
紹介したこれらの変更に伴い、数々の破壊的な変更となります。ほとんどの利用者は Terraform 0.12 へのアップグレードにあたり、設定ファイルを変更する必要はありません。ですが、小数グループの利用者は、いずれ設定変更が必要になります。

<!--
We've published the work-in-progress Terraform 0.12 upgrade guide so that users can begin preparing for Terraform 0.12. Please note that this upgrade guide may continue to change prior to the release of Terraform 0.12. In particular, we're continuing to look for ways to automate or completely obviate some of the breaking changes.
-->
私たちは[Terraform 0.12 アップグレード・ガイド](https://www.terraform.io/upgrade-guides/0-12.html)の公開に向けて作業を進めていますので、利用者の皆さんも Terraform 0.12 の準備を開始できます。アップグレード・ガイドは Terraform 0.12 をリリースするまで継続して変更する可能性があるため、ご注意ください。特に、私たちは自動的、もしくは破壊的な変更を完全に防ぐために取り組み続けます。

<!--
In addition to the upgrade guide, we are working on automated tooling for some of the breaking changes so that the work isn't completely manual. As configurations are being written, we always recommend constraining the Terraform version for safe execution.
-->
アップグレード・ガイドに加え、更新作業のすべてが手作業とならないよう、私たちは複数の破壊的な変更に対する自動化ツールに対応中です。設定ファイルの記述にあたっては、安全に実行できるよう、私たちは常に [Terraform バージョンに対する制約](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version) を推奨します。

<!--
Next
-->
### 次へ

<!--
We're very excited about the improvements coming in Terraform 0.12. Many of these features have been requested for years, but they have required resetting the foundation of Terraform's configuration handling. With this new foundation in place, Terraform is poised to quickly improve and support new features over upcoming releases.
-->
私たちは次期 Terraform 0.12 の開票にとてもワクワクしています。機能の多くは何年にもわたりご要望いただいていたものですが、Terraform の設定を処理する基礎部分をリセットする必要がありました。新しい基礎ができれば、Terraform は以降のリリースで素早い改良や新機能のサポートをできる準備が整います。

<!--
We apologize in advance for the breaking changes that we've introduced. We've already minimized the number of breaking changes over the past 6 months, and we will continue to evaluate if we can lower the number. Further, we've focused on the configuration changes in this release with the intention that future releases will not require any breaking changes.
-->
これまで紹介した破壊的な変更に先立ち、私たちは謝罪を申し上げます。私たちは過去６ヶ月にわたり破壊的な変更が最小限になるようにしてきました。また、可能であれば影響が最小限になるよう取り組み続けます。さらに、将来的なリリースでは破壊的な変更が必要としないように、私たちは今回のリリースでは設定（configuration）の変更にも注力しています。

<!--
If you have any feedback or concerns about the changes proposed, please get in touch with us via the public mailing list. We're happy to discuss any of these changes!
-->
フィードバックや変更に対する提案を検討中であれば、どうか公開メーリングリストにご参加ください。あらゆる変更に対する議論ができれば嬉しく思います！

<!--
Over the coming weeks we'll publish feature preview blog posts diving into the individual improvements of Terraform 0.12!
-->
来週以降の機能プレビューに関するブログ投稿では、Terraform 0.12 の個々の改良に対する詳細な情報をご案内します！

## 原文

* HashiCorp Terraform 0.12 Preview
  * https://www.hashicorp.com/blog/terraform-0-1-2-preview

