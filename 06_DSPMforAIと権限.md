# DSPM for AI と権限

過去の会話も含めて中身を確認したい場合、[02_ApplicationInsights連携セットアップ.md](02_ApplicationInsights連携セットアップ.md) では対応できません(接続設定を有効にした後の会話しか記録されないため)。この場合に検討するのが **DSPM for AI**(Microsoft Purview)経由でのアクセスです。

## DSPM for AI の有効化状況を確認する手順

1. **Microsoft Purview ポータルにアクセス**
   `https://purview.microsoft.com` にテナント管理者(またはコンプライアンス管理者)としてサインイン

2. **DSPM for AI のセクションを探す**
   左メニューから「Solutions(ソリューション)」→「Data Security Posture Management」→「for AI」を選択、もしくは左メニューの検索バーで「DSPM for AI」と検索

3. **セットアップ状況の確認**
   - 「Overview」や「Reports」タブが表示されれば、ある程度基盤は動いている状態
   - 「Setup」や「Get started」に該当する項目が残っている場合は、まだ完全に有効化されていない(初回セットアップウィザードが未完了の状態)
   - Copilot Studio のエージェントとの連携が有効か、「Data assessments」や「Activity explorer」で Copilot Studio 関連のアクティビティが表示されるかを見ると実態が分かる

## 必要な権限

この画面自体にアクセスするには、最低限以下のいずれかのロールが必要です。

| 目的 | 必要なロール |
|---|---|
| Purview ポータルへのアクセス | Microsoft Entra 管理センター側の **Compliance Administrator**(コンプライアンス管理者) |
| プロンプト/レスポンスの中身(トランスクリプト内容)まで見る | 上記に加えて **Content Explorer Content Viewer** |
| 監査ログ検索のみ | **Audit Reader** / **Audit Manager**(これだけで十分な場合もある) |

自身がテナント管理者でない場合は、社内の Microsoft 365 管理者やセキュリティ・コンプライアンス担当チームに、上記ロールの割り当て状況と DSPM for AI の有効化状況を確認してもらうのが早い場合があります。

権限を意図的に分離している理由(なぜ「監査ログを見る権限」と「会話の中身を見る権限」が別ロールになっているのか)については、[01_purview監査ログの設計思想.md](01_purview監査ログの設計思想.md) を参照してください。
