# GitHub Workflows

Android プロジェクト向けの再利用可能な GitHub Actions ワークフロー集です。

## 利用可能なワークフロー

| ワークフロー | 説明 |
|-------------|------|
| `android-ci-full.yml` | フルCI パイプライン（品質チェック + テスト + セキュリティ + デプロイ） |
| `android-code-quality.yml` | コード品質チェック（Ktlint, Android Lint） |
| `android-test.yml` | ユニットテスト + Koverカバレッジ |
| `android-security-scan.yml` | セキュリティスキャン（Gitleaks, OWASP） |
| `android-deploy-deploygate.yml` | DeployGateへのデプロイ |

## 使用方法

### フルパイプライン（推奨）

```yaml
# .github/workflows/ci.yml
name: Android CI

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]

jobs:
  ci:
    uses: <your-username>/github-workflows/.github/workflows/android-ci-full.yml@main
    with:
      java-version: '17'
      deploy-module: 'app'           # または 'mobile', 'wear' など
      deploygate-owner: 'YourOwner'
    secrets: inherit
```

### 個別ワークフローの使用

#### コード品質チェックのみ

```yaml
jobs:
  quality:
    uses: <your-username>/github-workflows/.github/workflows/android-code-quality.yml@main
    with:
      java-version: '17'
```

#### テストのみ

```yaml
jobs:
  test:
    uses: <your-username>/github-workflows/.github/workflows/android-test.yml@main
    with:
      java-version: '17'
      run-coverage: true
```

#### セキュリティスキャンのみ

```yaml
jobs:
  security:
    uses: <your-username>/github-workflows/.github/workflows/android-security-scan.yml@main
    with:
      java-version: '17'
      run-owasp: true
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 入力パラメータ

### 共通パラメータ

| パラメータ | デフォルト | 説明 |
|-----------|-----------|------|
| `java-version` | `'17'` | Java バージョン |
| `java-distribution` | `'temurin'` | Java ディストリビューション |

### android-ci-full.yml 固有

| パラメータ | デフォルト | 説明 |
|-----------|-----------|------|
| `run-coverage` | `true` | Koverカバレッジレポートを生成 |
| `run-owasp` | `true` | OWASP依存関係チェックを実行 |
| `deploy-module` | `''` | デプロイ対象モジュール（空の場合はスキップ） |
| `deploygate-owner` | `''` | DeployGateオーナー名 |
| `deploy-on-push-only` | `true` | pushイベント時のみデプロイ |

## 必要なシークレット

| シークレット | 必須 | 説明 |
|-------------|:----:|------|
| `GITHUB_TOKEN` | ✅ | GitHub トークン（自動提供） |
| `DEPLOYGATE_API_KEY` | デプロイ時 | DeployGate API キー |
| `KEYSTORE_BASE64` | リリース時 | Base64エンコードされたキーストア |
| `KEYSTORE_PASSWORD` | リリース時 | キーストアパスワード |
| `KEY_ALIAS` | リリース時 | キーエイリアス |
| `KEY_PASSWORD` | リリース時 | キーパスワード |

## セットアップ

1. このリポジトリを自分のGitHubアカウントにプッシュ
2. 各プロジェクトで `.github/workflows/ci.yml` を作成
3. 必要なシークレットを設定

## リポジトリの可視性

再利用可能なワークフローを使用するには、このリポジトリが以下のいずれかである必要があります：
- **Public** リポジトリ
- 同じ Organization 内の **Internal** または **Private** リポジトリ（GitHub Enterprise）
