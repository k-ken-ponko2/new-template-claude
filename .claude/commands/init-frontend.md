# フロントエンド環境初期化

プロジェクトのフロントエンド開発環境を対話形式で初期化します。

## 実行フロー

### Step 1: 技術スタック選択

以下の選択肢をユーザーに提示し、選択を求める：

#### CSS フレームワーク

1. **Tailwind CSS** - ユーティリティファースト、最も推奨
2. **CSS Modules** - スコープ付き CSS、シンプル
3. **styled-components** - CSS-in-JS、動的スタイリング向け
4. **Vanilla CSS** - フレームワークなし

#### UI コンポーネントライブラリ

1. **shadcn/ui** - Radix ベース、カスタマイズ性高（Tailwind 必須）
2. **Radix UI** - ヘッドレス、スタイル自由
3. **MUI (Material UI)** - 豊富なコンポーネント
4. **Chakra UI** - アクセシビリティ重視
5. **なし** - 自前で構築

#### コンポーネント配置

デフォルト: `src/components/ui/`
カスタム指定も可能

---

### Step 2: デザイントークン設定

技術スタック選択後、デザイントークンの設定方法を確認：

#### 質問

```
デザイントークン（カラー・スペーシング等）の設定方法を選択してください：

1. **Figmaから取得** - Figma VariablesのURLを指定
2. **手動で設定** - 対話形式でカラーパレット等を入力
3. **デフォルトを使用** - 汎用的な初期値で開始（後から変更可能）
```

---

### Step 2-A: Figma からトークン取得

#### 前提条件：Figma MCP のセットアップ

Figma 公式のリモート MCP サーバーを使用します。**API Token 不要**で、OAuth 認証のみで利用できます。

参考: [Figma 公式ドキュメント](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/)

##### Claude Code の場合（1 コマンドで完了）

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

認証手順：

1. Claude Code で `/mcp` と入力
2. `figma` を選択 → `Authenticate`
3. ブラウザで Figma にログイン → **Allow Access**
4. `Authentication successful` と表示されれば完了

##### VSCode / Cursor / Windsurf の場合

各エディタの MCP 設定に以下を追加：

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

##### 動作確認

Claude Code の場合：

```
/mcp
# figma: connected と表示されればOK
```

---

#### Figma URL の保存（config.local.md）

Figma URL は git 管理対象外の `config.local.md` に保存します。

##### .gitignore に追加（初回のみ）

```bash
# .gitignore
*.local.md
```

##### config.local.md の作成

```markdown
# design_system/config.local.md

## Figma 連携（このファイルは git 管理対象外）

FIGMA_URL=https://www.figma.com/design/xxxxx/ProjectName
```

以降、`/figma` や `/init-frontend sync` 実行時は `config.local.md` を自動参照します。

---

#### Figma URL の指定方法

```
FigmaファイルのURLを教えてください：
例: https://www.figma.com/design/xxxxx/ProjectName
```

**特定のフレーム/コンポーネントを指定する場合（推奨）**:

node-id 付き URL を使用すると、大規模ファイルから特定の要素だけを効率的に取得できます。

```
https://www.figma.com/design/xxxxx/ProjectName?node-id=123-456
```

**node-id の取得方法**:

1. Figma で対象のフレームを選択
2. 以下のいずれかで取得：
   - ブラウザ: アドレスバーに `node-id=X-Y` が追加される
   - 右クリック → **Copy/Paste as** → **Copy link**
   - ショートカット: `Cmd + L`（Mac）/ `Ctrl + L`（Win）

※ Figma Variables（ローカル変数）が定義されているファイルを指定してください
※ ファイルへのアクセス権限が必要です

#### 取得するトークン

Figma Variables から以下を抽出：

| Figma Collection    | 生成先               |
| ------------------- | -------------------- |
| Colors / Primitives | カラーパレット       |
| Colors / Semantic   | セマンティックカラー |
| Spacing             | スペーシング         |
| Border Radius       | 角丸                 |
| Font Size           | フォントサイズ       |
| Font Weight         | フォントウェイト     |

#### 抽出フロー

1. Figma MCP または API で Variables を取得
2. Collection/Mode ごとに整理
3. CSS フレームワークに応じた形式に変換
4. `design_system/tokens.md` に出力
5. Tailwind 選択時は `tailwind.config.js` のテーマ設定も生成

#### Figma トークン取得時の出力例

```markdown
## Figma から取得したトークン

### カラー（Colors/Primitives）

| Figma 変数名 | 値      | Tailwind Class                 |
| ------------ | ------- | ------------------------------ |
| blue/500     | #3b82f6 | `text-blue-500`, `bg-blue-500` |
| blue/600     | #2563eb | `text-blue-600`, `bg-blue-600` |
| gray/100     | #f3f4f6 | `text-gray-100`, `bg-gray-100` |

### カラー（Colors/Semantic）

| Figma 変数名 | 参照先   | 用途             |
| ------------ | -------- | ---------------- |
| primary      | blue/500 | メインアクション |
| secondary    | gray/500 | サブアクション   |
| destructive  | red/500  | 削除・警告       |
| background   | white    | 背景色           |
| foreground   | gray/900 | テキスト色       |

### スペーシング

| Figma 変数名 | 値   | Tailwind Class |
| ------------ | ---- | -------------- |
| space/1      | 4px  | `p-1`, `m-1`   |
| space/2      | 8px  | `p-2`, `m-2`   |
| space/4      | 16px | `p-4`, `m-4`   |
| space/6      | 24px | `p-6`, `m-6`   |

上記のトークンで初期化してよろしいですか？
修正があれば指定してください。
```

---

### Step 2-B: 手動でトークン設定

対話形式で以下をヒアリング：

#### 2-B-1: ブランドカラー

```
プロジェクトのブランドカラーを教えてください：

**Primary（メインカラー）**
- HEX値を入力（例: #3b82f6）
- または色名で指定（例: 青系、緑系）

**Secondary（サブカラー）**（任意）
- HEX値または色名

**Accent（アクセントカラー）**（任意）
- HEX値または色名
```

#### 2-B-2: カラーパレット生成

ブランドカラーから自動でパレットを生成：

```
Primary: #3b82f6 から以下のパレットを生成しました：

| シェード | 値 | 用途 |
|----------|-----|------|
| primary-50 | #eff6ff | 背景（薄） |
| primary-100 | #dbeafe | 背景 |
| primary-200 | #bfdbfe | ボーダー（薄） |
| primary-300 | #93c5fd | ボーダー |
| primary-400 | #60a5fa | ホバー |
| primary-500 | #3b82f6 | デフォルト ← 指定色 |
| primary-600 | #2563eb | アクティブ |
| primary-700 | #1d4ed8 | 強調 |
| primary-800 | #1e40af | テキスト（濃） |
| primary-900 | #1e3a8a | テキスト |

このパレットでよろしいですか？調整が必要な場合は指定してください。
```

#### 2-B-3: セマンティックカラー

```
以下のセマンティックカラーを確認してください：

| 用途 | デフォルト値 | 変更しますか？ |
|------|-------------|----------------|
| success（成功） | #22c55e（緑） | |
| warning（警告） | #f59e0b（黄） | |
| error（エラー） | #ef4444（赤） | |
| info（情報） | #3b82f6（青） | |

変更がある場合は「error: #dc2626」のように指定してください。
なければ「OK」と回答してください。
```

#### 2-B-4: スペーシング

```
スペーシングの基準値を選択してください：

1. **4px基準**（推奨） - 4, 8, 12, 16, 20, 24, 32, 40, 48, 64
2. **8px基準** - 8, 16, 24, 32, 40, 48, 64, 80, 96
3. **カスタム** - 独自の値を指定
```

#### 2-B-5: 角丸（Border Radius）

```
角丸のスタイルを選択してください：

1. **シャープ** - 0px, 2px, 4px, 8px
2. **標準**（推奨） - 0px, 4px, 8px, 12px, 16px, 9999px（pill）
3. **丸め** - 0px, 8px, 12px, 16px, 24px, 9999px（pill）
4. **カスタム** - 独自の値を指定
```

#### 2-B-6: タイポグラフィ

```
フォント設定を確認してください：

**フォントファミリー**
- 本文: system-ui, sans-serif（デフォルト）
- 見出し: 本文と同じ / 別フォントを指定
- コード: ui-monospace, monospace（デフォルト）

カスタムフォントを使用しますか？（Google Fonts等）
- 使用する場合はフォント名を指定（例: Inter, Noto Sans JP）
- 使用しない場合は「デフォルト」と回答
```

```
フォントサイズの基準を選択してください：

1. **標準**（推奨）
   - xs: 12px, sm: 14px, base: 16px, lg: 18px, xl: 20px, 2xl: 24px, 3xl: 30px

2. **コンパクト**
   - xs: 11px, sm: 13px, base: 14px, lg: 16px, xl: 18px, 2xl: 20px, 3xl: 24px

3. **ゆったり**
   - xs: 14px, sm: 16px, base: 18px, lg: 20px, xl: 24px, 2xl: 30px, 3xl: 36px
```

---

### Step 2-C: デフォルトを使用

汎用的な初期値で `design_system/tokens.md` を生成：

```markdown
# デザイントークン（デフォルト）

以下のデフォルト値で初期化します。
プロジェクトに合わせて後から変更可能です。

## カラー

- Primary: #3b82f6（青）
- Secondary: #64748b（グレー）
- Success: #22c55e, Warning: #f59e0b, Error: #ef4444

## スペーシング

- 4px 基準（4, 8, 12, 16, 24, 32, 48, 64）

## 角丸

- 標準（0, 4, 8, 12, 16, full）

## タイポグラフィ

- システムフォント
- 標準サイズ（12-30px）
```

---

### Step 3: 設定ファイル生成

選択・入力内容に基づき、以下のファイルを生成：

1. **design_system/config.md** - 選択した技術スタック・トークンソース
2. **design_system/tokens.md** - デザイントークン定義
3. **design_system/components.md** - コンポーネント一覧テンプレート
4. **design_system/figma-mapping.md** - Figma マッピングルール

#### Tailwind 選択時の追加生成

```typescript
// tailwind.config.ts（または.js）に追加するテーマ設定

export default {
  theme: {
    extend: {
      colors: {
        primary: {
          50: "#eff6ff",
          100: "#dbeafe",
          // ... Figmaまたは手動設定から生成
          500: "#3b82f6",
          // ...
        },
        secondary: {
          /* ... */
        },
        destructive: {
          /* ... */
        },
      },
      spacing: {
        // Figmaまたは手動設定から生成
      },
      borderRadius: {
        // Figmaまたは手動設定から生成
      },
      fontFamily: {
        // 指定されたフォント
      },
    },
  },
};
```

#### CSS Modules / styled-components 選択時の追加生成

```css
/* src/styles/tokens.css */
:root {
  /* Colors - Primary */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  /* ... */
  --color-primary-500: #3b82f6;
  /* ... */

  /* Colors - Semantic */
  --color-success: #22c55e;
  --color-warning: #f59e0b;
  --color-error: #ef4444;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  /* ... */

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  /* ... */

  /* Typography */
  --font-sans: system-ui, sans-serif;
  --font-mono: ui-monospace, monospace;
  --font-size-sm: 14px;
  --font-size-base: 16px;
  /* ... */
}
```

---

### Step 4: CLAUDE.md 更新

CLAUDE.md に以下のセクションを追加：

- フロントエンド規約
- 選択した技術スタック固有のルール
- デザイントークンの参照方法
- コンポーネント参照パス

---

### Step 5: 設定内容の確認

生成完了後、設定内容のサマリーを表示：

```markdown
## 初期化完了

### 技術スタック

- CSS: Tailwind CSS
- UI: shadcn/ui
- コンポーネント配置: src/components/ui/

### デザイントークン

- ソース: Figma（https://www.figma.com/design/xxxxx）
- カラー: 5 collections, 48 tokens
- スペーシング: 10 tokens
- 角丸: 6 tokens
- フォント: Inter, Noto Sans JP

### 生成ファイル

- ✅ design_system/config.md
- ✅ design_system/config.local.md（Figma 連携時、git 管理対象外）
- ✅ design_system/tokens.md
- ✅ design_system/components.md
- ✅ design_system/figma-mapping.md
- ✅ tailwind.config.ts（テーマ追記）
- ✅ CLAUDE.md（フロントエンド規約追記）
- ✅ .gitignore（\*.local.md 追加）

### 次のステップ

1. `design_system/tokens.md` を確認し、必要に応じて調整
2. 基本コンポーネント（Button, Input 等）を作成
3. `/figma` でデザインから実装を開始
```

---

## 生成テンプレート

### design_system/config.md

```markdown
# フロントエンド設定

## 技術スタック

- **CSS フレームワーク**: {選択したフレームワーク}
- **UI ライブラリ**: {選択したライブラリ}
- **コンポーネント配置**: {指定したパス}

## デザイントークン

- **ソース**: Figma / 手動設定 / デフォルト
- **Figma 連携**: `config.local.md` を参照（git 管理対象外）
- **最終同期**: {YYYY-MM-DD HH:MM}（Figma 連携時）

## 初期化日

{YYYY-MM-DD}
```

### design_system/config.local.md（Figma 連携時のみ生成）

⚠️ このファイルは git 管理対象外です（`.gitignore` に `*.local.md` を追加）

```markdown
# Figma 連携設定（このファイルは git 管理対象外）

## Figma URL

FIGMA_URL=https://www.figma.com/design/xxxxx/ProjectName

## 特定フレーム（任意）

FIGMA_NODE_ID=123-456

## メモ

- このファイルは `.gitignore` で除外されています
- `/figma` や `/init-frontend sync` 実行時に自動参照されます
- URL を変更したい場合はこのファイルを編集してください
```

### design_system/tokens.md

```markdown
# デザイントークン

> ソース: {Figma URL / 手動設定 / デフォルト}
> 最終更新: {YYYY-MM-DD}

## カラーパレット

### プリミティブカラー

| トークン名 | 値      | {CSS Framework 固有}           |
| ---------- | ------- | ------------------------------ |
| blue-50    | #eff6ff | `text-blue-50`, `bg-blue-50`   |
| blue-100   | #dbeafe | `text-blue-100`, `bg-blue-100` |
| ...        | ...     | ...                            |

### セマンティックカラー

| トークン名  | 参照/値  | 用途             | {CSS Framework 固有}         |
| ----------- | -------- | ---------------- | ---------------------------- |
| primary     | blue-500 | メインアクション | `text-primary`, `bg-primary` |
| secondary   | gray-500 | サブアクション   | `text-secondary`             |
| destructive | red-500  | 削除・警告       | `text-destructive`           |
| muted       | gray-400 | 非活性・補足     | `text-muted`                 |
| background  | white    | 背景             | `bg-background`              |
| foreground  | gray-900 | テキスト         | `text-foreground`            |
| border      | gray-200 | ボーダー         | `border-border`              |

## スペーシング

| トークン名 | 値   | {CSS Framework 固有}  |
| ---------- | ---- | --------------------- |
| space-1    | 4px  | `p-1`, `m-1`, `gap-1` |
| space-2    | 8px  | `p-2`, `m-2`, `gap-2` |
| space-3    | 12px | `p-3`, `m-3`, `gap-3` |
| space-4    | 16px | `p-4`, `m-4`, `gap-4` |
| space-6    | 24px | `p-6`, `m-6`, `gap-6` |
| space-8    | 32px | `p-8`, `m-8`, `gap-8` |

## 角丸（Border Radius）

| トークン名  | 値     | {CSS Framework 固有} |
| ----------- | ------ | -------------------- |
| radius-none | 0px    | `rounded-none`       |
| radius-sm   | 4px    | `rounded-sm`         |
| radius-md   | 8px    | `rounded-md`         |
| radius-lg   | 12px   | `rounded-lg`         |
| radius-xl   | 16px   | `rounded-xl`         |
| radius-full | 9999px | `rounded-full`       |

## タイポグラフィ

### フォントファミリー

| トークン名 | 値                           | {CSS Framework 固有} |
| ---------- | ---------------------------- | -------------------- |
| font-sans  | Inter, system-ui, sans-serif | `font-sans`          |
| font-mono  | ui-monospace, monospace      | `font-mono`          |

### フォントサイズ

| トークン名 | 値   | 行間 | {CSS Framework 固有} |
| ---------- | ---- | ---- | -------------------- |
| text-xs    | 12px | 16px | `text-xs`            |
| text-sm    | 14px | 20px | `text-sm`            |
| text-base  | 16px | 24px | `text-base`          |
| text-lg    | 18px | 28px | `text-lg`            |
| text-xl    | 20px | 28px | `text-xl`            |
| text-2xl   | 24px | 32px | `text-2xl`           |
| text-3xl   | 30px | 36px | `text-3xl`           |

### フォントウェイト

| トークン名    | 値  | {CSS Framework 固有} |
| ------------- | --- | -------------------- |
| font-normal   | 400 | `font-normal`        |
| font-medium   | 500 | `font-medium`        |
| font-semibold | 600 | `font-semibold`      |
| font-bold     | 700 | `font-bold`          |

## シャドウ

| トークン名 | 値                          | {CSS Framework 固有} |
| ---------- | --------------------------- | -------------------- |
| shadow-sm  | 0 1px 2px rgba(0,0,0,0.05)  | `shadow-sm`          |
| shadow-md  | 0 4px 6px rgba(0,0,0,0.1)   | `shadow-md`          |
| shadow-lg  | 0 10px 15px rgba(0,0,0,0.1) | `shadow-lg`          |
```

### design_system/components.md

```markdown
# コンポーネント一覧

## 登録済みコンポーネント

新規実装前に必ず確認し、既存コンポーネントを再利用すること。

| コンポーネント | パス                           | Props                              | 用途               |
| -------------- | ------------------------------ | ---------------------------------- | ------------------ |
| Button         | `{components_path}/Button.tsx` | variant, size, disabled, loading   | アクションボタン   |
| Input          | `{components_path}/Input.tsx`  | type, placeholder, error, disabled | テキスト入力       |
| Card           | `{components_path}/Card.tsx`   | -                                  | コンテンツラッパー |
| Modal          | `{components_path}/Modal.tsx`  | open, onClose, title               | ダイアログ         |

## コンポーネント追加時のルール

1. 上記テーブルに追記すること
2. Storybook があれば対応するストーリーを作成
3. 基本的な Props の型定義を含めること
4. `design_system/figma-mapping.md` にマッピングを追加
```

### design_system/figma-mapping.md

```markdown
# Figma → コード マッピングルール

> Figma ソース: {Figma URL}
> 最終同期: {YYYY-MM-DD}

## コンポーネントマッピング

| Figma Component    | Code Component                   | 備考 |
| ------------------ | -------------------------------- | ---- |
| Button/Primary     | `<Button variant="primary">`     |      |
| Button/Secondary   | `<Button variant="secondary">`   |      |
| Button/Ghost       | `<Button variant="ghost">`       |      |
| Button/Destructive | `<Button variant="destructive">` |      |
| Input/Default      | `<Input>`                        |      |
| Input/Error        | `<Input error>`                  |      |
| Card               | `<Card>`                         |      |
| Modal              | `<Modal>`                        |      |

## カラーマッピング

| Figma Variable     | トークン    | コード                               |
| ------------------ | ----------- | ------------------------------------ |
| Colors/Primary     | primary     | `text-primary`, `bg-primary`         |
| Colors/Secondary   | secondary   | `text-secondary`, `bg-secondary`     |
| Colors/Destructive | destructive | `text-destructive`, `bg-destructive` |
| Colors/Muted       | muted       | `text-muted`, `bg-muted`             |
| Colors/Background  | background  | `bg-background`                      |
| Colors/Foreground  | foreground  | `text-foreground`                    |
| Colors/Border      | border      | `border-border`                      |

## スペーシングマッピング

| Figma Variable | トークン | コード                |
| -------------- | -------- | --------------------- |
| Spacing/4      | space-1  | `p-1`, `m-1`, `gap-1` |
| Spacing/8      | space-2  | `p-2`, `m-2`, `gap-2` |
| Spacing/16     | space-4  | `p-4`, `m-4`, `gap-4` |
| Spacing/24     | space-6  | `p-6`, `m-6`, `gap-6` |

## 角丸マッピング

| Figma Variable | トークン    | コード         |
| -------------- | ----------- | -------------- |
| Radius/Small   | radius-sm   | `rounded-sm`   |
| Radius/Medium  | radius-md   | `rounded-md`   |
| Radius/Large   | radius-lg   | `rounded-lg`   |
| Radius/Full    | radius-full | `rounded-full` |

## 実装時の注意

1. **トークン優先**: Figma の値がトークンと 1-2px 異なる場合、トークンの値を使用
2. **ステート実装必須**: hover, focus, disabled, loading 状態を必ず実装
3. **レスポンシブ対応**: Figma にない場合も適切なブレークポイントを設定
4. **新規マッピング**: 追加時は上記テーブルを更新すること
```

---

## Figma トークン再同期

初期化後に Figma のトークンが更新された場合、再同期が可能：

```
ユーザー: /init-frontend sync

Claude:
> Figmaトークンを再同期します。
> 前回の同期: 2024-01-15
> Figma URL: https://www.figma.com/design/xxxxx
>
> 変更を検出しました：
> - 追加: Colors/Accent（#8b5cf6）
> - 変更: Spacing/6（20px → 24px）
> - 削除: Colors/Tertiary
>
> 以下のファイルを更新しますか？
> - design_system/tokens.md
> - design_system/figma-mapping.md
> - tailwind.config.ts
```

---

## 実行例

### 例 1: Figma 連携

```
ユーザー: /init-frontend

Claude:
> フロントエンド環境を初期化します。
>
> **1. CSSフレームワークを選択してください:**
> 1. Tailwind CSS（推奨）
> 2. CSS Modules
> 3. styled-components
> 4. Vanilla CSS

ユーザー: 1

Claude:
> **2. UIコンポーネントライブラリを選択してください:**
> 1. shadcn/ui（Tailwind必須）
> 2. Radix UI
> 3. MUI
> 4. Chakra UI
> 5. なし（自前構築）

ユーザー: 1

Claude:
> **3. コンポーネントの配置パスを指定してください:**
> デフォルト: `src/components/ui/`

ユーザー: デフォルトで

Claude:
> **4. デザイントークンの設定方法を選択してください:**
> 1. Figmaから取得
> 2. 手動で設定
> 3. デフォルトを使用

ユーザー: 1

Claude:
> FigmaファイルのURLを教えてください：

ユーザー: https://www.figma.com/design/abc123/MyDesignSystem

Claude:
> Figmaからトークンを取得しています...
>
> ## 取得したトークン
>
> ### カラー（32 tokens）
> | 変数名 | 値 |
> |--------|-----|
> | primary | #6366f1 |
> | secondary | #64748b |
> | ... | ... |
>
> ### スペーシング（8 tokens）
> | 変数名 | 値 |
> |--------|-----|
> | space-1 | 4px |
> | ... | ... |
>
> 上記のトークンで初期化してよろしいですか？

ユーザー: OK

Claude:
> ファイルを生成しています...
> ✅ design_system/config.md
> ✅ design_system/tokens.md
> ✅ design_system/components.md
> ✅ design_system/figma-mapping.md
> ✅ tailwind.config.ts（テーマ追記）
> ✅ CLAUDE.md（フロントエンド規約追記）
>
> 初期化が完了しました！
```

### 例 2: 手動設定

```
ユーザー: /init-frontend

[Step 1-3は省略]

Claude:
> **4. デザイントークンの設定方法を選択してください:**
> 1. Figmaから取得
> 2. 手動で設定
> 3. デフォルトを使用

ユーザー: 2

Claude:
> ブランドカラーを教えてください：
>
> **Primary（メインカラー）**
> HEX値または色名で指定

ユーザー: #10b981（緑系）

Claude:
> Primary: #10b981 から以下のパレットを生成しました：
>
> | シェード | 値 |
> |----------|-----|
> | primary-50 | #ecfdf5 |
> | primary-100 | #d1fae5 |
> | primary-500 | #10b981 | ← 指定色
> | primary-900 | #064e3b |
>
> このパレットでよろしいですか？
>
> 続いて **Secondary（サブカラー）** を指定しますか？（スキップ可）

ユーザー: スキップ

Claude:
> セマンティックカラーを確認してください：
> - success: #22c55e（緑）
> - warning: #f59e0b（黄）
> - error: #ef4444（赤）
>
> 変更がありますか？

ユーザー: OK

[以降、スペーシング・角丸・タイポグラフィを順に確認]
```
