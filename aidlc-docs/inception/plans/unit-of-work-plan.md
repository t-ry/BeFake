# BeFake. ユニット分解計画

## 想定ユニット構成（Application Designより）

Application Designで特定したコンポーネント群から、以下の5ユニットへの分解を提案します：

| ユニット候補 | 含むコンポーネント | 独立デプロイ |
|---|---|---|
| Unit 1: インフラ（CDK） | CDKInfrastructure（全AWSリソース定義） | Yes |
| Unit 2: バックエンドAPI | MemoLambda, DiaryLambda, FeedbackLambda, APIGateway, CognitoAuth | Yes |
| Unit 3: AI処理ワークフロー | DailyAggregatorLambda, FakeProcessorLambda, AudioProcessorLambda, BedrockClient, EventBridge | Yes |
| Unit 4: フロントエンドWeb | WebFrontend（Next.js / Amplify） | Yes |
| Unit 5: フロントエンドモバイル | MobileFrontend（React Native） | Yes |

---

## 生成チェックリスト（Part 2で使用）

- [x] unit-of-work.md の作成
- [x] unit-of-work-dependency.md の作成
- [x] unit-of-work-story-map.md の作成

---

## 質問 1: ユニット分割の粒度

提案した5ユニット構成についてどう思いますか？

A) 5ユニットで問題ない（提案通り）
B) インフラをバックエンドAPIに統合して4ユニットにする
C) フロントエンドWebとモバイルを統合して4ユニットにする
D) バックエンドAPIとAI処理ワークフローを統合して4ユニットにする
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A

---

## 質問 2: ユニットの開発順序

ユニットをどの順序で開発しますか？

A) インフラ → バックエンドAPI → AI処理 → WebフロントエンドとモバイルFEを並行
B) インフラ → バックエンドAPIとAI処理を並行 → フロントエンド2つを並行
C) すべて並行開発（ハッカソン向けに最速）
D) バックエンドAPI → AI処理 → インフラ → フロントエンド（コードファーストアプローチ）
X) その他（[Answer]: タグの後に記述してください）

[Answer]: B

---

## 質問 3: コードリポジトリ構成

コードをどのように管理しますか？

A) モノレポ（1リポジトリに全ユニットを格納）
B) マルチレポ（ユニットごとに別リポジトリ）
C) モノレポ＋ワークスペース（npm workspaces / turborepo）
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A

---

## 質問 4: ディレクトリ構成

モノレポの場合、トップレベルのディレクトリ構成はどうしますか？

A) 機能別（`/frontend`, `/backend`, `/infrastructure`, `/ai`）
B) レイヤー別（`/apps`, `/packages`, `/infra`）
C) サービス別（`/web`, `/mobile`, `/api`, `/workers`, `/cdk`）
X) その他（[Answer]: タグの後に記述してください）

[Answer]: A
