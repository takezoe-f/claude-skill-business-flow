# Miro REST API v2 リファレンス - 業務フロー図向け

業務フロー図を生成する際のMiro API呼び出しパターン。
SKILL.md のStep 5で参照する。

## 基本情報

- Base URL: `https://api.miro.com/v2`
- 認証: `Authorization: Bearer {MIRO_TOKEN}`
- 環境変数: `MIRO_TOKEN`, `MIRO_BOARD_ID`

## 1. API接続テスト

```bash
curl -s -o /dev/null -w "%{http_code}" \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}"
```

200が返ればOK。

## 2. 既存ボードのクリア（オプション）

ユーザーに確認の上、既存アイテムを削除する場合:

```bash
# ボード上の全アイテム取得
curl -s -H "Authorization: Bearer ${MIRO_TOKEN}" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/items?limit=50" | jq '.data[].id'

# 個別削除
curl -X DELETE -H "Authorization: Bearer ${MIRO_TOKEN}" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/items/${ITEM_ID}"
```

## 3. オブジェクト作成順序

必ずこの順序で作成する:
1. **フレーム**（スイムレーン）
2. **シェイプ**（ノード）
3. **テキスト**（タイムラインヘッダー、ラベル）
4. **コネクタ**（矢印）— ノードが存在しないと接続できないため最後

## 4. フレーム作成（スイムレーン）

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/frames" \
  -d '{
    "data": {
      "title": "営業部",
      "format": "custom",
      "type": "freeform"
    },
    "style": {
      "fillColor": "#F5F5F5"
    },
    "geometry": {
      "width": 2400,
      "height": 250
    },
    "position": {
      "x": 1200,
      "y": 125,
      "origin": "center"
    }
  }'
```

### スイムレーンレイアウト計算

- スイムレーン高さ: **250px** 固定
- スイムレーン幅: タイムラインのカラム数 × カラム幅
- Y座標: `レーンindex × 250`（レーン間のギャップなし or 10px程度）
- 左端にレーン名テキストを配置

## 5. シェイプ作成（ノード）

### ノード種別と色・形状

| 種別 | shape | fillColor | borderColor | 用途 |
|------|-------|-----------|-------------|------|
| 開始 | `circle` | `#D5F5E3` | `#27AE60` | フロー開始点 |
| 終了 | `circle` | `#D5D8DC` | `#7F8C8D` | フロー終了点 |
| タスク | `rectangle` | `#FFFFFF` | `#2C3E50` | 通常の業務ステップ |
| 判断 | `rhombus` | `#FEF9E7` | `#F39C12` | 分岐・判定 |
| システム | `round_rectangle` | `#EBF5FB` | `#3498DB` | 使用システム（Salesforce等） |
| データソース | `round_rectangle` | `#F5EEF8` | `#8E44AD` | DB・ファイル等 |
| ドキュメント | `round_rectangle` | `#E8F8F5` | `#1ABC9C` | ドキュメント・帳票 |
| 差戻し | `rectangle` | `#FADBD8` | `#E74C3C` | エラー・差戻し |

**重要**: 判断ノードのshapeは `rhombus`（`diamond` ではない）

### タスクノード作成例

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/shapes" \
  -d '{
    "data": {
      "content": "<p>売上データ入力</p>",
      "shape": "rectangle"
    },
    "style": {
      "fillColor": "#FFFFFF",
      "borderColor": "#2C3E50",
      "borderWidth": "2.0",
      "borderStyle": "normal",
      "fontFamily": "arial",
      "fontSize": "14",
      "textAlign": "center",
      "textAlignVertical": "middle"
    },
    "geometry": {
      "width": 140,
      "height": 60
    },
    "position": {
      "x": 350,
      "y": 50,
      "origin": "center"
    }
  }'
```

### 開始ノード

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/shapes" \
  -d '{
    "data": {
      "content": "<p>開始</p>",
      "shape": "circle"
    },
    "style": {
      "fillColor": "#D5F5E3",
      "borderColor": "#27AE60",
      "borderWidth": "2.0",
      "fontSize": "12",
      "textAlign": "center"
    },
    "geometry": {
      "width": 60,
      "height": 60
    },
    "position": {
      "x": 100,
      "y": 50,
      "origin": "center"
    }
  }'
```

### 判断ノード

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/shapes" \
  -d '{
    "data": {
      "content": "<p>差異あり？</p>",
      "shape": "rhombus"
    },
    "style": {
      "fillColor": "#FEF9E7",
      "borderColor": "#F39C12",
      "borderWidth": "2.0",
      "fontSize": "12",
      "textAlign": "center"
    },
    "geometry": {
      "width": 100,
      "height": 100
    },
    "position": {
      "x": 600,
      "y": 300,
      "origin": "center"
    }
  }'
```

### システムラベル（ノードの下に配置）

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/shapes" \
  -d '{
    "data": {
      "content": "<p>Salesforce</p>",
      "shape": "round_rectangle"
    },
    "style": {
      "fillColor": "#EBF5FB",
      "borderColor": "#3498DB",
      "borderWidth": "1.0",
      "fontSize": "11",
      "textAlign": "center"
    },
    "geometry": {
      "width": 100,
      "height": 30
    },
    "position": {
      "x": 350,
      "y": 90,
      "origin": "center"
    }
  }'
```

## 6. テキスト作成（タイムラインヘッダー）

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/texts" \
  -d '{
    "data": {
      "content": "<p><strong>毎月末日 17:00</strong></p>"
    },
    "style": {
      "fillColor": "transparent",
      "fontFamily": "arial",
      "fontSize": "14",
      "textAlign": "center"
    },
    "geometry": {
      "width": 180
    },
    "position": {
      "x": 350,
      "y": -30,
      "origin": "center"
    }
  }'
```

## 7. コネクタ作成

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/connectors" \
  -d '{
    "startItem": {
      "id": "START_ITEM_ID",
      "position": {
        "x": 1.0,
        "y": 0.5
      }
    },
    "endItem": {
      "id": "END_ITEM_ID",
      "position": {
        "x": 0.0,
        "y": 0.5
      }
    },
    "shape": "elbowed",
    "style": {
      "strokeColor": "#2C3E50",
      "strokeWidth": "2.0",
      "strokeStyle": "normal",
      "startStrokeCap": "none",
      "endStrokeCap": "stealth"
    }
  }'
```

### コネクタのバリエーション

| 用途 | strokeColor | strokeStyle | caption |
|------|-------------|-------------|---------|
| 通常フロー | `#2C3E50` | `normal` | — |
| Yes/承認 | `#27AE60` | `normal` | `Yes` |
| No/却下 | `#E74C3C` | `normal` | `No` |
| 差戻し | `#E74C3C` | `dashed` | `差戻し` |
| データ参照 | `#F39C12` | `dashed` | — |

### キャプション付きコネクタ

```bash
curl -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/connectors" \
  -d '{
    "startItem": { "id": "START_ID" },
    "endItem": { "id": "END_ID" },
    "shape": "elbowed",
    "captions": [
      {
        "content": "Yes",
        "position": "50%"
      }
    ],
    "style": {
      "strokeColor": "#27AE60",
      "strokeWidth": "2.0",
      "endStrokeCap": "stealth"
    }
  }'
```

### コネクタの接続点（position）

`startItem.position` / `endItem.position` で接続点を制御:
- 右端: `{ "x": 1.0, "y": 0.5 }`
- 左端: `{ "x": 0.0, "y": 0.5 }`
- 上端: `{ "x": 0.5, "y": 0.0 }`
- 下端: `{ "x": 0.5, "y": 1.0 }`

**重要**: 同じノードの同じ接続点に複数コネクタを接続しない。
複数の出力がある場合は異なるx/y座標を使う（例: 右上 `{x:1.0, y:0.3}` と 右下 `{x:1.0, y:0.7}`）。

## 8. 座標計算の指針

### レイアウト定数

```
SWIMLANE_HEIGHT = 250        # スイムレーンの高さ
SWIMLANE_LABEL_WIDTH = 120   # 左端のレーン名エリア幅
NODE_WIDTH = 140              # タスクノードの幅
NODE_HEIGHT = 60              # タスクノードの高さ
DECISION_SIZE = 100           # 判断ノードのサイズ
START_END_SIZE = 60           # 開始/終了ノードのサイズ
SYSTEM_LABEL_WIDTH = 100      # システムラベルの幅
SYSTEM_LABEL_HEIGHT = 30      # システムラベルの高さ
NODE_GAP_X = 50               # ノード間の横間隔
NODE_GAP_Y = 50               # ノード間の縦間隔
TIMELINE_COL_WIDTH = 200      # タイムラインカラム幅
SYSTEM_LABEL_OFFSET_Y = 40    # ノードの下にシステムラベルを配置する距離
```

### 座標計算ロジック

```
# スイムレーンY座標
swimlane_y(index) = index * SWIMLANE_HEIGHT

# ノードのY座標（スイムレーン中央に配置）
node_y(lane_index) = swimlane_y(lane_index) + SWIMLANE_HEIGHT / 2

# ノードのX座標（タイムラインカラム基準）
node_x(col_index) = SWIMLANE_LABEL_WIDTH + col_index * TIMELINE_COL_WIDTH + TIMELINE_COL_WIDTH / 2

# システムラベルY座標
system_label_y(node_y) = node_y + NODE_HEIGHT / 2 + SYSTEM_LABEL_OFFSET_Y
```

## 9. APIレート制限

Miro APIにはレート制限がある。大量のノードを作成する場合:
- リクエスト間に **100ms** 程度のスリープを入れる
- bash: `sleep 0.1` を各curlの後に挿入
- エラーが出たら待機時間を増やす

## 10. レスポンスからIDを取得

作成したアイテムのIDはレスポンスの `id` フィールドから取得する:

```bash
ITEM_ID=$(curl -s -X POST \
  -H "Authorization: Bearer ${MIRO_TOKEN}" \
  -H "Content-Type: application/json" \
  "https://api.miro.com/v2/boards/${MIRO_BOARD_ID}/shapes" \
  -d '{ ... }' | jq -r '.id')

echo "Created shape: ${ITEM_ID}"
```

コネクタ作成時にこのIDを使って接続先を指定する。
