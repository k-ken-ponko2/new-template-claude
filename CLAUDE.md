# Kiro スタイル仕様駆動開発テンプレート

このプロジェクトは Kiro スタイルの仕様駆動開発を採用しています。

## 仕様ファイル

- **specs/requirements.md**: ユーザーストーリーと要件
- **specs/design.md**: 技術アーキテクチャ
- **specs/tasks.md**: 実装タスクと進捗

## 開発フロー

1. 要件定義 → requirements.md に記載
2. 設計 → design.md に記載
3. タスク分割 → tasks.md に記載
4. 実装 → 各タスクを順次実装

## コマンド

- `/kiro`: 新しい機能の仕様を初期化
- `/makeRules`: コーディング規約や開発ルール・デザインルールの作成
- `/init-frontend`: フロントエンド環境の初期化（CSS/UI ライブラリ選択）
- `/figma`: Figma デザインからの実装

## 開発ルール

1. すべての機能は要件定義から始める
2. 要件を承認してから設計に進む
3. 設計を承認してから実装に進む
4. タスクは独立してテスト可能にする
5. KISS（Keep It Simple, Stupid）
   実装はできるだけシンプルに保つ。不要に複雑化しない。
6. YAGNI（You Aren't Gonna Need It）
   現時点で必要ない機能やコードは書かない。将来のための過剰設計を避ける。
7. SOLID 原則
   S: Single Responsibility Principle
   O: Open/Closed Principle
   L: Liskov Substitution Principle
   I: Interface Segregation Principle
   D: Dependency Inversion Principle
8. development_docs・development_docs 直下に .md file があれば、それを参照して code を生成する
9. design_system・design_system 直下に .md file があれば、それを参照して code を生成する
10. Think in English, but generate responses in Japanese (思考は英語、回答の生成は日本語で行うように)

---

## フロントエンド規約

### 初期化

フロントエンド開発を開始する前に `/init-frontend` を実行し、以下を設定すること：

- CSS フレームワーク（Tailwind / CSS Modules / styled-components / Vanilla）
- UI ライブラリ（shadcn/ui / Radix / MUI / Chakra / なし）
- コンポーネント配置パス

設定は `design_system/config.md` に記録される。

### 基本原則

1. **コンポーネント再利用優先**

   - 新規コンポーネント作成前に `design_system/components.md` を必ず確認
   - 既存コンポーネントの拡張・継承で対応可能か検討
   - 3 回以上使用するパターンは共通化を検討

2. **デザイントークン使用**

   - 色・スペーシング・フォントサイズは `design_system/tokens.md` を参照
   - ハードコードした値（マジックナンバー）は禁止
   - 新規トークンが必要な場合は追加提案 → 承認後に実装

3. **Figma マッピング遵守**
   - `design_system/figma-mapping.md` に従ってコンポーネントを使用
   - 1-2px のずれは許容し、トークンの最も近い値を使用
   - 完全一致より一貫性を優先

### 禁止事項

| 禁止                          | 理由               | 代替                     |
| ----------------------------- | ------------------ | ------------------------ |
| インラインスタイル `style={}` | 一貫性が保てない   | CSS フレームワークを使用 |
| マジックナンバー              | 意味不明、変更困難 | デザイントークンを参照   |
| コンポーネントの複製          | 重複コード         | 既存を拡張・Props で対応 |
| 無意味な`div`ネスト           | DOM 肥大化         | セマンティック HTML 使用 |

### ステート実装チェックリスト

インタラクティブ要素には以下のステートを実装すること（Figma に明示がなくても）：

- Default（通常）
- Hover（マウスオーバー）
- Focus（フォーカス時）
- Active（クリック中）
- Disabled（非活性）
- Loading（読み込み中）※該当する場合
- Error（エラー）※該当する場合

### コンポーネント設計

```typescript
// ✅ Good: 型定義、デフォルト値、JSDocコメント
interface ButtonProps {
  /** ボタンのバリアント */
  variant?: "primary" | "secondary" | "ghost" | "destructive";
  /** ボタンのサイズ */
  size?: "sm" | "md" | "lg";
  /** 読み込み中状態 */
  loading?: boolean;
  /** 非活性状態 */
  disabled?: boolean;
  /** クリックハンドラ */
  onClick?: () => void;
  /** 子要素 */
  children: React.ReactNode;
}

export function Button({
  variant = "primary",
  size = "md",
  loading = false,
  disabled = false,
  onClick,
  children,
}: ButtonProps) {
  // 実装
}

// ❌ Bad: any型、マジックナンバー、インラインスタイル
export function Button({ variant, size, onClick, children }: any) {
  return (
    <button
      style={{ padding: 12, backgroundColor: "#3b82f6" }}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

### ファイル配置

```
src/
├── components/
│   └── ui/           # 共通UIコンポーネント（init-frontendで指定）
│       ├── Button.tsx
│       ├── Input.tsx
│       ├── Card.tsx
│       └── index.ts  # barrel export
├── features/         # 機能別コンポーネント
│   └── auth/
│       ├── LoginForm.tsx
│       └── SignupForm.tsx
├── pages/            # ページコンポーネント（またはapp/）
└── styles/           # グローバルスタイル

design_system/        # デザイン定義（コード生成の参照元）
├── config.md         # 技術スタック設定
├── config.local.md   # Figma URL（git管理対象外）
├── tokens.md         # デザイントークン
├── components.md     # コンポーネント一覧
└── figma-mapping.md  # Figma変換ルール
```

⚠️ `*.local.md` は `.gitignore` に追加して git 管理対象外にすること。

---

## Figma 連携ワークフロー

### Figma URL の管理

Figma URL は `design_system/config.local.md` に保存（git 管理対象外）：

```markdown
# design_system/config.local.md

FIGMA_URL=https://www.figma.com/design/xxxxx/ProjectName
FIGMA_NODE_ID=123-456（任意）
```

`/figma` コマンド実行時、URL 指定がなければこのファイルを自動参照。

### `/figma` コマンド使用時のフロー

1. **URL 確認**: `config.local.md` を参照（指定があればそちらを優先）
2. **デザイン分析**: Figma から取得した情報を分析
3. **コンポーネント特定**: 既存/新規の判別
4. **計画提示**: 新規作成が必要な場合は先に確認
5. **承認後に実装**: ユーザー確認を経てから実装開始
6. **ドキュメント更新**: 完了後に `design_system/` 配下を更新

### Figma MCP 設定

Figma 公式のリモート MCP サーバーを使用します。**API Token 不要**で、OAuth 認証のみ。

参考: [Figma 公式ドキュメント](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/)

#### Claude Code（1 コマンドで完了）

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

認証: `/mcp` → `figma` → `Authenticate` → ブラウザでログイン

#### VSCode / Cursor / Windsurf

各エディタの MCP 設定に追加：

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

#### Figma URL の指定

**ファイル全体**: `https://www.figma.com/design/xxxxx/ProjectName`
**特定フレーム**: `https://www.figma.com/design/xxxxx/ProjectName?node-id=123-456`

node-id 付き URL を使うと、大規模ファイルから特定の要素だけを効率的に取得できます。

**node-id の取得**: フレーム選択 → `Cmd + L`（Mac）/ `Ctrl + L`（Win）

MCP なしでもスクリーンショット添付で実装可能。

---

## 仕様変更時の対応

仕様に変更があった場合は、関連するすべての仕様ファイル（requirements.md、design.md、tasks.md）を整合性を保ちながら更新します。

例：

- 「ユーザー認証機能を追加したい」
- 「データベースを PostgreSQL から MongoDB に変更したい」
- 「ダークモード機能は不要になったので削除して」

変更があった場合、以下の対応を行います。

1. requirements.md に要件を追加/修正/削除
2. design.md の設計を要件に合わせて更新
3. tasks.md のタスクを設計に基づいて調整
4. 各ファイル間の整合性を確認
5. **フロントエンド関連の場合**: `design_system/` 配下も更新
