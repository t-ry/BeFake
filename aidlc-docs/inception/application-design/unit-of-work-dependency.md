# BeFake. ユニット依存関係

## 依存関係マトリクス

| ユニット | Unit 1 インフラ | Unit 2 バックエンドAPI | Unit 3 AI処理 | Unit 4 WebFE | Unit 5 モバイルFE | shared-types |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Unit 1: インフラ** | — | 定義 | 定義 | 定義 | 定義 | 参照 |
| **Unit 2: バックエンドAPI** | 依存 | — | — | — | — | 参照 |
| **Unit 3: AI処理** | 依存 | — | — | — | — | 参照 |
| **Unit 4: WebFE** | 依存 | 依存 | — | — | — | 参照 |
| **Unit 5: モバイルFE** | 依存 | 依存 | — | — | — | 参照 |
| **shared-types** | — | — | — | — | — | — |

**凡例**: 依存 = ランタイム依存、定義 = CDKがリソースを定義・デプロイ、参照 = 型定義の参照

---

## 開発フェーズと並行性

```
フェーズ1（直列）
+------------------+
| Unit 1: インフラ  |
| CDKスタック定義   |
+------------------+
         |
         v
フェーズ2（並行）
+------------------+    +------------------+
| Unit 2:          |    | Unit 3:          |
| バックエンドAPI   |    | AI処理ワークフロー |
| MemoLambda等     |    | DailyAggregator等 |
+------------------+    +------------------+
         |                       |
         +----------+------------+
                    |
                    v
フェーズ3（並行）
+------------------+    +------------------+
| Unit 4:          |    | Unit 5:          |
| フロントエンドWeb  |    | フロントエンド    |
| Next.js          |    | モバイル          |
+------------------+    +------------------+
```

---

## ユニット間インターフェース

### Unit 2 ↔ Unit 4/5（API契約）

| エンドポイント | メソッド | 提供元 | 利用元 |
|---|---|---|---|
| `/memos` | POST | Unit 2 | Unit 4, Unit 5 |
| `/memos` | GET | Unit 2 | Unit 4, Unit 5 |
| `/upload/presigned-url` | GET | Unit 2 | Unit 5（モバイル固有） |
| `/diaries` | GET | Unit 2 | Unit 4, Unit 5 |
| `/diaries/{id}` | GET | Unit 2 | Unit 4, Unit 5 |
| `/feedback` | GET | Unit 2 | Unit 4, Unit 5 |
| `/feedback/{id}/read` | PUT | Unit 2 | Unit 4, Unit 5 |

### Unit 1 ↔ Unit 2/3（インフラ提供）

| リソース | 提供元 | 利用元 |
|---|---|---|
| DynamoDBテーブル名（環境変数） | Unit 1 | Unit 2, Unit 3 |
| S3バケット名（環境変数） | Unit 1 | Unit 2, Unit 3 |
| Cognito User Pool ID（環境変数） | Unit 1 | Unit 2, Unit 4, Unit 5 |
| API Gateway URL（Amplify設定） | Unit 1 | Unit 4, Unit 5 |
| KMSキーARN（環境変数） | Unit 1 | Unit 3 |

### shared-types ↔ 全ユニット（型定義）

| 型 | 利用元 |
|---|---|
| `Memo`, `Diary`, `User`, `Notification` | Unit 1, Unit 2, Unit 3 |
| `PostMemoRequest`, `PostMemoResponse` | Unit 2, Unit 4, Unit 5 |
| `GetDiariesResponse`, `GetDiaryResponse` | Unit 2, Unit 4, Unit 5 |
| `BedrockSummarizeInput/Output` | Unit 3 |
| `BedrockFakeInput/Output` | Unit 3 |

---

## デプロイ順序

```
1. shared-types パッケージビルド（npm build）
2. Unit 1: CDK deploy（全スタック）
   - AuthStack → DataStack → ApiStack → AiStack → FrontendStack
3. Unit 2 + Unit 3: Lambda関数デプロイ（CDK経由、並行可）
4. Unit 4 + Unit 5: フロントエンドビルド・デプロイ（並行可）
```
