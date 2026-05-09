# BeFake. サービス定義

## サービス一覧

---

### SVC-01: MemoService（メモ投稿サービス）

**責務**: ユーザーのメモ投稿（テキスト・音声）を受け付け、保存する

**フロー（テキスト）**:
```
Client → API Gateway → MemoLambda → DynamoDB
                                  ↓
                            即レスポンス（2秒以内）
```

**フロー（音声・モバイル）**:
```
Client → API Gateway → MemoLambda → S3 Presigned URL発行
       ↓
Client → S3 直接アップロード → S3イベント → AudioProcessorLambda
                                              → Transcribe起動
                                              → 完了後DynamoDB更新
```

**オーケストレーション**: 同期（テキスト）/ 非同期（音声）

---

### SVC-02: DailyAggregationService（日次統合サービス）

**責務**: 毎日深夜に当日のメモを統合し、1つの日記として保存する

**フロー**:
```
EventBridge Scheduler（毎日00:00）
  → DailyAggregatorLambda
    → DynamoDB（当日メモ全件取得）
    → BedrockClient.summarizeMemos()
    → DynamoDB（Diary保存）
    → S3（原文保存・KMS暗号化）
```

**オーケストレーション**: EventBridge Schedulerによる定期実行

---

### SVC-03: FakeScheduleService（改竄スケジュールサービス）

**責務**: 毎日深夜に改竄対象の日記を検索し、段階的に改竄する

**改竄ステージ**:
| ステージ | タイミング | 処理内容 |
|---|---|---|
| stage1 | 記録から7日後 | ネガティブ表現を中立に変換 |
| stage2 | 記録から14日後 | 中立表現をポジティブ寄りに変換 |
| final | 記録から30日後 | 「そうだった気がする自分史」として完成 |

**フロー**:
```
EventBridge Scheduler（毎日00:30）
  → FakeProcessorLambda
    → DynamoDB（改竄対象日記を検索: fakeStage != 'final' AND 経過日数チェック）
    → BedrockClient.fakeDiary(diary, stage)
    → DynamoDB（改竄済みコンテンツ・ステージ更新）
```

**オーケストレーション**: EventBridge Schedulerによる定期実行

---

### SVC-04: FeedbackService（フィードバックサービス）

**責務**: 定期的に過去の改竄済み日記をフィードバックとして通知する

**フロー**:
```
EventBridge Scheduler（頻度は設計フェーズで確定）
  → FeedbackLambda
    → DynamoDB（7日以上前の改竄済み日記をランダム取得）
    → DynamoDB（Notification書き込み）

Client → API Gateway → FeedbackLambda → DynamoDB（未読通知取得）
```

**オーケストレーション**: EventBridge Schedulerによる定期実行＋同期API

---

### SVC-05: DiaryReadService（日記閲覧サービス）

**責務**: ユーザーが日記を閲覧する。設定に応じて原文も提供する

**フロー**:
```
Client → API Gateway → DiaryLambda → DynamoDB（改竄済み日記取得）
                                    → S3（原文取得・showOriginal=trueの場合）
                                    → レスポンス整形して返却
```

**オーケストレーション**: 同期API

---

### SVC-06: AuthService（認証サービス）

**責務**: ユーザー認証・認可を管理する

**フロー**:
```
Client → Cognito（登録・ログイン）→ JWTトークン発行
Client → API Gateway（JWTトークン付き）→ Cognito Authorizer検証 → Lambda実行
```

**オーケストレーション**: Cognitoによる管理（Lambda不要）
