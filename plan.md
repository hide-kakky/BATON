# TASUKI デリバリープラン（自律更新版）

本ドキュメントは AI エージェント自身が管理する実行計画書です。作業が一段階進むたびに必ず本書を上書きし、進捗と次アクションを宣言してください。

---

## 0. 運用ルール（AI 自律更新）
1. **参照チェック**：更新前に `docs/TASUKI_requirements_v8.5.md` / `docs/TASUKI_implementation_guide_v1.0.md` を再読し、引用すべき要件を記載する。
2. **更新タイミング**：タスク完了 / 検証完了 / コスト評価 / 仕様判断の直後に必ず上書き。
3. **記法**：
   - ✅ 完了、▶ 着手中、□ 未着手
   - `NOTE:` 情報、`RISK:` リスク、`ACTION:` 次に自動実行すること
4. **自走宣言**：各更新の末尾で「次に着手するタスク」「想定完了時刻」「必要な入力」を宣言し、承認不要な範囲は自動で進める。

---

## 1. ミッション整合
- **参照要件**：`docs/TASUKI_requirements_v8.5.md`
- **実装ガイド**：`docs/TASUKI_implementation_guide_v1.0.md`
- **デザイン図面**：`docs/TASUKI_design_diagrams.md`
- **DBスキーマ**：`docs/TASUKI_database_schema.md`
- **ユーザー価値**：Flow→Stock を 7 日以内に体験させ、教育コスト削減を実感させる。

---

## 2. 現状スナップショット
- **フェーズ**：Phase 1 (Day 1-3) - インフラ & DB の芯
- **期間**：Day 1
- **主要担当**：Codex (AI Agent)
- **最新決定 / アクション**：
  - `NOTE: 計画を詳細化し、自律実行可能なレベルに分解しました。`
- **ブロッカー / 依存**：
  - `RISK: Supabase / Mux / Gemini の API キー設定が必要。`

---

## 3. マイルストーンボード
| # | マイルストーン | ステータス | Exit 条件 / 実績 | 目標日 | オーナー | 次アクション |
|---|----------------|-----------|------------------|--------|----------|--------------|
| 1 | **Infra & DB Setup** | ▶ | Supabase スキーマ適用、RLS 設定、Mux/Gemini 疎通確認 | Day 3 | Codex | DBスキーマ作成 |
| 2 | **Flow → Stock MVP** | □ | Flow 録画→Mux→Gemini→Draft 生成、**Google Docs 取り込み** | Day 10 | Codex |  |
| 3 | **Manager & Viewer UI** | □ | Manager 承認フロー、**一般ユーザー閲覧 UI**、AI 品質ログ | Day 20 | Codex |  |

---

## 4. 作業分解 & 進行ログ

### Phase 1: Infra & DB Setup (Day 1-3)
- **1.1 リポジトリ & 環境構成**
  - [ ] プロジェクト構造作成 (`apps/app`, `apps/edge`)
  - [ ] `.env` / `.envrc` テンプレート作成
- **1.2 データベース (Supabase)**
  - [ ] マイグレーションファイル作成 (`supabase/migrations/20251201000000_init_schema.sql`)
    - `organizations`, `stores`, `users`, `memberships`
    - `handovers`, `manuals`, `ai_jobs`, `manual_edits`
  - [ ] RLS ポリシー適用 (Owner only start)
  - [ ] シードデータ投入 (`supabase/seed.sql`)
- **1.3 外部サービス連携**
  - [ ] Mux Webhook 設定確認 (ドキュメントベース)
  - [ ] Gemini API 疎通テストスクリプト作成 (`scripts/test_gemini.ts`)

### Phase 2: Flow → Stock MVP (Day 4-10)
- **2.1 Flutter アプリ基盤**
  - [ ] Flutter プロジェクト作成 (`apps/app`)
  - [ ] 依存パッケージ導入 (`riverpod`, `supabase_flutter`, `isar`, `camera`)
  - [ ] Auth 画面 & 状態管理実装
- **2.2 Flow 録画 & アップロード**
  - [ ] 録画 UI 実装 (`RecordScreen`)
  - [ ] オフラインキュー実装 (`Isar` - `pending_uploads`)
  - [ ] Mux アップロード処理 (`VideoService`)
- **2.3 Edge Functions (Backend)**
  - [ ] `mux_webhook` 実装 (`supabase/functions/mux_webhook/index.ts`)
  - [ ] `ai_process_handover` 実装 (`supabase/functions/ai_process_handover/index.ts`)
    - Gemini 呼び出し (要約・手順・Tips 生成)
- **2.4 Google Docs 取り込み**
  - [ ] `import_google_doc` 実装 (`supabase/functions/import_google_doc/index.ts`)
    - URL から本文取得
    - Gemini で整形 (Optional)
    - `manuals` に `source_type='legacy_import'` で保存

### Phase 3: Manager & Viewer UI (Day 11-20)
- **3.1 Manager UI**
  - [ ] マニュアル一覧 & 詳細画面 (`ManagerManualList`, `ManagerManualDetail`)
  - [ ] 承認アクション (`approve`, `reject`)
  - [ ] **Google Docs 追加モーダル** 実装
- **3.2 一般ユーザー閲覧 UI**
  - [ ] タイムライン表示 (`TimelineScreen`)
  - [ ] カテゴリ検索 & フィルタ
  - [ ] マニュアル詳細 (`ManualDetailScreen`) - 動画 & テキスト統合表示
- **3.3 品質 & 運用**
  - [ ] `manual_edits` ログ記録実装
  - [ ] コスト監視 Cron (`supabase/functions/cost_monitor/index.ts`)

---

## 5. リスク & 対応
| リスク | 影響 | 状態 | 対応策 / 教訓 |
|--------|------|------|----------------|
| RLS 設計ミス | データ漏洩 | ▢ Open | `plan: owner-only→段階緩和` で慎重に進める |
| AI 成果ムラ | 承認遅延 | ▢ Open | `manual_edits` ログでプロンプト調整 |
| Google Docs 構造化 | 取り込み失敗 | ▢ Open | テキスト抽出ライブラリの選定とエラーハンドリング強化 |

---

## 6. 指標 / 検証ログ
- **ビルド検証**：未実施
- **Activation 指標**：未計測
- **コスト監視**：未計測

---

## 7. コミュニケーション / 次アクション
- **最新アップデート**：計画を詳細化し、Phase 1〜3 のタスクを具体的に定義しました。
- **次に着手するタスク**：`<ACTION: 1.1 リポジトリ & 環境構成 の作成>`
- **必要な支援**：Supabase プロジェクトの URL / Key、Mux / Gemini の API Key の準備をお願いします。

---
