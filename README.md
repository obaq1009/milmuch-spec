# milmuch SES Matching System – Specification

## 概要
本リポジトリは、milmuch（当社）における  
**SES営業支援システムの仕様書（設計書）一式** を管理するためのものです。

本システムは、日々受信するSES関連メールを起点に、

- 案件メール・人材メールの自動解析
- 案件 × 人材のマッチング
- マージン考慮を含む提案判断
- 提案後の返信・進捗管理
- 営業目標・実績の可視化
- 営業育成・属人化防止

を目的として設計されています。

---

## このリポジトリの使い方（重要）
- 本リポジトリは **実装用の仕様書置き場** です
- 実装対象は **Status: ACTIVE** と明記されている md のみです
- 実装判断が必要な場合は、口頭で進めず **仕様に戻って調整** します

---

## 想定技術スタック
- Ruby on Rails 7.x
- PostgreSQL（AWS RDS）
- AWS（EC2 / ALB / S3 / Sidekiq）
- ActiveJob / ActiveStorage
- Outlook連携（Bcc取り込み方式・Graphは後続）

---

## 全体構成（概要）

Outlook / Mail
↓（Bcc）
Mail Ingest
↓
Classification（案件 / 人材 / 事務 / 日程）
↓
Extraction（正規表現）
↓
Normalization（辞書）
↓
Matching（マージン考慮）
↓
Proposal（提案・進捗・育成）

---

## 仕様書一覧（読む順番）

### ① 全体把握
- CHANGELOG.md  
  → どこまで決まっているか・実装優先度

### ② メール解析
- classification_rules.md  
- extraction_rules.md  
- normalizer_rules.md  
- dictionary_ui.md  

### ③ マッチング・単価
- matching_rules.md  
- margin_rules.md  
- margin_ui.md  

### ④ 営業UI
- main_ui.md  
- proposal_template.md  
- affiliation_rules.md  

### ⑤ 進捗・育成
- reply_progress_tracking.md  
- sales_management.md
---

## 運用上の重要ルール
- 原文メールは **絶対に改変しない**
- 提案時は
  - 単価（原価＋マージン）
  - 所属（当社提示表記）
  を必ず明示する
- 提案後の返信は Bcc＋Tracking Code で追跡する
- AI生成文は **補足用途のみ** とする

---

## 実装優先順位（再掲）
1. ログイン / ユーザー管理
2. メール取込・分類・抽出
3. 辞書UI・正規化
4. マッチング（マージン込み）
5. メイン画面（案件↔人材）
6. 提案テンプレ（ON/OFF）
7. 返信・進捗管理
8. 営業目標・実績管理

---

## 注意事項
- 本リポジトリは **仕様専用**
- 実装コードは別リポジトリで管理する
- 不明点は仕様に追記・更新する

---

## 管理者
- Owner: milmuch





