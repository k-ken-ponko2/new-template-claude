# フロントエンド品質担保セットアップガイド

Claude Code での vibe coding 時に、フロントエンドコードの品質を担保するための設定ファイル一式です。

## ファイル構成

```
.claude/
├── commands/
│   ├── kiro.md           # 仕様駆動開発（既存）
│   ├── makeRules.md      # ルール作成（既存）
│   ├── init-frontend.md  # 【新規】フロントエンド初期化
│   └── figma.md          # 【新規】Figma実装
└── settings.local.json   # 権限設定（既存）

CLAUDE.md                  # 開発ルール（フロントエンド規約追加版）

design_system/             # 【init-frontend実行後に生成】
├── config.md              # 選択した技術スタック
├── tokens.md              # デザイントークン定義
├── components.md          # コンポーネント一覧
└── figma-mapping.md       # Figmaマッピング
```

## セットアップ手順

### 1. ファイル配置

```bash
# .claude/commands/ に配置
cp commands/init-frontend.md /path/to/project/.claude/commands/
cp commands/figma.md /path/to/project/.claude/commands/

# CLAUDE.md を置き換え（または既存にマージ）
cp CLAUDE.md /path/to/project/
```

### 2. フロントエンド初期化

Claude Code で以下を実行：

```
/init-frontend
```

#### Step 1: 技術スタック選択

対話形式で以下を選択：

- CSS フレームワーク（Tailwind / CSS Modules / styled-components / Vanilla）
- UI ライブラリ（shadcn/ui / Radix / MUI / Chakra / なし）
- コンポーネント配置パス

#### Step 2: デザイントークン設定

3 つの方法から選択：

| 方法               | 説明                       | おすすめケース                        |
| ------------------ | -------------------------- | ------------------------------------- |
| **Figma から取得** | Figma Variables を自動抽出 | デザイナーが Figma でトークン定義済み |
| **手動で設定**     | 対話形式でヒアリング       | ブランドカラーはあるが Figma 未整備   |
| **デフォルト使用** | 汎用的な初期値で開始       | とりあえず始めたい、後で調整          |

##### Figma から取得する場合

```
FigmaファイルのURLを入力
→ Figma Variables（ローカル変数）を自動抽出
→ カラー / スペーシング / 角丸 / フォントを取得
→ 確認後、tokens.md + tailwind.config.ts を生成
```

抽出対象：

- Colors/Primitives → カラーパレット
- Colors/Semantic → セマンティックカラー（primary, secondary 等）
- Spacing → スペーシング
- Border Radius → 角丸
- Font Size / Font Weight → タイポグラフィ

##### 手動で設定する場合

以下を順にヒアリング：

1. **ブランドカラー**

   - Primary（必須）: HEX 値または色名（例: `#3b82f6`, `青系`）
   - Secondary / Accent（任意）
   - → 指定色から 50〜900 のパレットを自動生成

2. **セマンティックカラー**

   - success / warning / error / info
   - デフォルト値を提示、変更があれば指定

3. **スペーシング**

   - 4px 基準（推奨） / 8px 基準 / カスタム

4. **角丸**

   - シャープ / 標準（推奨） / 丸め / カスタム

5. **タイポグラフィ**
   - フォントファミリー（システム or Google Fonts）
   - フォントサイズ基準（標準 / コンパクト / ゆったり）

#### 生成されるファイル

→ `design_system/` 配下にファイルが生成される

```
design_system/
├── config.md           # 技術スタック・トークンソース情報
├── config.local.md     # Figma URL（git管理対象外）
├── tokens.md           # デザイントークン定義
├── components.md       # コンポーネント一覧
└── figma-mapping.md    # Figma→コードマッピング

# Tailwind選択時は追加で生成/更新
tailwind.config.ts      # テーマ設定

# CSS Modules / styled-components選択時
src/styles/tokens.css   # CSS変数定義
```

⚠️ **重要**: Figma 連携時は `.gitignore` に `*.local.md` を追加してください。

### 3. Figma MCP 設定

Figma 公式のリモート MCP サーバーを使用します。**API Token 不要**で、OAuth 認証のみで利用できます。

参考: [Figma 公式ドキュメント](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/)

#### Claude Code の場合（1 コマンドで完了）

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

認証手順：

1. Claude Code で `/mcp` と入力
2. `figma` を選択 → `Authenticate`
3. ブラウザで Figma にログイン → **Allow Access**
4. `Authentication successful` と表示されれば完了

動作確認：

```
/mcp
# figma: connected と表示されればOK
```

#### VSCode の場合

`Cmd + Shift + P` → `MCP: Open User Configuration` で `mcp.json` を開く：

```json
{
  "servers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "type": "http"
    }
  }
}
```

保存後、`Start` をクリック → ブラウザで認証

#### Cursor の場合

[Figma MCP deep link](cursor://anysphere.cursor-deeplink/mcp/install?name=Figma&config=eyJ1cmwiOiJodHRwczovL21jcC5maWdtYS5jb20vbWNwIn0%3D) をクリックして自動設定

または `設定` → `MCP` で以下を追加：

```json
{
  "figma": {
    "url": "https://mcp.figma.com/mcp",
    "type": "http"
  }
}
```

#### Windsurf の場合

`~/.codeium/windsurf/mcp_config.json` を編集：

```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "type": "http"
    }
  }
}
```

※ MCP なしでもスクリーンショット添付で実装可能

## 使い方

### Figma URL の取得方法

#### ファイル全体の URL

**ブラウザ**: アドレスバーからコピー
**アプリ**: `Cmd + Shift + L`（Mac）/ `Ctrl + Shift + L`（Win）

```
https://www.figma.com/design/ABC123xyz/ProjectName
```

#### 特定のフレーム/コンポーネントの URL（node-id 付き）

1. 対象のフレームを選択
2. 以下のいずれかで取得：
   - ブラウザ: アドレスバーに `node-id=X-Y` が追加される
   - 右クリック → **Copy/Paste as** → **Copy link**
   - ショートカット: `Cmd + L`（Mac）/ `Ctrl + L`（Win）

```
https://www.figma.com/design/ABC123xyz/ProjectName?node-id=123-456
```

💡 **node-id を指定すると、大規模ファイルから特定の要素だけを効率的に取得できます**

### Figma デザインから実装

```
/figma https://www.figma.com/design/xxx/ProjectName
このページを実装して
```

または

```
/figma https://www.figma.com/design/xxx/ProjectName?node-id=123-456
このHeroセクションを実装して
```

スクリーンショットでも可：

```
/figma [スクリーンショット添付]
このデザインを実装して
```

### 実装の流れ

1. Claude がデザインを分析
2. 既存/新規コンポーネントを特定
3. 新規が必要なら計画を提示して確認
4. 承認後に実装
5. `design_system/` を自動更新

## Figma MCP 活用例

### デザイントークンの抽出

```
Figmaファイル（https://www.figma.com/design/xxxxx）の
カラー、タイポグラフィ、スペーシングをCSS変数として出力して
```

### コンポーネントのバリアント生成

```
このボタンコンポーネントを分析して、
Tailwind CSS + CVA でバリアント（サイズ: sm/md/lg、種類: primary/secondary/ghost）を持つ
Reactコンポーネントを実装して
```

### ステート付きコンポーネント

```
このボタンの4つの状態（通常、ホバー、アクティブ、無効）を含む
Reactコンポーネントを生成して
```

### デザイン仕様書の生成

```
このヘッダーコンポーネントの詳細な仕様書
（サイズ、色、間隔、フォントなど）をMarkdownで作成して
```

### デザインの一貫性チェック

```
このデザインのカラーパレットとタイポグラフィの使用を分析し、
デザインシステムから外れている部分があれば指摘して
```

## これで防げる問題

| 問題                 | 対策                         |
| -------------------- | ---------------------------- |
| CSS ベタ書き         | CSS フレームワーク使用を強制 |
| マジックナンバー     | デザイントークン参照を義務化 |
| コンポーネント重複   | 既存一覧確認を必須化         |
| 一貫性のなさ         | デザインシステム定義を参照   |
| ステート未実装       | チェックリストで漏れ防止     |
| Figma とコードの乖離 | Figma 連携で自動同期         |

## Figma トークン再同期

Figma でトークンが更新された場合、再同期できます：

```
/init-frontend sync
```

実行すると：

1. `design_system/config.local.md` から Figma URL を読み込み
2. MCP で Variables を再取得
3. 変更を検出（追加 / 変更 / 削除）
4. 差分を表示して確認
5. 承認後、以下を更新：
   - `design_system/tokens.md`
   - `design_system/figma-mapping.md`
   - `tailwind.config.ts`（Tailwind 使用時）

💡 URL を変更したい場合は `design_system/config.local.md` を編集してください。

## カスタマイズ

### デザイントークン追加

`design_system/tokens.md` を編集：

```markdown
## カラーパレット

| トークン名  | 値        | 用途                       |
| ----------- | --------- | -------------------------- |
| brand-green | `#10b981` | ブランドカラー（新規追加） |
```

### コンポーネント登録

`design_system/components.md` を編集：

```markdown
| ProfileCard | `src/components/ui/ProfileCard.tsx` | ユーザー情報表示 |
```

### Figma マッピング追加

`design_system/figma-mapping.md` を編集：

```markdown
| Avatar/Large | `<Avatar size="lg">` | |
```

## トラブルシューティング

### Claude が新しいコンポーネントを作りすぎる

→ `design_system/components.md` を充実させる
→ プロンプトで「既存コンポーネントを優先」と明示

### スタイルがデザインと合わない

→ `design_system/tokens.md` にトークンを追加
→ Figma のデザイントークンと同期（`/init-frontend sync`）

### MCP が動かない

→ `/mcp` で接続状態を確認
→ `figma: connected` でなければ再認証（`Authenticate`）
→ Figma にブラウザでログインできるか確認

### Figma から Variables が取得できない

→ Figma 側でローカル変数（Variables）が定義されているか確認
→ ファイルへのアクセス権限があるか確認
→ MCP なしの場合はスクリーンショットで対応

### トークンの値が Figma と微妙に違う

→ 仕様です。1-2px の差異はデザイントークンの値を優先
→ 完全一致より一貫性を重視する設計方針
