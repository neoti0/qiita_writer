# コードレビュー結果

## レビュー対象
`work/02_draft.md` に含まれるすべてのコードブロック

---

## コードブロック一覧と言語名確認

| コードブロック | 言語名 | 状態 |
|-------------|-------|------|
| エディタモデル概念 | （テキスト、コードブロック不使用） | ✅ |
| pip install | bash | ✅ |
| export ANTHROPIC_API_KEY | bash | ✅ |
| ディレクトリ構成 | （言語名なし） | ⚠️ `text` または言語名を追加推奨 |
| base_agent.py | python | ✅ |
| architect.py | python | ✅ |
| orchestrator.py | python | ✅ |
| bash実行コマンド | bash | ✅ |
| ディレクトリ構成（Claude Code版） | （言語名なし） | ⚠️ `text` を追加推奨 |
| CLAUDE.md記述例 | markdown | ✅ |
| Claude Code実行 | bash | ✅ |

---

## コードの動作正確性チェック

### base_agent.py

```python
def call_agent(system_prompt: str, user_message: str, model: str = "claude-haiku-4-5-20251001") -> str:
    message = client.messages.create(
        model=model,
        max_tokens=8096,
        messages=[
            {"role": "user", "content": user_message}
        ],
        system=system_prompt
    )
    return message.content[0].text
```

**評価**: ✅ 動作する。`system` パラメータはトップレベルに置く形式で正しい。

**指摘**: `message.content[0].text` はコンテンツブロックがテキスト型であることを前提としている。Tool useなど他のレスポンス型が返ってきた場合に `AttributeError` が発生する可能性があるが、このユースケースでは問題なし。

### architect.py

```python
def run(input_path: str = "input.json", output_path: str = "work/01_structure.md"):
    with open(input_path, "r", encoding="utf-8") as f:
        input_data = json.load(f)
    ...
    result = call_agent(system_prompt, user_message, model="claude-sonnet-4-6")
    write_file(output_path, result)
```

**評価**: ✅ ロジック的に問題なし。

**指摘**:
- `from .base_agent import ...` は相対インポートのため、`agents/` がパッケージである必要がある（`__init__.py` が必要）。記事中では触れられていない。

### orchestrator.py

```python
with concurrent.futures.ThreadPoolExecutor(max_workers=2) as executor:
    future_tech = executor.submit(technical.run)
    future_code = executor.submit(code_review.run)
    concurrent.futures.wait([future_tech, future_code])
```

**評価**: ⚠️ 動作はするが、例外処理が欠如している。

**問題点**:
- `future_tech.result()` / `future_code.result()` を呼ばないと、スレッド内で発生した例外が握りつぶされる
- `concurrent.futures.wait` はタスク完了を待つが、例外を再raiseしない

**推奨修正**:
```python
with concurrent.futures.ThreadPoolExecutor(max_workers=2) as executor:
    future_tech = executor.submit(technical.run)
    future_code = executor.submit(code_review.run)
    # 例外を検出するためにresult()を呼ぶ
    future_tech.result()
    future_code.result()
```

### bashのhere-document

```bash
cat > input.json << 'EOF'
{
  "overview": "...",
  ...
}
EOF
```

**評価**: ✅ 正しいheredoc構文。シングルクォートの`'EOF'`により変数展開を無効化しているのも適切。

---

## セキュリティチェック

### [重要] APIキーの扱い

```bash
export ANTHROPIC_API_KEY="your-api-key-here"
```

**指摘**: この形式はシェル履歴に残る可能性がある。また、`.env`ファイルと`python-dotenv`の使用を推奨するか、少なくとも注記を追加すること。

**推奨追加文**:
:::note warn
APIキーはソースコードにハードコードしないでください。`.env`ファイルと`python-dotenv`ライブラリの使用を推奨します。また、`.env`ファイルは`.gitignore`に追加してください。
:::

### [低] 入力ファイルのサニタイズ

`input.json` の内容をそのままLLMプロンプトに埋め込んでいる。悪意あるプロンプトインジェクション（`input.json`に「前の指示を無視して...」などを書き込む）のリスクがある。内部ツールとして使う分には許容範囲内。

---

## ベストプラクティス・設計原則のチェック

### ✅ 良い設計
- `read_file` / `write_file` を共通化している
- `os.makedirs(exist_ok=True)` で安全にディレクトリ作成
- エンコーディングを明示（`encoding="utf-8"`）

### ⚠️ 改善提案
- `agents/__init__.py` の存在について言及がない
- `orchestrator.py` の `from agents import ...` は `agents/` 配下の全モジュールをインポートしているが、`__init__.py` での再エクスポートが必要

---

## 総合評価

コードは概ね正確に動作する。主な問題点は：
1. `concurrent.futures` の例外処理（サイレント失敗のリスク）
2. `agents/__init__.py` への言及がない
3. APIキーのセキュリティに関する注意書きが不足

根本的な動作不能なバグはなく、読者が試して動く品質。ただし上記3点の修正で信頼性が高まる。
