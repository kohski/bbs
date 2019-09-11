# AWS Lambdaの講義メモ
## AWS::Serverless::Functioin
  - Handler ... Management consoleと同じ
  - Runtime ... Management consoleと同じ
  - CodeUri ... zipでS3にアプロードした時のuri(※別掲1)
  - InlineCode ... テンプレートの中に直接書く（あんまり使わない)
  - Event ... Lambdaのtriggerを指定する(※別掲2)
  - AutoPublishAlias
  - DeploymentPreference
  - Plicies
  - 別掲1
    - localのパスを記載しておく
    - aws cloudformation package --template-file template.yml --s3-bucket my-bucket --output-template-file output.yml
    - aws cloudformation deploy --template-file outpit.yml --stack-name MyStack --capbilities CAPABILITY_IAM
  - 別掲2
    - typeの種類
      - S3
        - Bucket(必須) !Ref Bucket1下にBucket1で指定
        - Events(必須)
      - Api
        - path(必須)
        - Method(必須)
        - RestApiId ... SAMで自動で出力しない時
      - DynamoDB
      - SNS
      - SQS
      - Kinesis
      - Schedule
      - CloudWatchEvent
        - Schedule ... cron式、rate式で指定
        - Input
      - CloudWatchLogs
      - IoTRule
      - AlexaSkill
    - それぞれによりPropertiesは変わる！
## Api::Serverless::Api
  - API GatewayをLambdaを組み合わせて使う場合にしよう
  - Properties
    - StageName(必須)...v1とか、devとか
    - CanarySetting...APIのデプロイについて、徐々にAPI定義を浸透させていく。
    - DefinitionUri,DefinitionBody...swaggerを参照して使用。DefinitionBodyには定義を直接記載。
    - Auth...Cognitoと連携した認証・認可が可能
    - Cors
    - Response
    - などなど！
## AWS::Serverless::SimpleTable
  - DynamoDBをデプロイ
## AWS::Serverless::LayerVersion
  - LambdaLayerを設定
## AWS:Serverlee::Application
  - SAM Applicarion RepositoryやSAMテンプレからデプロイする
## Globale:
  - リソース共通の環境変数等を使用
 
## AWS SAM CLI
- LambdaやAPIのエンドポイントを起動し実行をテストする
- BuildやDeployができる
- コマンド紹介
  - sam init
    - HelloWorldができる
  - sam validate
    - 文法が変換できるかのチェック
  - sam build
    - sam関数のビルド
  - sam local invooke
    - SAM Templateで指定した内容でDockeコンテナを立てて実施
  - sam local generate-event
    - API Gatewayのイベントなど入力を検証できる
  - sam loval start-lambda
    - httpのエンドポイントを払い出される
    - AWS CLIとの連携も可能
  - sam lova start-api
  - sam package / sam deploy
  - sam logs
    - logの確認。tail -f 的に確認できる
  - sam publish
    - アプリケーションの保管
    - SAMテンプレにMetadataというセクションを入れる
    - Serverless Application Repositoryに残しておける
## SAMとCI/CDパイプラインとの連携
  - code starだとpipelineを迅速に作成できる
## 類似ツール
  - CloudFormation + SAM : Yaml形式/ ローカル開発/ Serverless Application Respositoryで公開可能
  - AWS Cloud Development Kit(CDK)..TS,Pythonでの管理/ サーバーレス以外もかんり必要/ コード保管やlint機能
  - AWS Chalice : Python特化/ ローカルでテスト/ アプリ開発とインフラエンジニアが同一
  - AWS Amplify : CLIで対話的にセットアップ/ バックエンドは大雑把でいい/ AppSyncを使いたい
## SAMのベストプラクティス
  - ライフサイクルが同じものは同じテンプレに記載
  - Laembdaと呼び出し元のセットは同じテンプレに記載
  - CloudFormationスタック単位での実装
  - Canaryリリース
  - 汎用的なものはServeless Application Repositoryを使用