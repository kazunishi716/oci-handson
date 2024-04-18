# OCIハンズオン-監視編
- ハンズオン-作業イメージ&前提
- モニタリング情報の確認
- 通知先の作成
- アラームの作成
- 動作確認

## ハンズオン-作業イメージ&前提
- 作業用コンパートメント及び、VCN、Computeが作成されていること。

このハンズオンではCPU使用率が閾値を超えた際に担当者へメール通知するケースを想定しています
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/3f0b5807-3563-4f5a-abdc-9fb0107990ec)

## モニタリング情報の確認
事前に対象コンパートメントにて参照できるメトリックを確認します。
1. コンピュート → インスタンス → インスタンス名を をクリックし、インスタンス詳細へアクセスします。
2. 画面下部にてメトリック詳細を確認することができます
  1. CPU使用率
  2. メモリ使用率
  3. ディスク読取り I/
  4. ディスク書取り I/O
  5. ディスク読取り バイト
  6. ディスク書取り バイト
  7. ネットワーク受信 バイト
  8. ネットワーク送信 バイト
  9. ロード平均
  10. メモリ割り当てのストール
   
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/37915f26-e2f4-4472-93a8-fe468f8ad372)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/e3d6eced-2e1a-4fe1-8f75-7cfcfe35d3ac)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/f09afec8-1886-44f8-abdb-a5160c0134c0)

## 通知先の作成
1. メニューより「開発者サービス > 通知」を選択します
2. トピックの作成をクリックし、以下内容を入力します
  1. 名前 - handson-topic
  2. 説明 - 空白 


![image](https://github.com/kazunishi716/oci-handson/assets/153155301/fb849606-891d-4dfb-839d-0555bc0b2af7)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/18da9080-4b30-4635-a619-408edd85a16e)

3. 作成したトピック(=handson-topic)をクリックし、詳細画面にアクセスします
4. サブスクリプションの作成をクリックし、以下内容を入力します
   1. プロトコル - 電子メール
   2. 電子メール - <自身のメールアドレスを入力>

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/0e0d40ce-68c1-42bd-bd0c-c659522361d9)

5. 登録したメール・アドレス宛てに検証メールが送付されるので、Confirm subscriptionをクリックします

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/b2f3155f-3270-4ffe-9ea7-816a86c160c5)

6. サブスクリプションのステータスが"Pending"→"Active"になっていることを確認します

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/cc9618e0-da53-41c4-8ec0-d3dc754125b9)

## アラームの作成
1. メニューより「監視及び管理 > アラーム定義」を選択します

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/61d3c5c9-6eff-41b8-86e4-95016d88f78c)

2. アラームの作成をクリックし、以下内容を入力します
   1. アラーム定義
      1. アラーム名 - handson-alarm
   2. メトリックの説明
      1. コンパートメント - handson-compartment
      2. ネームスペース - oci_computeagent
      3. リソース・グループ・オプション - 空白
      4. メトリック名 - CpuUtilization
      5. 間隔 - 1分
      6. 統計 - Mean
  3. メトリック・ディメンション
     1. ディメンション名 - resourcDisplayName
     2. ディメンション値 - 事前に作成したインスタンスを指定
  4. トリガー・ルール
     1. 演算子 - 次より大きい
     2. 値 - 20
     3. トリガー遅延分数 - 1分
     4. アラームの重大度 - クリティカル
     5. アラーム本体オプション - ハンズオン：CPUアラート
  5. アラーム通知の定義
     1. 宛先サービス - 通知
     2. コンパートメント - handson
     3. トピック - handson-topic
  6. メッセージのグループ化 -  メトリック・ストリーム全体で通知をグループ化
  7. メッセージの書式 - フォーマットされたメッセージの送信

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/31f95004-c293-4c46-8607-d0f7604e327e)
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/99b61a3c-a97a-41c8-8ead-d32c9a4fe955)
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/09054fca-a508-4ec0-9636-bf7798bcd570)

## 動作確認
対象インスタンスでCPU使用率20%を1分間超過すると指定したEメールアドレスに通知が来る設定が完了したので、検証のためにインスタンスのCPU使用率を高めます。

> [!WARNING]
> 下記コマンドはサーバスペックによっては瞬時にCPU使用率100%に到達します。検証時はシステム影響など十分に留意の上、実行してください

1. 該当サーバにssh接続し、以下コマンドを3回ほど実行します
```
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
```

2. しばらくすると指定したメールアドレス宛に閾値を超過した旨のメールが届きます。
※メールタイトルが"OK_TO_FIRING"になっています

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/116b2871-f773-4406-906e-343a062d5b5d)
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/b19df186-baeb-40c4-aba6-38e72102a15f)

4. 停止させる場合は下記コマンドにて番号を確認します
```
jobs
```

4.下記コマンドでプロセスキルして停止します（番号はjobsコマンドで確認したものを指定してください）
```
kill %1 %2 %3
```

実際にOCIのコンソール画面を確認するとCPU使用率が向上していることが確認できます

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/ec271add-0d40-48f1-87c1-097a48a8a700)

なお、閾値より下回ると再度メールにて、問題が解除されたことを通知してくれます。
※メールタイトルが"FIRING_TO_OK"になっています
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/7d6bdd70-557d-4a69-b228-6425a9d3e2d7)


今回はCPU使用率についてのハンズオンを実施しました。
メモリ使用率や通知をメールではなく、Slackなどとも連携できるので運用や自環境によって随時カスタマイズください。

## 参考 モニタリング ダッシュボードについて
OCIのインスタンスのCPU使用率やメモリ使用率、OCIコストなどをダッシュボード化できる機能について紹介します

1. OCI Webコンソールの"ダッシュボード"ボタンをクリックします
![image](https://github.com/kazunishi716/oci-handson/assets/153155301/f03fc1dc-7bca-4b6a-bb77-e3c662d870ae)


2. 新規ダッシュボードをクリックし、以下内容を入力します
   1. 最初から作成
   2. ダッシュボード名 - custom
   3. 説明 - 空白
   4. リソース・コンパートメント - 任意
   5. 新規ダッシュボードグループを作成
   6. ダッシュボード・グループ名 - custom-group

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/2c446b8e-e0c4-48b3-b73f-4bffbad95098)

3. ウィジェットの追加
   1. 各種必要な項目を任意で追加します

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/ef836cc4-2e3f-4902-b66e-575514f88503)

  2. 今回は以下内容で簡単にダッシュボードを作成してみました
     1. インフラストラクチャ請求 (テナント全体のコスト請求額が確認可能)
     2. インフラストラクチャのコスト分析 (コンパートメント単位やリソース単位で任意の日/週/月でグラフ化する機能)
     3. モニタリング(コンピュート等のメトリック情報を参照する機能)
     4. リソース・エクスプローラー(対象コンパートメントの各リソース数をまとめて数値化する機能)

![image](https://github.com/kazunishi716/oci-handson/assets/153155301/446deaba-aef7-400b-8c2a-158fe6c4cc05)

ダッシュボード自体は複数作成可能で、切り替えも可能なので各種コスト分析用のダッシュボードやリソースのメトリックをまとめて確認するためのダッシュボードなど複数を使い分けることで運用工数を削減できます。
各社/各自の運用に沿ったダッシュボードを作成ください。
