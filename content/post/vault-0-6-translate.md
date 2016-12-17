+++
date = "2016-07-09T07:30:00+09:00"
draft = false
slug = "vault-0-6-translate"
title = "【参考訳】Vault 0.6"
categories = ["Vault"]
tags = ["HashiCorp", "Vault","translation"]
+++

## 概要

6月14日に、[HashiCorp](https://www.hashicorp.com) の blog に[Vault 0.6 のリリースに関する解説記事](https://www.hashicorp.com/blog/vault-0.6.html)がありました。こちらも例によって訳しましたので、参考程度にどうぞ。

※ 今回は Vault 独特の用語が多く、翻訳内容には通常よりも多くの意訳箇所があります。そのため、普段より "日本語（英語）" のように、原文併記している箇所が多く読みづらい所がありますが、ご容赦願います。 この意図は、HasihCorp の Vault について、話題に上げたり議論できる人を増やしたいなぁという思いからです。そのため、内容について気になる箇所がありましたら、随時ご指摘いただけますと幸いです。

## Vault 0.6

私たちは [Vault](https://www.vaultproject.io/) [0.6](https://www.vaultproject.io/downloads.html) のリリースを誇りに思います。Vault はシークレット（secret）管理用のツールです。API 鍵や暗号化された細心の注意を払うべきデータを、完全な内部認証局（CA）を持つ Vault に置けます。つまり、シークレット管理に必要な全ての解決策（solution）を Vault が提供します。

今回のリリースでは、重要な新機能をいくつか追加しました。ほかにも、新しいセキュア・ワークフローの拡張や、多くの改良、バグ修正を行いました。特に集中したのはトークン管理（token management)とトークン/認証ワークフローです。

詳細：

* [Token Accessors（トークン・アクセス機構）](#token-accessors)
* [Token Authentication Backend Roles（トークン認証バックエンド・ロール）](#token-authentication-backend-roles)
* [Response Wrapping（レスポンス・ラッピング）](#response-wrapping)
* [AWS EC2 Authentication Backend（AWS EC2 認証バックエンド）](#aws-ec2-authentication-backend)

[概要](#他の機能)：

* Codebase Audit（コードベースの監査）
* Integrated Consul Health Checks（Consulヘルスチェック統合）
* Lisner Certificate Reloading（リスナー証明書の再読み込み）
* MSSQL Credential Generation（MSSQL 証明書生成）
* Azure Data Store（Azureデータ・ストア）
* Swift Data Store（Swiftデータ・ストア）

（いくつかの機能は 0.5.1 と 0.5.2 で搭載したものですが、これまでブログに解説を投稿していませんでした。）

変更に関する全ての詳細は [Vault 0.6 CHANGELOG](https://github.com/hashicorp/vault/blob/v0.6.0/CHANGELOG.md) をご覧ください。さらに、この投稿の最後に書いた更新に関する情報は必ずお読みください。

いつもの通り、アイディア、バグ報告、プルリクエストをいただいた私たちのコミュニティに対して、大変感謝を申しあげます。

Vault 0.6 の主な新機能について学ぶには、このまま読み進めてください。

## Token Accessors

トークンの作成時、２つの値を返します。トークン自身と認証機構（accessor；アクセッサ）です。

```
$ vault token-create -policy=default
Key             Value
---             -----
token           82c5fb97-da1b-1d2c-cfd5-23fa1dca7c85
token_accessor  dd256e17-b9d9-172d-981b-a70422e12cb8
token_duration  2592000
token_renewable true
token_policies  [default]
```

トークンに対する知識がなくても、トークンの表示（lookup）や無効化（revocation）の処理を認証機構で行えます。使うには ``auth/token/lookup-accessor`` /`` auth/token/revoke-accessor`` を通すか、各 CLI コマンドで ``-accessor`` フラグを指定します。認証機構で表示機能を使っても、トークン ID を返しません。

認証機構を持つクライアントを識別するための情報を保管するため、他のクライアントが作成したサービス用トークンに関する情報の表示や、必要があれば後でトークンを無効化できます。たとえば、トークンを作成した従業員が会社を辞めたとします。認証機構を使えば、保管されているトークン ID がなくてもトークンを無効化できます。

これを簡単に行えるだけでなく、トークン・ストアを直接使わずに作成したトークンのワークフローにも便利です。また、認証機構（accessor）値の HMAC （ハッシュ・ベースのメッセージ認証符号）化をオプション設定を無効にもできます。（デフォルトの HMAC 化は、認証機構の値にアクセス可能な誰もが、トークンでの認証を無効化するサービス拒否攻撃を行えてしまいます。そして、たいていのログは厳密に管理されていません。） 監査バックエンド（audit backend）を有効にしている場合は、設定オプションで ``hmac_accessor=false`` を指定すると、この挙動が有効になります。

## Token Authentication Backend Roles

[トークン認証バックエンド（Token Authentication Backend）](https://www.vaultproject.io/docs/auth/token.html) はロール（役割）の概念を取り入れました。ロールが提供するのは管理度（degree of administrative）の柔軟さであり、許可されたユーザがトークンの作成時、ユニークな属性（properties）や他の behalf を含められます。

トークン・アクセス機構と一緒にこれらのロールを有効にすると、直接トークンを作成するよりも優れたワークフローになります。管理者は上限（limit）や機能（capabilities）を指定するためにロールを設定できます。そして、信頼するユーザやサービスに対してはトークンを作成できるロールを割り当て可能です。サービスごとにトークンを指定できるので、認証機構の値（accessor）は監査ログ（audit logs）に文字列（plaintext）で書き込めます。たとえば、無効化（revocation）が必要であれば、認証機構を通して特定のトークンの無効化や、特定のロールが発行したトークンを全て無効化できます。

もう１つ書き加えておきます。 ``auth/token/create``  でトークンを直接作成する時、再利用性（renewability）と最大 TTL の明示も制御できます。

## Response Wrapping

トークン認証バックエンド・ロールとトークン認証機構は、きめ細かな権限の管理を可能にします。しかし、生成したトークンを安全に配布する問題は解決していません。安全な導入が難しい課題です。その理由の１つは、高度に安全な通信経路の確保が大変だからです。

Vault 0.6 でレスポンス・ラッピング（response wrapping）を導入しました。これはセキュリティ・プリミティブ（訳者注：セキュリティ上の構造、の意味）です。Vault が生成したトークンを配布するだけでなく、簡単かつ安全に行えるようにします。

詳細な仕組みを説明する前に、読者の皆さんには [Cubbyhole 認証パラダイム](https://www.hashicorp.com/blog/vault-cubbyhole-principles.html) に関する投稿をあらかじめお読みになったほうが良いかもしれません。以降の段落に関する概要を説明していますが、全くのオプションです。ですが、レスポンス・ラッピングはパラダイムの一部を完全に実装したものであり、より優れたセキュリティを提供します。そのためには、スタンドアロンのプロセスが ``temp`` と ``perm`` トークンを管理します。この実装の詳細については先の投稿の通りであり、 ``/sys`` のパスを除き、レスポンス・ラッピングは Vault に到達するあらゆる応答に使えます。

レスポンス・ラッピングは以下の通りに動作します。ここで想定するシナリオは、クライアントが Vault からシークレットを取得するにあたり、利用可能なシークレットしか表示できないように制限します。

1. クライアントのリクエストに、応答をラッピングするためのラッピング・トークン（wrapping token）の適用期間を指定します。CLI 上で使うには、グローバル・クライアント・フラグの ``-wrap-ttl``  を指定します。

2. Vault はリクエストを通常通り処理します。ですが、通常通りに応答するのではなく、JSON シリアル化（JSON-serialize）した応答をします。この応答には、新しいトークンの cubbyhole を含みます。新しいトークンを使うのは、クライアントで指定した TTL の間のみであり、延長できません。

3. Vault は新しいレスポンスを生成して応答します。ここでラッピングする情報には、ラッピング・トークン ID とラッピング・トークンの有効期間です。

4. クライアントは対象のターゲットに対し、ラッピング・トークンを渡します。

5. 対象のターゲットは、トークンを使い ``unwrap`` （ラッピングされていない）処理をします。水面下で行っているのは、をトークンの cubbyhole が既知の場所から、オリジナルの応答単に受け取るだけです。そして JSON でパースし、通常通り応答します。API や CLI を透過するだけであり、Vault から直接応答を得るのと、ラッピング・トークンから読み込みパースするのと、両者に差違はありません。

6. 対象のターゲットが権限がないと拒否された場合、トークンの TTL と作成時間を比較し、有効期間を超えていないか確認します（対象のターゲットから取得するには、おそらく時間がかかるでしょう）。もしも有効期限が切れていなければ、ターゲットはセキュリティ警告をあげます。警告の内容は、応答がラッピングされていない可能性と、シークレットが意図しない第三者に漏洩している可能性です。

この流れを説明するため、以下では CLI コマンド ``token-create``  を２回実行します。１つは正常ですが、もう片方はラッピングされていない処理です。

```
#
# 通常の token-create コマンド
#

$ vault token-create -policy=default
Key             Value
---             -----
token           ff999148-077e-2629-8d9a-cb0a7b97e811
token_accessor  8c243823-5820-ff8f-f641-6999290c60c0
token_duration  2592000
token_renewable true
token_policies  [default]

#
# レスポンス・ラッピングに対応した token-create コマンド
#

$ vault token-create -policy=default -wrap-ttl=60s
Key                             Value
---                             -----
wrapping_token:                 7a39125a-2001-be9b-e363-3ba85a16c311
wrapping_token_ttl:             60
wrapping_token_creation_time:   2016-06-14 06:02:11.171558903 +0000 UTC
wrapped_accessor:               ed96a7ae-dfdb-3c09-0a60-a02b53dfe6b2

#
# ラッピングしていない応答
#

$ vault unwrap 7a39125a-2001-be9b-e363-3ba85a16c311
Key             Value
---             -----
token           d878b2ed-564e-0790-4154-67bcf7386268
token_accessor  ed96a7ae-dfdb-3c09-0a60-a02b53dfe6b2
token_duration  2592000
token_renewable true
token_policies  [default]

#
# 無効なトークン（使用済み）の検出
#

$ vault unwrap 7a39125a-2001-be9b-e363-3ba85a16c311
error reading cubbyhole/response: Error making API request.

URL: GET https://127.0.0.1:8200/v1/cubbyhole/response
Code: 400. Errors:

* permission denied
```

この例でご覧の通り、ラッピングした応答にトークンを含む場合、トークン認証機構はラッピング情報を提供します。これにより、特権を持つ実行者（privileged caller；訳者注、管理者の意味）が生成したトークンにより、そのクライアントが対象とした認証機構の追跡を簡単にします。これは万が一の時にトークンを無効化するためです。

レスポンス・ラッピングは Vault リクエストのほとんどで利用可能です。そのため、あらゆるシークレットに対し、転送時のセキュリティ拡張をもたらします。

```
#
# K/V (generic) バックエンド
#

$ vault write secret/foo bar=baz
Success! Data written to: secret/foo

$ vault read -wrap-ttl=60s secret/foo
Key                             Value
---                             -----
wrapping_token:                 3a63ff9f-1c38-af43-0a85-dc1155f2469d
wrapping_token_ttl:             60
wrapping_token_creation_time:   2016-06-10 13:46:14.155857566 -0400 EDT

$ vault unwrap 3a63ff9f-1c38-af43-0a85-dc1155f2469d
Key                     Value
---                     -----
refresh_interval        2592000
bar                     baz

#
# Transit バックエンド
#

$ echo "bar" | base64 | vault write transit/encrypt/foo plaintext=-
Key             Value
---             -----
ciphertext      vault:v1:j4Z1/6WLK+snlhIyooxa1zW0yEmBqFSfZTL88bN/Qt4=


$ vault write -wrap-ttl=60s transit/decrypt/foo ciphertext="vault:v1:j4Z1/6WLK+snlhIyooxa1zW0yEmBqFSfZTL88bN/Qt4="
Key                             Value
---                             -----
wrapping_token:                 e93a78ec-cced-dc94-d1f8-d71e84faddde
wrapping_token_ttl:             60
wrapping_token_creation_time:   2016-06-07 15:59:18.251657838 -0400 EDT

$ vault unwrap -field=plaintext e93a78ec-cced-dc94-d1f8-d71e84faddde | base64 -d
bar
```

## AWS EC2 Authentication Backend

[AWS EC2 認証バックエンド](https://www.vaultproject.io/docs/auth/aws-ec2.html) は、 EC2 インスタンスの自動的な Vault 認証と Vault トークンの取得を可能にします。対応クライアントでの AMI 操作は、ユーザ側の調整や設定変更を行わずに認証できます。

バックエンドは AWS を信頼できるサード・パーティとして扱います。特に AWS のインスタンスが提供するのは、秘密鍵で署名されたメタデータと、確認可能な公開済みの公開鍵です。ハイレベルでは、バックエンドは与えられたメタデータの整合性を確認します。具体的には、インスタンス ID が過去に使われていないか（TOFU あるいは Trust On First Use ＝間違いなく初めて使う、の原則として知られています。）と、そのインスタンスが実行中の状態かです。また、バックエンドはクライアントが指定した一時的な値も受け付けます。たとえば、初期のトークンの有効期限が切れたら、新しいトークンの取得に使います。そして、他のクライアントがメタデータを再利用しようと試みても、それを拒否します。

ローレベルでは、セキュリティ・ポリシーと組織における広範囲のワークフローに一致するよう、様々な制限（modification）をサポートしています。詳細についてはドキュメントをご覧ください。

インスタンスを認証するには、インスタンス内で動作するクライアントが Vault と通信し、Vault の API を通して必要な情報を渡す必要があります。Vault Enterprise をご利用のお客さまには、デプロイのワークフローを簡単にするため、HashiCorp が構築したスタンドアロンのバイナリを提供しています。クライアントは与えられたトークンを更新し続けます。トークンの有効期限が切れれば、再認証して新しいトークンを取得します。

Vault Enterprise クライアント・バイナリは単純にバックエンド API を使うだけです。しかし、Vault 内部の挙動はオープンソースで公開されているものとは異なります。Vault Enterprise をお使いではない組織でも、必要があれば自分でクライアントを構築できます。あるいは [HashiCorp に Vault Enterprise に関する情報をお訊ねください](https://www.hashicorp.com/vault.html) 。

## 他の機能

今回のリリースでは、多くの新機能追加と改良を施しました。その中でも、いくつかの重要な項目を紹介します：

* **Codebase Audit（コードベースの監査）** … Vault 0.5 のソースコード群は iSEC パートナによって監査済みです（私たちには監査内容の一般公開が許されていません）。
* **Integrated Consul Health Checks（Consulヘルスチェック統合）** … Consul データ・ストア自動的に ``vaule`` サービスを登録し、Consul でヘルスチェックを実施します。デフォルトでは、アクティブなノードは ``active.vault.service.consul`` から確認できます。また、スタンバイ・ノードは ``standby.vaule.service.consul`` から確認できます。デフォルトの Consul のサービス・ディスカバリでは、封印済みの（sealed） vault を critical（クリティカル）と認識するため、一覧に表示しません。詳細は [ドキュメント](https://www.vaultproject.io/docs/config/index.html#path)をご覧ください。
* **Lisner Certificate Reloading（リスナー証明書の再読み込み）** … Vault の設定リスナーは、Vault が SIGHUP を送ると TLS 証明書を秘密鍵を再読み込みします。これにより、Vault 証明書の再読み込みにあたり、Vault の停止や封印解除（unseal）の作業が不要になります。
* **MSSQL Credential Generation（MSSQL 証明書生成）** … Vault が既にサポートしてる Postgres や MySQL と同様、証明書を生成するバックエンドに MSSQL が使えます。
* **Azure Data Store（Azureデータ・ストア）** … Vault が既にサポートしている S3 と同様、Vault の物理データ・ストアとして Azure blob オブジェクト・ストレージが使えます。
* **Swift Data Store（Swiftデータ・ストア）** … Vault が既にサポートしている S3 と同様、Vault の物理データ・ストアとして Swift オブジェクト・ストレージが使えます。

## 更新の詳細

Vault 0.6 は大規模な変更を伴うため、更新する前に様々な理解が必要です。[一般的な更新手順](https://www.vaultproject.io/docs/install/upgrade.html)だけでなく、[バージョン固有の手順](https://www.vaultproject.io/docs/install/upgrade-to-0.6.html)もご覧ください。ご注意いただきたいのは、扱っているのは 0.5.x からのバージョンアップです。さらに前のバージョンから更新する場合は、それぞれのリリースに関するブログの投稿をご覧ください。

私たちが推奨するのはいつもの通り、独立した環境でテストした後、更新を試みてください。もし何らかの問題があれば、[GitHub の Vault の issue トラッカー](https://github.com/hashicorp/vault/issues) や [Vault メーリングリスト](https://groups.google.com/group/vault-tool) にお訊ねください。

それでは、Vault 0.6 をお楽しみください！

## 原文

* Vault 0.6 - HashiCorp
  * https://www.hashicorp.com/blog/vault-0.6.html

