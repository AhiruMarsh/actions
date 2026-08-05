# actions

GitHub Actions の再利用可能ワークフロー (reusable workflows) を管理するリポジトリです。  
各リポジトリの `.github/workflows/*.yml` から `workflow_call` で呼び出して使用します。

## 構成

```
.github/
├── dependabot.yaml          # GitHub Actions の週次自動更新
└── workflows/
    ├── claude.yaml                    # Claude Code 連携
    ├── generate-terraform-docs.yaml   # terraform-docs の生成・PRへの反映
    └── release.yaml                   # 年月ベースのバージョニングでリリース作成
```

## ワークフロー一覧

### release.yaml — リリース作成

`vYYYY.M.N` 形式（年.月.当月内の連番）でタグを採番し、GitHub Release を自動生成します。  
同じ年月内での2回目以降のリリースは連番 (`N`) がインクリメントされ、月が変わると `N` は `0` にリセットされます。

呼び出し側リポジトリでの利用例:

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    uses: AhiruMarsh/actions/.github/workflows/release.yaml@main
    permissions:
      contents: write
```

### generate-terraform-docs.yaml — Terraform ドキュメント生成

[terraform-docs](https://github.com/terraform-docs/terraform-docs) を実行し、生成結果を対象ディレクトリの `README.md` に inject した上で PR ブランチへ push します。

| 入力 | 必須 | 内容 |
|---|---|---|
| `working-dir` | ✅ | terraform-docs を実行するディレクトリ |

呼び出し側リポジトリでの利用例:

```yaml
name: Generate terraform docs

on:
  pull_request:

jobs:
  docs:
    uses: AhiruMarsh/actions/.github/workflows/generate-terraform-docs.yaml@main
    with:
      working-dir: terraform/
    permissions:
      contents: write
```

### claude.yaml — Claude Code 連携

Issue / PR コメントや Issue 本文中の `@claude` メンションをトリガーに [Claude Code](https://github.com/anthropics/claude-code-action) を起動します。  
トリガー元ユーザーが `AhiruMarsh` の場合のみ実行される制限が入っています。

呼び出し側リポジトリでの利用例:

```yaml
name: Claude

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    uses: AhiruMarsh/actions/.github/workflows/claude.yaml@main
    secrets: inherit
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
      actions: read
```

## 依存関係の自動更新

Dependabot により GitHub Actions のバージョンが週次で自動更新されます。
