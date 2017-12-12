---
title: Android アプリ開発を始める
date: 2017-11-04
---
講座で解説のために作成するアプリの紹介とプロジェクトの作成

<!-- toc -->

# Android Studio セットアップ
Android Studioの初回起動時には利用環境のセットアップ処理が行われるので手順を記載します。

1.Windows ホーム画面の左下にある```検索欄```に```Android```と入力し**Android Studio**をクリックし起動します。
<img src="sad01.png" alt="alt" title="SetUp" width="600">

2.android SDK Componentのインストールが開始されます
<img src="sad02.png" alt="alt" title="SetUp" width="450">

3.Welcome画面では```Next```ボタンをクリックします
<img src="sad03.png" alt="alt" title="SetUp" width="450">

4.Install Typeは**stanard**を選択し```Next```をクリックします
<img src="sad04.png" alt="alt" title="SetUp" width="450">

5.Android Studio内のテーマカラーを選択します、好みで選択し、```Next```をクリックします
<img src="sad05.png" alt="alt" title="SetUp" width="450">

6.セットアップ情報の確認画面が表示されるので```Finish```をクリックします
<img src="sad06.png" alt="alt" title="SetUp" width="450">

7.Androidアプリの開発に必要なComponentのダウンロードは始まります
<img src="sad07.png" alt="alt" title="SetUp" width="450">

8.ダウンロードが終わったら```Finish```をクリックします
<img src="sad08.png" alt="alt" title="SetUp" width="450">

9.Android Studioの起動画面が表示されたら完了です
<img src="sad09.png" alt="alt" title="SetUp" width="450">

# 新しいプロジェクトを作る

1.```Start a new Android Studio project```をクリック
<img src="sad10.png" alt="alt" title="SetUp" width="450">

2.新規プロジェクト作成時にはアプリ名などを入力します
以下の表の通りに項目を設定して```Next```をクリックします。

|項目名                 |説明                |入力内容        |
|:---------------------|:-----------------:|:------------:|
|Application name      |アプリ名             |BookDiscovery |
|Company domain        |全世界の独自ドメイン   |example.com   |
|Project location      |プロジェクトの保存先   |任意設定        |
|include C++ support   |C++ライブラリ取込み   |チェックを外す   |
|include Kotlin support|Kotlinライブラリ取込み|チェックを外す   |

**Company domain**は講義の説明で作成するアプリはリリースしませんので独自ドメインをお持ちでなければ初期値の
ままで構いません。
今回の講義ではC++での実装はありませんのでチェックを外しておきます。
また、Kotlinは後半で利用方法を紹介しますので一旦チェックを外しておいてください。
<img src="sad11.png" alt="alt" title="SetUp" width="450">

3.今回作成するAndroidアプリでサポート対象とするOSバージョンの下限設定を行います
**Phone and Tablet**のみチェックをつけ、
> API 23: Android 6.0 (Marshmallow)

を選択したら、```Next```をクリックします
<img src="sad12.png" alt="alt" title="SetUp" width="450">

4.**Empty Activity**を選択し```Next```をクリックします
<img src="sad13.png" alt="alt" title="SetUp" width="450">

5.初期画面となるActivity設定を行います
以下の表の通りに項目を設定して```Next```をクリックします

|項目名                             |説明                |入力内容       |
|:---------------------------------|:-----------------:|:------------:|
|Activity Name                     |画面名              |MainActivity  |
|Generate Layout File              |デザインファイル作成有無|チェックを入れる|
|Layout Name                       |デザインファイル名    |activity_main |
|Backwards Compatibility(AppCompat)|下方互換性の有無      |チェックを入れる |
<img src="sad14.png" alt="alt" title="SetUp" width="450">

6.「5.」で設定した内容でAndroidアプリの開発のデフォルト環境を構築が実行されます
インストールが終了し**done**の文字が表示されたら```Finish```をクリックします。
<img src="sad15.png" alt="alt" title="SetUp" width="450">

7.構築が終了すると開発画面に遷移します
<img src="sad16.png" alt="alt" title="SetUp" width="450">

<img src="sad17.png" alt="alt" title="SetUp" width="450">

<img src="sad18.png" alt="alt" title="SetUp" width="450">

<img src="sad19.png" alt="alt" title="SetUp" width="450">

<img src="sad20.png" alt="alt" title="SetUp" width="450">
