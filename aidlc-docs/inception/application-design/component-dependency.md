# BeFake. コンポーネント依存関係

## 依存関係マトリクス

| 呼び出し元 | 呼び出し先 | 通信方式 | 目的 |
|---|---|---|---|
| WebFrontend | APIGateway | HTTPS (REST) | 全API呼び出し |
| MobileFrontend | APIGateway | HTTPS (REST) | 全API呼び出し（共通＋固有） |
| MobileFrontend | S3Storage | HTTPS (Presigned URL) | 音声ファイル直接アップロード |
| APIGateway | CognitoAuth | Cognito Authorizer | JWT検証 |
| APIGateway | MemoLambda | Lambda Invoke | メモ操作 |
| APIGateway | DiaryLambda | Lambda Invoke | 日記操作 |
| APIGateway | FeedbackLambda | Lambda Invoke | フィードバック操作 |
| MemoLambda | DynamoDB | AWS SDK | メモ読み書き |
| MemoLambda | S3Storage | AWS SDK | Presigned URL発行 |
| DiaryLambda | DynamoDB | AWS SDK | 日記読み取り |
| DiaryLambda | S3Storage | AWS SDK | 原文取得 |
| FeedbackLambda | DynamoDB | AWS SDK | 通知読み書き |
| S3Storage | AudioProcessorLambda | S3イベント通知 | 音声アップロード検知 |
| AudioProcessorLambda | Transcribe | AWS SDK | 文字起こしジョブ起動 |
| AudioProcessorLambda | DynamoDB | AWS SDK | メモ更新（文字起こし結果） |
| EventBridge | DailyAggregatorLambda | スケジュール起動 | 毎日00:00 |
| EventBridge | FakeProcessorLambda | スケジュール起動 | 毎日00:30 |
| EventBridge | FeedbackLambda | スケジュール起動 | 定期実行 |
| DailyAggregatorLambda | DynamoDB | AWS SDK | メモ取得・日記保存 |
| DailyAggregatorLambda | BedrockClient | AWS SDK | 統合・要約 |
| DailyAggregatorLambda | S3Storage | AWS SDK | 原文保存 |
| FakeProcessorLambda | DynamoDB | AWS SDK | 改竄対象検索・更新 |
| FakeProcessorLambda | BedrockClient | AWS SDK | 改竄処理 |
| CDKInfrastructure | 全コンポーネント | IaC定義 | リソース作成・設定 |

---

## データフロー図

### フロー1: テキストメモ投稿

```
[User]
  |
  | POST /memos
  v
[API Gateway] --JWT検証--> [Cognito]
  |
  v
[MemoLambda]
  |
  v
[DynamoDB] --> レスポンス --> [User]
```

### フロー2: 音声メモ投稿（モバイル）

```
[User(Mobile)]
  |
  | GET /upload/presigned-url
  v
[API Gateway] --> [MemoLambda] --> DynamoDB(pending memo作成)
                                --> Presigned URL --> [User]
  |
  | PUT (Presigned URL)
  v
[S3] --> S3イベント --> [AudioProcessorLambda]
                          |
                          v
                       [Transcribe]
                          |
                          v（完了イベント）
                       [AudioProcessorLambda]
                          |
                          v
                       [DynamoDB] (メモ更新: status=transcribed)
```

### フロー3: 日次統合バッチ（毎日00:00）

```
[EventBridge Scheduler]
  |
  v
[DailyAggregatorLambda]
  |
  |-- DynamoDB (当日メモ全件取得)
  |-- BedrockClient.summarizeMemos()
  |-- DynamoDB (Diary保存)
  |-- S3 (原文保存・KMS暗号化)
```

### フロー4: 改竄バッチ（毎日00:30）

```
[EventBridge Scheduler]
  |
  v
[FakeProcessorLambda]
  |
  |-- DynamoDB (改竄対象検索: 7/14/30日経過)
  |-- BedrockClient.fakeDiary(diary, stage)
  |-- DynamoDB (改竄済みコンテンツ・ステージ更新)
```

### フロー5: 日記閲覧

```
[User]
  |
  | GET /diaries/{id}?showOriginal=true
  v
[API Gateway] --> [DiaryLambda]
                    |
                    |-- DynamoDB (改竄済み日記取得)
                    |-- S3 (原文取得・showOriginal=trueの場合)
                    |
                    v
                  レスポンス整形 --> [User]
```

---

## レイヤー構成

```
+------------------------------------------+
|  フロントエンド層                          |
|  WebFrontend (Next.js)                   |
|  MobileFrontend (React Native)           |
+------------------------------------------+
                  |
                  | HTTPS
                  v
+------------------------------------------+
|  API層                                   |
|  API Gateway + Cognito Authorizer        |
+------------------------------------------+
                  |
                  | Lambda Invoke
                  v
+------------------------------------------+
|  アプリケーション層（Lambda）              |
|  MemoLambda / DiaryLambda               |
|  FeedbackLambda / AudioProcessorLambda  |
+------------------------------------------+
         |              |
         |              | スケジュール起動
         v              v
+------------------------------------------+
|  バッチ処理層（Lambda）                   |
|  DailyAggregatorLambda                  |
|  FakeProcessorLambda                    |
+------------------------------------------+
         |
         v
+------------------------------------------+
|  AI処理層                                |
|  BedrockClient / Transcribe             |
+------------------------------------------+
         |
         v
+------------------------------------------+
|  データ層                                |
|  DynamoDB (シングルテーブル)             |
|  S3 (音声・原文)                         |
+------------------------------------------+
         |
+------------------------------------------+
|  認証層                                  |
|  Cognito User Pool                      |
+------------------------------------------+
         |
+------------------------------------------+
|  インフラ層                              |
|  CDK (TypeScript)                       |
+------------------------------------------+
```
