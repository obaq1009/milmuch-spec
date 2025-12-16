# 案件一覧画面仕様（Project List UI）

- Status: ACTIVE
- Version: v1.0
- Last Updated: 2025-12-16
- Owner: milmuch / ChatGPT
- Target: Ruby on Rails 7.x
- Depends On:
  - Login UI v1.0（ログイン必須）
  - Classification / Extraction / Normalizer（project/candidate）
  - Matching Rules v1.1（マージン込み）
  - Margin Settings UI v1.0（default/project/override）
  - Proposal Template v1.1（テンプレON/OFF＋原文保持）

---

## 1. 目的
営業が「案件起点」で以下を最短で行えるようにする。

- 案件を探す（フィルタ）
- 案件を選ぶ
- 候補人材（スコア・単価・マージン込み）を見る
- 必要ならマージンを調整する
- 提案文を作る（原文崩さず）
- 提案済み管理をする

---

## 2. URL / 画面名
- GET `/matching/projects`
- 画面名：案件一覧

※ ログイン必須（未ログインは /login へリダイレクト）

---

## 3. レイアウト（3カラム）
- 左：検索・フィルタ（固定幅）
- 中央：案件一覧（スクロール）
- 右：選択案件の候補人材（TOP20）＋アクション

---

## 4. 左カラム：検索・フィルタ

### 4.1 対象データ
- `email_parses.mail_type = project`
- `email_parses` は最新の parser_version のみを対象（運用で決める。MVPは最新1件）

### 4.2 フィルタ項目（MVP必須）
- キーワード（部分一致）
  - 対象：title / description / skills_required / skills_optional
- 単価（万円）
  - min_man（整数）
  - max_man（整数）
  - 対象：project.price_min_man / price_max_man
- リモート種別
  - full / partial / none / unknown
- 場所（テキスト部分一致）
  - project.location_text
- 開始（テキスト部分一致）
  - project.start_text（例：1月）
- 外国籍条件（トグル）
  - `外国籍不可のみ`
- 期間（受信日）
  - from_date / to_date（sent_at or created_at）

### 4.3 操作
- [検索] ボタン
- [リセット] ボタン

---

## 5. 中央：案件一覧（リスト）

### 5.1 表示順（初期）
- 受信日（sent_at）降順（新しい順）

### 5.2 行表示項目（MVP必須）
- タイトル（project.title or subject）
- 単価（表示優先順）
  1) `min-max`（例：65-67万）
  2) `～max`（例：～55万）
  3) `price_text`（例：スキル見合い）
- 場所（location_text）
- リモート（remote_typeバッジ）
- 開始（start_text）
- 必須スキル（skills_required：最大3件チップ）
- 面談回数（interview_count：あれば）
- 募集人数（positions：あれば）
- 外国籍（nationality_rule：あれば）

### 5.3 行クリック挙動（必須）
- クリックした行を「選択状態」にする（ハイライト）
- 右カラムを当該案件の候補人材に切り替える
- URLは変えなくてOK（SPA不要、RailsでOK）
  - v1：同一ページ内で再描画（TurboでもOK）

### 5.4 初期選択（推奨）
- 初回表示時、先頭（最新）の案件を自動選択する

---

## 6. 右カラム：候補人材（TOP20）

### 6.1 対象
- `email_parses.mail_type = candidate`

### 6.2 候補算出
- 当該案件（project）と各人材（candidate）でマッチ計算
- 表示は TOP20（score desc）

※ 性能上厳しい場合は matches テーブルキャッシュ（別仕様参照）

### 6.3 表示項目（MVP必須）
各候補行に必ず表示：

- `total_score`（0〜100）
- 上位理由（reasons上位2〜3件）
- 氏名/イニシャル（candidate.name_or_initial）
- 年齢（あれば）
- スキル（最大5件チップ）
- 稼働開始（availability_text）
- リモート希望（work_style_pref.remote_pref）

#### 6.3.1 単価・マージン表示（必須）
- 原価（candidate_cost_base_man）
- 適用マージン（applied_margin_man）
- 適用元（source：上書き/案件別/デフォルト）
- 実効原価（effective_cost_man）
- 案件単価（project price）

表示例（固定文言推奨）：
- `原価65 + マージン8（案件別）= 実効73 / 案件65-67`

### 6.4 行アクション（MVP必須）
- [詳細]（人材詳細へ：/matching/candidates/:id など）
- [マージン上書き]（Override Margin：MGN-03）
- [提案メール作成]（Proposal Template：テンプレON/OFF選択）
- [提案済みにする]（status=sentに変更：MVP）

### 6.5 提案済み表示（必須）
- proposals に同一（project,candidate）が存在し status=sent の場合：
  - 候補行に `提案済み` バッジ表示
- 提案済みは並び順を下げる（任意）
  - v1推奨：提案済みは下部へ（toggleで「提案済みも表示」も可）

---

## 7. 画面上部（任意だが推奨）
共通ヘッダー
- 左：milmuch（ロゴ）
- 中：ナビ
  - 案件一覧
  - 人材一覧
  - 進捗（提案ボード）
  - 営業目標（マイページ）
- 右：ログアウト

※ 権限に応じて表示制御（adminのみ管理画面）

---

## 8. エッジケース（v1での扱い）
- 案件の必須スキルが抽出できない → needs_review になっている前提
- 単価不明（スキル見合い） → 単価一致点は弱一致（matching_rules参照）
- 国籍不明の人材 → 外国籍不可案件でも hard fail しない（maybe扱い推奨）
- 大量件数で遅い場合 → matchesキャッシュを導入

---

## 9. 受入基準（Acceptance Criteria）
- ログイン後 `/matching/projects` が表示できる
- フィルタで案件を絞り込める
- 案件行をクリックすると、右に候補人材TOP20が表示される
- 候補人材行に「スコア」「理由」「原価/マージン/実効原価/案件単価」が表示される
- マージン上書きで当該候補の単価評価が即反映される
- 提案メール作成でテンプレON/OFF選択ができ、原文を崩さずコピーできる
- 提案済みバッジが表示される

---

## 10. 非スコープ（v2以降）
- おすすめ（今日やること）自動生成
- 高度な検索（全文検索エンジン）
- AIによる自動返信分類（OK/NG/面談希望）
- 添付（PDF/Excel）の自動解析

---
