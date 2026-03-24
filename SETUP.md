# Claude Codeへの配置ガイド

## 概要

このスキルは Claude Code の Skills 機能を使って配置します。
配置後、自然言語で「業務フロー図を作って」などと言うだけで自動的にスキルが起動します。
出力先はMiro（メイン）とFigJam（フォールバック）の2系統に対応。

---

## 1. スキルの配置

### 方法A: グローバルスキル（推奨・全プロジェクトで使える）

```bash
mkdir -p ~/.claude/skills/business-flow-generator/references
cp SKILL.md ~/.claude/skills/business-flow-generator/
cp references/miro-api-patterns.md ~/.claude/skills/business-flow-generator/references/
cp references/mermaid-patterns.md ~/.claude/skills/business-flow-generator/references/
```

### 方法B: プロジェクトスキル（特定プロジェクトのみ）

```bash
mkdir -p .claude/skills/business-flow-generator/references
cp SKILL.md .claude/skills/business-flow-generator/
cp references/* .claude/skills/business-flow-generator/references/
```

---

## 2. Miro APIの設定（メイン出力先）

### トークン取得
1. https://miro.com/app/settings/user-profile/ → 「Your apps」
2. アプリを新規作成 → `boards:read boards:write` スコープを付与
3. OAuthトークンを取得

### 環境変数の設定
```bash
export MIRO_TOKEN="your-miro-oauth-token"
export MIRO_BOARD_ID="your-board-id"
```

ボードIDは、MiroボードURLの `https://miro.com/app/board/` の後の文字列。

### 永続化（.bashrc / .zshrc に追記）
```bash
echo 'export MIRO_TOKEN="your-miro-oauth-token"' >> ~/.zshrc
echo 'export MIRO_BOARD_ID="your-board-id"' >> ~/.zshrc
```

---

## 3. FigJam MCPの設定（フォールバック）

Miroが使えない場合にFigJamにMermaid図を生成する。

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

---

## 4. 動作確認

```bash
# Claude Code を起動して
この業務フローの会話からフロー図を作って
```

スキルが自動的に:
1. Miro接続を確認 → OKなら Miro に出力
2. Miro不可 → Figma MCP → FigJam に出力
3. 両方不可 → Mermaid構文をテキスト出力

### スラッシュコマンドでの起動
```
/business-flow-generator
```

---

## 5. 使い方の例

### テキストファイルを渡す
```
この議事録から業務フロー図をMiroに作って
```

### 直接テキストを貼り付け
```
以下のヒアリング内容から業務フローを可視化して:

田中（営業）: お客さんから新規案件の問い合わせが来て...
鈴木（開発）: 見積もりの技術部分は...
```

### 段階的に確認
```
この内容の業務フローを整理して。まず構造を見せて、確認してから図にしたい
```

### 出力先を指定
```
FigJamで作って（Miroではなく）
```

---

## ディレクトリ構成

```
~/.claude/skills/business-flow-generator/
├── SKILL.md                           # メインのスキル定義
└── references/
    ├── miro-api-patterns.md           # Miro REST API パターン集
    └── mermaid-patterns.md            # Mermaid構文パターン集（FigJam用）
```

---

## トラブルシューティング

### スキルが自動起動しない場合
CLAUDE.md に追記:
```markdown
## 利用可能なスキル
- business-flow-generator: テキストから業務フロー図を生成しMiro/FigJamに配置する
```

### Miro APIエラー
- **401**: トークン期限切れ → Miroで再発行
- **404**: ボードIDが不正 → URLを再確認
- **429**: レート制限 → 自動的にリトライされる

### Figma MCP が接続できない場合
- `/mcp` で状態確認
- `claude mcp remove figma` → 再追加
