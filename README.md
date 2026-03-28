# GitHub Workflows

Android プロジェクト向けの再利用可能な GitHub Actions ワークフロー集です。全プロジェクトの CI/CD 設定を本リポジトリで一元管理し、各プロジェクトは thin caller（15〜25行）で呼び出します。

## エコシステムにおける位置づけ

本リポジトリは 3 層開発・運用基盤の **CI/CD レイヤー** に位置します。

| レイヤー | リポジトリ / ツール | 役割 |
|---------|-------------------|------|
| 開発 | ローカルスクリプト (`dev-cycle.sh`) | TDD自動サイクル（実装→テスト→Lint） |
| **CI/CD** | **github-workflows**（本リポジトリ） | **Push/PR時のビルド・テスト・セキュリティ・デプロイ** |
| 運用 | [claude-ops](https://github.com/Rintaro-Ko/claude-ops) | 24/7 監視・自動対応・コンテンツ生成 |

### 利用プロジェクト

| プロジェクト | モジュール構成 | デプロイ |
|-------------|--------------|---------|
| MyTeslaWearApp | mobile / wear / shared | Play Console: `com.example.myteslawearapp` |
| RideVoiceNavi | androidApp / shared | Play Console: `com.example.ridevoicenavi` |
| FuelEfficiency_AAOS | app | Play Console: `com.rintaro.truefuel` |

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

## セキュリティワークフロー

### LLMマルチペルソナセキュリティレビュー (`android-security-llm.yml`)

Claude Code を活用した4ペルソナ（中堅/上級/シニア承認者/EU当局）による多角的セキュリティレビュー。

| モード | トリガー | 内容 |
|--------|---------|------|
| **differential** | Push毎（`android-ci-smart.yml` 経由） | 変更ファイルのみをHaikuで高速スキャン |
| **full** | 週次 cron schedule | 全コードを4ペルソナ全員でフルレビュー |

追加 inputs: `scan-mode`, `privacy-policy-repo`, `privacy-policy-paths`, `fail-on-critical`
追加 secrets: `ANTHROPIC_API_KEY`

### DAST + Maestro E2E (`android-security-dast.yml`)

APKビルド → Maestro E2E セキュリティテスト（エミュレータ上） + APKバイナリ解析（逆コンパイル、権限監査、exported コンポーネント、証明書ピンニング検査）。

### クラウドセキュリティ監査 (`android-security-cloud.yml`)

Firebase設定監査: セキュリティルール（Firestore/Storage/RTDB）、IAM/サービスアカウント、network_security_config.xml、.gitignore漏れチェック。

### スマートCIでのLLMセキュリティ統合

`android-ci-smart.yml` に `run-llm-security: true` を指定すると、Push毎に差分ベースのLLMセキュリティレビューを自動実行:

```yaml
jobs:
  ci:
    uses: Rintaro-Ko/github-workflows/.github/workflows/android-ci-smart.yml@main
    with:
      primary-module: app
      run-llm-security: true
      privacy-policy-repo: Rintaro-Ko/taro-autodev-app
      privacy-policy-paths: "docs/privacy-policy.html docs/terms-of-service.html"
    secrets: inherit
```

## セットアップ

1. このリポジトリを自分のGitHubアカウントにプッシュ
2. 各プロジェクトで `.github/workflows/android.yml` を作成（上記テンプレート参照）
3. Firebase プロジェクト作成 + サービスアカウント設定
4. GitHub リポジトリに Secrets/Variables を設定
5. LLMセキュリティ使用時: `ANTHROPIC_API_KEY` を GitHub Secrets に追加

## リポジトリの可視性

再利用可能なワークフローを使用するには、このリポジトリが以下のいずれかである必要があります：
- **Public** リポジトリ
- 同じ Organization 内の **Internal** または **Private** リポジトリ（GitHub Enterprise）
