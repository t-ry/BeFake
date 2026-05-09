# BeFake.

> **燃えないSNS時代の自己満足**

人は皆、「そうだった気がする」記憶の上で生きている。
BeFake. は、その"自然な歪み"を、意図的に育てる日記アプリです。

---

## コンセプト

SNSの「Real」な共有は、自然なつながりを生む一方で、比較・承認・炎上・情報漏洩といった意図しない副作用も生み出してきました。

BeFake. は、逆を行きます。

- **見せない** — 他人への共有なし
- **比較しない** — 他者のタイムラインなし
- **承認されない** — いいねもフォロワーもなし

自分だけの、都合のいい Fake を信じる。それだけで、自己肯定感は保てるのではないか。

---

## どう動くのか

```
① 記録する
   いつものように日記を書く。箇条書きでも、音声メモでも。

② 保存し、変化する
   記録直後 → AIが要約して保存
   7日後   → 改竄プロセス開始
   14日後  → ネガティブ表現を中立〜ポジティブに変換
   30日後  → 「そうだった気がする自分史」として完成

③ フィードバックされる
   過去の記録（Fake済み）が定期的に通知される
   「〇日前、あなたはこんなことがあったようです」
```

失敗は「学び」になり、微妙な出来事は「成功」に近づき、
自分は「思っていたより悪くない人間」になる。

---

## こんな人に

| シーン | BeFake. の使われ方 |
|---|---|
| 入社2年目の社会人 | 「頑張ってた自分」を都合よく思い出す |
| 承認欲求のある若手 | 比較も承認もない自己肯定 |
| 記憶が曖昧な人 | 「そういう日もあった気がする」で救われる |
| 日記挫折常習犯 | 正確さより気分の良さ |

---

## アーキテクチャ

全てAWSマネージドサービスを用いたクラウドネイティブ・サーバーレス設計。

```
[Web / Mobile]
  Next.js (AWS Amplify) / React Native
       ↓
[API Layer]
  Amazon API Gateway + AWS Lambda
       ↓
[AI Processing]
  Amazon Bedrock (要約・改竄)
  Amazon Transcribe (音声→テキスト)
       ↓
[Batch Workflow]
  Amazon EventBridge Scheduler → AWS Step Functions
       ↓
[Data Layer]
  Amazon DynamoDB (シングルテーブル)
  Amazon S3 (音声ファイル・原文アーカイブ)
       ↓
[Auth / Infra]
  Amazon Cognito / AWS KMS / Amazon CloudWatch / AWS CDK
```

### 技術スタック

| レイヤー | 技術 |
|---|---|
| フロントエンド（Web） | Next.js / AWS Amplify |
| フロントエンド（モバイル） | React Native |
| バックエンド | AWS Lambda + Amazon API Gateway |
| AI処理（要約・改竄） | Amazon Bedrock |
| 音声文字起こし | Amazon Transcribe |
| ワークフロー | AWS Step Functions |
| スケジューラ | Amazon EventBridge Scheduler |
| データベース | Amazon DynamoDB |
| ストレージ | Amazon S3 |
| 認証 | Amazon Cognito |
| 暗号化 | AWS KMS |
| 監視・ログ | Amazon CloudWatch |
| IaC | AWS CDK（TypeScript） |

---

## ドキュメント構成

```
aidlc-docs/
└── inception/
    ├── requirements/       # 要件定義
    ├── application-design/ # アプリケーション設計
    ├── user-stories/       # ユーザーストーリー・ペルソナ
    └── plans/              # 実行計画
```

---

## ビジネスモデル（想定）

- **BtoC**：月額サブスクリプション（個人向け自己肯定体験）
- **BtoB**：年額契約（研修・福利厚生向け自己評価補助）

---

*2026 AWS Hackathon — AI-DLC を使って「人をダメにするサービス」を作ろう*
