# Application Insights 連携セットアップ

Purview 監査ログでは会話の中身が "Redacted" になりますが([01_purview監査ログの設計思想.md](01_purview監査ログの設計思想.md) 参照)、Copilot Studio エージェントを Application Insights に接続すると、ユーザーメッセージ・ボットの応答・トピックのトリガー・イベント・カスタムログといったテレメトリを別経路で取得できるようになります。

## 前提として押さえておくべき注意点

### 遡及的な復元はできない

Application Insights は「接続設定を有効にした後に発生した会話」のみを記録します。すでに Redacted になっている過去の会話を後から復元する手段ではなく、**今後の会話を可視化するための別経路のログ収集** です。

過去の会話も含めて中身を確認したい場合は、Application Insights ではなく [DSPM for AI](06_DSPMforAIと権限.md) 経由でのアクセスを検討する必要があります。「今後の会話を監視したい」のか「過去の会話も遡って確認したい」のかで、選ぶべき手段が変わります。

### Redacted 表示そのものは解除されない

Application Insights 側の設定を変えても、Purview 監査ログの "Redacted" 表示は直りません。Purview 側は「会話の全文は含めず、トランスクリプトのスレッド ID のみを記録する」という設計自体が変わらないためです。Application Insights は Purview とは完全に別の経路(別のログの流れ)としてデータを取得しているだけです。

イメージで言うと以下の通りです。

- **Purview 監査ログ** = 「誰が・いつ・何のエージェントと話したか」を記録する台帳(中身は原則見えない設計)
- **Application Insights** = その台帳とは別に、会話の中身そのものを見るための独立した監視カメラを別途設置するようなもの

つまり「Redacted を解除する」のではなく、「Redacted のままの経路とは別に、中身が見える経路を新しく作る」というのが正確な理解です。

## セットアップ手順

### 1. Azure 側で Application Insights リソースを作成

1. Azure Portal → 「リソースの作成」→ Application Insights
2. リソースグループ、リージョンを指定して作成
3. 作成後、リソースの「概要」画面から「接続文字列(Connection String)」をコピー

### 2. Copilot Studio 側でエージェントに接続

1. 対象のエージェントを開く → 設定(Settings) → 詳細設定(Advanced) → Application Insights
2. 先ほどコピーした接続文字列を貼り付け
3. 以下の2つのオプションを有効化する
   - **アクティビティのログ記録(Log activities)**
   - **機密性の高いアクティビティプロパティのログ記録(Log sensitive Activity properties)**

   この2つ目を有効化すると、ユーザー ID・名前・実際のテキスト・発話内容(`userid`, `name`, `text`, `speak`)まで取得可能になります。
4. 保存

> **セキュリティ上の注意**:「機密性の高いアクティビティプロパティ」を有効にすると、ユーザーの発話内容そのものが Application Insights に平文で蓄積されます。アクセス権限の絞り込みと、Application Insights 側のデータ保持期間・アクセス制御(Azure RBAC)の設計は必須です。

### 3. トピック側で補強(任意・推奨)

トピック内で「カスタムテレメトリイベントのログ」ノードを使うと、キー・バリュー形式で任意のデータを Application Insights に送信できます。ナレッジソース検索クエリなどを可視化したい場合は、ここで明示的にログを仕込む必要があります。

### 4. Azure Portal で Kusto クエリを実行

1. 対象の Application Insights リソース → 「ログ」
2. 主に `customEvents` テーブルを参照(会話メッセージやトピック実行、カスタムイベントが格納される)

```kusto
customEvents
| where timestamp > ago(7d)
| where name == "Message" or name == "TopicStart"
| project timestamp, name, customDimensions
| order by timestamp desc
```

クエリの書き方の詳細は [03_KQL基礎.md](03_KQL基礎.md)、より実践的なクエリ例は [04_会話ログの確認クエリ集.md](04_会話ログの確認クエリ集.md) を参照してください。

### 5. 可視化(任意)

Copilot Studio Analytics Template Workbook などの既存テンプレートを使うと、時系列チャートやフィルタ付きダッシュボードを組めます。

## 押さえておくべき限界

ツール呼び出しは Application Insights で `TopicStart` イベントとして表示され、どのツールやコネクタが呼ばれたかは分かりますが、Dataverse トランスクリプトほどの詳細さはありません。以下の情報は Application Insights には含まれません。

- どのナレッジソースが検索されたか
- 検索にかかった時間
- 意図認識の信頼度スコア
- オーケストレーションの計画
- セッションの結果(解決・エスカレーション・放棄)

より詳細なトレースが必要な場合は、テストペインからの「スナップショット保存」(`dialog.json`)や Dataverse の会話トランスクリプトテーブル(デフォルト30日保持)も併用するのが実務上のセットです。
