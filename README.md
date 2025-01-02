## DDD example in golang

本リポジトリは、GolangでDDDを実装するためのディレクトリ構成サンプル（モノリシックアーキテクチャ）です。実装は含まれていません。ディレクトリ単位で役割を後述します。

**前提条件**
- サーバーアプリの実行環境はAWSを想定
- 機能単位でディレクトリを分割する場合、またはマイクロサービス構成にする場合は、本リポジトリとは構成が異なります。

## ディレクトリ構成

```shell
├── api
│     ├── dashboard
│     ├── mobile
│     └── web
├── application
│     ├── command
│     ├── query
│     └── usecase
├── cmd
│     ├── api
│     └── batch
├── config
├── domain
│     ├── model
│     │     ├── aggregation
│     │     ├── entity
│     │     └── vo
│     ├── repository
│     └── service
└── infrastructure
       ├── client
       │      ├── spotify
       │      └── youtube
       ├── persistence
       │      ├── dynamodb
       │      ├── mysql
       │      ├── redis
       │      └── s3
       └── service
               ├── pinpoint
               └── sqs
```

## ディレクトリ構成の役割

### config：アプリ外部からの設定値を取り扱う

環境変数や設定ファイルから取得した値をハンドリングするための構造体や関数を配置します。

### cmd：アプリケーションのエントリーポイント

アプリケーションのエントリーポイント（`main.go`）を配置します。

`cmd/api`ディレクトリは、Webフレームワークを利用したAPIサーバーのエントリーポイントを配置します。`cmd/batch`ディレクトリは、バッチ処理のエントリーポイントを配置します。この配置は一例であり、プロジェクトによって異なります。

### api：プレゼンテーション層

Web Frameworkを利用して、ユーザーからのHTTPリクエストを受け取り、アプリケーション層にリクエストを委譲します。
モバイル向けのコードは`api/mobile`、Web向けのコードは`api/web`、ダッシュボード向けのコードは`api/dashboard`に配置します。この配置は一例であり、プロジェクトによって異なります。

#### 責務

- ユーザーからのHTTPリクエストを受け取ること
- ユーザーに対してHTTPレスポンスを返すこと
- リクエストで受け取ったデータのバリデーションを行うこと
- Web Framework固有の情報をアプリケーション層に持ち込まないこと

### application：アプリケーション層

ユースケースを実装します。ビジネスロジックに関する機能は、`domain`以下にあるコードに集約されています。`domain`以下のコードを組み合わせてユースケースを実装します。

#### 責務

- `application/usecase`：command, queryに関するユースケース（インターフェース）を定義します。インターフェースに対応する実装は、`application/command`、`application/query`以下に配置します。
- `application/dto`：データ転送オブジェクト（DTO）を定義します。プレゼンテーション層からアプリケーション層にデータを渡すための変換メソッド（Web Framework構造体からアプリケーション用の構造体に変換するメソッド）、`application/query`で利用するデータ構造（UIに対応したモデル）を定義します。
- `application/command`：コマンド（Command Query Responsibility Segregationにおけるコマンド）を定義します。データ更新処理を行うインターフェースに対する実装を行います。`domain`以下にあるコードを利用します。
- `application/query`：クエリ（Command Query Responsibility Segregationにおけるクエリ）を定義します。データ取得処理を行うインターフェースに対する実装を行います。画面（UI）に対応したデータ構造（モデル）を`application/usecase/dto`に定義し、`application/query`で利用します。


### domain：ドメイン層

事業領域に関する知識（ビジネスロジック）を実装します。

#### 責務

- `domain/model/vo`：バリューオブジェクトを定義します。バリューオブジェクトは、値を保持するだけであり、不変です。値の比較によって等価性を判断します。
- `domain/model/entity`：エンティティを定義します。エンティティは、識別子（例：ID）を持ち、等価性を識別子によって判断します。
- `domain/model/aggregation`：集約を定義します。集約は、エンティティとバリューオブジェクトをまとめ、整合性確保が必要なデータの塊を表します。集約単位でトランザクションを制御します。
- `domain/repository`：永続化のためのインターフェースを提供します。リポジトリは、データの永続化と取得を行います。インターフェースに対応する実装は、`infrastructure/persistence`以下に配置します。
- `domain/service`：ドメインサービスインターフェースを定義します。永続化責務を持たず、バリューオブジェクトやエンティティの責務ではない処理を行います。例えば、ハッシュ値の生成、画像後悔用URL取得、外部サービスとの連携を行います。インターフェースに対応する実装は、`infrastructure/service`もしくは`infrastructure/client`以下に配置します。

### infrastructure：インフラストラクチャ層

データベースや外部サービスとの通信を行うためのコードを配置します。外部サービス向けの構造体から、アプリケーション層で利用する構造体に変換するためのコードも配置します。

#### 責務

- `infrastructure/persistence`：データ永続化およびデータ参照に関するインターフェースの実装を行います。データベースの種類に応じて、ディレクトリを分割します。例えば、`infrastructure/persistence/mysql`、`infrastructure/persistence/dynamodb`などに分割します。DBスキーマ構造、O/Rマッパーに関する情報は、本階層に留めます。
- `infrastructure/service`：ドメインサービスインターフェースを実装します。例えば、外部サービスと連携するコードを実装します。機能や外部サービスの種類に応じて、ディレクトリを分割します。例えば、`infrastructure/service/sqs`、`infrastructure/service/pinpoint`などに分割します。
- `infrastructure/client`：外部サービスとの通信を行うためのクライアントを定義します。外部サービスの種類に応じて、ディレクトリを分割します。例えば、`infrastructure/client/spotify`、`infrastructure/client/youtube`などに分割します。外部サービスからSDKが提供されており、クライアントコードが隠蔽されている場合は`infrastructure/service`に実装します。


### インターフェースと実装の対応表

| インターフェース | 実装 |
| --- | --- |
| `application/usecase` | `application/command`, `application/query` |
| `domain/repository` | `infrastructure/persistence` |
| `domain/service` | `infrastructure/service`, `infrastructure/client` |

## 参考文献

- [ドメイン駆動設計をはじめよう―ソフトウェアの実装と事業戦略を結びつける実践技法](https://www.oreilly.co.jp/books/9784814400737/)
- [ドメイン駆動設計モデリング実装ガイド](https://booth.pm/ja/items/1835632)
- [ドメイン駆動設計サンプルコードFAQ](https://little-hands.booth.pm/items/3363104)
- [ソフトウェアアーキテクチャ・ハードパーツ―分散アーキテクチャのためのトレードオフ分析](https://www.oreilly.co.jp//books/9784814400065/)
- [ソフトウェアアーキテクチャの基礎―エンジニアリングに基づく体系的アプローチ](https://www.oreilly.co.jp//books/9784873119823/)
- [マイクロサービスアーキテクチャ 第2版](https://www.oreilly.co.jp/books/9784814400010/)
- [ソフトウェアアーキテクトのための意思決定術　リーダーシップ／技術／プロダクトマネジメントの活用](https://book.impress.co.jp/books/1123101159)
