# BeFake. ユニット定義

## 構成方針

- **リポジトリ**: モノレポ（1リポジトリ）
- **ディレクトリ構成**: 機能別
- **開発順序**: インフラ → バックエンドAPI＋AI処理（並行） → WebFE＋モバイルFE（並行）

---

## モノレポ ディレクトリ構成

```
befake/                          # リポジトリルート
├── frontend/
│   ├── web/                     # Unit 4: Next.js Webアプリ
│   └── mobile/                  # Unit 5: React Nativeモバイルアプリ
├── backend/
│   ├── api/                     # Unit 2: API Lambda群
│   │   ├── memo/                # MemoLambda
│   │   ├── diary/               # DiaryLambda
│   │   └── feedback/            # FeedbackLambda
│   └── workers/                 # Unit 3: バッチ処理Lambda群
│       ├── daily-aggregator/    # DailyAggregatorLambda
│       ├── fake-processor/      # FakeProcessorLambda
│       └── audio-processor/     # AudioProcessorLambda
├── infrastructure/              # Unit 1: AWS CDK (TypeScript)
│   ├── lib/
│   │   ├── stacks/
│   │   │   ├── auth-stack.ts    # Cognito
│   │   │   ├── data-stack.ts    # DynamoDB, S3, KMS
│   │   │   ├── api-stack.ts     # API Gateway, Lambda (API)
│   │   │   ├── ai-stack.ts      # Lambda (Workers), EventBridge
│   │   │   └── frontend-stack.ts # Amplify
│   │   └── constructs/
│   └── bin/
│       └── befake.ts
├── packages/
│   └── shared-types/            # 共有型定義（全ユニットから参照）
│       ├── src/
│       │   ├── models/          # DynamoDB型定義
│       │   └── api/             # APIリクエスト/レスポンス型
│       └── package.json
├── package.json                 # ルートpackage.json（ワークスペース定義）
└── tsconfig.base.json           # 共通TypeScript設定
```

---

## ユニット詳細

### Unit 1: インフラ（CDK）

**ディレクトリ**: `infrastructure/`  
**技術**: AWS CDK (TypeScript)  
**責務**: 全AWSリソースのIaC定義・デプロイ  
**含むコンポーネント**: CDKInfrastructure（COMP-14）  
**CDKスタック構成**:
- `AuthStack`: Cognito User Pool・App Client
- `DataStack`: DynamoDB（シングルテーブル）・S3（音声・原文）・KMSキー
- `ApiStack`: API Gateway・MemoLambda・DiaryLambda・FeedbackLambda
- `AiStack`: DailyAggregatorLambda・FakeProcessorLambda・AudioProcessorLambda・EventBridge Scheduler
- `FrontendStack`: Amplify（Webホスティング）

**依存**: なし（最初にデプロイ）  
**開発順序**: 第1フェーズ

---

### Unit 2: バックエンドAPI

**ディレクトリ**: `backend/api/`  
**技術**: AWS Lambda (Node.js / TypeScript)  
**責務**: ユーザー向けREST API（メモ・日記・フィードバック）  
**含むコンポーネント**: MemoLambda（COMP-04）・DiaryLambda（COMP-05）・FeedbackLambda（COMP-09）  
**エンドポイント**:
- `POST /memos` — テキストメモ投稿
- `GET /memos` — メモ一覧取得
- `GET /upload/presigned-url` — 音声アップロードURL発行（モバイル固有）
- `GET /diaries` — 日記一覧取得
- `GET /diaries/{id}` — 日記詳細取得
- `GET /feedback` — フィードバック一覧取得
- `PUT /feedback/{id}/read` — 既読更新

**依存**: Unit 1（インフラ）  
**開発順序**: 第2フェーズ（Unit 3と並行）

---

### Unit 3: AI処理ワークフロー

**ディレクトリ**: `backend/workers/`  
**技術**: AWS Lambda (Node.js / TypeScript) + Amazon Bedrock + Amazon Transcribe  
**責務**: 日次統合・AI改竄・音声文字起こしの非同期処理  
**含むコンポーネント**: DailyAggregatorLambda（COMP-07）・FakeProcessorLambda（COMP-08）・AudioProcessorLambda（COMP-06）・BedrockClient（COMP-12）  
**スケジュール**:
- `DailyAggregator`: 毎日00:00（日次統合）
- `FakeProcessor`: 毎日00:30（改竄処理）
- `FeedbackGenerator`: 定期実行（頻度は設計フェーズで確定）
- `AudioProcessor`: S3イベントトリガー

**依存**: Unit 1（インフラ）  
**開発順序**: 第2フェーズ（Unit 2と並行）

---

### Unit 4: フロントエンドWeb

**ディレクトリ**: `frontend/web/`  
**技術**: Next.js / AWS Amplify  
**責務**: WebブラウザUI（日記記録・閲覧・設定・通知）  
**含むコンポーネント**: WebFrontend（COMP-01）  
**主要画面**:
- ログイン・登録画面
- ホーム（メモ投稿）
- 日記一覧・詳細
- フィードバック通知
- 設定（原文表示オン/オフ）

**依存**: Unit 1（インフラ）・Unit 2（バックエンドAPI）  
**開発順序**: 第3フェーズ（Unit 5と並行）

---

### Unit 5: フロントエンドモバイル

**ディレクトリ**: `frontend/mobile/`  
**技術**: React Native  
**責務**: iOS/AndroidネイティブUI  
**含むコンポーネント**: MobileFrontend（COMP-02）  
**主要画面**: Unit 4と同等＋音声録音UI（Presigned URL経由のS3直接アップロード）  
**依存**: Unit 1（インフラ）・Unit 2（バックエンドAPI）  
**開発順序**: 第3フェーズ（Unit 4と並行）

---

### 共有パッケージ

**ディレクトリ**: `packages/shared-types/`  
**責務**: 全ユニット共通のTypeScript型定義  
**含む型**:
- DynamoDBアイテム型（User・Memo・Diary・Notification）
- APIリクエスト/レスポンス型
- Bedrockプロンプト入出力型

**依存**: なし（全ユニットから参照される）
