# 設定手順

GitHub Actionsを変更せず、DfCのGitHubコネクタ経由でAgentless code scanningを有効にする手順。

## 事前条件

- Defender for Cloudを利用できるAzureサブスクリプション
- Azure側：サブスクリプションの共同作成者など、コネクタ作成に必要な権限
- GitHub側：対象OrganizationのOwner権限
- 対象リポジトリの管理者への連絡と、ソースコードをスキャンサービスへ提供することについての承認
- 対応リージョン（例：East Asia）とデータ所在地、料金の確認

参考：[GitHubのオンボード手順](https://learn.microsoft.com/ja-jp/azure/defender-for-cloud/quickstart-onboard-github)、[必要権限](https://learn.microsoft.com/en-us/azure/defender-for-cloud/devops-support)

## 作業手順

1. Azure Portalで **Microsoft Defender for Cloud → Environment settings → Add environment → GitHub** を開く。
2. コネクタ名、サブスクリプション、リソースグループ、リージョンを指定する。
3. **Authorize** からGitHubにログインし、Defender for Cloud GitHub Appを対象Organizationにインストールする。
4. 最初はPoC用リポジトリのみアクセスを許可する。Organization全体への接続は検証後に判断する。
5. コネクタ作成時または設定の編集画面で **Agentless code scanning（Preview）** を有効にする。
6. スキャン対象は可能なら **Inclusion mode** で検証用リポジトリに限定する。Terraform / Bicep / 依存関係に必要なスキャナーを有効にする。
7. コネクタを保存し、初回のリポジトリ検出とスキャンを待つ。コード・IaCのスキャンは以後1日1回。
8. **Defender for Cloud → Recommendations** で検出内容と対象リポジトリを確認する。

表示名や配置はポータルの更新によって変わる場合がある。作業中は公式ドキュメントの最新画面も確認する。

## 運用上の考え方

PoCではDefender CSPMやGHAS、Dependabotを追加しない。まず標準連携で何が検出されるかを確認する。

本番導入時は、アクセス対象を絞ったGitHub Appの権限、スキャン対象の自動検出、結果の確認担当、例外の扱いを決める。

参考：[Agentless code scanningの構成とスコープ](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning#customize-scanner-coverage-and-scope)
