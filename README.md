# AWS WAF Managed Rules Deployment Notifier

AWS WAF Managed Rulesの更新通知をAWS Chatbot経由でSlack/Teamsに配信するためのCloudFormationテンプレート

## 概要

このプロジェクトは、AWS WAF Managed Rulesの更新通知を受信し、AWS Chatbotを通じてSlackやTeamsなどのコラボレーションツールに通知を送信するサーバーレスソリューションです。EventBridge Pipesを使用してメッセージを変換し、コードメンテナンスが不要な構成を実現しています。

AWS WAF Managed Rulesは定期的に更新され、新しい脅威への対応や誤検知の修正が行われます。このソリューションにより、これらの更新をリアルタイムで把握し、必要に応じて迅速に対応できます。

## アーキテクチャ

```
[AWS WAF SNS Topic] → [SQS Queue] → [EventBridge Pipes] → [SNS Topic] → [AWS Chatbot] → [Slack/Teams]
```

### コンポーネント

- **SQS Queue**: AWS WAF SNS Topicからのメッセージを受信
- **EventBridge Pipes**: メッセージをChatbot形式に変換（ノーコード）
- **SNS Topic**: 変換されたメッセージをChatbotに配信
- **AWS Chatbot**: Slack/Teamsへの通知を管理
- **Dead Letter Queue**: 失敗したメッセージの保管

## 特徴

- 🚀 **完全サーバーレス**: Lambda関数不要
- 🛠️ **メンテナンスフリー**: コード管理不要
- 💰 **低コスト**: 月数回の更新なら実質無料
- 🔄 **自動リトライ**: DLQ付きで信頼性向上
- 📊 **カスタマイズ可能**: メッセージフォーマットを簡単に変更可能

## セットアップ

### 前提条件

- AWS CLIがインストールされ、設定済みであること
- AWS WAF Managed Rulesを使用していること
- （オプション）Slackワークスペースまたは Microsoft Teams チャネル

### サポートリージョン

AWS WAF Managed Rulesの更新通知SNSトピックは`us-east-1`リージョンでのみ利用可能です。ただし、このソリューションは他のリージョンにもデプロイ可能で、クロスリージョンでSNSサブスクリプションを作成します。

### デプロイ手順

1. **パラメータファイルの編集**

   `parameters.json`を編集して、必要な設定を行います：

   ```json
   {
     "Parameters": [
       {
         "ParameterKey": "WafSnsTopicArn",
         "ParameterValue": "arn:aws:sns:us-east-1:247893642050:aws-managed-rules-deploy-notifications"
       },
       {
         "ParameterKey": "ChatbotWorkspaceId",
         "ParameterValue": "YOUR_SLACK_WORKSPACE_ID"
       },
       {
         "ParameterKey": "ChatbotChannelId",
         "ParameterValue": "YOUR_SLACK_CHANNEL_ID"
       },
       {
         "ParameterKey": "NotificationEmail",
         "ParameterValue": "your-email@example.com"
       }
     ]
   }
   ```

2. **CloudFormationスタックのデプロイ**

   ```bash
   aws cloudformation create-stack \
     --stack-name waf-managed-rules-notifier \
     --template-body file://template.yaml \
     --parameters file://parameters.json \
     --capabilities CAPABILITY_NAMED_IAM \
     --region us-east-1
   ```

3. **AWS Chatbotの設定（Slack/Teams使用時）**

   - [AWS Chatbotコンソール](https://console.aws.amazon.com/chatbot/)にアクセス
   - Slack WorkspaceまたはTeamsチャネルを設定
   - 作成されたSNS TopicをChatbot設定に追加

## 通知フォーマット

通知は以下の形式で配信されます：

```
🛡️ AWS WAF Managed Rules Update

ℹ️ New version available for rule group AWSManagedRulesCommonRuleSet

📦 Rule Group: AWSManagedRulesCommonRuleSet
🏷️ Version: v1.5

📝 Details:
Welcome to AWSManagedRulesCommonRuleSet version 1.5! We've updated the regex
specification in this version to improve protection coverage, adding protections
against insecure deserialization. For details about this change, see
http://updatedPublicDocs.html.

🕐 Timestamp: 2021-08-24T11:12:19.810Z
```

### 通知に含まれる情報

- **Subject**: 更新の概要（どのルールグループが更新されたか）
- **Rule Group**: 更新されたルールグループ名
- **Version**: 新しいメジャーバージョン番号
- **Details**: 更新内容の詳細説明とドキュメントリンク
- **Timestamp**: 更新通知の送信時刻

## モニタリング

### CloudWatch Logs

EventBridge Pipeのログは以下で確認できます：

```bash
aws logs tail /aws/vendedlogs/pipes/waf-managed-rules-notifier-waf-notification-pipe
```

### Dead Letter Queue

失敗したメッセージの確認：

```bash
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT_ID/waf-managed-rules-notifier-waf-notifications-dlq
```

## トラブルシューティング

### 通知が届かない場合

1. SNSサブスクリプションが確認済みか確認
2. SQSキューにメッセージが到達しているか確認
3. EventBridge Pipeのログでエラーを確認
4. Chatbotの設定を確認

### メッセージフォーマットの変更

`template.yaml`の`InputTemplate`セクションを編集して、メッセージフォーマットをカスタマイズできます。

利用可能なフィールド：
- `$.Subject` - 更新の概要
- `$.Message` - 更新の詳細内容
- `$.MessageAttributes.managed_rule_group.Value` - ルールグループ名
- `$.MessageAttributes.major_version.Value` - バージョン番号
- `$.Timestamp` - 更新通知のタイムスタンプ

## リソースのクリーンアップ

不要になった場合は、以下のコマンドでリソースを削除できます：

```bash
aws cloudformation delete-stack --stack-name waf-managed-rules-notifier --region us-east-1
```

## コスト

月数回の更新の場合、実質的にコストは発生しません：

- SQS: 最初の100万リクエスト無料
- EventBridge Pipes: $0.40/100万リクエスト
- SNS: 最初の100万リクエスト無料
- CloudWatch Logs: 最初の5GB無料
