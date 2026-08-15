# AWS Gateway Load Balancer - 集中型トラフィック検査プロジェクト

[English Version](README.md)

AWS GWLB を使用してトラフィックを検査し、プライベート AWS リソースへの不正アクセス試行をログに記録する集中型セキュリティアーキテクチャ。

## プロジェクト概要
このプロジェクトは、AWS Gateway Load Balancer (GWLB) を使用して異なる VPC 間を流れるトラフィックを検査するセキュアなネットワークアーキテクチャを示しています。GENEVE カプセル化を使用した集中型検査モデルを実装し、「Testing/Consumer」環境と「Appliance/Security」環境を効果的に分離する方法を紹介します。

## 実装と検証

### 1. ターゲットグループの設定 (GENEVE プロトコル)
GWLB のトラフィックルーティングに不可欠な UDP ポート 6081 で GENEVE プロトコルを使用するようにターゲットグループを設定しました。
![Target Group](images/1-target-group.png)

### 2. GWLB リスナールーティング
すべての受信トラフィックを指定されたターゲットグループに転送するように GWLB リスナーを設定しました。
![GWLB Listener](images/2-listener.png)

### 3. 重要なルーティングロジック (プライベートルートテーブル)
すべての送信トラフィック (`0.0.0.0/0`) を Gateway Load Balancer エンドポイント (GWLBE) に強制的にルーティングするようにプライベートルートテーブルを変更しました。
![Private Route Table](images/3-private-route.png)

### 4. トラフィックの生成とテスト
踏み台ホスト (Bastion Host) 経由でプライベートサーバーに正常に接続し、検査パスをテストするための ICMP トラフィックを生成しました。
![SSH Connection](images/4-ssh-test.png)

### 5. パケット検査の証明
アプライアンスサーバー上で `tcpdump` を使用して GENEVE カプセル化パケットをキャプチャし、GWLB がトラフィックを正常に傍受してルーティングしていることを確認しました。
![tcpdump Verification](images/5-tcp.png)

## 仕組み

1. トラフィックは Testing プライベートサブネット内の EC2 インスタンスから発生します。
2. ルートテーブルは、このトラフィックを GWLB エンドポイントに誘導します。
3. エンドポイントは GENEVE を使用してトラフィックをカプセル化し、Packet Trace VPC 内の GWLB に転送します。
4. GWLB はトラフィックをアプライアンスサーバーの UDP ポート 6081 に転送します。
5. アプライアンスはトラフィックを検査し、カプセル化を解除して、セキュリティルールに基づいて許可または拒否を行います。

---
ステータス: アーキテクチャ設計および手動実装完了。
