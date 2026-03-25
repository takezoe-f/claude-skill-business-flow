# Business Flow Generator

あらゆるテキスト情報（議事録、会話ログ、業務マニュアル、要件定義書、Slack/チャットログ、メールなど）から、スイムレーン付きの業務フロー図を自動生成する Claude Code スキル。

## 機能

- テキストから業務フローの構造化分析（アクター・タスク・分岐・タイムライン・使用システムの自動抽出）
- Miro ボードへのスイムレーン付きフロー図の自動配置（メイン出力）
- FigJam への Mermaid ダイアグラム生成（フォールバック）
- ノード種別に応じた色分け・コネクタの自動接続（判断ノード、差戻し、分岐など）
- 中間データのユーザー確認フロー（Human-in-the-loop）

## 前提条件

### Miro（メイン出力）
- 環境変数 `MIRO_TOKEN` と `MIRO_BOARD_ID` が設定されていること
- `curl` が利用可能であること

### FigJam（フォールバック）
- Figma MCP サーバーが接続済みであること（`generate_diagram` ツールが利用可能）
- Miro も Figma MCP も利用不可の場合は Mermaid 構文をテキスト出力

## インストール

```bash
# Claude Code のスキルディレクトリにクローン
git clone https://github.com/takezoe-f/claude-skill-business-flow.git ~/.claude/skills/business-flow-generator/
```

## 使い方

以下のようなフレーズで起動します：

- `フロー図を作って`
- `業務フローを可視化して`
- `この内容をMiroに起こして`
- `部署間の連携を図にして`
- テキストやファイルを渡して `フローチャートにして`

## ライセンス

MIT
