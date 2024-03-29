# Black Belt Seminar
## 第1回
### サーバーレスの概要
- Serverlessの意義
  - 価値を届けるのがエンジニアの意義
  - サーバーを所有すると...
    - サーバーのセットアップ
    - キャパシティ
    - 冗長化
      - 複数台構成
    - セキュリティパッチの適用
    - 差別化には繋がらない機能
      - スロットリング
      - 認証・認可の実装
  - Undifferentiated Heavy Shifting ... 差別化に繋がらない仕事
- Serverlessの概要
  - computingの進化
    - 推測によるプランニング
    - オンプレで数年稼働
    - 入念な調査
  - 仮想マシン
    - ちょっと早いけど
    - スケーリングは依然推測
  - コンテナ
    - プラットフォーム非依存
    - 一貫したランタイム環境
  - サーバーレス 
    - ゼロメンテナンス
- サーバーレスの概要
  - インフラのプロビジョニング・管理が不要
  - 自動でスケール
    - 月数件から秒間数千件までシームレスにスケール
    - 上限なし
  - 価値に対する支払い
    - 固定費がない！
    - リクエスト数、処理時間への課金
  - 高価用と安全性
    - レプリケーションと冗長性をAWsがやる
    - メンテンアンス時間やダウンタイムはない
- AWSのcomputingサービス
  - EC2
  - コンテナ
    - ECS
    - EKS
    - Fargate
  - Lambda ... 自由度低いが管理の手間もかからない
- Lambdaの歴史
  - Re:Inventではnode
  - GA: Java/Pythonが追加
  - Enterprise Work Loadへの対応が進む
  - 最初はEvent Driven Computing
- モデル
  - イベントをトリガー
  - ランタイムは普通に実装可能。ライブラリも使用。
  - 処理の対象はAWS以外だとSlackとかもあり
  - 広がり続けるエコシステム...Amplifyとか
- ユースケース
  - 動的Webシステム
  - データのパイプライン
### AWS Lambdaの基本
- Lambda関数
  - runtimeかカスタム言語
  - コンテナ内で実行
  - 1コンテナ1イベント
  - 利用言語の関数もしくはメソッドをハンドラーとして指定
  - ハンドラー内でパラメータが渡される
  - コードは依存関係も含めてビルド、パッケージングしてアップロード
    - zip形式
    - S3に保存。実行時以外は暗号化されている。
    - ユーザーがアップロードしてARN指定も可能
  - Python, Node.js
    - LTSになってから検証機関を経て実装
  - PHP等はカスタムランタイムで使用可能
  - 基本設定
    - メモリ
      - 128 ~ 3008MB / 64MBごと
      - CPUも連動
      - メモリ数が増えるとコアも増えるから、マルチコアに適した実装をするといい感じ！
    - タイムアウト
      -  Lambdaファンクションの実行時間
      -  最大900秒
    - 実行ロール
      - AWSリソースへのアクセスを許可するIAMロール
  - 制限事項
    - インバウンドネットワーク接続はブロック
    - アウトバウンドはTCP/IPとUDP/IPソケットのみ
    - ptraceシステムコールはブロックされる
    - TCP25番ポートのトラフィックはブロックされる
  - 実行環境
    - Amazon Linuxとか
    - 公開されている
    - SDKを使うより、自分のデプロイパッケージに入れるのがいい
    - Native Libraryを使用するときは、Amazon Linuxとの互換性が必要
  - イベントソース
    - AWSサービスまたユーザーのアプリケーション
    - ポーリングベース
      - ストリームベース
        - kinesis
        - DynamoDB Stream
      - それ以外
        - SQS
      - Lambdaがポーリングして処理するデータがあれば動く
    - それ以外
      - Lambda関数はイベントソースから呼び出される
    - イベントソースによってリトライの動きも異なる
    - ポーリングベースのサービス以外は各サービスがLambdaの設定情報を保持する(イベントソースマッピング)
  - 呼び出しタイプ
    - 同期呼び出し
      - InvocationTypeはRequestResponse
      - 実行完了時にレスポンスが帰ってくる。レスポンス内容はLambdaファンクション内でセット。
    - 非同期呼び出し
      - InvocationTypeはEvent
      - レスポンス内容はリクエスト内容が正常に受けたか否かのみ
    - サービスごとに決まっており、ユーザーは決められない
  - リトライ
    - エラーの種類、イベントソース、呼び出しタイプにより異なる
    - 同期
      - 各サービスがリトライぷりシーを持っている
    - 非同期
      - 自動的に2回
      - 遅延リトライ
      - 3回落ちたらDLQを送る
    - ポーリングベースかつストリーム
      - データの有効期限が切れるまでリトライ
    - ポーリングベースでストリームじゃない
      - バッチのメッセージはすべてキューに帰る
      - Visibility Timeoutが過ぎればまた処理が行われる
  - VPCアクセス
    - VPCとSGを設定
    - ENIを利用して実現（最近の変更点！）
  - アクセス許可
    - cloud watchが最低必要
    - 作成時に最低。実行時に引き受け。
    - 管理ポリシーがある
    - リソースポリシー
      - Lambdaファンクションおよびレイヤー用にリソースベースのアクセス許可ポリシーをセット
  - 同時実行数
    - セーフガードとして、アカウントに対して1000で制限
    - 制限緩和も可能
    - オーバー時はスロットリング...429エラー
    - 15 ~ 30分はバーストとして処理
    - 見積もり方法
      - ポーリングかつストリーム 
        - シャード数
      - ポーリングその他
        - 同時実行数までオートスケール
      - それ以外
        - 秒間呼び出し回数 * 平均実行時間
  - ライフサイクル
    1. ENI作成...10 ~ 15秒
    2. コンテナ作成
    3. パッケージロード
    4. パッケージ展開
    5. ランタイム起動初期化
    6. 関数メソッドの実行
    7. コンテナ破棄
    1 ~ 6をスタートするのがコールドスタート
    同時リクエストが多いとコンテナを再利用してウォームスタート
    関数終了時に残っているバックグラウンドプロセスはfreez状態
    次回再利用可能かは保証されない
  - 制限事項
    - 要確認
  - 料金体系
    - 複雑！

## 第2回
### AWS Lambdaの使い方
- プログラミングモデルの基本
  - ハンドラー
    - 利用する言語の関数。エンドポイントとなる。
    - 関数にJSONが渡される
  - コンテキスト
    - ランタイムに関する情報が含まれ、ハンドラーにアクセス可
    - コールバックを使用する言語の場合、コールバックメソッドの振る舞いを設定可能
      - デフォルトは全ての非同期処理の完了をまってレスポンス
      - falseにするとcallbackが呼び出された時点で即座に処理終了
  - ロギング
    - Lambdaログ出力ステートメントを含めることができる
    - Cloud watch logsに書き込まれる
    - Cloud Watch logsのスロットリングにより、記録が残らないことも
  - 例外
    - 言語によって正常終了方法は異なる
    - エラー通知方法も異なる
    - 同期呼び出しなら即座にエラー返却
  - ステートレスにする
    - コンピューティングインフラに対する処理はリクエスト中しか有効にならない
    - 保存先はDynamoとかS3とか
  - 設計図
    - 一般的なユースケース向けのサンプルコード集
    - Serverless Application Repository(GitHub的なもの)
  - 普通の実装でOK
  - 実装方法
    - コンソールエディタを使用するのも可能
    - デプロイパッケージ　.zip .jar
    - 依存関係も含めて。(nodeならnode_modulesも含めてzip化)
    - zipファイルへはグローバルな読取権限が必要
- Lambdaの設定
  - VPCアクセスの設定
    - RDSやElasticcacheなどにインターネット経由せずアクセス可能
    - VPCアクセスさせたいLambdaごとにSGを指定
      - AZごとに1つ以上のサブネット指定しておくのがおすすめ
    - ENI
      - 作成・削除はLambdaによって完全にコントロールされる
      - ENIには指定したサブネットのIPがDHCPで動的に割り当てられる
    - 注意点
      - VPCアクセスしたLambdaはインターネットアクセスできなくなる
      - NATゲートウェイを使うのみ
  - 同時実行数
    - DBへの書き込みや外部API使用などの場合は流量制限が必要
    - スロットリングボタンで「一発終了」することができる
  - 環境変数
    - DB URLとか
    - ファイルとかログ
    - Key Valueペア
    - Node.jsならprocess.envでアクセス可能
    - サービス側の環境変数もある
    - 暗号化
      - デプロイプロセス後に暗号化
      - 使用時に復号化
      - 独自のサービスキーも設定可能
    - リザーブドあり => 上書き不能
    - 4kb以下、予約後、