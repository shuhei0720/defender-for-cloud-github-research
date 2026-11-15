# GitHubリポジトリのセキュリティ検査（Defender for Cloud）

Microsoft Defender for Cloud（DfC）のGitHub連携とAgentless code scanningについて調べた記録です。
目的は、GitHub Actionsを変更せずにリポジトリを定期検査し、検出結果をDfCの推奨事項で確認できるかを整理することです。

調査日：2026-11-15

## 分かったこと

- GitHubコネクタでリポジトリとCI/CDのセキュリティ設定を評価できる。
- Agentless code scanning（プレビュー）でコード、IaC、依存パッケージを定期検査できる。既存のWorkflowは変更不要。
- リポジトリの検出・設定評価は約8時間ごと、コードスキャンは1日1回。スキャン対象は既定ブランチ。
- Terraform、Bicep、ARM、Python、JavaScript/TypeScriptなどを検査できる。
- **`uses:`でSHA固定したAction自体のCVE検知は未確認。** SyftのSBOM対応だけでは、DfCの脆弱性判定まで保証されない。
- GitHubのDependabot Alertsも、SHA指定のActionには脆弱性アラートを生成しない。SHA固定は維持したまま検証する。

## 資料

| ファイル | 内容 |
| --- | --- |
| [調査結果](docs/findings.md) | 対応範囲、スキャン方式、ライセンス、制約、公式根拠 |
| [設定手順](docs/setup.md) | GitHubコネクタとAgentless code scanningの導入 |
| [検証項目](docs/verification.md) | DfC推奨事項とSHA固定Actionの確認方法 |

## 現時点の判断

DfCの標準連携だけで、GitHubリポジトリの設定不備、IaC、一般的な依存関係を定期確認する用途には対応できる。
ただし、今回の主目的である**SHA固定ActionのCVE検知をDfCだけで完結できるかは未確認**。これを確認するまでは、用途を満たしたと判断しない。

MDASH、Dependabot、GitHub Advanced Securityは、この初期検証では追加しない。

> このリポジトリは公開ドキュメントの調査結果です。DfCの実環境を接続・検証した記録ではありません。
