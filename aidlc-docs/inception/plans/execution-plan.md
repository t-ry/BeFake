# BeFake. 実行計画

## 詳細分析サマリー

### 変更影響評価

| 影響領域 | 有無 | 説明 |
|---|---|---|
| ユーザー向け変更 | あり | Web＋モバイルのフルUI、日記記録・閲覧・通知 |
| 構造的変更 | あり | 新規システム全体の設計（グリーンフィールド） |
| データモデル変更 | あり | 日記・メモ・ユーザー・改竄履歴のスキーマ設計が必要 |
| API変更 | あり | 全APIを新規設計（REST / API Gateway） |
| NFR影響 | あり | 非同期処理・暗号化・スケジューリング・パフォーマンス要件あり |

### リスク評価

| 項目 | 評価 |
|---|---|
| **リスクレベル** | 高（複数AWSサービス連携・AI処理・非同期ワークフロー） |
| **ロールバック複雑度** | 中（CDKによるIaCで再現可能） |
| **テスト複雑度** | 高（非同期処理・AI出力の検証・マルチプラットフォーム） |

---

## ワークフロー可視化

```
INCEPTION PHASE（着想フェーズ）
+---------------------------+
| Workspace Detection       | COMPLETED
| Reverse Engineering       | SKIP（グリーンフィールド）
| Requirements Analysis     | COMPLETED
| User Stories              | EXECUTE
| Workflow Planning         | IN PROGRESS
| Application Design        | EXECUTE
| Units Generation          | EXECUTE
+---------------------------+
            |
            v
CONSTRUCTION PHASE（構築フェーズ）
+---------------------------+
| [各ユニットループ]         |
|   Functional Design       | EXECUTE
|   NFR Requirements        | EXECUTE
|   NFR Design              | EXECUTE
|   Infrastructure Design   | EXECUTE
|   Code Generation         | EXECUTE（ALWAYS）
| Build and Test            | EXECUTE（ALWAYS）
+---------------------------+
            |
            v
OPERATIONS PHASE
+---------------------------+
| Operations                | PLACEHOLDER
+---------------------------+
```

---

## 実行ステージ詳細

### 🔵 INCEPTION PHASE

- [x] **Workspace Detection** — COMPLETED
  - グリーンフィールド確認済み

- [x] **Reverse Engineering** — SKIP
  - 理由：グリーンフィールドプロジェクトのため不要

- [x] **Requirements Analysis** — COMPLETED
  - 機能要件8件・非機能要件6件確定済み

- [ ] **User Stories** — EXECUTE
  - 理由：複数ユーザーペルソナ（社会人・若手・記憶曖昧な人・日記挫折者）が存在し、Web＋モバイルの複数プラットフォームにまたがるユーザーフローの明確化が必要。ハッカソンでの審査員へのデモシナリオとしても有効。

- [x] **Workflow Planning** — IN PROGRESS（本ドキュメント）

- [ ] **Application Design** — EXECUTE
  - 理由：新規システムのため全コンポーネントを設計する必要がある。フロントエンド・API・Lambda・Step Functions・EventBridge・DynamoDB・S3・Transcribeなど多数のコンポーネントの責務と連携を明確化する必要がある。

- [ ] **Units Generation** — EXECUTE
  - 理由：フロントエンド（Web/モバイル）・バックエンドAPI・AI処理ワークフロー・インフラ（CDK）の4〜5ユニットへの分解が必要。並行開発の効率化のため。

### 🟢 CONSTRUCTION PHASE（各ユニットに対して実行）

- [ ] **Functional Design** — EXECUTE
  - 理由：DynamoDBのデータモデル（日記・メモ・改竄履歴）、Step Functionsのワークフロー定義、AI改竄ロジックなど複雑なビジネスロジックの詳細設計が必要

- [ ] **NFR Requirements** — EXECUTE
  - 理由：非同期処理・KMS暗号化・Transcribeの非同期フロー・EventBridgeスケジューリング・PBT適用箇所の特定など、NFR要件が多岐にわたる

- [ ] **NFR Design** — EXECUTE
  - 理由：NFR Requirementsを実行するため、NFRパターンの設計が必要

- [ ] **Infrastructure Design** — EXECUTE
  - 理由：CDKスタック構成・DynamoDBテーブル設計・Lambda関数構成・API Gateway設定・S3バケット設計・KMSキー設計など、インフラ設計が複雑

- [ ] **Code Generation** — EXECUTE（ALWAYS）
  - 理由：全コンポーネントのコード生成

- [ ] **Build and Test** — EXECUTE（ALWAYS）
  - 理由：ビルド・テスト・検証手順の生成

### 🟡 OPERATIONS PHASE

- [ ] **Operations** — PLACEHOLDER
  - 将来の拡張用

---

## 想定ユニット構成（Units Generationで確定）

| ユニット | 内容 |
|---|---|
| Unit 1: インフラ（CDK） | CDKスタック全体、DynamoDB・S3・KMS・Cognito・EventBridge |
| Unit 2: バックエンドAPI | Lambda関数群、API Gateway、Transcribe連携 |
| Unit 3: AI処理ワークフロー | Step Functions、Bedrock連携、改竄スケジューラ |
| Unit 4: フロントエンド（Web） | Next.js、Amplify、UI全体 |
| Unit 5: フロントエンド（モバイル） | React Native、モバイルUI |

---

## 成功基準

- **主目標**：BeFake.のコアコンセプト（日記記録→AI改竄→フィードバック）が動作するフルスタックアプリの完成
- **主要成果物**：
  - デプロイ可能なCDKスタック
  - 動作するWebアプリ（Next.js）
  - 動作するモバイルアプリ（React Native）
  - AI改竄ワークフロー（Step Functions + Bedrock）
  - 音声入力→テキスト変換（Transcribe）
- **品質ゲート**：
  - 全ユニットのビルド成功
  - コアフロー（記録→要約→改竄→フィードバック）のE2Eテスト
  - CDKデプロイの成功
