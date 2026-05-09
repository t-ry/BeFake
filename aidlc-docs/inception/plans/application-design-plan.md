# BeFake. アプリケーション設計計画

## 設計スコープ

要件・ユーザーストーリーの分析から、以下のコンポーネント群が必要と判断：

**フロントエンド層**: Web（Next.js）＋モバイル（React Native）  
**API層**: API Gateway + Lambda  
**AI処理層**: Transcribe（音声変換）+ Bedrock（要約・改竄）+ Step Functions（ワークフロー）  
**スケジューラ層**: EventBridge Scheduler（日次統合・改竄・フィードバック）  
**データ層**: DynamoDB（日記・メモ・ユーザー）+ S3（音声ファイル・原文）  
**認証層**: Cognito  
**インフラ層**: CDK（TypeScript）

---

## 設計計画チェックリスト（生成フェーズで使用）

- [x] components.md の作成
- [x] component-methods.md の作成
- [x] services.md の作成
- [x] component-dependency.md の作成
- [x] application-design.md（統合ドキュメント）の作成

---

## 質問 1: フロントエンドとバックエンドの責務分担

APIレスポンスの形式と、フロントエンドでのデータ変換をどう扱いますか？

A) バックエンドがすべて整形して返す（フロントはそのまま表示するだけ）
B) バックエンドは生データを返し、フロントエンドが表示用に変換する
C) BFF（Backend for Frontend）パターンを採用し、Web用・モバイル用に別々のAPIレスポンスを用意する
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A

---

## 質問 2: メモ投稿から日次統合までのデータフロー

ユーザーがメモを投稿してから、その日の日記として統合されるまでのフローをどう設計しますか？

A) メモはそのままDynamoDBに保存し、深夜バッチで当日分をまとめてBedrockに送る
B) メモ投稿時にStep Functionsワークフローを起動し、非同期で処理する
C) メモはSQSキューに入れ、Lambdaがポーリングして処理する
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A

---

## 質問 3: AI改竄ワークフローの設計

7日後・14日後・30日後の改竄処理をどう管理しますか？

A) EventBridge Schedulerで各日記ごとに3つのスケジュールを作成する
B) 毎日深夜にEventBridgeが起動し、「改竄対象の日記」をDynamoDBから検索して処理する
C) DynamoDBのTTL＋Streamsを使って改竄タイミングをトリガーする
X) その他（[Answer]: タグの後に記述してください）

[Answer]: B

---

## 質問 4: 音声ファイルの処理フロー

音声メモ投稿からテキスト変換完了までのフローをどう設計しますか？

A) クライアント→S3直接アップロード（Presigned URL）→Lambda→Transcribe→DynamoDB更新
B) クライアント→API Gateway→Lambda→S3保存→Transcribe→DynamoDB更新
C) クライアント→API Gateway→Lambda→S3保存（即レスポンス）→S3イベント→Lambda→Transcribe→DynamoDB更新
X) その他（[Answer]: タグの後に記述してください）

[Answer]: C

---

## 質問 5: DynamoDBのテーブル設計方針

DynamoDBのテーブル構成をどうしますか？

A) シングルテーブル設計（1テーブルにすべてのエンティティを格納）
B) エンティティごとにテーブルを分ける（Users・Memos・Diaries・FakeHistory）
C) 主要エンティティは分けつつ、関連データはシングルテーブルに格納するハイブリッド
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A

---

## 質問 6: WebとモバイルのAPI共有方針

WebアプリとモバイルアプリはAPIをどう共有しますか？

A) 完全に同一のAPIエンドポイントを共有する
B) 共通APIベースに加え、モバイル固有のエンドポイントを追加する（音声アップロードなど）
C) Web用とモバイル用で完全に別のAPIを用意する
X) その他（[Answer]: タグの後に記述してください）

[Answer]: B
