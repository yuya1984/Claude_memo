# Claude_memo

Microsoft 365 / Copilot Studio におけるログ・監査・トレーサビリティに関する調査メモ集です。
「エージェントとユーザーの会話をどこで、どこまで確認できるのか」という問いを起点に、Microsoft が提供する複数のログ経路の役割分担と、それぞれの具体的な確認手順を整理しています。

## 全体像:3つのログ経路

Copilot Studio エージェントの利用状況は、目的の異なる3つの経路で記録されています。どれも「ログ」と呼ばれますが、見えるものと見えないものが明確に分かれています。

| 経路 | 目的 | 想定利用者 | 会話の中身 |
|---|---|---|---|
| **Purview 監査ログ** | 「誰が・いつ・どのエージェントを使ったか」の証跡(コンプライアンス) | 全社共通の管理者 | 原則含まない(スレッドIDのみ、"Redacted"表示) |
| **DSPM for AI**(Purview) | センシティブな内容を含むコンプライアンス調査 | 限定されたコンプライアンス担当 | 追加権限があれば中身にアクセス可 |
| **Application Insights** | 開発・運用のためのパフォーマンス監視・デバッグ | 開発・運用担当 | 設定を有効化すれば記録可(平文) |
| **Azure Activity Log** | Azure リソースに対する管理操作の記録 | Azure 全体の管理者 | 会話内容は対象外(リソース操作のみ) |

この違いを理解しておくと、「Purview で見ると Redacted なのに、なぜ Application Insights では中身が見えるのか」といった疑問がすぐに解消できます。詳細は [01_purview監査ログの設計思想.md](01_purview監査ログの設計思想.md) を参照してください。

## 目次

1. [Purview 監査ログの設計思想](01_purview監査ログの設計思想.md) — なぜ会話の中身が"Redacted"になるのか、その設計意図
2. [Application Insights 連携セットアップ](02_ApplicationInsights連携セットアップ.md) — Copilot Studio と Application Insights を接続し、会話ログを取得できるようにする手順
3. [KQL(Kusto Query Language)基礎](03_KQL基礎.md) — ログを検索するための専用クエリ言語の書き方
4. [会話ログの確認クエリ集](04_会話ログの確認クエリ集.md) — customEvents / traces / dependencies テーブルを使った実践クエリ
5. [Azure Activity Log](05_AzureActivityLog.md) — Azure リソースの管理操作ログ(会話ログとは別物)
6. [DSPM for AI と権限](06_DSPMforAIと権限.md) — コンプライアンス調査に必要なロールと確認手順
7. [付録:Microsoft サポート専門家プロンプト](appendix_AIプロンプトテンプレート.md) — Microsoft 製品サポートに特化した AI エージェント用システムプロンプト
