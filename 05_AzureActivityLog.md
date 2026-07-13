# Azure Activity Log

Azure Activity Log は、これまでの各ファイルで扱ってきた「会話ログ」とは別物です。**Azure リソースそのものに対する管理操作**を記録する仕組みで、Copilot Studio の会話内容は対象に含まれません。

参照:[Azure Activity Log 公式ドキュメント](https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log?tabs=portal-1%2Clog-analytics%2Cportal-2)

## 要点

- **目的**:Azure リソースに対する「管理操作(コントロールプレーン操作)」を記録する(例:仮想マシンの作成、設定の変更、リソースの削除など)
- **収集**:デフォルトで有効になっており、特別な設定は不要。記録されたエントリをユーザーが変更・削除することはできない
- **保持期間と料金**:デフォルトで **90日間** 保持され、その後自動的に削除される。この90日間の記録に対して追加料金はかからない
- **長期保存**:90日を超えてログを保持・分析したい場合は、「診断設定」を作成し、Log Analytics ワークスペースやストレージアカウントなど別の場所にエクスポートする必要がある
- **アクセス方法**:Azure Portal 画面、Azure CLI、Azure PowerShell、REST API からログの確認や取得が可能

## 一言で言えば

「Azure 環境内で『誰が・いつ・何をしたか(管理操作)』をデフォルトで90日間無料で記録してくれる監査用ログ機能」です。

Copilot Studio の会話内容を確認したい場合は、Azure Activity Log ではなく [02_ApplicationInsights連携セットアップ.md](02_ApplicationInsights連携セットアップ.md) や [06_DSPMforAIと権限.md](06_DSPMforAIと権限.md) を参照してください。
