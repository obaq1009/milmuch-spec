# CHANGELOG

本ファイルは、milmuch SES Matching System の仕様変更履歴を管理する。  
実装対象は **Status: ACTIVE** の仕様のみとする。

---

## v1.0 – 2025-12-15
### Initial Scope Definition
#### Added
- システム全体構想の確定（メール起点 SES マッチング）
- Rails 7.x 前提の設計方針
- AWS 最小構成（EC2 / RDS / S3 / ALB / Sidekiq）
- raw メール保存方針（再解析前提）

---

## v1.1 – 2025-12-15
### Mail Classification & Extraction
#### Added
- メール分類ルール（project / candidate / ops / schedule / other / needs_review）
- 抽出ルール（案件・人材）
- 正規表現ベースの抽出仕様
- JSONB による柔軟なフィールド保持

#### Changed
- 単価は「万円」のみをスコープ内と定義
- 時給（円/時）はスコープ外に明示

---

## v1.2 – 2025-12-15
### Dictionary & Normalization
#### Added
- 表記揺らぎ対応の辞書DB設計
- 辞書編集UI（営業/管理者が追加・編集可能）
- 再解析（Re-parse）機能
- 正規化（Normalizer）仕様（スキル/勤務地/リモート/ラベル揺れ）

---

## v1.3 – 2025-12-16
### Matching Logic (Base)
#### Added
- ルールベースのマッチングロジック
- スコア設計（0〜100）
- Hard Fail 条件（外国籍不可、リモート条件不一致 等）
- スコア理由（Explainability）必須化

---

## v1.4 – 2025-12-16
### Margin Model Introduction
#### Added
- マージン3層モデル
  - Default Margin（全体）
  - Project Margin（案件別）
  - Override Margin（案件×人材）
- 実効原価（原価 + マージン）の定義
- マージン設定UI（全体/案件別/組み合わせ別）

#### Changed
- 単価評価を「人材原価」→「実効原価」ベースに変更

---

## v1.5 – 2025-12-16
### Main UI (Qoala-like)
#### Added
- 案件一覧（project_list_ui.md）
- 人材一覧（candidate_list_ui.md）
- 案件詳細（project_detail_ui.md：サマリー＋アコーディオン）
- 人材詳細（candidate_detail_ui.md：サマリー＋アコーディオン）

---

## v1.6 – 2025-12-16
### Proposal Template (Safe Mode)
#### Added
- 提案メール作成（テンプレ ON / OFF 選択）
- 原文（人材メール本文）保持
- 必須差替項目の固定化
  - 単価（実効原価）
  - 所属（当社提示表記）

---

## v1.7 – 2025-12-16
### Affiliation Handling
#### Added
- 所属の抽出・正規化仕様
- affiliation_alias 辞書
- 原文保持＋当社提示表記の二重構造

---

## v1.8 – 2025-12-16
### Reply & Progress Tracking
#### Added
- 提案単位（proposal）での進捗管理
- Bcc 取り込みによる送受信ログ管理
- Tracking Code による紐付け
- 未紐付け受信箱
- 進捗ボード（営業/管理者）

---

## v1.9 – 2025-12-16
### Tracking Health & Operation Safety
#### Added
- Bcc 入れ忘れ対策（運用ルール）
- 追跡漏れ検知ジョブ
- tracking_alerts（放置・未追跡検知）
- アラート一覧UI

---

## v2.0 – 2025-12-16
### Authentication & Sales Management
#### Added
- ログイン機能（login_ui.md）
- ユーザー管理（user_management_ui.md）
- 担当者（owner）可視化前提
- 営業目標・実績管理（sales_management.md）
  - 月次目標（売上/粗利）
  - 月次実績（売上/粗利）
  - 日次目標（提案数）
  - 日次実績（提案数：システム集計＋手入力）

---

## 実装優先順位（近藤さん向け）
1. ログイン / ユーザー管理
2. メール取込・分類・抽出（Classification/Extraction）
3. 辞書UI・正規化（Normalizer/Re-parse）
4. マッチング（マージン込み）
5. メイン画面（案件 ↔ 人材）
6. 提案メール作成（テンプレON/OFF・原文保持）
7. 返信・進捗管理（Reply/Tracking）
8. 営業目標・実績管理（Sales Dashboard）
9. 追跡漏れ検知（Tracking Health）

---

## 注意事項
- 実装対象は **Status: ACTIVE** の仕様のみ
- 不明点は口頭で埋めず、md に追記してから進める