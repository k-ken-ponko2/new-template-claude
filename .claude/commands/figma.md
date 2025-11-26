# Figma デザイン実装

Figma のデザインをコードに MCP または手動で変換する際のワークフローを定義します。

## 前提条件

- `/init-frontend` でフロントエンド環境が初期化済みであること
- `design_system/` 配下のファイルが存在すること
- Figma MCP が設定済みであること（MCP 経由の場合）

---

## 実行フロー

### Step 0: Figma URL の確認

1. まず `design_system/config.local.md` が存在するか確認
2. 存在する場合、`FIGMA_URL` を読み込んでデフォルトとして使用
3. ユーザーが URL を指定した場合はそちらを優先

```markdown
# config.local.md から読み込み例

FIGMA_URL=https://www.figma.com/design/xxx/ProjectName
FIGMA_NODE_ID=123-456（任意）
```

### Step 1: デザイン情報の取得

#### パターン 1: URL 指定なし（config.local.md を使用）

```
ユーザー: /figma このページを実装して

Claude: config.local.md から Figma URL を読み込みました。
        https://www.figma.com/design/xxx/ProjectName
        このファイルからデザイン情報を取得します。
```

#### パターン 2: URL 指定あり（指定 URL を優先）

```
ユーザー: /figma https://www.figma.com/design/yyy/AnotherProject?node-id=456-789
```

#### パターン 3: スクリーンショット（MCP 不要）

MCP が設定されていない場合や、簡易的に実装したい場合に使用。

##### スクリーンショットの取得方法

**方法 A: Figma 内でコピー（推奨）**

1. フレームを選択
2. 右クリック → **Copy/Paste as** → **Copy as PNG**
3. Claude に貼り付け

**方法 B: OS 標準のスクリーンショット**

- Mac: `Cmd + Shift + 4` → 範囲選択
- Windows: `Win + Shift + S` → 範囲選択

**方法 C: Figma からエクスポート**

1. フレームを選択
2. 右パネル → **Export** → PNG/JPG
3. ファイルを Claude にアップロード

##### 使用例

```
ユーザー: /figma [スクリーンショット添付] このデザインを実装して
```

##### 補足情報の提供（推奨）

スクリーンショットでは以下が**推測**になるため、可能なら補足すると精度が向上：

| 項目              | 推測になる理由                  |
| ----------------- | ------------------------------- |
| カラーコード      | 画像から正確な HEX 値は取得困難 |
| スペーシング      | px 値の正確な判定が難しい       |
| フォント情報      | フォント名・サイズ・ウェイト    |
| ホバー/フォーカス | 静止画では状態変化が見えない    |

##### 補足情報付きの依頼例

```
ユーザー: /figma [スクリーンショット添付]

このログインフォームを実装して。

補足情報:
- Primary色: #3b82f6
- 背景色: #f8fafc
- 角丸: 8px
- フォント: Inter
- ボタンホバー時: 少し暗く

技術スタック:
- React + Tailwind CSS
- バリデーションはzod
- 既存の Button, Input コンポーネントを使用
```

##### MCP vs スクリーンショット 比較

| 項目           | MCP 経由                         | スクリーンショット     |
| -------------- | -------------------------------- | ---------------------- |
| セットアップ   | 必要（1 コマンド）               | **不要**               |
| 正確な数値     | ✅ 取得可能                      | ❌ 推測                |
| Variables 取得 | ✅ 可能                          | ❌ 不可                |
| 手軽さ         | 初期設定後は楽                   | **すぐ使える**         |
| 適したケース   | 正確な実装、デザインシステム構築 | プロトタイプ、ラフ実装 |

##### スクリーンショット使用時の Claude の対応

1. 画像を解析してレイアウト・配色・構成を把握
2. `design_system/tokens.md` を参照して近いトークンを適用
3. `design_system/components.md` で既存コンポーネントを確認
4. 不明な値は妥当な推測値を使用（後で調整可能な形で実装）
5. 推測した値はコメントで明示

```tsx
// 例: 推測値をコメントで明示
export function Card({ children }: CardProps) {
  return (
    <div
      className={cn(
        "bg-white rounded-lg p-6", // padding: 推測値（Figmaで要確認）
        "shadow-md", // shadow: 画像から推測
        "border border-gray-200"
      )}
    >
      {children}
    </div>
  );
}
```

### Step 2: デザイン分析

取得したデザインを分析し、以下を特定：

1. **使用コンポーネント**: 既存コンポーネントとのマッチング
2. **新規コンポーネント**: 作成が必要なもの
3. **デザイントークン**: 使用されている色・スペーシング・タイポグラフィ

出力例：

```markdown
## デザイン分析結果

### 既存コンポーネントで対応

- ヘッダー部分 → `<Button variant="primary">`
- カード部分 → `<Card>`

### 新規作成が必要

- `ProfileAvatar` - ユーザーアバター表示（既存にない）
- `StatCard` - 統計表示カード（Card の派生だが専用化推奨）

### 確認事項

- カラー `#6366f1` はトークン未定義 → `primary-500` として追加？
- 24px のスペーシングが多用 → `space-lg` として定義済み
```

### Step 3: 実装計画の提示

新規コンポーネント作成前に、ユーザーに確認を求める：

```markdown
## 実装計画

### 新規コンポーネント

1. `ProfileAvatar.tsx` - サイズ: sm/md/lg、fallback 対応
2. `StatCard.tsx` - Card を拡張、アイコン+数値+ラベル

### デザイントークン追加

- `tokens.md` に以下を追加：
  - `avatar-sm`: 32px
  - `avatar-md`: 48px
  - `avatar-lg`: 64px

### 進め方

上記の計画で実装を開始してよろしいですか？
修正があればお知らせください。
```

### Step 4: 実装

承認後、以下の順序で実装：

1. **デザイントークン更新**: 必要に応じて `design_system/tokens.md` を更新
2. **新規コンポーネント作成**: `design_system/components.md` に登録しながら作成
3. **ページ/機能の実装**: 上記コンポーネントを使用して構築
4. **Figma マッピング更新**: 新規コンポーネントを `figma-mapping.md` に追加

---

## 実装ルール

### コンポーネント作成時

```typescript
// ✅ Good: Props型定義、デフォルト値、コメント
interface ProfileAvatarProps {
  /** アバター画像URL */
  src?: string;
  /** ユーザー名（fallback表示用） */
  name: string;
  /** サイズ */
  size?: 'sm' | 'md' | 'lg';
}

export function ProfileAvatar({
  src,
  name,
  size = 'md'
}: ProfileAvatarProps) {
  // 実装
}

// ❌ Bad: 型なし、マジックナンバー
export function ProfileAvatar({ src, name, size }) {
  return <img style={{ width: 48 }} ... />
}
```

### スタイリング時

#### Tailwind CSS の場合

```typescript
// ✅ Good: トークンベース
const sizeClasses = {
  sm: "w-8 h-8", // avatar-sm: 32px
  md: "w-12 h-12", // avatar-md: 48px
  lg: "w-16 h-16", // avatar-lg: 64px
};

// ❌ Bad: 任意値
<div className="w-[48px] h-[48px]" />;
```

#### CSS Modules の場合

```css
/* ✅ Good: CSS変数使用 */
.avatar {
  width: var(--avatar-md);
  height: var(--avatar-md);
}

/* ❌ Bad: ハードコード */
.avatar {
  width: 48px;
  height: 48px;
}
```

### Figma との差異対応

| 状況                 | 対応                                 |
| -------------------- | ------------------------------------ |
| 1-2px のずれ         | デザイントークンの最も近い値を使用   |
| 未定義の色           | トークン追加を提案、承認後に実装     |
| 複雑なアニメーション | 基本動作を先に実装、アニメは後で追加 |
| レスポンシブ指定なし | 標準ブレークポイントで適切に対応     |

---

## ステート実装チェックリスト

Figma に明示されていなくても、以下のステートを実装すること：

- [ ] **Default**: 通常状態
- [ ] **Hover**: マウスオーバー時
- [ ] **Focus**: フォーカス時（キーボードナビゲーション）
- [ ] **Active**: クリック/タップ中
- [ ] **Disabled**: 非活性状態
- [ ] **Loading**: 読み込み中（該当する場合）
- [ ] **Error**: エラー状態（該当する場合）

---

## 完了時の更新

実装完了後、以下のファイルを更新：

1. **design_system/components.md**: 新規コンポーネントを追加
2. **design_system/tokens.md**: 追加したトークンを記載
3. **design_system/figma-mapping.md**: マッピング表を更新

---

## 実行例

### 例 1: Figma URL 指定

```
ユーザー: /figma https://www.figma.com/design/abc123/MyApp?node-id=1-100
         ユーザープロフィールページを実装して
```

Claude:

> Figma デザインを分析しています...
>
> ## デザイン分析結果
>
> ### 既存コンポーネントで対応
>
> - ページタイトル → `<h1 className="text-2xl font-bold">`
> - 編集ボタン → `<Button variant="secondary">`
> - 情報カード → `<Card>`
>
> ### 新規作成が必要
>
> - `ProfileHeader` - アバター + 名前 + ステータス
> - `InfoRow` - ラベル + 値の行表示
>
> 上記の計画で実装を進めてよろしいですか？

ユーザー: OK

Claude:

> 実装を開始します...
> [コンポーネント作成・ページ実装]
>
> ✅ 完了しました
>
> 作成ファイル:
>
> - src/components/ui/ProfileHeader.tsx
> - src/components/ui/InfoRow.tsx
> - src/pages/profile.tsx
>
> 更新ファイル:
>
> - design_system/components.md（2 件追加）
> - design_system/figma-mapping.md（2 件追加）

### 例 2: スクリーンショット指定

```
ユーザー: /figma [画像添付] このモーダルを実装して
```

Claude:

> 添付画像を分析しています...
>
> ## デザイン分析結果
>
> 確認モーダルのデザインですね。
>
> ### 既存コンポーネントで対応
>
> - モーダル枠 → `<Modal>`
> - キャンセルボタン → `<Button variant="ghost">`
> - 確認ボタン → `<Button variant="destructive">`
>
> ### 新規作成不要
>
> 既存コンポーネントの組み合わせで実装可能です。
>
> 実装を進めてよろしいですか？
