# Amazon SageMakerでのセキュリティ
## 考慮するべきポイント
- データの保護
  - 保存のデータ暗号化
    - S3データの暗号化
      - Server Side Encryption: SSE
        - SSE-S3、SSE-KMS、SSE-Cの中から選ぶ
        - サーバーサイド暗号化のため、S3側でデータの絞り込みが効き、中身の参照にCMK:Customer Master Keyへのアクセス権が必要となるため、SSE-KMSがおすすめ
        - AWS Key Management Serviceの詳細は[こちら](https://aws.amazon.com/jp/kms/)
      - Client Side Encription: CSE
- 通信の保護
  - 閉域に閉じた環境
    - VPCから閉域網に閉じたあk達でサービスのエンドポイントにアクセス
  - 通信のデータ暗号化
- 権限管理
  - データに対して最小の見解を与える
  - データレイクアカウント、開発環境アカウント、本番環境アカウントなど、用途や組織に応じてクロスアカウントでの環境分離を行う
  - データの分類
    - パーソナルデータ：匿名化する
    - 売上・取引先データ：取引先名ではなくてIDのみ使用
- ガバナンス
  - CloudWatch Logsによるログの出力及び集約
  - CloudTrailによるAPI実行歴の蓄積

## セキュアなjupyter notebooks 開発環境
- ノートブックインスタンス作成APIを、ダイレクトコネクトとVPCエンドポイント経由で閉域に閉じる
- すべての通信はTLSで暗号化
- ノートブックにアクセスする際の権限管理
  - プリサインドURLを作成する。この際、作成ようAPIを叩くIAMポリシーに`sourceIP`や`sourceVPC`を指定すると閉域内からしかURLを作成できなくなる。
  - `sourceIp`や`sourceVpc`を指定したポリシーを使って生成したURLは，同じ`sourceIp`や`sourceVpc`からしかアクセスできない
