# Amazon SageMakerでのセキュリティ
## データの保護 
#### [SageMakerのインスタンス上での保存のデータ暗号化](https://docs.aws.amazon.com/ja_jp/sagemaker/latest/dg/encryption-at-rest.html)
- AWS Key Management Service キーを Amazon SageMaker ノートブック、トレーニングジョブ、ハイパーパラメータ調整ジョブ、バッチ変換ジョブ、エンドポイントに渡すことで、アタッチされた機械学習 (ML) ストレージボリュームを暗号化することができる
- SageMakerのそれぞれのストレージボリュームからS3にデータを移す場合には、必要に応じてAWS KMS暗号化を使用する

#### S3データの暗号化
- Server Side Encryption: SSE
  - SSE-S3、SSE-KMS、SSE-Cの中から選ぶ
  - サーバーサイド暗号化のため、S3側でデータの絞り込みが効き、中身の参照にCMK:Customer Master Keyへのアクセス権が必要となるため、SSE-KMSがおすすめ
  - AWS Key Management Serviceの詳細は[こちら](https://aws.amazon.com/jp/kms/)
- Client Side Encription: CSE
  - Amazon Macie などの高度なマネージドセキュリティサービスを使用します。これにより、Amazon S3 に保存される個人データの検出と保護が支援されます

#### 通信の保護
- 閉域に閉じた環境
  - VPCから閉域網に閉じた形でサービスのエンドポイントにアクセス
  - [AWS PrivateLink](https://aws.amazon.com/jp/privatelink/)を使用可能なVPCエンドポイントをサポートしている[サービス一覧](https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/vpc-endpoints.html)にSageMakerのノートブックインスタンスとランタイムが入っている。
- 通信のデータ暗号化
  - SSL(Source Socket Layer)やTSL(Transport Layer Security)を使うことで盗聴や改ざんを防ぐことをおすすめ
  - 転送中のネットワーク間データはすべて、TLS 1.2 暗号化をサポートしています
  - 一部暗号化されない通信もある
    - サービスコントロールプレーンとトレーニングジョブインスタンス (顧客データではない) の間のコマンドとコントロールの通信
    - 分散トレーニングジョブ (ネットワーク内) のノード間の通信
## 権限管理
- データに対して最小の見解を与える
- データレイクアカウント、開発環境アカウント、本番環境アカウントなど、用途や組織に応じてクロスアカウントでの環境分離を行う
- データの分類
  - パーソナルデータ：匿名化する
  - 売上・取引先データ：取引先名ではなくてIDのみ使用
## ガバナンス
- CloudWatch Logsによるログの出力及び集約
- CloudTrailによるAPI実行歴の蓄積
  
## その他
  - PCI DSS(Payment Card Industry Data Security Standard)対応済
  - HIPAA(Health Insurance Portability and Accountability Act)対応済

## セキュアなjupyter notebooks 開発環境
- ノートブックインスタンス作成APIを、ダイレクトコネクトとVPCエンドポイント経由で閉域に閉じる
- すべての通信はTLSで暗号化
- ノートブックにアクセスする際の権限管理
  - プリサインドURLを作成する。この際、作成ようAPIを叩くIAMポリシーに`sourceIP`や`sourceVPC`を指定すると閉域内からしかURLを作成できなくなる。
  - `sourceIp`や`sourceVpc`を指定したポリシーを使って生成したURLは，同じ`sourceIp`や`sourceVpc`からしかアクセスできない
- SageMaker VPCエンドポイントを用いることで閉域に閉じた形でS3やECRへアクセス可能
- notebookインスタンス上のデータもS3上のデータと同様にSSE-KMSで暗号化、ECRはサービス側で自動でイメージを暗号化
- Lifecycle Configを用いることで必ず実行させたいセキュリティモジュールなどのインストールなどをノートブック起動時に自動化

## セキュアな学習環境
- 学習ジョブの実行APIを、ダイレクトコネクトとVPCエンドポイント経由で閉域に閉じる
- SageMaker学習インスタンスは閉域内に立ち上がる？
- SageMaker VPCエンドポイントを使えば、立ち上げたコンテナへコードとトレーニングデータを閉域内で渡して学習を実行可能
- トレーニングインスタンス間の通信はTLSで実施可能、たださい計算時間が増える可能性がある
- CloudWatchにログを書き出し、ログはCMKで暗号化

## セキュアな推論環境
- 学習済みモデルを読み込んで，https リクエストを受け付ける
- 推論インスタンスへは直接ログインは不可
- 外部からリクエストを受けたアプリサーバが推論エンドポイントに対して，Sig. V4 ベースの認証を用いてリクエスト
