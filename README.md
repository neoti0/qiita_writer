# Qiita記事執筆マルチエージェントシステム

Claude Codeを使ってQiita技術記事を自動執筆するパイプラインです。
**APIキー不要・追加課金なし**（Claude Proサブスクリプションのみで動作）

## セットアップ

```bash
# Claude Codeのインストール（未インストールの場合）
npm install -g @anthropic-ai/claude-code

# このフォルダに移動
cd qiita-writer

# Claude Codeを起動
claude
```

## 使い方

### 1. input.jsonを編集する

```json
{
  "overview": "記事として伝えたい内容の概要",
  "goal": "この記事を読んだ後に読者が達成できること",
  "audience": "想定する読者の詳細（技術レベル・使用技術など）",
  "unique_experience": "執筆者自身がハマったこと・気づき・失敗談など（記事の独自性となる体験）"
}
```

### 2. Claude Codeに依頼する

Claude Codeが起動したら、以下のように指示するだけ：

```
input.jsonを読んでCLAUDE.mdのパイプラインに従って記事を執筆してください
```

### 3. 出力を確認する

```
qiita-writer/
├── work/
│   ├── 01_structure.md         ← 記事構成
│   ├── 02_draft.md             ← 初稿
│   ├── 03a_technical_review.md ← 技術検証結果
│   ├── 03b_code_review.md      ← コードレビュー結果
│   ├── 04_proofreading.md      ← 校正結果
│   ├── 05a_critic.md           ← 批判的査読結果
│   ├── 05b_reader.md           ← 読者視点チェック結果
│   ├── 06_todos.md             ← 修正Todoリスト
│   ├── 07_corrected.md         ← 修正済み記事
│   └── 08_with_diagrams.md     ← 図追加済み記事
└── output/
    └── final_article.md        ← ★ 最終記事（これをQiitaに投稿）
```

## Nano Banana 2（Gemini画像生成）について

図示化エージェントが複雑な図に対して、Geminiへの指示プロンプトを生成します。

1. `output/final_article.md` を開く
2. `<NANO_BANANA_2_PROMPT>` の箇所を探す
3. プロンプト内容をコピーする
4. [Geminiアプリ](https://gemini.google.com) を開き「🍌 画像を作成」を選択
5. プロンプトを貼り付けて画像を生成
6. 生成された画像を記事に挿入する

## パイプライン構成

```
input.json
    │
    ▼
Stage 1: アーキテクト（構成設計）
    │
    ▼
Stage 2: ライター（執筆）
    │
    ├─────────────────────┐
    ▼                     ▼
Stage 3a: テクニカル   Stage 3b: コードレビュー
    │                     │
    └──────────┬──────────┘
               ▼
Stage 4: 校正
               │
    ┌──────────┴──────────┐
    ▼                     ▼
Stage 5a: 批判        Stage 5b: 読者視点
    │                     │
    └──────────┬──────────┘
               ▼
Stage 6: 修正Todo洗い出し
               │
               ▼
Stage 7: 訂正
               │
               ▼
Stage 8: 図示化（Mermaid + Nano Banana 2プロンプト生成）
               │
               ▼
Stage 9: SEO最適化
               │
               ▼
    output/final_article.md
```

## サンプル成果物

このリポジトリには実際にパイプラインを実行して生成された成果物が含まれています。

- `work/` — 各ステージの中間ファイル
- `output/final_article.md` — 最終記事（Qiita投稿用）
