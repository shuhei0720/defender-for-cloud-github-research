# 調査結果

確認日：2026-11-15

## 1. 対象の機能

| 機能 | 何を確認できるか | 実行方式 |
| --- | --- | --- |
| GitHubコネクタ（DevOps Security） | リポジトリ、パイプライン、接続情報などのセキュリティ設定 | 接続直後、その後約8時間ごとに評価 |
| Agentless code scanning（プレビュー） | コードの問題、IaC設定不備、OSS依存関係の既知脆弱性 | 既定ブランチを接続直後、その後1日1回スキャン |
| DfCの推奨事項 | 検出された問題、重大度、修正方法 | DfCポータルで確認 |

リポジトリごとにGitHub Actionsを追加する必要はない。PRごとに実行するスキャンでもない。ビルドを失敗させるゲートとしては使えない。

根拠：[Agentless code scanning](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning)

## 2. スキャナーと対応範囲

| 対象 | ツール | 備考 |
| --- | --- | --- |
| Python | Bandit | 静的解析 |
| JavaScript / TypeScript | ESLint | 静的解析 |
| Terraform、ARM、Bicepほか | Checkov | IaC設定不備 |
| ARM、Bicep | Template Analyzer | IaC設定不備 |
| npm、pip、NuGetなどの依存関係 | Trivy | manifest / lockfileから既知脆弱性を調査 |
| GitHub Actionsを含むSBOM | Syft | パッケージの識別。CVE検知を保証するものではない |

PowerShellスクリプト自体の静的解析は、公開されているAgentless code scanningの対応言語一覧にはない。

根拠：[対応ツールとファイル形式](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning#supported-scanning-tools)

## 3. SHA固定のGitHub Actionsについて

現状のWorkflowは `uses: owner/action@<40文字のコミットSHA>` のような指定を前提とする。

- GitHubのDependabot AlertsはGitHub ActionsのSemVer指定には対応するが、SHA指定にはアラートを生成しない。
- Dependabot Version UpdatesはSHA固定されたActionの更新には対応する。ただし更新PRの作成と、現在のSHAの既知CVE検知は別機能。
- DfCはSyftでGitHub ActionsをSBOMの対象として扱うと記載している。一方で、SHAをバージョン・修正コミットに対応付けてCVEを推奨事項として表示するかどうかは公式資料で確認できない。

SHA固定をタグ指定に変更する必要はない。むしろGitHubは、タグの差し替え対策としてコミット固定を推奨している。

今回確認すべきなのは「Actionを認識できるか」だけではなく「そのSHAに該当するCVEがDfCの推奨事項に表示されるか」。

根拠：

- [Dependabot Alertsの制限](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-alerts)
- [DependabotのGitHub Actions対応](https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories)
- [GitHub Actionsの安全な利用](https://docs.github.com/en/actions/reference/security/secure-use)

## 4. ライセンス・料金

DfCのDevOps Securityは、基本的な姿勢評価と高度な機能で前提が異なる。

- リポジトリの検出、設定不備の推奨事項、Agentlessによるコード・IaC検査は、公式の機能対応表でFoundational CSPM側にも掲載されている。
- Defender CSPMは、攻撃経路分析、Cloud Security Explorer、コードとクラウドの関連付けなどの高度な機能に必要。
- GitHub Advanced Security（GHAS）由来のシークレット・OSS脆弱性の結果を取り込む機能には、GHASが前提として記載されている。
- これとは別に、AgentlessはTrivyの依存関係スキャンとDfCの依存関係推奨事項を案内している。**GHAS由来の結果とTrivy由来の結果を混同しない。**

金額はサブスクリプションの契約・プランによるため、この資料では見積もっていない。導入前にAzureの料金画面で確認する。

根拠：

- [DevOps securityの対応範囲と前提条件](https://learn.microsoft.com/en-us/azure/defender-for-cloud/devops-support)
- [Agentless code scanningの検出結果](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning#view-and-manage-scan-results)

## 5. 制約と注意事項

- Agentless code scanningはプレビュー機能。
- 対象リポジトリは1 GB未満、既定ブランチのみ。PR上の変更はマージ前には検査対象にならない。
- スキャン内容は選択できるが、ツール固有の細かなルール調整には制限がある。
- Agentless code scanningの対応リージョンにEast Asiaは含まれるが、日本リージョンは記載されていない。
- 公式資料にはソースコードのスキャン処理地域がUS/EUのgeoである旨の記載がある。リージョン選択とコードの処理場所は同一と決めつけず、データ所在地の要件に照らして確認する。
- Microsoftの推奨事項は個別検出結果への移行が完了している。旧来のグループ化された推奨事項名だけで運用やクエリを作らない。

根拠：

- [Agentless code scanningの制限とデータ取扱い](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning)
- [推奨事項の個別化](https://learn.microsoft.com/en-us/azure/defender-for-cloud/transition-grouped-individual-recommendations)

## 6. 公式資料

1. [GitHubコネクタの作成](https://learn.microsoft.com/ja-jp/azure/defender-for-cloud/quickstart-onboard-github)
2. [Agentless code scanning](https://learn.microsoft.com/en-us/azure/defender-for-cloud/agentless-code-scanning)
3. [DevOps security：サポートと権限](https://learn.microsoft.com/en-us/azure/defender-for-cloud/devops-support)
4. [DfCの推奨事項の確認](https://learn.microsoft.com/en-us/azure/defender-for-cloud/review-security-recommendations)
5. [GitHub Dependabot Alertsの制限](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-alerts)
