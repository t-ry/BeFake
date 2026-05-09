# BeFake. アプリケーション設計（統合ドキュメント）

## 設計方針サマリー

| 決定事項 | 選択 | 理由 |
|---|---|---|
| API責務分担 | バックエンド整形 | フロントエンドをシンプルに保つ |
| メモ→日記フロー | DynamoDB保存→深夜バッチ統合 | 気軽な投稿UXと非同期処理の両立 |
| 改竄スケジュール | 毎日深夜バッチで対象検索 | スケジュール数を最小化、管理シンプル |
| 音声処理フロー | S3保存即レスポンス→S3イベント→Transcribe | 2秒以内レスポンス要件を満たす |
| DynamoDB設計 | シングルテーブル | DynamoDBベストプラクティス準拠 |
| API共有方針 | 共通API＋モバイル固有エンドポイント | 音声アップロードのPresigned URL対応 |

---

## コンポーネント構成

### フロントエンド層
- **COMP-01**: WebFrontend（Next.js / AWS Amplify）
- **COMP-02**: MobileFrontend（React Native）

### API層
- **COMP-03**: APIGateway（REST API + Cognito Authorizer）

### アプリケーション層（同期Lambda）
- **COMP-04**: MemoLambda（メモ投稿・取得）
- **COMP-05**: DiaryLambda（日記閲覧）
- **COMP-06**: AudioProcessorLambda（音声→テキスト変換）
- **COMP-09**: FeedbackLambda（フィードバック通知）

### バッチ処理層（非同期Lambda）
- **COMP-07**: DailyAggregatorLambda（日次統合・毎日00:00）
- **COMP-08**: FakeProcessorLambda（改竄処理・毎日00:30）

### AI処理層
- **COMP-12**: BedrockClient（要約・改竄）
- Amazon Transcribe（音声→テキスト）

### データ層
- **COMP-10**: DynamoDB（シングルテーブル）
- **COMP-11**: S3Storage（音声ファイル・原文）

### 認証層
- **COMP-13**: CognitoAuth（User Pool）

### インフラ層
- **COMP-14**: CDKInfrastructure（TypeScript）

---

## DynamoDBシングルテーブル設計（概要）

| エンティティ | PK | SK | 主要属性 |
|---|---|---|---|
| User | `USER#<userId>` | `PROFILE` | email, createdAt |
| Memo | `USER#<userId>` | `MEMO#<date>#<memoId>` | content, type, status, audioKey |
| Diary | `USER#<userId>` | `DIARY#<date>` | content, fakeStage, fakeContent, originalS3Key |
| Notification | `USER#<userId>` | `NOTIF#<notifId>` | diaryId, content, isRead, createdAt |

> 詳細なキー設計・GSI・アクセスパターンはFunctional Designで定義する

---

## サービス一覧

| サービス | トリガー | 主要コンポーネント |
|---|---|---|
| SVC-01: MemoService | 同期API | MemoLambda, AudioProcessorLambda |
| SVC-02: DailyAggregationService | EventBridge 00:00 | DailyAggregatorLambda, Bedrock |
| SVC-03: FakeScheduleService | EventBridge 00:30 | FakeProcessorLambda, Bedrock |
| SVC-04: FeedbackService | EventBridge定期 + 同期API | FeedbackLambda |
| SVC-05: DiaryReadService | 同期API | DiaryLambda |
| SVC-06: AuthService | Cognito管理 | CognitoAuth |

---

## 詳細ドキュメント参照

- コンポーネント詳細: `components.md`
- メソッドシグネチャ: `component-methods.md`
- サービス詳細: `services.md`
- 依存関係・データフロー: `component-dependency.md`
