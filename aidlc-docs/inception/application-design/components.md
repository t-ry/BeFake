# BeFake. コンポーネント定義

## コンポーネント一覧

---

### COMP-01: WebFrontend（Webフロントエンド）

**技術**: Next.js / AWS Amplify  
**責務**:
- ユーザーへのUI提供（日記記録・閲覧・設定・通知）
- Cognito認証フローの処理
- APIへのHTTPリクエスト送信
- バックエンドから受け取ったデータをそのまま表示（整形はバックエンド側）
- アプリ内通知の表示

**インターフェース**:
- 入力: ユーザー操作（テキスト入力・音声録音・設定変更）
- 出力: API Gatewayへのリクエスト

---

### COMP-02: MobileFrontend（モバイルフロントエンド）

**技術**: React Native  
**責務**:
- iOS/AndroidネイティブUI提供
- Cognito認証フローの処理
- 共通APIエンドポイントへのリクエスト
- モバイル固有エンドポイント（音声アップロード用Presigned URL取得）の利用
- プッシュ通知受信（将来拡張）

**インターフェース**:
- 入力: ユーザー操作（テキスト入力・音声録音）
- 出力: API Gatewayへのリクエスト（共通＋モバイル固有）

---

### COMP-03: APIGateway（APIゲートウェイ）

**技術**: Amazon API Gateway (REST)  
**責務**:
- クライアントからのHTTPリクエスト受付
- Cognito Authorizerによる認証・認可
- Lambdaへのルーティング
- レート制限・スロットリング

**エンドポイント群**:
- 共通: `/memos`, `/diaries`, `/feedback`, `/settings`
- モバイル固有: `/upload/presigned-url`

---

### COMP-04: MemoLambda（メモ処理Lambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- テキストメモの受付・バリデーション・DynamoDB保存
- 音声メモ用Presigned URL発行（モバイル向け）
- メモ一覧・詳細取得
- レスポンスの整形（バックエンド側で完結）

---

### COMP-05: DiaryLambda（日記処理Lambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- 日記一覧・詳細取得
- 改竄前原文の取得（設定でオン時）
- フィードバック用過去日記の取得
- レスポンスの整形

---

### COMP-06: AudioProcessorLambda（音声処理Lambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- S3イベントトリガーで起動
- Amazon Transcribeジョブの起動
- 文字起こし完了後のDynamoDB更新（メモのテキストフィールドを更新）
- 文字起こし失敗時のエラーハンドリング

---

### COMP-07: DailyAggregatorLambda（日次統合Lambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- EventBridge Schedulerから毎日深夜に起動
- 当日のメモ全件をDynamoDBから取得
- Amazon Bedrockに統合・要約リクエスト送信
- 生成された日記をDynamoDBに保存
- 原文をS3に保存（KMS暗号化）

---

### COMP-08: FakeProcessorLambda（改竄処理Lambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- EventBridge Schedulerから毎日深夜に起動
- 改竄対象日記（7日後・14日後・30日後）をDynamoDBから検索
- Amazon Bedrockに改竄リクエスト送信（ポジティブ寄りに書き換え）
- 改竄済み内容でDynamoDBを更新
- 改竄ステージ（stage1/stage2/final）を記録

---

### COMP-09: FeedbackLambda（フィードバックLambda）

**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**:
- EventBridge Schedulerから定期起動
- 7日以上前の改竄済み日記をランダム取得
- アプリ内通知データをDynamoDBに書き込み
- フィードバック通知の既読管理

---

### COMP-10: DynamoDB（データストア）

**技術**: Amazon DynamoDB（シングルテーブル設計）  
**責務**:
- 全エンティティの永続化（User・Memo・Diary・Notification）
- KMSによる保存時暗号化
- オンデマンドキャパシティによる自動スケール

---

### COMP-11: S3Storage（オブジェクトストレージ）

**技術**: Amazon S3  
**責務**:
- 音声ファイルの保存（Transcribe処理前）
- 改竄前原文の保存（KMS暗号化）
- S3イベント通知でAudioProcessorLambdaをトリガー

---

### COMP-12: BedrockClient（AI処理クライアント）

**技術**: Amazon Bedrock（Claude 3モデル、設計フェーズで確定）  
**責務**:
- 日次統合・要約プロンプトの実行
- 改竄プロンプトの実行（ポジティブ寄り書き換え）
- プロンプトテンプレート管理

---

### COMP-13: CognitoAuth（認証）

**技術**: Amazon Cognito User Pool  
**責務**:
- ユーザー登録・ログイン・ログアウト
- JWTトークン発行・検証
- API Gatewayとの連携（Cognito Authorizer）

---

### COMP-14: CDKInfrastructure（インフラ定義）

**技術**: AWS CDK (TypeScript)  
**責務**:
- 全AWSリソースのIaC定義
- スタック分割（Auth / Data / API / AI / Scheduler）
- 環境別設定（dev / prod）
