+++
date = "2016-06-23T23:15:00+09:00"
draft = false
slug = "docker-1-12-swarm-mode-and-ingress-load-balancing"
title = "Docker 1.12: swarm モードと Ingress Load Balancing 概要"
categories = ["Docker"]
tags = ["Docker", "Swarm"]
+++

## 概要

　Docker Engine 1.12 RC1 から、Docker Engine に swarm モードが搭載されています。また、機能として Ingress オーバレイ・ネットワークが標準で利用でき、負荷分散機能も使えます。このトピックでは新しい用語や概念の整理をし、実際のクラスタを作成と、nginx サービスとしてのコンテナを実行するまでの手順を整理します。

## 「swarm」 は「クラスタ」の意味

　Docker Engine に swarm モードは、従来の Docker Engine のクラスタを管理する Docker Swarm が Docker Engine に機能統合されたものです。これまで Docker Engine のクラスタを作るためには、 [Swarm コンテナ](https://hub.docker.com/_/swarm/) か `swarm` バイナリをセットアップする必要がありました。ですが、今後は **SwarmKit** という内蔵機能に統合されているため、追加セットアップなしに `docker` コマンドでクラスタを管理できます。

　そして、Docker Engine でこのバージョンから Docker Engine のクラスタを **swarm** と呼びます。swarm は「群れ」の意味があります。そして swarm クラスタを扱うには、Docker Engine の **swarm モード（swarm mode）** を利用します。

## Docker 1.12 RC をセットアップ

　現時点で swarm モードを使えるのは RC （リリース候補）版のみです。そのため、通常のセットアップ方法とは異なります。バイナリは [GitHub の releases](https://github.com/docker/docker/releases) から入手できます。あるいは、次のコマンドを実行し、自動的に RC 版をセットアップすることもできます（ Ubuntu 14.04, 16.04, CentOS 7.2 で動作確認）。

```
$ curl -fsSL https://test.docker.com/ | sh
```

　CentOS 7.2 では Docker Engine が自動起動しないため、 `systemctl` で起動する必要があります。

```
# systemctl start docker
```

　起動後の動作確認は、 ``docker version`` コマンドを実行します。正常に動作していれば、クライアントとサーバの両方のバージョンを表示できます。

```
# docker version
Client:
 Version:      1.12.0-rc2
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   906eacd-unsupported
 Built:        Fri Jun 17 21:21:56 2016
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.0-rc2
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   906eacd-unsupported
 Built:        Fri Jun 17 21:21:56 2016
 OS/Arch:      linux/amd64
```

## ノード

　ここでは３台の Docker Engine でクラスタを作成します。

* ``node-01`` 192.168.39.1 … マネージャ・ノード兼ワーカ・ノード
* ``node-02`` 192.168.39.2 … ワーカ・ノード
* ``node-03`` 192.168.39.3 … ワーカ・ノード

　３台のサーバで Docker Engine （ ``dockerd`` デーモン ）を動かします。このクラスタ内の Docker Engine を、それぞれ swarm の **ノード（node）** と呼びます。そして、このノードには２つの役割があります。

　 １つは **マネージャ・ノード（manager node）** であり、ここで **サービス（service）** の期待状態（desired state）を定義します。サービスとは「 ``nginx`` コンテナを３つ動作する」といった状態を指します。また、swarm クラスタ全体を管理する ``docker node <命令>`` コマンドを処理できるのもマネージャ・ノードです。

　もう１つは **ワーカ・ノード（worker node）** です。マネージャ・ノードで定義されたサービスがこのワーカ・ノードに送られ、コンテナを実行・停止などの処理を行います。また、ワーカ・ノードは自分の状態をマネージャ・ノードに伝える役割もあります。

　なお、デフォルトではマネージャ・ノードはワーカーノードの役割も兼ねます。オプションの設定で、マネージャのみの役割で動作させることもできます。

## サービスとタスクについて

　swarm クラスタ上で何らかのコンテナを実行するには、サービスを定義します。このサービス定義に従って実際にコンテナを起動するのが **タスク（task）** です。タスクはコンテナの最小スケジューリング単位です。スケジューリングとは、どのノードでどのコンテナを実行するのかを検討し、実際に起動する処理を指します。

　そして、サービスには２種類あります。

* 複製サービス（replicated service） … マネージャが各ノードに対し、指定した数の複製タスクを作成します。
* グローバル・サービス（global service） … あるタスクをクラスタ内の全ノード上で実行します。

　複製サービスが適しているのは、ウェブなどのスケールさせる必要があるタスクです。グローバル・サービスは全ノード上で実行しますので、監視やログ収集などのタスク実行に適していると考えられます。

## swarm の初期化

[<img src="/images/swarm-mode-1.png" width="100%">](/images/swarm-mode-1.png)

　ここからは、実際にクラスタを構築します。swarm モードを使うためには、まず、swarm クラスタの初期化をします。初期化のためには ``docker swarm init`` コマンドを使います（init = initialize = 初期化）。 ``--lissten-addr <ホスト>:<ポート>`` で、swam マネージャが利用するインターフェースとポート番号を指定します。

```
$ docker swarm init --listen-addr 192.168.39.1:2377
```

　swarm クラスタ上のノード一覧を確認するには ``docker node ls`` コマンドを使います。

```
$ docker node ls
ID                           NAME     MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
dhrbvl6o9xqvprsq2uzq8o9ev *  node-01  Accepted    Ready   Active        Leader
```

　``docker swarm init`` コマンドを実行した ``node-01`` ノードの情報が表示されます。 ``MANAGER STATUS`` の部分に何らかの表示があれば、マネージャとして動作しています。ここでは初めて実行したマネージャのため、 ``Leader`` （リーダ）の役割です。

## swarm クラスタに他のノードを追加

[<img src="/images/swarm-mode-2.png" width="100%">](/images/swarm-mode-2.png)

　次は残りの２台をクラスタに追加します。クラスタに追加するには、 ``node-02`` ``node-03`` の各ノード上で ``docker swarm join <マネージャ>`` コマンドを実行します。ここでは、次のコマンドを実行します。

```
$ docker swarm join 192.168.39.1:2377
```

　それから、 ``node-01`` 上で ``docker node ls`` コマンドを実行したら、各ノードが swarm に追加されているのが確認できます。

```
docker@node-01:~$ docker node ls
ID                           NAME     MEMBERSHIP  STATUS  AVAILABILITY  MANAGER STATUS
6puvbl7tgxislfha5iaha40jm    node-03  Accepted    Ready   Active
9zj16or3durdh5i7c6khlench    node-02  Accepted    Ready   Active
dhrbvl6o9xqvprsq2uzq8o9ev *  node-01  Accepted    Ready   Active        Leader
```

　この情報はとは別に、クラスタの情報は ``docker info``  でも確認できます。

```
$ docker info
(省略)
Swarm: active
 NodeID: dhrbvl6o9xqvprsq2uzq8o9ev
 IsManager: Yes
 Managers: 1
 Nodes: 3
```

　マネージャは１つ、ノードが３つあることが分かります。

　これでサービスを実行する準備が整いました。

## nginx サービスの定義と実行

[<img src="/images/swarm-mode-3.png" width="100%">](/images/swarm-mode-3.png)

　次は ``nginx`` コンテナを実行するサービスを定義します。サービスの定義は ``docker service create`` コマンドを使います。ここでは ``web`` という名前のサービスで（``--name web`` ）、コンテナを１つ実行（ ``--replicas 1`` ）します。また、サービス側のポート 80 をコンテナ内のポート 80 に割り当て（ ``-p 80:80`` ）します。

```
$ docker service create --replicas 1 –name web -p 80:80 nginx
f218o6xshkyt7zzxujhnz1a2h
```

　実行後の文字列 ``f218o6xshkyt7zzxujhnz1a2h`` はサービス ID と呼ばれるものです。サービス名かサービス ID を使い、サービスの停止・更新・削除などの操作時に使います。

　そして、サービスの状況を確認するには ``docker service ls`` コマンドを使います。

```
docker@node-01:~$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
13765buws9fr  web   0/1       nginx
```

　ここでは ``REPLICAS`` （レプリカ）の表示が ``0/1`` になっているのに注目します。先ほど指定した ``--replicas 1`` とは、タスク（コンテナ）の期待数が ``1`` でした。しかし、実際には ``0`` 、つまり、まだタスク（としてのコンテナ）が起動しちていない状態です。

　この時、スケジューリング処理が行われています。３つのワーカのうち、どこでタスク（としてのコンテナ）を実行するか決め、その後、 ``nginx`` イメージのダウンロードと Docker コンテナを起動します。この状態を確認するには ``docker service tasks <サービス名>`` コマンドを実行します。

```
$ docker service tasks web
ID                         NAME   SERVICE  IMAGE  LAST STATE             DESIRED STATE  NODE
emjve46lgwjx22o6pxpbetfed  web.1  web      nginx  Running About an hour  Running        node-02
```

　ここでは ``node-02`` で ``nginx`` イメージを使ったコマンドが実行中だと分かります。状況はリアルタイムに変わりますので、 ``watch``  コマンドを組みあわせると、状況の監視に便利です（例： ``watch -n 1 'docker service tasks web'`` ）。

　タスク起動後にサービスの一覧を確認したら、今度は ``1/1`` に切り替わります。

```
docker@node-01:~$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
13765buws9fr  web   1/1       nginx
```

## ingress オーバレイ・ネットワークと負荷分散

　インターネット側など、外部のネットワークからコンテナにアクセスするには、ポート 80 に接続します。接続は、 ``node-02`` の IP アドレス ``192.168.39.2 `` でアクセス可能なだけではありません。swarm モードで動作しているノードであれば、どこからでもアクセス可能です。

　これは、各ノード上では ``ingress`` という名前のオーバレイ・ネットワーク（overlay network）が作成されているからです。 ``docker network ls`` コマンドを実行し、ノード上の Docker ネットワーク一覧を表示します。

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
bbb5e37b01e8        bridge              bridge              local
6550c72b075f        docker_gwbridge     bridge              local
84baeb90cbb2        host                host                local
9i5hmgqr20jh        ingress             overlay             swarm
7ab4c321bbe5        none                null                local
```

　注目すべきは ``ingress`` という名前のネットワークです。これは swarm モードで自動作成されるオーバレイ・ネットワークです。これは複数のノード間がつながっている共有ネットワークです。そして ``SCOPE`` には ``local`` と ``swam`` があります。 ``local`` はノードの中でのみ有効なネットワーク、 ``swarm``  は swarm モードのクラスタで有効なネットワークです。

　そして、 ``ingress`` ネットワークの特長として、このネットワーク上で公開しているポートは、どのノードへアクセスがあっても、タスクを実行しているノードに自動ルーティングしてくれます。

[<img src="/images/swarm-mode-4.png" width="100%">](/images/swarm-mode-4.png)

　さらに Ingress ネットワークには負荷分散機能も入っています。この負荷分散はタスクに応じて自動的に処理されます。つまり、swarm モードではサービスとして公開しているポートが分かれば、どのノードにアクセスしても、自動的に対象となるタスク（コンテナ）に到達可能です。内部では自動的にタスクの監視を行うため、スケールアップ・スケールダウンによりコンテナが増減しても、外部から内部の状況を意識する必要がなくなります。

[<img src="/images/swarm-mode-5.png" width="100%">](/images/swarm-mode-5.png)

　これを応用したら、この Ingress Load Balancing の上にロードバランサやプロキシを置くなり、あるいは DNS ラウンドロビンなどで可用性を高めるサービスが作られるかもしれません。実際には性能面での検証は必要と思われますが、Nginx などリバース・プロキシをたてなくても、良い感じで分散してくれる仕組みは、使いどころがありそうだと感じています。

## サービスの更新

　次は、サービスの数を３つに増やします。状態を変更するのは ``docker service update``  コマンドです。

```
$ docker service update --replicas 3 web
web
```

　サービス一覧を確認したら、期待数が 3 になっているのが分かります。

```
$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
13765buws9fr  web   2/3       nginx
```

　タスクの一覧で状況を確認してみます。

```
$ docker service tasks web
ID                         NAME   SERVICE  IMAGE  LAST STATE           DESIRED STATE  NODE
emjve46lgwjx22o6pxpbetfed  web.1  web      nginx  Running 2 hours      Running        node-02
8owd8swtc99coag9evqz4j0dx  web.2  web      nginx  Running 8 seconds    Running        node-01
6z7svrkbpjyfp7gme4r1c2qh4  web.3  web      nginx  Preparing 8 seconds  Running        node-03
```

　ここでは３台のノードに分散して nginx コンテナを起動しようとしています。 ``node-01`` ``node-02`` は ``Running`` （稼働中）と分かりますが、 ``node-03`` は ``Preparing`` （準備中）です。そのため、 ``docker service ls`` の ``REPLICAS`` が ``2/3`` でした。

　もうしばらくしてから ``docker service ls`` を実行したら、最終的に３つのタスクが実行中になります。

```
$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
13765buws9fr  web   3/3       nginx
```

　なお３つのノードに分散してスケジューリングされたのは、デフォルトでは Spread （スプレッド）というコンテナをノードに分散配置するストラテジが適用されているからです。

　次はまたタスクの数を減らしましょう。

```
$ docker service update --replicas 1 web
web
$ docker service tasks web
ID                         NAME   SERVICE  IMAGE  LAST STATE         DESIRED STATE  NODE
6z7svrkbpjyfp7gme4r1c2qh4  web.3  web      nginx  Running 4 minutes  Running        node-03
```

　このように、再び ``docker service update`` コマンドを実行するだけで、スケールダウンも可能です。

　最後にサービスを削除するには ``docker service rm`` コマンドを使います。

```
$ docker service rm web
web
$ docker service ls
ID  NAME  REPLICAS  IMAGE  COMMAND
```

　これでサービスの削除完了です。

　このように、``docker service`` コマンドでサービスを定義したら、 ``docker`` コマンドを使わなくても、swarm クラスタ上でコンテナの起動・更新・停止が可能になります。

　なお、この検証を行った Docker 1.12RC2 はリリース候補版のため、実際の Docker 1.12 では仕様が変わる可能性もありますのでご了承ください。現段階では開発中ですが、色々と評価してみる分には、面白いのではないかと思います。

## 参考

* Swarm mode overview
  * https://docs.docker.com/engine/swarm/
* Swarm mode key concepts
  * https://docs.docker.com/engine/swarm/key-concepts/
* Getting started with swarm mode
  * https://docs.docker.com/engine/swarm/swarm-tutorial/
* Docker 1.12: Now with Built-in Orchestration! | Docker Blog
  * https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/

