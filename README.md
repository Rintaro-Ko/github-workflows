# GitHub Workflows

Android プロジェクト向けの再利用可能な GitHub Actions ワークフロー集です。

## 利用可能なワークフロー

| ワークフロー | 説明 |
|-------------|------|
| `android-ci-smart.yml` | スマートCI パイプライン（変更検知 + 品質 + テスト + セキュリティ + Firebase/Play Console デプロイ） |
| `android-code-quality.yml` | コード品質チェック（Ktlint, Android Lint） |
| `android-test.yml` | ユニットテスト + Koverカバレッジ |
| `android-security-scan.yml` | セキュリティスキャン（Gitleaks, OWASP） |
| `android-security-full.yml` | フルセキュリティスキャン（OWASP MASVS準拠） |

## 使用方法

### スマートCIパイプライン（推奨）

`android-ci-smart.yml` は変更検知により必要なジョブのみ実行し、Firebase App Distribution / Play Console Internal Testing へのデプロイを行います。

```yaml
# .github/workflows/android.yml
name: Android CI

on:
  push:
    branches: [ "main", "master", "develop" ]
  pull_request:
    branches: [ "main", "master", "develop" ]

jobs:
  ci:
    uses: Rintaro-Ko/github-workflows/.github/workflows/android-ci-smart.yml@main
    with:
      primary-module: app
      deploy-module: app
      firebase-app-id: ${{ vars.FIREBASE_APP_ID }}
      firebase-tester-groups: MyApptest_Users
      play-package-name: com.example.app
    secrets: inherit
```

### ブランチ戦略

| ブランチ | デプロイ先 | ビルドタイプ |
|---------|-----------|------------|
| `develop` | Firebase App Distribution | Debug APK |
| `main/master` | Play Console Internal Testing | Release AAB |

### 入力パラメータ (android-ci-smart.yml)

| パラメータ | 必須 | デフォルト | 説明 |
|-----------|:----:|-----------|------|
| `primary-module` | ✅ | - | メインモジュール名 (e.g., `app`, `mobile`, `androidApp`) |
| `secondary-module` | - | `''` | セカンダリモジュール名 (e.g., `wear`, `shared`) |
| `tertiary-module` | - | `''` | 第3モジュール名 |
| `deploy-module` | - | `''` | デプロイ対象モジュール（空でスキップ） |
| `firebase-app-id` | - | `''` | Firebase App ID |
| `firebase-tester-groups` | - | `'testers'` | Firebase テスターグループ名 |
| `play-package-name` | - | `''` | Play Console パッケージ名 |
| `pre-test-script` | - | `''` | テスト前実行スクリプト |
| `security-script` | - | `''` | カスタムセキュリティスクリプトのパス |
| `sast-exclude-pattern` | - | `''` | SAST除外パターン |

### 必要なSecrets/Variables

| 名前 | 種別 | 必須 | 説明 |
|------|------|:----:|------|
| `FIREBASE_APP_ID` | Variable | Firebase時 | Firebase App ID |
| `FIREBASE_SERVICE_ACCOUNT` | Secret | Firebase時 | Firebase サービスアカウント JSON |
| `GOOGLE_PLAY_SERVICE_ACCOUNT` | Secret | Play Console時 | Play Console サービスアカウント JSON |
| `KEYSTORE_BASE64` | Secret | リリース時 | Base64エンコードされたキーストア |
| `KEYSTORE_PASSWORD` | Secret | リリース時 | キーストアパスワード |
| `KEY_ALIAS` | Secret | リリース時 | キーエイリアス |
| `KEY_PASSWORD` | Secret | リリース時 | キーパスワード |

## セットアップ

1. このリポジトリを自分のGitHubアカウントにプッシュ
2. 各プロジェクトで `.github/workflows/android.yml` を作成（上記テンプレート参照）
3. Firebase プロジェクト作成 + サービスアカウント設定
4. GitHub リポジトリに Secrets/Variables を設定

## リポジトリの可視性

再利用可能なワークフローを使用するには、このリポジトリが以下のいずれかである必要があります：
- **Public** リポジトリ
- 同じ Organization 内の **Internal** または **Private** リポジトリ（GitHub Enterprise）
