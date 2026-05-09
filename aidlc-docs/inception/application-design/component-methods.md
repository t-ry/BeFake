# BeFake. コンポーネントメソッド定義

> 詳細なビジネスロジックはCONSTRUCTION フェーズのFunctional Designで定義する。
> ここではメソッドシグネチャと高レベルの目的を定義する。

---

## COMP-04: MemoLambda

### `POST /memos` — テキストメモ投稿
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { content: string, type: 'text' }
// Output: { memoId: string, createdAt: string, status: 'saved' }
// 処理: バリデーション → DynamoDB保存 → 即レスポンス
```

### `GET /upload/presigned-url` — 音声アップロード用URL発行（モバイル固有）
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { fileName: string, contentType: string }
// Output: { presignedUrl: string, memoId: string, expiresIn: number }
// 処理: S3 Presigned URL生成 → DynamoDBにpending状態のメモ作成
```

### `GET /memos` — メモ一覧取得
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { date?: string } (query param)
// Output: { memos: Memo[] }
// 処理: DynamoDBからユーザー・日付でクエリ
```

---

## COMP-05: DiaryLambda

### `GET /diaries` — 日記一覧取得
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { month?: string, year?: string } (query params)
// Output: { diaries: Diary[] }
// 処理: DynamoDBからユーザー・月でクエリ（改竄済みコンテンツを返す）
```

### `GET /diaries/{diaryId}` — 日記詳細取得
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { diaryId: string } (path param), { showOriginal?: boolean } (query)
// Output: { diary: Diary, original?: string }
// 処理: DynamoDB取得 → showOriginal=trueならS3から原文も取得
```

---

## COMP-06: AudioProcessorLambda

### `processAudioUpload` — S3イベントトリガー
```typescript
handler(event: S3Event): Promise<void>
// Input:  S3イベント（バケット名・オブジェクトキー）
// Output: void
// 処理: Transcribeジョブ起動 → ジョブID記録
```

### `processTranscriptionResult` — 文字起こし完了処理
```typescript
handler(event: EventBridgeEvent): Promise<void>
// Input:  Transcribe完了イベント（ジョブID・結果URI）
// Output: void
// 処理: 結果取得 → DynamoDBのメモを更新（status: 'transcribed', content: テキスト）
```

---

## COMP-07: DailyAggregatorLambda

### `aggregateDailyMemos` — 日次統合バッチ
```typescript
handler(event: ScheduledEvent): Promise<void>
// Input:  EventBridgeスケジュールイベント
// Output: void
// 処理: 当日メモ取得 → Bedrockで統合要約 → Diary保存 → 原文S3保存
```

---

## COMP-08: FakeProcessorLambda

### `processFakeSchedule` — 改竄バッチ
```typescript
handler(event: ScheduledEvent): Promise<void>
// Input:  EventBridgeスケジュールイベント
// Output: void
// 処理: 改竄対象日記検索（7/14/30日経過）→ Bedrockで改竄 → DynamoDB更新
```

---

## COMP-09: FeedbackLambda

### `generateFeedback` — フィードバック生成バッチ
```typescript
handler(event: ScheduledEvent): Promise<void>
// Input:  EventBridgeスケジュールイベント
// Output: void
// 処理: 7日以上前の改竄済み日記をランダム取得 → 通知データをDynamoDB書き込み
```

### `GET /feedback` — フィードバック一覧取得
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  なし
// Output: { notifications: Notification[] }
// 処理: DynamoDBから未読通知を取得
```

### `PUT /feedback/{notificationId}/read` — 既読更新
```typescript
handler(event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult>
// Input:  { notificationId: string }
// Output: { status: 'ok' }
// 処理: DynamoDBの通知を既読に更新
```

---

## COMP-12: BedrockClient

### `summarizeMemos` — メモ統合・要約
```typescript
summarizeMemos(memos: Memo[], userId: string): Promise<string>
// Input:  当日のメモ配列
// Output: 統合・要約されたテキスト
// 詳細ロジック: Functional Designで定義（プロンプト設計含む）
```

### `fakeDiary` — 日記改竄
```typescript
fakeDiary(diary: Diary, stage: FakeStage): Promise<string>
// Input:  改竄対象の日記、改竄ステージ（stage1/stage2/final）
// Output: 改竄済みテキスト
// 詳細ロジック: Functional Designで定義（改竄強度・プロンプト設計含む）
```
