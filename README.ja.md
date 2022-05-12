[![FIWARE Banner](https://fiware.github.io/tutorials.Historic-Context-Flume/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Historic-Context-Flume.svg)](https://opensource.org/licenses/MIT)

[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware-cygnus)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

<!-- prettier-ignore -->
このチュートリアルは、コンテキスト・データをサードパーティのデータベースに保存し
てコンテキストの履歴ビューを作成するために使用する汎用イネーブラである
、[FIWARE Cygnus](https://fiware-cygnus.readthedocs.io/en/latest/) の概要です。
このチュートリアルでは
、[前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent)で接続した
IoT センサをアクティブにし、これらのセンサからの測定値をデータベースに保存してさ
らに分析します。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用してい
ますが
、[Postman documentation](https://fiware.github.io/tutorials.Historic-Context-Flume/)
も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/4824d3171f823935dcab)

## コンテンツ

<details>
<summary>詳細 <b>(クリックして拡大)</b></summary>

-   [データの永続性](#data-persistence)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker と Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [起動](#start-up)
-   [MongoDB - コンテキスト・データをデータベースに永続化](#mongodb---persisting-context-data-into-a-database)
    -   [MongoDB - データベース・サーバの設定](#mongodb---database-server-configuration)
    -   [MongoDB - Cygnus の設定](#mongodb---cygnus-configuration)
    -   [MongoDB - 起動](#mongodb---start-up)
        -   [Cygnus サービスの健全性をチェック](#checking-the-cygnus-service-health)
        -   [コンテキスト・データの生成](#generating-context-data)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes)
    -   [MongoDB - データベースからデータを読み込む](#mongodb----reading-data-from-a-database)
        -   [MongoDB サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-mongodb-server)
        -   [サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-server)
-   [PostgreSQL - コンテキスト・データをデータベースに永続化](#postgresql---persisting-context-data-into-a-database)
    -   [PostgreSQL - データベース・サーバの設定](#postgresql---database-server-configuration)
    -   [PostgreSQL - Cygnus の設定](#postgresql---cygnus-configuration)
    -   [PostgreSQL - 起動](#postgresql---start-up)
        -   [Cygnus サービスの健全性をチェック](#checking-the-cygnus-service-health-1)
        -   [コンテキスト・データの生成](#generating-context-data-1)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-1)
    -   [PostgreSQL - データベースからデータを読み込む](#postgresql---reading-data-from-a-database)
        -   [PostgreSQL サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-postgresql-server)
        -   [PostgreSQL サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-postgresql-server)
-   [ElasticSearch - コンテキスト・データをデータベースに永続化](#elasticsearch---persisting-context-data-into-a-database)
    -   [ElasticSearch - データベース・サーバの設定](#elasticsearch---database-server-configuration)
    -   [ElasticSearch - Cygnus の設定](#elasticsearch---cygnus-configuration)
    -   [ElasticSearch - 起動](#elasticsearch---start-up)
        -   [Cygnus サービスの健全性をチェック](#checking-the-cygnus-service-health-2)
        -   [コンテキスト・データの生成](#generating-context-data-2)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-2)
    -   [ElasticSearch - データベースからデータを読み込む](#elasticsearch---reading-data-from-a-database)
        -   [ElasticSearch サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-elasticsearch-server)
        -   [ElasticSearch サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-elasticsearch-server)
-   [MySQL - コンテキスト・データをデータベースに永続化](#mysql---persisting-context-data-into-a-database)
    -   [MySQL - データベース・サーバの設定](#mysql---database-server-configuration)
    -   [MySQL - Cygnus の設定](#mysql---cygnus-configuration)
    -   [MySQL - 起動](#mysql---start-up)
        -   [Cygnus サービスの健全性をチェック](#checking-the-cygnus-service-health-3)
        -   [コンテキスト・データの生成](#generating-context-data-3)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-3)
    -   [MySQL - データベースからデータを読み込む](#mysql---reading-data-from-a-database)
        -   [MySQL サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-mysql-server)
        -   [MySQL サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-mysql-server)
-   [マルチ・エージェント - 複数のデータベースへのコンテキスト・データの永続化](#multi-agent---persisting-context-data-into-a-multiple-databases)
    -   [マルチ・エージェント - 複数のデータベースのための Cygnus 設定](#multi-agent---cygnus-configuration-for-multiple-databases)
    -   [マルチ・エージェント - 起動](#multi-agent---start-up)
        -   [Cygnus サービスの健全性をチェック](#checking-the-cygnus-service-health-4)
        -   [コンテキスト・データの生成](#generating-context-data-4)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-4)
    -   [マルチ・エージェント - 永続化データの読み込み](#multi-agent---reading-persisted-data)
-   [次のステップ](#next-steps)

</details>

<a name="data-persistence"></a>

# データの永続性

> "History will be kind to me for I intend to write it."
>
> — Winston Churchill

以前のチュートリアルでは、実世界の状態の測定値を提供する IoT センサと、**Orion
Context Broker** と **IoT Agent** の 2 つの FIWARE コンポーネントを紹介しました
。このチュートリアルでは、新しいデータ永続化コンポーネント FIWARE **Cygnus** を
紹介します。

これまでのシステムは、現在のコンテキストを扱うように構築されています。つまり、与
えられた瞬間に現実のオブジェクトの状態を定義するデータ・エンティティを保持してい
ます。

この定義から、コンテキストはシステムの**現在**の状態のみに関心があります。システ
ムの履歴状態を報告するのは既存コンポーネントの責任ではありません。コンテキストは
、各センサが Context Broker に送信した最後の測定値に基づいています。

これを行うためには、コンテキストが更新されるたびに、状態の変化をデータベースに永
続化するために、既存のアーキテクチャを拡張する必要があります。

過去のコンテキスト・データを永続化することは、大規模なデータ分析に役立ちます。傾
向を発見するために使用することができます。また、データをサンプリングして集約して
、外部データ測定の影響を取り除くこともできます。ただし、各スマート・ソリューショ
ンでは、各エンティティ型の重要性が異なり、エンティティと属性を異なるレートでサン
プリングする必要があります。

コンテキスト・データを使用するためのビジネス要件はアプリケーションごとに異なりま
すので、履歴データの永続化のための標準的な使用例は 1 つもありません。それぞれの
状況は固有のものであり、1 つのサイズがすべてに適合するケースではありません。した
がって、Context Broker に履歴的なコンテキスト・データの永続性を与えることではな
く、この役割は、構成可能な別のコンポーネント **Cygnus** に分離されています。

ご想像のとおり、オープンソース・プラットフォームの一部としての **Cygnus** は、デ
ータの永続化に使用されるデータベースに関する技術に依存しません。使用するデータベ
ースは、ビジネス・ニーズに応じて異なります。

ただし、この柔軟性を提供するにはコストがかかります。システムの各部分を個別に設定
し、必要な最小限のデータだけを通知するように通知を設定する必要があります。

#### デバイス・モニタ

このチュートリアルの目的のために、一連のダミー IoT デバイスが作成され、Context
Broker に接続されます。使用しているアーキテクチャとプロトコルの詳細は
、[IoT Sensors チュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-v2)に
あります。各デバイスの状態は、次の UltraLight デバイス・モニタの Web ページで確
認できます : `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Historic-Context-Flume/img/device-monitor.png)

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは
、[前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent/)で作成した
コンポーネントと ダミー IoT デバイスをベースにしています。3 つの FIWARE コンポー
ネントを使用します
。[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/),
[IoT Agent for Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/),
コンテキスト・データをデータベースに永続化するための
[Cygnus Generic Enabler](https://fiware-cygnus.readthedocs.io/en/latest/) を導入
しました。Orion Context Broker と IoT Agent の両方が
[MongoDB](https://www.mongodb.com/) テクノロジを利用して保持している情報の永続性
を維持しています。**MySQL**, **PostgreSQL**, **MongoDB**, **ElasticSearch** データベースのいずれか
で、履歴コンテキスト・データを永続化します。

したがって、全体のアーキテクチャーは以下の要素で構成されます :

-   3 つの **FIWARE 汎用イネーブラー** :
    -   FIWARE
        [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)は
        、[NGSI](https://fiware.github.io/specifications/ngsiv2/latest/) を使用
        してリクエストを受信します
    -   FIWARE
        [IoT Agent for Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/)
        は
        、[Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        フォーマットのダミー IoT デバイスからノース・バウンドの測定値を受信し
        、Context Broker がコンテキスト・エンティティの状態を変更するための
        [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) リクエス
        トに変換します
-   FIWARE Cygnus はコンテキストの変更をサブスクライブし、データベース
    (**MySQL** , **PostgreSQL** , **MongoDB**) に保持します。
-   以下の**データベース**の 1 つ、2 つ、3 つまたは 4 つ :
    -   基礎となる [MongoDB](https://www.mongodb.com/) データベース :
        -   **Orion Context Broker** が、データ・エンティティなどのコンテキスト
            ・データ情報を保持し、サブスクリプション、レジストレーションするため
            に使用します
        -   **IoT Agent** がデバイスの URL やキーなどのデバイス情報を保持するた
            めに使用します
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして潜在的に
            使用します
    -   追加の [PostgreSQL](https://www.postgresql.org/) データベース :
        -   履歴データを保持するためのデータ・シンクとして潜在的に使用します
    -   追加の [MySQL](https://www.mysql.com/) データベース :
        -   履歴データを保持するためのデータ・シンクとして潜在的に使用します
    -   追加の [ElasticSearch](https://www.elastic.co) データベース :
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして使用します
-   3 つの**コンテキストプロバイダ** :
    -   **在庫管理フロントエンド**は、このチュートリアルで使用していません。これ
        は以下を行います :
        -   店舗情報を表示し、ユーザーがダミー IoT デバイスと対話できるようにし
            ます
        -   各店舗で購入できる商品を表示します
        -   ユーザが製品を購入して在庫数を減らすことを許可します
    -   HTTP 上で動作する
        [Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用して
        、[ダミー IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-v2)の
        セットとして機能する Web サーバ
    -   このチュートリアルでは、**コンテキスト・プロバイダの NGSI proxy** は使用
        しません。これは以下を行います :
        -   [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使
            用してリクエストを受信します
        -   独自の API を独自のフォーマットで使用して、公開されているデータ・ソ
            ースへのリクエストを行います
        -   [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) 形式
            でコンテキスト・データ Orion Context Broker に返します

要素間のすべての対話は HTTP リクエストによって開始されるため、エンティティはコン
テナ化され、公開されたポートから実行されます。

チュートリアルの各セクションの具体的なアーキテクチャについては、以下で説明します
。

<a name="prerequisites"></a>

# 前提条件

<a name="docker-and-docker-compose"></a>

## Dokcer と Docker Compose

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に
分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行する
ためのツールです
。[YAML file](https://github.com/FIWARE/tutorials.Historic-Context-Flume/tree/master/docker-compose)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つ
まり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます
。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部と
してインストールされますが、Linux ユーザ
は[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要
があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージ
ョンを確認できます :

```console
docker-compose -v
docker version
```

Docker バージョン 20.10 以降と Docker Compose 1.29 以上を使用していることを確認
し、必要に応じてアップグレードしてください。

<a name="cygwin-for-windows"></a>

## Cygwin for Windows

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは
[cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディスト
リビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>

# 起動

開始する前に、必要な Docker イメージをローカルで取得または構築しておく必要があり
ます。リポジトリを複製し、以下のコマンドを実行して必要なイメージを作成してくださ
い :

```console
git clone https://github.com/FIWARE/tutorials.Historic-Context-Flume.git
cd tutorials.Historic-Context-Flume
git checkout NGSI-v2

./services create
```

その後、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Historic-Context-Flume/blob/NGSI-v2/services)
の Bash スクリプトを実行することによって、コマンドラインからすべてのサービスを初
期化できます :

```console
./services <command>
```

ここで、`<command>` は、有効にしたいデータベースによって異なります。このコマンド
は、前のチュートリアルのシード・データをインポートし、起動時にダミー IoT センサ
をプロビジョニングします。

> :information_source: **注:** クリーンアップをやり直したい場合は、次のコマンド
> を使用して再起動することができます :
>
> ```console
> ./services stop
> ```

<a name="mongodb---persisting-context-data-into-a-database"></a>

# MongoDB - コンテキスト・データをデータベースに永続化

MongoDB テクノロジーを使用して、履歴コンテキスト・データを永続化することは
、Orion Context Broker と IoT Agent に関連するデータを保持するために既に MongoDB
インスタンスを使用しているため、比較的簡単に構成できます。MongoDB インスタンスは
標準 `27017` ポートをリッスンしており、全体のアーキテクチャは以下のようになりま
す :

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/cygnus-mongo.png)

<a name="mongodb---database-server-configuration"></a>

## MongoDB - データベース・サーバの設定

```yaml
mongo-db:
    image: mongo:4.2
    hostname: mongo-db
    container_name: db-mongo
    ports:
        - "27017:27017"
    networks:
        - default

```

<a name="mongodb---cygnus-configuration"></a>

## MongoDB - Cygnus の設定

```yaml
cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - "5080"
    ports:
        - "5051:5051"
        - "5080:5080"
    environment:
        - "CYGNUS_MONGO_HOSTS=mongo-db:27017"
        - "CYGNUS_MONGO_SERVICE_PORT=5051"
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_API_PORT=5080"
        - "CYGNUS_SERVICE_PORT=5051"
```

`cygnus` コンテナは、2 つのポートでリッスンしています :

-   Cygnus のサブスクリプション・ポート : `5051` は、サービスが Orion context
    broker からの通知をリッスンするポートです
-   Cygnus の管理ポート : `5080`は、純粋にチュートリアル・アクセスのために公開さ
    れているため、cUrl または Postman は同じネットワークの一部ではなくプロビジョ
    ニング・コマンドを作成できます

`cygnus` コンテナは、示されているように環境変数によって制御できます :

| キー                | 値               | 説明                                                                                         |
| ------------------- | ---------------- | -------------------------------------------------------------------------------------------- |
| CYGNUS_MONGO_HOSTS  | `mongo-db:27017` | Cygnus が履歴コンテキスト・データを保持するために接続する MongoDB サーバのカンマ区切りリスト |
| CYGNUS_LOG_LEVEL    | `DEBUG`          | Cygnus のログレベル                                                                          |
| CYGNUS_SERVICE_PORT | `5051`           | コンテキスト・データの変更をサブスクライブするときに Cygnus がリッスンする通知ポート         |
| CYGNUS_API_PORT     | `5080`           | Cygnus が操作上の理由でリッスンするポート                                                    |

<a name="mongodb---start-up"></a>

## MongoDB - 起動

**MongoDB** データベースのみでシステムを起動するには、次のコマンドを実行します :

```console
./services mongodb
```

<a name="checking-the-cygnus-service-health"></a>

### Cygnus サービスの健全性をチェック

Cygnus が動作したら、公開されている `CYGNUS_API_PORT` ポートへの HTTP リクエスト
を行うことでステータスを確認できます。レスポンスがブランクの場合、これは通常
、Cygnus が実行されていないか、別のポートでリッスンしているためです。

#### :one: リクエスト :

```console
curl -X GET \
  'http://localhost:5080/v1/version'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "success": "true",
    "version": "1.18.0.SNAPSHOT...etc"
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうなりますか？
>
> -   Dokcer コンテナが動作していることを確認するには :
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが走っているのを確認してください。`cygnus` が実行されていな
> い場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されるシステムを監視する必要が
あります。ダミー IoT センサを使用してこれを行うことができます
。`http://localhost:3000/device/monitor` でデバイス・モニタのページを開き、**ス
マート・ドア**のロックを解除し、**スマート・ランプ**をオンにします。これは、ドロ
ップ・ダウン・リストから適切なコマンドを選択し、`send` ボタンを押すことによって
行うことができます。デバイスからの測定値のストリームは、同じページに表示されます
:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/door-open.gif)

<a name="subscribing-to-context-changes"></a>

### コンテキスト変更のサブスクライブ

動的コンテキスト・システムが起動したら、**Cygnus** にコンテキストの変更を通知す
る必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに POST リクエ
ストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、サブスクリプションをフィ
    ルタリングして、接続されている IoT センサからの測定値だけをリッスンするため
    に使用します。センサがこれらの設定を使用してプロビジョニングされているためで
    す
-   リクエスト・ボディの `idPattern` は、すべてのコンテキスト・データの変更が
    Cygnus に通知されるようにします
-   通知 `url` は設定された `CYGNUS_MONGO_SERVICE_PORT` と一致する必要があります
-   Cygnus は現在、古い NGSI v1 形式の通知のみを受け付けているため
    、`attrsFormat=legacy` が必要です
-   `throttling` 値は、変更がサンプリングされる割合を定義します

#### :two: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Cygnus (Mongo-DB) of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://cygnus:5051/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを保持するために使用されるデータベースは、サブ
スクリプションの詳細に影響を与えません。各データベースで同じです。レスポンスは
**201 - Created** です。

> :information_source: **注 :** **Cygnus** ログ内で次のフォームのエラーが表示さ
> れた場合 :
>
> ```
> Received bad request from client.
> cygnus         | org.apache.flume.source.http.HTTPBadRequestException: 'fiware-servicepath' header
> value does not match the number of notified context responses
> ```
>
> これは通常、`"attrsFormat": "legacy"` フラグが省略されているためです。

サブスクリプションが作成されている場合は、`/v2/subscriptions` エンドポイントに対
して GET リクエストを出すことで、それが起動しているかどうかを確認することができ
ます。

#### :three: リクエスト :

```console
curl -X GET \
  'http://localhost:1026/v2/subscriptions/' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス :

```json
[
    {
        "id": "5b39d7c866df40ed84284174",
        "description": "Notify Cygnus (Mongo-DB) of all context changes",
        "status": "active",
        "subject": {
            "entities": [
                {
                    "idPattern": ".*"
                }
            ],
            "condition": {
                "attrs": []
            }
        },
        "notification": {
            "timesSent": 158,
            "lastNotification": "2018-07-02T07:59:21.00Z",
            "attrs": [],
            "http": {
                "url": "http://cygnus:5050/notify"
            },
            "lastSuccess": "2018-07-02T07:59:21.00Z"
        },
        "throttling": 5
    }
]
```

レスポンスの `notification` セクション内には、サブスクリプションの健全性を表すい
くつかの追加の `attributes` があります

サブスクリプションの基準が満たされている場合、`timesSent` は`0` より大きくなけれ
ばなりません。ゼロの値は、サブスクリプションの `subject` が間違っているか、サブ
スクリプションが間違った `fiware-service-path` または `fiware-service` ヘッダで
作成されたことを示します

`lastNotification` は最近のタイム・スタンプでなければなりません。そうでなければ
、デバイスは定期的にデータを送信しません。 **スマート・ドア**のロックを解除し
、**スマート・ランプ**をオンにすることを忘れないでください

`lastSuccess` は `lastNotification` の日付と一致する必要があります。そうでない場
合、**Cygnus** はサブスクリプションを正しく受信していません。 ホスト名とポートが
正しいことを確認してください。

最後に、サブスクリプションの `status` が `active` であることを確認します。期限切
れのサブスクリプションは起動しません。

<a name="mongodb----reading-data-from-a-database"></a>

## MongoDB - データベースからデータを読み込む

コマンドラインから `mongo-db` データを読み込むには、コマンドライン・プロンプトを
表示するために、`mongo` ツールにアクセスして `mongo` イメージのインタラクティブ
なインスタンスを実行する必要があります :

```console
docker run -it --network fiware_default  --entrypoint /bin/bash mongo
```

次のようにコマンドラインを使用して、実行中の `mongo-db` データベースにログインで
きます :

```bash
mongo --host mongo-db
```

<a name="show-available-databases-on-the-mongodb-server"></a>

### MongoDB サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のように文を実行します :

#### クエリ :

```
show dbs
```

#### 結果 :

```
admin          0.000GB
iotagentul     0.000GB
local          0.000GB
orion          0.000GB
orion-openiot  0.000GB
sth_openiot    0.000GB
```

結果には、デフォルトで MongoDB によって設定されている、2 つのデータベース
`admin` と `local` があり、FIWARE プラットフォームによって作成された 4 つのデー
タベースも含まれています。Orion Context Broker は、それぞれの `fiware-service`
に 2 つのデータベース・インスタンスを作成しました。

-   IoT デバイスのエンティティは、`openiot` `fiware-service` ヘッダを使用して作
    成され、別々に保持されるのに対し、ストア・エンティティは、`fiware-service`
    を定義することなく作成され、したがって `orion` データベース内に保持されます
    。 IoT Agent は、IoT センサ・データ`iotagentul` という別の **MongoDB** デー
    タベースに保持するように初期化されました。

Orgn Context Broker に Cygnus をサブスクリプションした結果、`sth_openiot` という
新しいデータベースが作成されました。履歴コンテキストを保持する **MongoDB** デー
タベースのデフォルト値は、`sth_` プレフィックスの後ろに `fiware-service` ヘッダ
が続くため、`sth_openiot` は IoT デバイスの履歴コンテキストを保持します。

<a name="read-historical-context-from-the-server"></a>

### サーバから履歴コンテキストを読み込む

#### クエリ :

```
use sth_openiot
show collections
```

#### 結果 :

```
switched to db sth_openiot

sth_/_Door:001_Door
sth_/_Door:001_Door.aggr
sth_/_Lamp:001_Lamp
sth_/_Lamp:001_Lamp.aggr
sth_/_Motion:001_Motion
sth_/_Motion:001_Motion.aggr
```

`sth_openiot` 内を見ると、一連のテーブルが作成されていることがわかります。各テー
ブルの名前は、`sth_` プレフィックスの後に `fiware-servicepath` ヘッダとそれに続
くエンティティ id が続きます。各エンティティごとに 2 つのテーブルが作成されます
。`.aggr` テーブルには、後でチュートリアルでアクセスするいくつかの集計データが格
納されています。生データは、`.aggr` サフィックスなしのテーブルで見ることができま
す。

履歴データは各テーブル内のデータを確認するで見ることができます。デフォルトでは、
各行には単一の属性のサンプリング値が含まれます。

#### クエリ :

```
db["sth_/_Door:001_Door"].find().limit(10)
```

#### 結果 :

```
{ "_id" : ObjectId("5b1fa48630c49e0012f7635d"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "TimeInstant", "attrType" : "ISO8601", "attrValue" : "2018-06-12T10:46:30.836Z" }
{ "_id" : ObjectId("5b1fa48630c49e0012f7635e"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "close_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f7635f"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "lock_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76360"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "open_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76361"), "recvTime" : ISODate("2018-06-12T10:46:30.836Z"), "attrName" : "refStore", "attrType" : "Relationship", "attrValue" : "Store:001" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76362"), "recvTime" : ISODate("2018-06-12T10:46:30.836Z"), "attrName" : "state", "attrType" : "Text", "attrValue" : "CLOSED" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76363"), "recvTime" : ISODate("2018-06-12T10:45:26.368Z"), "attrName" : "unlock_info", "attrType" : "commandResult", "attrValue" : " unlock OK" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76364"), "recvTime" : ISODate("2018-06-12T10:45:26.368Z"), "attrName" : "unlock_status", "attrType" : "commandStatus", "attrValue" : "OK" }
{ "_id" : ObjectId("5b1fa4c030c49e0012f76385"), "recvTime" : ISODate("2018-06-12T10:47:28.081Z"), "attrName" : "TimeInstant", "attrType" : "ISO8601", "attrValue" : "2018-06-12T10:47:28.038Z" }
{ "_id" : ObjectId("5b1fa4c030c49e0012f76386"), "recvTime" : ISODate("2018-06-12T10:47:28.081Z"), "attrName" : "close_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
```

通常の **MongoDB** クエリ構文は、適切なフィールドと値をフィルタリングするために
使用できます。たとえば、`id=Motion:001_Motion` の**モーション・センサ**が蓄積し
ているレートを読み取るには、次のようにクエリを作成します :

#### クエリ :

```
db["sth_/_Motion:001_Motion"].find({attrName: "count"},{_id: 0, attrType: 0, attrName: 0 } ).limit(10)
```

#### 結果 :

```
{ "recvTime" : ISODate("2018-06-12T10:46:18.756Z"), "attrValue" : "8" }
{ "recvTime" : ISODate("2018-06-12T10:46:36.881Z"), "attrValue" : "10" }
{ "recvTime" : ISODate("2018-06-12T10:46:42.947Z"), "attrValue" : "11" }
{ "recvTime" : ISODate("2018-06-12T10:46:54.893Z"), "attrValue" : "13" }
{ "recvTime" : ISODate("2018-06-12T10:47:00.929Z"), "attrValue" : "15" }
{ "recvTime" : ISODate("2018-06-12T10:47:06.954Z"), "attrValue" : "17" }
{ "recvTime" : ISODate("2018-06-12T10:47:15.983Z"), "attrValue" : "19" }
{ "recvTime" : ISODate("2018-06-12T10:47:49.090Z"), "attrValue" : "23" }
{ "recvTime" : ISODate("2018-06-12T10:47:58.112Z"), "attrValue" : "25" }
{ "recvTime" : ISODate("2018-06-12T10:48:28.218Z"), "attrValue" : "29" }
```

MongoDB クライアントを離れて、インタラクティブ・モードを終了するには、次のコマン
ドを実行します :

```console
exit
```

```console
exit
```

<a name="postgresql---persisting-context-data-into-a-database"></a>

# PostgreSQL - コンテキスト・データをデータベースに永続化

履歴データ**PostgreSQL** などの代替データベースに保存するには、PostgreSQL サーバ
をホストする追加のコンテナが必要です。このデータのデフォルトの Docker イメージを
使用できます。PostgreSQL インスタンスは標準 `5432` ポートをリッスンしており、全
体のアーキテクチャは以下のようになります :

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/cygnus-postgres.png)

MongoDB コンテナには、Orion Context Broker と IoT Agent に関連するデータを保持す
る必要があるため、2 つのデータベースを持つシステムが用意されています。

<a name="postgresql---database-server-configuration"></a>

## PostgreSQL - データベース・サーバの設定

```yaml
postgres-db:
    image: postgres:latest
    hostname: postgres-db
    container_name: db-postgres
    expose:
        - "5432"
    ports:
        - "5432:5432"
    networks:
        - default
    environment:
        - "POSTGRES_PASSWORD=password"
        - "POSTGRES_USER=postgres"
        - "POSTGRES_DB=postgres"
```

`postgres-db` は、コンテナは単一のポートでリッスンします :

-   ポート `5432` は PostgreSQL サーバのデフォルト・ポートです。必要に応じてデー
    タベースのデータを表示する `pgAdmin4` ツールを実行できるように公開されていま
    す

`postgres-db` は、次に示されているように環境変数によって制御されます :

| キー              | 値         | 説明                                        |
| ----------------- | ---------- | ------------------------------------------- |
| POSTGRES_PASSWORD | `password` | PostgreSQL データベース・ユーザのパスワード |
| POSTGRES_USER     | `postgres` | PostgreSQL データベース・ユーザのユーザ名   |
| POSTGRES_DB       | `postgres` | PostgreSQL データベースの名前               |

> :information_source: **注:** このようなプレーン・テキストの環境変数にユーザ名
> とパスワードを渡すことはセキュリティ上のリスクです。 これはチュートリアルでは
> 許容される方法ですが、プロダクション環境では
> 、[Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

<a name="postgresql---cygnus-configuration"></a>

## PostgreSQL - Cygnus の設定

```yaml
cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    networks:
        - default
    depends_on:
        - postgres-db
    expose:
        - "5080"
    ports:
        - "5055:5055"
        - "5080:5080"
    environment:
        - "CYGNUS_POSTGRESQL_HOST=postgres-db"
        - "CYGNUS_POSTGRESQL_PORT=5432"
        - "CYGNUS_POSTGRESQL_USER=postgres"
        - "CYGNUS_POSTGRESQL_PASS=password"
        - "CYGNUS_POSTGRESQL_ENABLE_CACHE=true"
        - "CYGNUS_POSTGRESQL_SERVICE_PORT=5055"
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_API_PORT=5080"
        - "CYGNUS_SERVICE_PORT=5055"
```

`cygnus` コンテナは、2 つのポートでリッスンしています :

-   Cygnus のサブスクリプション・ポート : `5055` は、サービスが Orion Context
    Broker からの通知をリッスンするポートです
-   Cygnus の管理ポート : `5080` は純粋にチュートリアルのアクセスのために公開さ
    れているため、cUrl または Postman は同じネットワークの一部ではなくプロビジョ
    ニング・コマンドを作成できます

`cygnus` コンテナは、次のように環境変数によって制御されます :

| キー                           | 値            | 説明                                                                                 |
| ------------------------------ | ------------- | ------------------------------------------------------------------------------------ |
| CYGNUS_POSTGRESQL_HOST         | `postgres-db` | 履歴コンテキスト・データの永続化に使用される PostgreSQL サーバのホスト名             |
| CYGNUS_POSTGRESQL_PORT         | `5432`        | PostgreSQL サーバがコマンドをリッスンするために使うポート                            |
| CYGNUS_POSTGRESQL_USER         | `postgres`    | PostgreSQL データベース・ユーザのユーザ名                                            |
| CYGNUS_POSTGRESQL_PASS         | `password`    | PostgreSQL データベース・ユーザのパスワード                                          |
| CYGNUS_LOG_LEVEL               | `DEBUG`       | Cygnus のログレベル                                                                  |
| CYGNUS_SERVICE_PORT            | `5055`        | コンテキスト・データの変更をサブスクライブするときに Cygnus がリッスンする通知ポート |
| CYGNUS_API_PORT                | `5080`        | Cygnus が操作上の理由でリッスンするポート                                            |
| CYGNUS_POSTGRESQL_ENABLE_CACHE | `true`        | PostgreSQL 設定内でキャッシングを有効にするためのスイッチ                            |

> :information_source: **注:** このようなプレーン・テキストの環境変数にユーザ名
> とパスワードを渡すことはセキュリティ上のリスクです。 これはチュートリアルでは
> 許容される方法ですが、プロダクション環境では、`CYGNUS_POSTGRESQL_USER` と
> `CYGNUS_POSTGRESQL_PASS` は
> 、[Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を使用して渡す必要があります。

<a name="postgresql---start-up"></a>

## PostgreSQL - 起動

**PostgreSQL** データベースを使用してシステムを起動するには、次のコマンドを実行
します :

```console
./services postgres
```

<a name="checking-the-cygnus-service-health-1"></a>

### Cygnus サービスの健全性をチェック

Cygnus が動作したら、公開されている `CYGNUS_API_PORT` ポートへの HTTP リクエスト
を行うことでステータスを確認できます。レスポンスがブランクの場合、これは通常
、Cygnus が実行されていないか、別のポートでリッスンしているためです。

#### :four: リクエスト :

```console
curl -X GET \
  'http://localhost:5080/v1/version'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "success": "true",
    "version": "1.18.0.SNAPSHOT...etc"
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうなりますか？
>
> -   Dokcer コンテナが動作していることを確認するには
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが走っているのを確認してください。`cygnus` が実行されていな
> い場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-1"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されるシステムを監視する必要が
あります。ダミー IoT センサを使用してこれを行うことができます
。`http://localhost:3000/device/monitor` でデバイス・モニタのページを開き、**ス
マート・ドア**のロックを解除し、**スマート・ランプ**をオンにします。これは、ドロ
ップ・ダウン・リストから適切なコマンドを選択し、`send` ボタンを押すことによって
行うことができます。デバイスからの測定値のストリームは、同じページに表示されます
:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/door-open.gif)

<a name="subscribing-to-context-changes-1"></a>

### コンテキスト変更のサブスクライブ

動的コンテキスト・システムが起動したら、**Cygnus** にコンテキストの変更を通知す
る必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに POST リクエ
ストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、サブスクリプションをフィ
    ルタリングして、接続されている IoT センサからの測定値だけをリッスンするため
    に使用します。センサがこれらの設定を使用してプロビジョニングされているためで
    す
-   リクエスト・ボディの `idPattern` は、すべてのコンテキスト・データの変更が
    Cygnus に通知されるようにします
-   通知 `url` は設定された `CYGNUS_POSTGRESQL_SERVICE_PORT` と一致する必要があります
-   Cygnus は現在、古い NGSI v1 形式の通知のみを受け付けているため
    、`attrsFormat=legacy` が必要です
-   `throttling` 値は、変更がサンプリングされる割合を定義します

#### :five: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Cygnus (Postgres) of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://cygnus:5055/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを保持するために使用されるデータベースは、サブ
スクリプションの詳細に影響を与えません。各データベースで同じです。レスポンスは
**201 - Created** です。

<a name="postgresql---reading-data-from-a-database"></a>

## PostgreSQL - データベースからデータを読み込む

コマンドラインから PostgreSQL データを読み取るには、`postgres` クライアントにア
クセスする必要があります。これを行うには、コマンドライン・プロンプトを表示するた
めに接続文字列を指定した `postgresql-client` イメージのインタラクティブなインス
タンスを実行します :

```console
docker run -it --rm  --network fiware_default jbergknoff/postgresql-client \
   postgresql://postgres:password@postgres-db:5432/postgres
```

<a name="show-available-databases-on-the-postgresql-server"></a>

### PostgreSQL サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のように文を実行します :

#### クエリ :

```
\list
```

#### 結果 :

```
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```

結果には、2 つのテンプレート・データベース `template0` と `template1` と、Dokcer
コンテナが起動したときの `postgres` データベースの設定が含まれています。

使用可能なスキーマのリストを表示するには、次のように文を実行します :

#### クエリ :

```
\dn
```

#### 結果 :

```
  List of schemas
  Name   |  Owner
---------+----------
 openiot | postgres
 public  | postgres
(2 rows)
```

Orion Context Broker に Cygnus をサブスクリプションした結果、`openiot` という新
しいスキーマが作成されました。スキーマの名前は `fiware-service` ヘッダに一致しま
す。したがって、`openiot` は、IoT デバイスの履歴コンテキストを保持します。

<a name="read-historical-context-from-the-postgresql-server"></a>

### PostgreSQL サーバから履歴コンテキストを読み込む

Docker コンテナをネットワーク内で実行すると、実行中のデータベースに関する情報を
取得することができます。

#### クエリ :

```sql
SELECT table_schema,table_name
FROM information_schema.tables
WHERE table_schema ='openiot'
ORDER BY table_schema,table_name;
```

#### 結果 :

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` は、コンテキスト・データとともに提供される `fiware-service` ヘッ
ダと一致します。

テーブル内のデータを読み込むには、次のように select 文を実行します :

#### クエリ :

```sql
SELECT * FROM openiot.motion_001_motion limit 10;
```

#### 結果 :

```
  recvtimets   |         recvtime         | fiwareservicepath |  entityid  | entitytype |  attrname   |   attrtype   |        attrvalue         |                                    attrmd
---------------+--------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:05.423Z | []
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | count       | Integer      | 7                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:05.423Z"}]
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:05.423Z"}]
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:35.480Z | []
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | count       | Integer      | 10                       | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:35.480Z"}]
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:35.480Z"}]
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:41.520Z | []
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | count       | Integer      | 12                       | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:41.520Z"}]
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:41.520Z"}]
 1528803047545 | 2018-06-12T11:30:47.545Z | /
```

通常の **PostgreSQL** クエリ構文を使用して、適切なフィールドと値をフィルタリング
することができます。たとえば、`id=Motion:001_Motion` の**モーション・センサ**が
蓄積しているレートを読み取るには、次のようにクエリを作成します :

#### クエリ :

```sql
SELECT recvtime, attrvalue FROM openiot.motion_001_motion WHERE attrname ='count'  limit 10;
```

#### 結果 :

```
         recvtime         | attrvalue
--------------------------+-----------
 2018-06-12T11:30:05.491Z | 7
 2018-06-12T11:30:35.501Z | 10
 2018-06-12T11:30:41.563Z | 12
 2018-06-12T11:30:47.545Z | 13
 2018-06-12T11:31:02.617Z | 15
 2018-06-12T11:31:32.718Z | 20
 2018-06-12T11:31:38.733Z | 22
 2018-06-12T11:31:50.780Z | 24
 2018-06-12T11:31:56.825Z | 25
 2018-06-12T11:31:59.790Z | 26
(10 rows)
```

Postgres クライアントを終了してインタラクティブ・モードを終了するには、次のコマ
ンドを実行します :

```console
\q
```

その後、コマンドラインに戻ります。

<a name="elasticsearch---persisting-context-data-into-a-database"></a>

# ElasticSearch - コンテキスト・データをデータベースに永続化

履歴コンテキスト・データを **ElasticSearch** などの代替データベースに永続化するには、ElasticSearch サーバをホストする
追加のコンテナが必要になります。このデータのデフォルトの Docker イメージを使用できます。ElasticSearch シンクは
Elasticsearch のバージョン6.3および7.6でテストされていることに注意してください。ElasticSearch インスタンスは標準の
`9200` ポートでリッスンしており、全体的なアーキテクチャを以下に示します:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/cygnus-elasticsearch.png)

Orion Context Broker と IoT Agent に関連するデータを保持するために MongoDB コンテナが引き続き必要であるため、2つの
データベースを備えたシステムができました。

<a name="elasticsearch---database-server-configuration"></a>

## ElasticSearch - データベース・サーバの設定

```yaml
elasticsearch-db:
    image: elasticsearch:${ELASTICSEARCH_VERSION}
    hostname: elasticsearch
    container_name: db-elasticsearch
    expose:
        - "${ELASTICSEARCH_PORT}"
    ports:
        - "${ELASTICSEARCH_PORT}:${ELASTICSEARCH_PORT}"
    networks:
        - default
    volumes:
        - es-db:/usr/share/elasticsearch/data
    environment:
        ES_JAVA_OPTS: "-Xmx256m -Xms256m"
        ELASTIC_PASSWORD: changeme
        # Use single node discovery in order to disable production mode and avoid bootstrap checks.
        # see: https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html
        discovery.type: single-node
    healthcheck:
        test: curl http://localhost:${ELASTICSEARCH_PORT} >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
        interval: 30s
        timeout: 10s
        retries: 5
```

`elasticsearch-db` コンテナは1つのポートのみをリッスンしています:

-   ポート `9200` (ELASTICSEARCH_PORT) は、ElasticSearchサーバーのデフォルトのポートです

> 注: 複数の ElasticSearch ノードをデプロイする場合は、対応するポートを指定する必要があります。デフォルトでは、
> ノード通信用の ElasticSearch の場合は `9300` (ELASTICSEARCH_NODES_COMMUNICATION_PORT) である必要があります。

`elasticsearch-db` コンテナは、示されているように環境変数によって駆動されます:

| キー             | バリュー            | 詳細                                                 |
| ---------------- | ------------------- | ---------------------------------------------------- |
| ES_JAVA_OPTS     | `-Xmx256m -Xms256m` | JVM ヒープサイズの設定。実稼働環境ではお勧めしません |
| ELASTIC_PASSWORD | `changeme`          | PostgreSQL データベース・ユーザのパスワード          |

> :information_source: **注:** このようなプレーン・テキストの環境変数でパスワードを渡すことは、セキュリティ上の
> リスクです。これはチュートリアルでは許容できる方法ですが、本番環境では、
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> シークレットを適用することでこのリスクを回避できます。

<a name="elasticsearch---cygnus-configuration"></a>

## ElasticSearch - Cygnus の設定

```yaml
cygnus:
    image: fiware/cygnus-ngsi:${CYGNUS_VERSION}
    hostname: cygnus
    container_name: fiware-cygnus
    depends_on:
        - elasticsearch-db
    networks:
        - default
    expose:
        - "${CYGNUS_API_PORT}"
        - "${CYGNUS_ELASTICSEARCH_SERVICE_PORT}"
    ports:
        - "${CYGNUS_ELASTICSEARCH_SERVICE_PORT}:${CYGNUS_ELASTICSEARCH_SERVICE_PORT}" # localhost:5058
        - "${CYGNUS_API_PORT}:${CYGNUS_API_PORT}" # localhost:5088
    environment:
        - "CYGNUS_ELASTICSEARCH_HOST=elasticsearch-db:${ELASTICSEARCH_PORT}"
        - "CYGNUS_ELASTICSEARCH_PORT=${CYGNUS_ELASTICSEARCH_SERVICE_PORT}"
        - "CYGNUS_ELASTICSEARCH_SSL=false"
        - "CYGNUS_API_PORT=${CYGNUS_API_PORT}" # Port that Cygnus listens on for operational reasons
        - "CYGNUS_LOG_LEVEL=DEBUG" # The logging level for Cygnus
    healthcheck:
        test: curl --fail -s http://localhost:${CYGNUS_API_ADMIN_PORT}/v1/version || exit 1
```

`cygnus` コンテナは2つのポートでリッスンしています:

-   Cygnus のサブスクリプション・ポート CYGNUS_ELASTICSEARCH_SERVICE_PORT, `5058` は、サービスが Orion Context Broker
    からの通知をリッスンする場所です
-   Cygnus の管理ポート CYGNUS_API_PORT, `5080` は、純粋にチュートリアル・アクセス用に公開されているため、cUrl または
    Postman は同じネットワークの一部でなくてもプロビジョニング・コマンドを実行できます

`cygnus` コンテナは、示されているように環境変数によって駆動されます:

| キー                      | バリュー                                 | 詳細                                                                                        |
| ------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------- |
| CYGNUS_ELASTICSEARCH_HOST | `elasticsearch-db:${ELASTICSEARCH_PORT}` | 履歴コンテキスト・データを永続化するために使用される ElasticSearch サーバのホスト名とポート |
| CYGNUS_ELASTICSEARCH_PORT | `${CYGNUS_ELASTICSEARCH_SERVICE_PORT}`   | Cygnus が履歴コンテキスト・データを永続化するために使用するポート。デフォルトでは `5058`    |
| CYGNUS_ELASTICSEARCH_SSL  | `false`                                  | SSL は通信用に構成されていません                                                            |
| CYGNUS_API_PORT           | `${CYGNUS_API_PORT}`                     | Cygnus が操作上の理由でリッスンするポート。デフォルトでは `5080`                            |
| CYGNUS_LOG_LEVEL          | `DEBUG`                                  | Cygnus のログレベル                                                                         |

<a name="elasticsearch---start-up"></a>

## ElasticSearch - 起動

**ElasticSearch** データベースを使用してシステムを起動するには、次のコマンドを実行します:

```console
./services elasticsearch
```

<a name="checking-the-cygnus-service-health-2"></a>

### Cygnus サービスの健全性をチェック

Cygnus が実行されたら、公開された `CYGNUS_API_PORT` ポートに HTTP リクエストを送信することで、ステータスを
確認できます。レスポンスが空白の場合、これは通常、Cygnus が実行されていないか、別のポートでリッスンしていることが
原因です。

#### リクエスト:

```console
curl -X GET \
  'http://localhost:5080/v1/version'
```

#### レスポンス:

レスポンスは次のようになります:

```json
{
    "success": "true",
    "version": "1.18.0_SNAPSHOT.etc"
}
```

> **トラブルシューティング:** レスポンスが空白の場合はどうなりますか？
>
> -   Docker コンテナが実行されていることを確認するには、
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが実行されているのが見えるはずです。`cygnus` が実行されていない場合は、必要に応じてコンテナを
> 再起動できます。

<a name="checking-the-cygnus-service-health-2"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを監視する必要があります。ダミー IoT センサを
使用してこれを行うことができます。`http://localhost:$TUTORIAL_APP_PORT/device/monitor` でデバイス・モニタのページ
を開きます。**Smart Door** のロックを解除し、**Smart Lamp** をオンにします。変数 `Tutorial_APP_PORT` は `.env`
ファイルで定義されていることに注意してください。これは、ドロップ・ダウン・リストから適切なコマンドを選択し、
`send` ボタンを押すことで実行できます。デバイスからの測定値の流れは、同じページに表示されます:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/door-open.gif)

<a name="subscribing-to-context-changes-2"></a>

### コンテキスト変更のサブスクライブ

動的コンテキストシステムが稼働したら、コンテキストの変更を **Cygnus** に通知する必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに POST リクエストを行うことで実行されます。

-   `fiware-service` ヘッダと `fiware-servicepath` ヘッダは、これらの設定を使用してプロビジョニングされているため、
    接続された IoT センサからの測定値のみをリッスンするようにサブスクリプションをフィルタリングするために使用
    されます
-   リクエスト・ボディの `idPattern` は、Cygnus にすべてのコンテキスト・データの変更が通知されるようにします
-   通知 (notification) `url` は設定された `CYGNUS_ELASTICSEARCH_SERVICE_PORT` と一致する必要があります
-   `throttling` 値は、変更がサンプリングされるレートを定義します

#### リクエスト:

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Cygnus ElasticSearch of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://cygnus:5058/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データの永続化に使用されるデータベースは、サブスクリプションの詳細に影響を与えません。
各データベースで同じです。レスポンスは **201 - Created** になります。

<a name="elasticsearch---reading-data-from-a-database"></a>

## ElasticSearch - データベースからデータを読み込む

コマンドラインから ElasticSearch データを読み取るために、一連の HTTP リクエストを実行してデータを取得します。
データにアクセスするための特定のクエリを作成する方法を知りたい場合は、現在のバージョンの
[Elastic Search API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/search-your-data.html)
を参照してください。

<a name="show-available-databases-on-the-elasticsearch-server"></a>

### ElasticSearch サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のようにステートメントを実行します:

### クエリ:

```console
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

### 結果:

```
health status index                                        uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   cygnus-openiot--motion-003-motion-2021.04.16 lh3Y2NT8SJaqNrkLDlmFOg   1   1        288            0    119.4kb        119.4kb
yellow open   cygnus-openiot--lamp-004-lamp-2021.04.16     XZ4zor2rTkeXBtO-uvJ0KA   1   1        451            0     92.8kb         92.8kb
yellow open   cygnus-openiot--motion-001-motion-2021.04.16 TSMdVhXhSeOygOGAhgqHtw   1   1         16            0      267kb          267kb
yellow open   cygnus-openiot--motion-004-motion-2021.04.16 K_RN-eVIQGGDlRVNpmiDow   1   1        400            0     95.4kb         95.4kb
yellow open   cygnus-openiot--lamp-002-lamp-2021.04.16     Nveh6Ka1TQ2mDcxqh_YvXg   1   1        522          102    181.9kb        181.9kb
yellow open   cygnus-openiot--lamp-003-lamp-2021.04.16     Mo5paVjeQwabf6PjGuNzWw   1   1        627            0    109.1kb        109.1kb
yellow open   cygnus-openiot--door-001-door-2021.04.16     jWZccKX3Tjayd2nUrjuxSw   1   1         32            4    380.8kb        380.8kb
yellow open   cygnus-openiot--lamp-001-lamp-2021.04.16     9L-1ZbX_TxmgyQXpQN_6lA   1   1        957            0    170.5kb        170.5kb
yellow open   cygnus-openiot--motion-002-motion-2021.04.16 mvvS-IGvR2C4cy7KryJy1g   1   1        440            0    108.6kb        108.6kb
```

結果には、インデックスの完全なリスト、レジストリの数、および各テーブルのストア・サイズが含まれます。この情報を取得
したら、各センサの情報をリクエストできます:

<a name="read-historical-context-from-the-elasticsearch-server"></a>

### ElasticSearch サーバから履歴コンテキストを読み込む

ネットワーク内で Docker コンテナを実行すると、実行中のデータベースに関する情報を取得できます。この場合、`motion003`
エンティティにアクセスし、結果を2つに制限します。

### クエリ:

```console
curl -XGET 'localhost:9200/_sql?format=json' -H 'Content-Type: application/json' -d'
{
  "query": " select * from \"cygnus-openiot--motion-003-motion-2021.04.16\" limit 2 "
}'
```

### 結果:

```json
{
    "columns": [
        {
            "name": "attrMetadata.name",
            "type": "text"
        },
        {
            "name": "attrMetadata.type",
            "type": "text"
        },
        {
            "name": "attrMetadata.value",
            "type": "datetime"
        },
        {
            "name": "attrName",
            "type": "text"
        },
        {
            "name": "attrType",
            "type": "text"
        },
        {
            "name": "attrValue",
            "type": "text"
        },
        {
            "name": "entityId",
            "type": "text"
        },
        {
            "name": "entityType",
            "type": "text"
        },
        {
            "name": "recvTime",
            "type": "datetime"
        }
    ],
    "rows": [
        [
            "TimeInstant",
            "DateTime",
            "2021-04-16T09:23:38.418Z",
            "supportedProtocol",
            "Text",
            "[\"ul20\"]",
            "Motion:003",
            "Motion",
            "2021-04-16T09:23:38.418Z"
        ],
        [
            "TimeInstant",
            "DateTime",
            "2021-04-16T09:23:38.418Z",
            "function",
            "Text",
            "[\"sensing\"]",
            "Motion:003",
            "Motion",
            "2021-04-16T09:23:38.418Z"
        ]
    ]
}
```

<a name="mysql---persisting-context-data-into-a-database"></a>

# MySQL - コンテキスト・データをデータベースに永続化

同様に、履歴コンテキスト・データ**MySQL** に永続化するには、MySQL サーバをホスト
する追加のコンテナが必要になります。このデータのデフォルトの Docker イメージも使
用できます。MySQL インスタンスは標準 `3306` ポートでリッスンしており、全体のアー
キテクチャは以下のようになります :

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/cygnus-mysql.png)

MongoDB コンテナは、Orion Context Broker と IoT Agent に関連するデータを保持する
必要があるため、2 つのデータベースを持つシステムがあります。

<a name="mysql---database-server-configuration"></a>

## MySQL - データベース・サーバの設定

```yaml
mysql-db:
    restart: always
    image: mysql:5.7
    hostname: mysql-db
    container_name: db-mysql
    expose:
        - "3306"
    ports:
        - "3306:3306"
    networks:
        - default
    environment:
        - "MYSQL_ROOT_PASSWORD=123"
        - "MYSQL_ROOT_HOST=%"
```

> :information_source: **注:** デフォルトの `root` ユーザを使用し、このような環
> 境変数にパスワードを表示することはセキュリティ上のリスクです。これはチュートリ
> アルでは受け入れられるものですが、本番環境では別のユーザを設定して
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

`mysql-db` コンテナは、単一ポートで待機しています :

-   ポート `3306` は MySQL サーバのデフォルト・ポートです。これは公開されている
    ので、必要に応じて他のデータベース・ツールを実行してデータを表示することもで
    きます

`mysql-db` コンテナは、次のように環境変数によって制御されます :

| キー                | 値         | 説明                                                                                                                                                                                             |
| ------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| MYSQL_ROOT_PASSWORD | `123`      | MySQL `root` アカウントに設定されているパスワードを指定します                                                                                                                                    |
| MYSQL_ROOT_HOST     | `postgres` | デフォルトでは、MySQL によって `root'@'localhost` アカウントが作成されます。このアカウントはコンテナ内からのみ接続できます。この環境変数を設定すると、他のホストからのルート接続が可能になります |

<a name="mysql---cygnus-configuration"></a>

## MySQL - Cygnus の設定

```yaml
cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    networks:
        - default
    depends_on:
        - mysql-db
    expose:
        - "5080"
    ports:
        - "5050:5050"
        - "5080:5080"
    environment:
        - "CYGNUS_MYSQL_HOST=mysql-db"
        - "CYGNUS_MYSQL_PORT=3306"
        - "CYGNUS_MYSQL_USER=root"
        - "CYGNUS_MYSQL_PASS=123"
        - "CYGNUS_MYSQL_SERVICE_PORT=5050"
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_API_PORT=5080"
        - "CYGNUS_SERVICE_PORT=5050"
```

> :information_source: **注:** このようなプレーン・テキストの環境変数にユーザ名
> とパスワードを渡すことはセキュリティ上のリスクです。 これはチュートリアルでは
> 許容される方法ですが、プロダクション環境では、`CYGNUS_MYSQL_USER` と
> `CYGNUS_MYSQL_PASS` は
> 、[Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を使用して渡す必要があります。

`cygnus` コンテナは、2 つのポートでリッスンしています :

-   Cygnus のサブスクリプション・ポート : `5050` は、サービスが Orion context
    broker からの通知をリッスンするポートです
-   Cygnus の管理ポート : `5080`は、純粋にチュートリアル・アクセスのために公開さ
    れているため、cUrl または Postman は同じネットワークの一部ではなくプロビジョ
    ニング・コマンドを作成できます

`cygnus` コンテナは、図のように環境変数によって制御されます :

| キー                | 値         | 説明                                                                          |
| ------------------- | ---------- | ----------------------------------------------------------------------------- |
| CYGNUS_MYSQL_HOST   | `mysql-db` | Hostname of the MySQL server used to persist historical context data          |
| CYGNUS_MYSQL_PORT   | `3306`     | Port that the MySQL server uses to listen to commands                         |
| CYGNUS_MYSQL_USER   | `root`     | Username for the MySQL database user                                          |
| CYGNUS_MYSQL_PASS   | `123`      | Password for the MySQL database user                                          |
| CYGNUS_LOG_LEVEL    | `DEBUG`    | The logging level for Cygnus                                                  |
| CYGNUS_SERVICE_PORT | `5050`     | Notification Port that Cygnus listens when subcribing to context data changes |
| CYGNUS_API_PORT     | `5080`     | Port that Cygnus listens on for operational reasons                           |

<a name="mysql---start-up"></a>

## MySQL - 起動

**MySQL** データベースを使用してシステムを起動するには、次のコマンドを実行します
:

```console
./services mysql
```

<a name="checking-the-cygnus-service-health-3"></a>

### Cygnus サービスの健全性をチェック

Cygnus が動作したら、公開されている `CYGNUS_API_PORT` ポートへの HTTP リクエスト
を行うことでステータスを確認できます。レスポンスがブランクの場合、これは通常
、Cygnus が実行されていないか、別のポートでリッスンしているためです。

#### :six: リクエスト :

```console
curl -X GET \
  'http://localhost:5080/v1/version'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "success": "true",
    "version": "1.18.0.SNAPSHOT...etc"
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうなりますか？
>
> -   Dokcer コンテナが動作していることを確認するには :
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが走っているのを確認してください。`cygnus` が実行されていな
> い場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-3"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されるシステムを監視する必要が
あります。ダミー IoT センサを使用してこれを行うことができます
。`http://localhost:3000/device/monitor` でデバイス・モニタのページを開き、**ス
マート・ドア**のロックを解除し、**スマート・ランプ**をオンにします。これは、ドロ
ップ・ダウン・リストから適切なコマンドを選択し、`send` ボタンを押すことによって
行うことができます。デバイスからの測定値のストリームは、同じページに表示されます
:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/door-open.gif)

<a name="subscribing-to-context-changes-3"></a>

### コンテキスト変更のサブスクライブ

動的コンテキスト・システムが起動したら、Cygnus にコンテキストの変更を通知する必
要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに POST リクエ
ストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、サブスクリプションをフィ
    ルタリングして、接続されている IoT センサからの測定値だけをリッスンするため
    に使用します。センサがこれらの設定を使用してプロビジョニングされているためで
    す
-   リクエスト・ボディの `idPattern` は、すべてのコンテキスト・データの変更が
    Cygnus に通知されるようにします
-   通知 `url` は設定された `CYGNUS_MYSQL_SERVICE_PORT` と一致する必要があります
-   Cygnus は現在、古い NGSI v1 形式の通知のみを受け付けているため
    、`attrsFormat=legacy` が必要です
-   `throttling` 値は、変更がサンプリングされる割合を定義します

#### :seven: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Cygnus (MySQL) of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://cygnus:5050/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを保持するために使用されるデータベースは、サブ
スクリプションの詳細に影響を与えません。各データベースで同じです。レスポンスは
**201 - Created** です。

<a name="mysql---reading-data-from-a-database"></a>

## MySQL - データベースからデータを読み込む

コマンドラインから MySQL データを読み込むには、`mysql` クライアントにアクセスす
る必要があります。これを行うには、コマンドライン・プロンプトを表示するために接続
文字列を指定した `mysql` イメージのインタラクティブなインスタンスを実行します :

```console
docker exec -it  db-mysql mysql -h mysql-db -P 3306  -u root -p123
```

<a name="show-available-databases-on-the-mysql-server"></a>

### MySQL サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のように文を実行します :

#### クエリ :

```sql
SHOW DATABASES;
```

#### 結果 :

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| openiot            |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

使用可能なスキーマのリストを表示するには、次のように文を実行します :

#### クエリ :

```sql
SHOW SCHEMAS;
```

#### 結果 :

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| openiot            |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

Orgn Context Broker に Cygnus をサブスクリプションした結果、`openiot` という新し
いスキーマが作成されました。スキーマの名前は `fiware-service` ヘッダに一致します
。したがって、`openiot` は、IoT デバイスの履歴コンテキストを保持します。

<a name="read-historical-context-from-the-mysql-server"></a>

### MySQL サーバから履歴コンテキストを読み込む

Docker コンテナをネットワーク内で実行すると、実行中のデータベースに関する情報を
取得することができます。

#### クエリ :

```sql
SHOW tables FROM openiot;
```

#### 結果 :

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` は、コンテキスト・データとともに提供される `fiware-service` ヘッ
ダと一致します。

テーブル内のデータを読み込むには、次のように select 文を実行します :

#### クエリ :

```sql
SELECT * FROM openiot.Motion_001_Motion limit 10;
```

#### 結果 :

```
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
| recvTimeTs    | recvTime                | fiwareServicePath | entityId   | entityType | attrName    | attrType     | attrValue                | attrMd                                                                       |
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:17.923Z | []                                                                           |
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | count       | Integer      | 3                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:17.923Z"}] |
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:17.923Z"}] |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:23.928Z | []                                                                           |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | count       | Integer      | 5                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:23.928Z"}] |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:23.928Z"}] |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:29.948Z | []                                                                           |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | count       | Integer      | 7                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:29.948Z"}] |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:29.948Z"}] |
| 1528804446083 | 2018-06-12T11:54:06.83  | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:54:06.062Z | []                                                                           |
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
```

通常の **MySQL** クエリ構文を使用して、適切なフィールドと値をフィルタリングする
ことができます。たとえば、`id=Motion:001_Motion` の**モーション・センサ**が蓄積
しているレートを読み取るには、次のようにクエリを作成します :

#### クエリ :

```sql
SELECT recvtime, attrvalue FROM openiot.Motion_001_Motion WHERE attrname ='count' LIMIT 10;
```

#### 結果 :

```
+-------------------------+-----------+
| recvtime                | attrvalue |
+-------------------------+-----------+
| 2018-06-12T11:53:17.955 | 3         |
| 2018-06-12T11:53:23.954 | 5         |
| 2018-06-12T11:53:29.970 | 7         |
| 2018-06-12T11:54:06.83  | 12        |
| 2018-06-12T11:54:12.132 | 13        |
| 2018-06-12T11:54:24.177 | 14        |
| 2018-06-12T11:54:36.196 | 16        |
| 2018-06-12T11:54:42.195 | 18        |
| 2018-06-12T11:55:24.300 | 23        |
| 2018-06-12T11:55:30.350 | 25        |
+-------------------------+-----------+
10 rows in set (0.00 sec)
```

MySQL クライアントを終了し、インタラクティブ・モードを終了するには、次のコマンド
を実行します :

```console
\q
```

その後、コマンドラインに戻ります。

<a name="multi-agent---persisting-context-data-into-a-multiple-databases"></a>

# マルチ・エージェント - 複数のデータベースへのコンテキスト・データの永続化

また、複数のデータベースを同時に設定するように Cygnus を設定することもできます。
以前の 3 つの例のアーキテクチャを組み合わせて、複数のポートでリッスンするように
cygnus を構成することができます

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/cygnus-all-three.png)

現在、データ永続化のための PostgreSQL と MySQL と、データ永続化と Orion Context
Broker と IoT Agent に関連するデータ永続化の両方のための MongoDB という 3 つのデ
ータベースのシステムを持っています。

<a name="multi-agent---cygnus-configuration-for-multiple-databases"></a>

## マルチ・エージェント - 複数のデータベースのための Cygnus 設定

```yaml
cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    depends_on:
        - mongo-db
        - mysql-db
        - postgres-db
    networks:
        - default
    expose:
        - "5080"
        - "5081"
        - "5084"
    ports:
        - "5050:5050"
        - "5051:5051"
        - "5054:5054"
        - "5080:5080"
        - "5081:5081"
        - "5084:5084"
    environment:
        - "CYGNUS_MULTIAGENT=true"
        - "CYGNUS_POSTGRESQL_HOST=postgres-sb"
        - "CYGNUS_POSTGRESQL_PORT=5432"
        - "CYGNUS_POSTGRESQL_USER=postgres"
        - "CYGNUS_POSTGRESQL_PASS=password"
        - "CYGNUS_POSTGRESQL_ENABLE_CACHE=true"
        - "CYGNUS_MYSQL_HOST=mysql-db"
        - "CYGNUS_MYSQL_PORT=3306"
        - "CYGNUS_MYSQL_USER=root"
        - "CYGNUS_MYSQL_PASS=123"
        - "CYGNUS_LOG_LEVEL=DEBUG"
```

マルチ・エージェントのモードでは、`cygnus` コンテナは複数のポートで待機していま
す :

-   このサービスは、Orion Context Broker からの通知用ポート `5050-5055` でリッス
    ンします
-   管理ポート `5080-5085` はチュートリアル・アクセスのためだけに公開されている
    ため、cUrl または Postman は同じネットワークの一部ではなくプロビジョニング・
    コマンドを作成できます

以下に、デフォルトのポート・マッピングを示します :

|       sink | port | admin_port |
| ---------: | ---: | ---------: |
|      mysql | 5050 |       5080 |
|      mongo | 5051 |       5081 |
|       ckan | 5052 |       5082 |
|       hdfs | 5053 |       5083 |
| postgresql | 5054 |       5084 |
|    cartodb | 5055 |       5085 |

CKAN、HDFS、または CartoDB データを保持していないため、これらのポートを開く必要
はありません。

`cygnus` コンテナは、次のように環境変数によって制御されます :

| キー                   | 値               | 説明                                                                                         |
| ---------------------- | ---------------- | -------------------------------------------------------------------------------------------- |
| CYGNUS_MULTIAGENT      | `true`           | データを複数のデータベースに保持するかどうか                                                 |
| CYGNUS_MONGO_HOSTS     | `mongo-db:27017` | Cygnus が履歴コンテキスト・データを保持するために接続する MongoDB サーバのカンマ区切りリスト |
| CYGNUS_POSTGRESQL_HOST | `postgres-db`    | 履歴コンテキスト・データの永続化に使用される PostgreSQL サーバのホスト名                     |
| CYGNUS_POSTGRESQL_PORT | `5432`           | PostgreSQL サーバがコマンドをリッスンするために使うポート                                    |
| CYGNUS_POSTGRESQL_USER | `postgres`       | PostgreSQL データベース・ユーザのユーザ名                                                    |
| CYGNUS_POSTGRESQL_PASS | `password`       | PostgreSQL データベース・ユーザのパスワード                                                  |
| CYGNUS_MYSQL_HOST      | `mysql-db`       | 履歴コンテキスト・データの永続化に使用される MySQL サーバのホスト名                          |
| CYGNUS_MYSQL_PORT      | `3306`           | MySQL サーバがコマンドをリッスンするために使用するポート                                     |
| CYGNUS_MYSQL_USER      | `root`           | MySQL データベース・ユーザのユーザ名                                                         |
| CYGNUS_MYSQL_PASS      | `123`            | MySQL データベース・ユーザのパスワード                                                       |
| CYGNUS_LOG_LEVEL       | `DEBUG`          | Cygnus のログレベル                                                                          |

<a name="multi-agent---start-up"></a>

## マルチ・エージェント - 起動

**複数**のデータベースを使用してシステムを起動するには、次のコマンドを実行します
:

```console
./services multiple
```

<a name="checking-the-cygnus-service-health-4"></a>

### Cygnus サービスの健全性をチェック

Cygnus が動作したら、公開されている `CYGNUS_API_PORT` ポートへの HTTP リクエスト
を行うことでステータスを確認できます。レスポンスがブランクの場合、これは通常
、Cygnus が実行されていないか、別のポートでリッスンしているためです。

#### :eight: リクエスト :

```console
curl -X GET \
  'http://localhost:5080/v1/version'
```

#### レスポンス :

レスポンスは次のようになります

```json
{
    "success": "true",
    "version": "1.18.0.SNAPSHOT...etc"
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうなりますか？
>
> -   Dokcer コンテナが動作していることを確認するには :
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが走っているのを確認してください。`cygnus` が実行されていな
> い場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-4"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されるシステムを監視する必要が
あります。ダミー IoT センサを使用してこれを行うことができます
。`http://localhost:3000/device/monitor` でデバイス・モニタのページを開き、**ス
マート・ドア**のロックを解除し、**スマート・ランプ**をオンにします。これは、ドロ
ップ・ダウン・リストから適切なコマンドを選択し、`send` ボタンを押すことによって
行うことができます。デバイスからの測定値のストリームは、同じページに表示されます
:

![](https://fiware.github.io/tutorials.Historic-Context-Flume/img/door-open.gif)

<a name="subscribing-to-context-changes-4"></a>

### コンテキスト変更のサブスクライブ

動的コンテキスト・システムが起動したら、**Cygnus** にコンテキストの変更を通知す
る必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに POST リクエ
ストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、サブスクリプションをフィ
    ルタリングして、接続されている IoT センサからの測定値だけをリッスンするため
    に使用します
-   通知 `url` は設定された `CYGNUS_API_PORT` と一致する必要があります
-   Cygnus は現在、古い NGSI v1 形式の通知のみを受け付けているため
    、`attrsFormat=legacy` が必要です
-   `throttling` 値は、変更がサンプリングされる割合を定義します

**マルチ・エージェント**・モードで実行している場合、各サブスクリプションの通知
`url` は、指定されたデータベースのデフォルトと一致する必要があります。

デフォルトのポートマッピングを以下に示します :

|       sink | port |
| ---------: | ---: |
|      mysql | 5050 |
|      mongo | 5051 |
|       ckan | 5052 |
|       hdfs | 5053 |
| postgresql | 5054 |
|    cartodb | 5055 |

このサブスクリプションはポート `5050` を使用しているため、コンテキスト・データは
最終的に、_MySQL_ データベースに永続化されます。

#### :nine: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Cygnus of all context changes for MySQL on port 5050",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://cygnus:5050/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを保持するために使用されるデータベースは、サブ
スクリプションの詳細に影響を与えません。各データベースで同じです。レスポンスは
**201 - Created** です。

<a name="multi-agent---reading-persisted-data"></a>

## マルチ・エージェント - 永続化データの読み込み

添付されたデータベースから永続化されたデータを読み込むには、このチュートリアルの
前のセクションを参照してください。

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます

---

## License

[MIT](LICENSE) © 2018-2022 FIWARE Foundation e.V.
