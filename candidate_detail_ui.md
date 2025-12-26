# 人材詳細画面仕様（Candidate Detail UI）

- Status: ACTIVE
- Version: v1.0
- Last Updated: 2025-12-16
- Owner: milmuch / ChatGPT
- Target: Ruby on Rails 7.x
- Depends On:
  - Candidate List UI v1.0
  - Classification / Extraction / Normalizer
  - Proposal Template v1.1（原文保持）
  - Matching Rules v1.1（マージン込み）
  - Affiliation Rules v1.2

---

## 1. 目的
人材一覧では判断しきれない場合に、
**原文確認・スキル詳細・条件確認・添付確認**を安全かつ効率的に行う。

- 原文を崩さず、紹介トラブルを防止
- 「出せる人材かどうか」を短時間で判断
- 案件提案前の最終確認に使う

---

## 2. URL / 画面名
- GET `/matching/candidates/:id`
- 画面名：人材詳細

※ 人材一覧（/matching/candidates）からの遷移が主  
※ 新規タブ or モーダル表示でも可（MVPは画面遷移でOK）

---

## 3. 画面構成（重要）

### 上部：サマリー（常に表示）
### 下部：アコーディオン（必要な人だけ展開）

※ **全情報を一気に表示しない**（Qoala準拠）

---

## 4. 上部サマリー（MVP必須）

### 4.1 表示項目
- 氏名 / イニシャル（name_or_initial）
- 年齢（age）
- 単価（原価）
  - `min-max` / `min-` / `price_text`
- 稼働開始（availability_text）
- リモート希望（work_style_pref.remote_pref バッジ）
- 最寄駅（nearest_station）
- 主要スキル（skills：最大7件チップ表示）
- 所属（affiliation_presented：当社提示用表記）

### 4.2 表示ルール
- サマリーだけで「この人を出せるか」が判断できること
- 単価・稼働開始・リモートは特に目立たせる
- 所属は **当社提示用表記のみ** を表示（原文は下で確認）

---

## 5. 下部アコーディオン構成（MVP）

### 🔽 B-1：人材原文（最重要）
**目的：原文確認・トラブル防止**

- 人材紹介メール本文（Email.body_text）
- **改変なし**
- 引用形式で表示（各行に `>` を付与）

表示氏名：M.K（女性/37歳/韓国籍）
希望単価：80〜85万円希望
参画時期：11/1〜
最寄駅：五反田
所属：弊社所属フリーランス

---

### 🔽 B-2：スキル詳細
**目的：経験・強みの正確な把握**

表示項目：
- 言語 / FW / DB / ツール（抽出結果）
- 経験年数（あれば）
- 正規化済みスキル名を表示

※ v1では「原文どおりの詳細記述」は B-1 に委ねる

---

### 🔽 B-3：希望条件
**目的：案件可否判断**

表示項目（あれば）：
- 単価希望（price_min_man / price_max_man / price_text）
- リモート希望詳細（work_style_pref.remote_detail）
- 稼働条件
- NG条件（記載があれば）

---

### 🔽 B-4：添付ファイル
**目的：スキルシート・補足資料確認**

- 添付一覧（ActiveStorage）
- 表示：
  - ファイル名
  - 種別（PDF / Excel 等）
  - サイズ
- 操作：
  - ダウンロード
  - プレビュー（v2以降）

---

### 🔽 B-5：所属・契約情報（内部用）
**目的：提案文作成時の裏確認**

- affiliation_raw（原文所属表記）
- affiliation_category（正規化カテゴリ）
- affiliation_presented（当社提示用表記）

※ admin のみ表示推奨  
※ sales はサマリーの提示表記のみを見る

---

### 🔽 B-6：システム情報（内部用）
**目的：管理・デバッグ**

- 受信日時（sent_at）
- 送信元ドメイン
- parser_version
- needs_review フラグ

※ admin のみ表示

---

## 6. 画面内アクション（MVP）

### 6.1 基本アクション
- [一覧に戻る]
- [人材一覧でこの人材を選択状態にする]（任意）

### 6.2 営業アクション
- [案件一覧で候補案件を見る]（/matching/candidates 右カラムへ戻る）
- [提案メール作成]（Proposal Template：テンプレON/OFF選択）

---

## 7. UXルール（重要）
- 初期状態では **全アコーディオンは閉じる**
- 最重要の「人材原文」を一番上に配置
- 展開・折りたたみはワンクリック
- 一覧画面の思考を壊さない

---

## 8. 受入基準（Acceptance Criteria）
- 人材一覧から人材詳細に遷移できる
- サマリーだけで人材の全体像が把握できる
- 原文が改変なしで表示される
- スキル・条件・添付がアコーディオンで確認できる
- admin のみ内部情報が見える

---

## 9. 非スコープ（v2以降）
- 原文の自動要約（AI）
- 添付ファイルの全文検索
- 経験年数の自動算出

---
