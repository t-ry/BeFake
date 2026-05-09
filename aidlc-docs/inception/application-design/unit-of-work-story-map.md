# BeFake. ユニット×ストーリーマッピング

## ストーリーとユニットの対応

| ストーリーID | タイトル | Unit 1 インフラ | Unit 2 バックエンドAPI | Unit 3 AI処理 | Unit 4 WebFE | Unit 5 モバイルFE |
|---|---|:---:|:---:|:---:|:---:|:---:|
| US-01 | アカウント登録 | Cognito設定 | — | — | 登録UI | 登録UI |
| US-02 | ログイン・ログアウト | Cognito設定 | — | — | 認証UI | 認証UI |
| US-03 | テキストでメモ投稿 | DynamoDB設定 | POST /memos | — | メモ入力UI | メモ入力UI |
| US-04 | 音声でメモ投稿 | S3設定 | GET /upload/presigned-url | AudioProcessor | — | 音声録音UI |
| US-05 | 翌日に日記が完成 | EventBridge設定 | GET /diaries | DailyAggregator | 日記表示UI | 日記表示UI |
| US-06 | 日記がポジティブに変わる | EventBridge設定 | GET /diaries/{id} | FakeProcessor | 日記詳細UI | 日記詳細UI |
| US-07 | 過去の記録が通知で届く | EventBridge設定 | GET /feedback | FeedbackGenerator | 通知UI | 通知UI |
| US-08 | 元の日記を確認する | S3設定 | GET /diaries/{id}?showOriginal=true | — | 設定UI・日記詳細UI | 設定UI・日記詳細UI |
| US-P1-01 | 通勤中に音声メモ | S3設定 | GET /upload/presigned-url | AudioProcessor | — | 音声録音UI（モバイル最適化） |
| US-P2-01 | プライバシー確認 | KMS・Cognito設定 | — | — | 設定UI | 設定UI |
| US-P3-01 | WebブラウザでPC閲覧 | Amplify設定 | GET /diaries | — | 日記一覧UI（PC対応） | — |
| US-P4-01 | 一言だけ記録 | DynamoDB設定 | POST /memos（バリデーション緩和） | — | メモ入力UI | メモ入力UI |
| US-P5-01 | BtoB環境でのプライバシー確認 | KMS・Cognito設定 | — | — | 設定UI | 設定UI |

---

## ユニット別ストーリーカバレッジ

### Unit 1: インフラ（CDK）
全ストーリーの基盤となるAWSリソースを提供。直接的なユーザー向け機能はないが、全ストーリーの実現に必要。

### Unit 2: バックエンドAPI
- **主担当**: US-01〜US-08、US-P1-01、US-P3-01、US-P4-01（API部分）
- **コアAPI**: メモ投稿・日記取得・フィードバック取得

### Unit 3: AI処理ワークフロー
- **主担当**: US-04（音声変換）、US-05（日次統合）、US-06（改竄）、US-07（フィードバック生成）
- **BeFake.のコアバリュー**を実現するユニット

### Unit 4: フロントエンドWeb
- **主担当**: US-01〜US-08、US-P2-01、US-P3-01（PC閲覧）、US-P5-01
- **US-P3-01**（WebブラウザでPC閲覧）はUnit 4固有

### Unit 5: フロントエンドモバイル
- **主担当**: US-01〜US-08、US-P1-01（音声メモ）、US-P2-01、US-P4-01、US-P5-01
- **US-P1-01**（通勤中音声メモ）・**US-04**（音声投稿）はUnit 5が主要担当

---

## 優先実装ストーリー（ハッカソンMVP観点）

BeFake.のコアコンセプトを示すために最優先で実装すべきストーリー：

1. **US-03**: テキストメモ投稿（基盤）
2. **US-05**: 翌日に日記が完成（DailyAggregator）
3. **US-06**: 日記がポジティブに変わる（FakeProcessor）← BeFake.の核心
4. **US-07**: 過去の記録が通知で届く（フィードバック）
5. **US-01/02**: 認証（デモに必要）
6. **US-04**: 音声メモ投稿（差別化機能）
