# API Gateway Endpoint Type Hands-on

このプロジェクトは、AWS API Gatewayの異なるエンドポイントタイプ（Edge、Regional）を使用したLambda関数の統合を学習するためのハンズオンです。

## 概要

- **Edge API**: エッジロケーションで実行され、グローバルな低遅延を提供
- **Regional API**: 特定のリージョンで実行され、リージョン固有の機能を利用可能

## アーキテクチャ

```
┌─────────────────┐    ┌─────────────────┐
│   Edge API      │    │  Regional API   │
│   (us-east-1)   │    │ (ap-northeast-1)│
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          └──────────┬───────────┘
                     │
              ┌──────▼──────┐
              │   Lambda    │
              │  Function   │
              └─────────────┘
```

## ファイル構成

```
templates/
├── lambda.yaml          # Lambda関数とIAMロール
├── edge-api.yaml        # Edge API Gateway
└── regional-api.yaml    # Regional API Gateway
```

## デプロイ手順

### 1. Lambda関数のデプロイ

```bash
aws cloudformation deploy \
  --stack-name lambda \
  --template-file ./templates/lambda.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### 2. Edge API Gatewayのデプロイ

```bash
aws cloudformation deploy \
  --stack-name edge-api \
  --template-file ./templates/edge-api.yaml \
  --region ap-northeast-1
```

### 3. Regional API Gatewayのデプロイ

```bash
aws cloudformation deploy \
  --stack-name regional-api \
  --template-file ./templates/regional-api.yaml \
  --region ap-northeast-1
```

## テスト方法

### Edge APIのテスト

```bash
# エンドポイントURLを取得
EDGE_API_ID=$(aws cloudformation describe-stacks \
  --stack-name edge-api \
  --query 'Stacks[0].Outputs[?OutputKey==`EdgeApiInvokeAPI`].OutputValue' \
  --output text \
  --region ap-northeast-1)

# APIをテスト
curl $EDGE_API_ID
```

### Regional APIのテスト

```bash
# エンドポイントURLを取得
REGIONAL_API_ID=$(aws cloudformation describe-stacks \
  --stack-name regional-api \
  --query 'Stacks[0].Outputs[?OutputKey==`RegionalApiInvokeAPI`].OutputValue' \
  --output text \
  --region ap-northeast-1)

# APIをテスト
curl $REGIONAL_API_ID
```

## 主な違い

| 項目 | Edge API | Regional API |
|------|----------|--------------|
| 実行場所 | エッジロケーション | 指定リージョン |
| デフォルトリージョン | us-east-1 | 指定リージョン |
| 遅延 | グローバルで低遅延 | リージョン内で低遅延 |
| リージョン固有機能 | 利用不可 | 利用可能 |
| 料金 | エッジ転送料金 | リージョン転送料金 |

## トラブルシューティング

### よくあるエラー

1. **Missing Authentication Token**
   - エンドポイントURLが間違っている可能性
   - 正しいパス（`/test/test`）を指定

2. **Internal Server Error**
   - Lambda関数のログを確認
   ```bash
   aws logs tail /aws/lambda/HelloFunction --region ap-northeast-1
   ```

3. **Permission Denied**
   - Lambda関数の権限設定を確認
   - API Gatewayの統合設定を確認

## クリーンアップ

```bash
# スタックの削除
aws cloudformation delete-stack --stack-name regional-api --region ap-northeast-1
aws cloudformation delete-stack --stack-name edge-api --region ap-northeast-1
aws cloudformation delete-stack --stack-name lambda --region ap-northeast-1
```

## 参考資料

- [API Gateway エンドポイントタイプ](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-endpoint-types.html)
- [CloudFormation テンプレート](https://docs.aws.amazon.com/cloudformation/)
- [Lambda 関数](https://docs.aws.amazon.com/lambda/) 