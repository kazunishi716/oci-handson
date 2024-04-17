# OCIハンズオン-ロードバランサー編

- ハンズオン-作業イメージ&前提
- 負荷分散用のWebサーバ構築
- ロードバランサー用のネットワーク作成
- ロードバランサー作成
- 負荷分散の確認

## ハンズオン-作業イメージ&前提
- LoadBalancer / Computeを作成するために必要なポリシーが付与されていること。
- 作業用コンパートメント及び、VCNが作成されていること。まだの場合は[こちら](https://github.com/kazunishi716/oci-handson/blob/a96607cfd3e6e39e4820a4219272872bd765b8af/VCN%26Compute.md?plain=1#L29)から作成ください。
- 各Computeインスタンスにインターネット経由でssh接続できること。

下記がこのハンズオンで作成できる構成イメージです。
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/f69b28e9-a31b-4a6b-9fcd-5037b571d568)

## 負荷分散用のWebサーバ構築

Computeを2台作成します。Computeを作成した後の手順を記載します。
Computeの作成手順については[こちら](https://github.com/kazunishi716/oci-handson/blob/a96607cfd3e6e39e4820a4219272872bd765b8af/VCN%26Compute.md?plain=1#L55)をご参照ください。

該当インスタンスにssh接続します。
1. Apache HTTPサーバーをインストールします
```
sudo yum -y install httpd
```
2. TCPの80番(http)および443番(https)ポートをオープンします
```
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
```
3. ファイアウォールを再ロードします
```
 sudo firewall-cmd --reload
```
4. Webサーバーを起動します
```
sudo systemctl start httpd
```
5. index.html ファイルを作成し、それぞれにどちらのWebサーバーかを示す文字列を記述します
```
1台目： sudo sh -c 'echo "Web Server 1 (host:`hostname`)" > /var/www/html/index.html'
```
```
2台目： sudo sh -c 'echo "Web Server 2 (host:`hostname`)" > /var/www/html/index.html'
```

## ロードバランサー用のネットワーク作成
ロードバランサはWebサーバと同サブネットにも配置可能ですが、
今回はWebサーバーが配置されているサブネットとは別のパブリック・サブネットを作成し、
そこにロードバランサーを配置していきます。

### ロードバランサ用のセキュリティ・リストの追加
1. ネットワーキング → 仮想・クラウドネットワーク を選択し、Webサーバーが存在するVCNのリンクをクリックします
2. 左下部の リソース メニューにある セキュリティ・リスト を選択し、セキュリティ・リストの作成 ボタンをクリックします
3. セキュリティ・リストの作成 ウィンドウに以下の項目を入力します
   1. 名前 - LB_Security_List
   2. コンパートメントに作成 – handson
   3. Ingress Rule と Egress Ruleは空白

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/0a20e6a1-7515-4638-9ade-ef2a4805dcc1)

### ロードバランサ用のルート表の追加
1. VCNリソース メニューにある ルート表 を選択し、ルート表の作成 ボタンを押します
2. ルート表の作成 ウィンドウに以下の項目を入力し、ルート表の作成 ボタンを押します
   1. 名前 - LB_Route_Table
   2. コンパートメントに作成 – handson
   3. ルート・ルール(オプション) – “+別のルート・ルール”をクリック
   4. ターゲット・タイプ - インターネット・ゲートウェイ
   5. 宛先CIDRブロック - 0.0.0.0/0
   6. ターゲット・インターネット・ゲートウェイ - VCNのインターネット・ゲートウェイを選択

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/cc76d30e-ea66-48a4-91eb-8b60b763d435)

### ロードバランサ用のサブネットの追加
1. VCNリソース メニューにあるサブネットを選択し、サブネットの作成 ボタンを押します
2. サブネットの作成 ウィンドウに以下の項目を入力し、サブネットの作成 ボタンを押します
  1. 名前 - LB_Subnet
  2. コンパートメントに作成 – handon
  3. サブネット・タイプ - リージョナル（推奨）を選択
  4. CIDR ブロック - 10.0.3.0/24
  5. サブネット・アクセス - パブリック・サブネット を選択
  6. DNS 解決 - チェックを外す
  7. DHCPオプション - 入力なしのまま
  8. セキュリティ・リスト - LB_Security_List

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/9d6d1847-3ccb-48a6-b364-67cd6ece9f91)

## ロードバランサ構築
1. ネットワーキング → ロード・バランサ を選択し、ロード・バランサの作成 ボタンを押します

### 詳細の追加
1. 詳細追加画面にて以下の項目を入力し、“次”アイコンをクリックします
  1. ロード・バランサ名 - Handson_LB
  2. 可視性タイプの選択 - パブリックを選択
  3. パブリックIPアドレスの割当て - エフェメラルIPアドレスを選択
  4. シェイプ - 最小帯域幅：10 Mbps、最大帯域幅：50 Mbps と入力
  5. 仮想クラウド・ネットワーク – handson-vcn
  6. サブネット - LB_Subnet

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/8ead7319-a827-47bd-9558-1c002fee34ac)

### バックエンドの選択
1. バックエンドの選択画面にて以下の項目を入力し、“次”アイコンをクリックします
   1. ロード・バランシング・ポリシーの指定 - 重み付けラウンド・ロビン を選択
   2. バックエンド・サーバーの選択 - バックエンドの追加を押して作成した2インスタンス(web-instance1/web-instance2)を選択
   3. ヘルス・チェック・ポリシーの指定 - すべてデフォルト
      1. プロトコル - HTTP (デフォルト)
      2. ポート - 80 (デフォルト)
      3. 間隔(ミリ秒) - 100000 (デフォルト)
      4. タイムアウト(ミリ秒) - 3000 (デフォルト)
      5. 再試回数 - 3 (デフォルト)
      6. ステータス・コード - 200 (デフォルト)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/7837a066-f174-4b2f-8010-2981e87a7e26)

### リスナーの構成
1. リスナーの構成の選択画面にて以下の項目を入力し、“次”アイコンをクリックします
   1. リスナー名 - LB_Listener
   2. リスナーで処理するトラフィックのタイプを指定します - HTTP を選択
   3. リスナーでイングレス・トラフィックをモニターするポートを指定します - 80 (デフォルト)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/db5dc113-8e0e-42c7-8344-f9dcc9751863)

### ロギングの管理
1. ロギングの管理の選択画面にて以下の項目を入力し、“送信”アイコンをクリックします
   1. エラー・ログ -「無効」
   2. アクセス・ログ -「無効」（デフォルト）

## ロードバランサーへのhttp通信許可の設定
ここまでの作業で以下のような構成が完成しています。
現状足りていない設定である、ロードバランサーへのhttp(ポート80)通信の許可設定を実施します。
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/7e47b90a-e7c1-4b3e-9a41-1be81bb72f64)

1. コンソールメニューから ネットワーキング → 仮想クラウド・ネットワーク を選択し、handson-vcnをクリックします
2. 左下メニューから セキュリティ・リスト を選択し"LB_Security_List"を選択します
3. イングレス・ルールの追加 ボタンを押し、以下を入力し"イングレスルールの追加"をクリックします
   1. ステートレス - いいえ (デフォルト)
   2. ソースCIDR - 0.0.0.0/0
   3. IPプロトコル - TCP を選択
   4. ソース・ポート範囲 - ALL
   5. 宛先ポート範囲 - 80
   6. 説明 - LBへのhttp通信用

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/30d8b8bf-dc55-4eb6-ba4c-825fd83188ed)


## ロードバランサーの動作の確認
以下の構成の作成が完了したのでロードバランサーを介して通信かつ、負荷分散ができるか動作確認をします
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/7e389613-b6d2-429d-85ad-a0071f780863)

1. 作成したロードバランサーの割り当てされたパブリックIPを確認します
2. WebブラウザにてIPアドレスを入力して ENTER キーを押し、Web Server 1 または Web Server 2 と表示されることを確認します
3. ブラウザ更新を実施し、Web Server 1 または Web Server 2の表示が切り替わることを確認します

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/85f4d2d4-4760-444c-8c5b-1fad51b25227)
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/2436c9d7-dd65-4417-9a0b-90f52f0cb5b8)

ここまでの作業が完了することでロードバランサーを介した負荷分散は完了となります。


