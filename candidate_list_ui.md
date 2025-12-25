# 人材一覧画面仕様（Candidate List UI）

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
営業が「人材起点」で以下を最短で行えるようにする。

- 人材を探す（フィルタ）
- 人材を選ぶ
- 候補案件（スコア・単価・マージン込み）を見る
- 必要ならマージンを調整する（案件別 / 組み合わせ別）
- 提案文を作る（原文崩さず）
- 提案済み管理をする

---

## 2. URL / 画面名
- GET `/matching/candidates`
- 画面名：人材一覧

※ ログイン必須（未ログインは /login へリダイレクト）

---

## 3. レイアウト（3カラム）
- 左：検索・フィルタ（固定幅）
- 中央：人材一覧（スクロール）
- 右：選択人材の候補案件（TOP20）＋アクション

---

## 4. 左カラム：検索・フィルタ

### 4.1 対象データ
- `email_parses.mail_type = candidate`
- `email_parses` は最新の parser_version のみを対象（MVPは最新1件）

### 4.2 フィルタ項目（MVP必須）
- キーワード（部分一致）
  - 対象：name_or_initial / pr_text / skills / subject
- 単価（万円）
  - min_man / max_man
  - 対象：candidate.price_min_man / price_max_man
- リモート希望
  - full / partial / none / unknown
- 最寄駅（テキスト部分一致）
  - nearest_station
- 稼働開始（テキスト部分一致）
  - availability_text（例：1月、11/1〜）
- 国籍（トグル）
  - `日本のみ`（または `外国籍除外`）
- 期間（受信日）
  - from_date / to_date（sent_at or created_at）

### 4.3 操作
- [検索] ボタン
- [リセット] ボタン

---

## 5. 中央：人材一覧（リスト）

### 5.1 表示順（初期）
- 受信日（sent_at）降順（新しい順）

### 5.2 行表示項目（MVP必須）
- 氏名/イニシャル（name_or_initial）
- 年齢（あれば）
- 単価（表示優先順）
  1) `min-max`（例：80-85万）
  2) `min-`（例：65万〜）
  3) `price_text`（例：応相談）
- 最寄駅（nearest_station）
- リモート希望（remote_pref バッジ）
- 稼働開始（availability_text）
- スキル（skills：最大5件チップ）
- 所属（affiliation_raw：あれば短く表示）

### 5.3 行クリック挙動（必須）
- クリックした行を「選択状態」にする（ハイライト）
- 右カラムを当該人材の候補案件に切り替える
- URLは変えなくてOK（Railsで再描画/Turbo可）

### 5.4 初期選択（推奨）
- 初回表示時、先頭（最新）の人材を自動選択する

---

## 6. 右カラム：候補案件（TOP20）

### 6.1 対象
- `email_parses.mail_type = project`

### 6.2 候補算出
- 当該人材（candidate）と各案件（project）でマッチ計算
- 表示は TOP20（score desc）

### 6.3 表示項目（MVP必須）
各候補行に必ず表示：

- `total_score`（0〜100）
- 上位理由（reasons上位2〜3件）
- 案件タイトル（project.title）
- 場所（location_text）
- リモート（remote_type バッジ）
- 開始（start_text）
- 必須スキル（最大3件チップ）

#### 6.3.1 単価・マージン表示（必須）
人材起点では「この人材をこの案件に当てるといくらで提案するか」を明示する。

- 原価（candidate_cost_base_man）
- 適用マージン（applied_margin_man）
- 適用元（source：上書き/案件別/デフォルト）
- 実効原価（effective_cost_man）
- 案件単価（project price）

表示例：
- `原価65 + マージン5（デフォ）= 実効70 / 案件75-80`

### 6.4 行アクション（MVP必須）
- [詳細]（案件詳細へ：/matching/projects/:id など）
- [マージン上書き]（Override Margin：MGN-03）
- [提案メール作成]（Proposal Template：テンプレON/OFF選択）
- [提案済みにする]（status=sentに変更）

### 6.5 提案済み表示（必須）
- proposals に同一（project,candidate）が存在し status=sent の場合：
  - 候補行に `提案済み` バッジ表示
- 提案済みを下げる（任意）：
  - v1推奨：提案済みは下部へ（toggleで切替可）

---

## 7. 補足：案件別マージンの編集導線（推奨）
人材起点で見ている時でも「この案件は強気で行く（案件別マージン）」を設定したくなる。

- 候補案件行に小リンク：`案件マージン編集`
  - クリックで案件詳細（またはモーダル）へ
  - project_margin_man を編集可能（権限：sales以上）

※ MVPでは案件詳細で編集でも可（導線だけ用意）

---

## 8. 受入基準（Acceptance Criteria）
- ログイン後 `/matching/candidates` が表示できる
- フィルタで人材を絞り込める
- 人材行をクリックすると、右に候補案件TOP20が表示される
- 候補案件行に「スコア」「理由」「原価/マージン/実効原価/案件単価」が表示される
- マージン上書きで当該候補の単価評価が即反映される
- 提案メール作成でテンプレON/OFF選択ができ、原文を崩さずコピーできる
- 提案済みバッジが表示される

---

## 9. 非スコープ（v2以降）
- おすすめ（今日やること）自動生成
- 高度な検索（全文検索）
- 添付（PDF/Excel）の自動解析
- 返信内容のAI分類（OK/NG/面談希望）

---