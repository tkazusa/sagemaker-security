# Amazon SageMakerでのセキュリティ
## 考慮するべきポイント
- データの保護
  - 保存のデータ暗号化
    - S3データの暗号化
      - Server Side Encryption: SSE
        - SSE-S3、SSE-KMS、SSE-Cの中から選ぶ
        - 基本的におすすめはSSE-KMS
        - AWS Key Management Serviceの詳細は[こちら](https://aws.amazon.com/jp/kms/)
      - Client Side Encription: CSE
- 通信の保護
  - 閉域に閉じた環境
  - 通信のデータ暗号化
- 権限管理
- ガバナンス
