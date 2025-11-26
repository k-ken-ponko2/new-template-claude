$ARGUMENTS の機能について、以下の仕様ファイルを作成してください：

技術スタックの選定に関して
使用言語の default は Typescript
フロント・バックエンドに使用するフレームワークについては対話形式で確認しながら選定を行ってください

1. **specs/requirements.md**

   - ユーザーストーリー形式： "As a [ユーザー], I want [目標] so that [利点]"
   - 各ストーリーに受け入れ基準を含める

2. **specs/design.md**

   - システムアーキテクチャ
   - 技術スタックの選定
   - コンポーネント構成

3. **specs/tasks.md**
   - 実装タスクの一覧
   - 各タスクに[ ] チェックボックスを付ける
   - 依存関係を明記

まず要件を確認してから、他のファイルを作成してください。

4. **ドメイン別履歴自動記録**
   - 機能内容を分析してドメイン分類を決定
   - 適切なドメイン履歴ファイルに詳細記録を追加
   - メイン history.md の統計情報を更新

---

## ドメイン履歴記録テンプレート

### ファイル配置ルール

```
specs/history/{domain}/{feature}.md
```

### 詳細記録テンプレート

```markdown
# {Domain} - {Feature} History

## YYYY-MM-DD - [機能・変更内容]

**Command**: `/kiro $ARGUMENTS`

**Category**: {Domain}
**Phase**: Planning / Development / Testing / Deployment
**Impact**: High / Medium / Low
**Tags**: #{tag1} #{tag2} #{tag3} #{category} #{phase}

### Generated Files

- [作成・更新されたファイル一覧とその内容概要]

### Key Decisions Made

#### **[決定事項 1]**

- [詳細説明]
- [技術的根拠]

#### **[決定事項 2]**

- [詳細説明]
- [技術的根拠]

### Achievements

- ✅ [達成事項 1]
- ✅ [達成事項 2]
- ✅ [達成事項 3]

### Impact Analysis

- **[影響領域 1]**: [具体的影響・改善効果]
- **[影響領域 2]**: [具体的影響・改善効果]
- **[影響領域 3]**: [具体的影響・改善効果]

### Technical Implementation

- [実装詳細・技術仕様]
- [設計判断・制約事項]
- [将来拡張性・考慮事項]

**Status**: ✅ Complete / ⏸️ In Progress / ❌ Blocked

---
```

### メインインデックス更新

```markdown
# specs/history/index.md - Global Statistics 更新

- **Total History Files**: [数値更新]
- **Active Histories**: [ドメインリスト更新]
- **Last Updated**: [日付更新]

# 対象ドメイン overview.md - 統計更新

- Total Entries: [数値更新]
- Last Updated: [日付更新]
- Key Decisions: [主要決定事項追加]
```

## 実行フロー

1. **要件確認**: $ARGUMENTS の機能内容を分析
2. **ドメイン判定**: 機能内容からドメイン分類を動的決定
3. **仕様書作成**: requirements.md → design.md → tasks.md 順に作成
4. **履歴記録**: 詳細テンプレートで該当ドメインに記録
5. **統計更新**: メインインデックスとドメイン overview 更新
