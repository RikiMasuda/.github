# Commit rules (Conventional Commits)

このリポジトリ群では Conventional Commits を採用します。

## フォーマット

```
[#issue-number] tipe(scope): subject [#PR-number]
```

> **注記**: `tipe` はユーザー指定の表記です。一般的な Conventional Commits では `type` と綴りますが、ここでは指定どおりに記載しています。

- `[#issue-number]` は**必須** — 関連する Issue 番号を先頭に記載する（例: `[#42]`）
- `tipe` は必須
- `scope` は任意（影響範囲が明確なときのみ）
- `subject` は必須（短く、命令形で）
- `[#PR-number]` は**任意** — PR がすでに作成されている場合のみ末尾に記載する（例: `[#123]`）

> **ポイント**: Issue 番号は常に必須です。PR 番号は PR 作成前のコミットでは省略してください。

例:

```
[#42] feat: add user search
[#10] fix(api): handle null token [#55]
[#7] docs: update contributing guide
```

## type 一覧

| type       | 用途                                       |
| ---------- | ------------------------------------------ |
| `feat`     | 新機能                                     |
| `fix`      | バグ修正                                   |
| `docs`     | ドキュメントのみの変更                     |
| `refactor` | 仕様変更なしの内部改善                     |
| `test`     | テスト追加 / 修正                          |
| `chore`    | 雑務（依存関係更新、生成物、ツール設定など）|
| `perf`     | パフォーマンス改善                         |
| `build`    | ビルド関連（bundler / packaging など）     |
| `ci`       | CI 関連                                    |
| `revert`   | 差し戻し                                   |

## subject のルール

- 1 行で要点が分かるように短く書く
- 命令形で書く（`add` / `fix` / `update` / `remove` など）
- 末尾に句点・ピリオドは付けない
- 実装詳細ではなく「何を変えたか」を優先する

## 本文（任意だが推奨）

複雑な変更は本文に以下を含める:

- 変更理由（Why）
- 影響範囲
- 互換性・移行手順（必要なら）

## 破壊的変更（Breaking Change）

破壊的変更がある場合は以下のいずれかを使う:

- `type!:` のように `!` を付ける
- 本文に `BREAKING CHANGE:` を含める

例:

```
feat!: remove legacy auth
```
