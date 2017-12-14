---
title: Android アプリ開発の環境構築
date: 2017-11-01
---
Androidアプリの開発を行うための環境構築をおこないます。

<!-- toc -->

Windows / Mac の各PCごとに手順が違うためそれぞれの環境構築手順を以下に記述します。

# 環境構築(Windows PC)
Android アプリの開発を始めるには最低限２つソフトウェアのインストールと設定を行う必要があります。
これ以降にwindows OS環境でのAndroid アプリ開発をおこなく準備作業を記載します。
* java(JDK)
* Android Studio

## java(JDK)

まずはAndroid アプリの開発を行うためのプログラミング言語javaを実行できるようにする必要があるため、
java(JDK)をインストールします。

### JDKのダウンロード
インストールするにはまず、java(JDK)をダウンロードする必要があるため、以下のリンクからjava(JDK)をダウンロードします。

**java(JDK)のバージョンとして動作保証がされているjava8系のダウンロードを行います**

下記リンクをクリックしJDKダウンロードページにアクセスします。
[JDK8 ダウンロードページ](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
※ 資料作成時の最新版**java 8u152**バージョンのダウンロード手順で記載していきます。

下図の通り```Java SE Development Kit 8u152```の項目から
```Accept License Agreement```にチェックを入れ、```Windows x64```のDownload列のリンクをクリックしてexeファイルをダウンロードします。
<img src="j8_dl.png" alt="alt" title="java se download" width="520">

### JDKのインストール
ダウンロードしたJDKのインストーラを起動してPCにJDKをインストールします。

1. ダウンロードしたexeファイルをダブルクリックしインストーラーを起動します。
<img src="j8_i_01.png" alt="alt" title="java(JDK) install" width="520">
2. 確認ダイアログが表示された場合は、```はい```をクリックします。
<img src="j8_i_02.png" alt="alt" title="java(JDK) install" width="450">
3. JDKのインストール開始画面が表示されたら```次(N) >```をクリックし、その後も画面に従いインストールを進めます。
<img src="j8_i_03.png" alt="alt" title="java(JDK) install" width="450">
4. **開発ツール**が選択されていることを確認し、```次(N) >```をクリックします
<img src="j8_i_04.png" alt="alt" title="java(JDK) install" width="450">
5. JDKの保存先を設定できます、変更は不要なので```次(N) >```をクリックします
<img src="j8_i_06.png" alt="alt" title="java(JDK) install" width="450">
6. インストールが始まります、完了して画面が変わるのを待ちます
<img src="j8_i_07.png" alt="alt" title="java(JDK) install" width="450">
7. インストールの完了画面が表示されたら```閉じる(C)```をクリックします
<img src="j8_i_08.png" alt="alt" title="java(JDK) install complete" width="450">

以上でJDKのインストールが完了です。


---

### JDKのインストール 確認
次にコマンドプロンプトを使いJDKが正しくインストールされたことを確認します。

1. Windows ホーム画面の左下にある```検索欄```に```cmd```と入力し**コマンドプロンプト**をクリックし起動します。
<img src="j8_c_01.png" alt="alt" title="java(JDK) install confirm" width="520">
2. コマンドプロンプトに```java -version```と入力しエンターキーを押します。
<img src="j8_c_02.png" alt="alt" title="java(JDK) install confirm" width="600">

以上でjava(JDK)のインストールの確認ができました

---

## Android Studio
最後にAndroid Studioの環境構築を行います。
Android Studioをインストールするため、exeファイルを下記リンクからダウンロードページにアクセスします。

[Android Studio ダウンロードページ](https://developer.android.com/studio/index.html?hl=ja)
※ 資料作成時の最新版は Android Studio 3.0.1

1. **Android Studio Androidの公式 IDE**下にある```Android Studio 3.0.1 FOR WINDOWS...```のボタンをクリックしexeファイルをダウンロードします。
<img src="as_i_01.png" alt="alt" title="Android Studio download" width="520">
2. ダウンロードの確認画面が表示されたら**利用規約**を確認の上、チェックをクリックし、```ANDROID STUDIO FOR WINDOWS ダウンロード```をクリックします。
<img src="as_i_02.png" alt="alt" title="Android Studio download" width="520">
3. ダウンロードしたexeファイルをダブルクリックし、Android Studioのインストーラーを起動します。
<img src="as_i_03.png" alt="alt" title="Android Studio download" width="520">
4. 確認ダイアログが表示された場合は```はい```をクリックします
<img src="as_i_04.png" alt="alt" title="Android Studio install" width="450">
5. インストーラの起動画面が表示されたら```Next >```をクリックします
<img src="as_i_05.png" alt="alt" title="Android Studio install" width="450">
6. インストールするコンポーネント設定が表示されたら"Android Virtual Device"がチェックされていることを確認し、```Next >```をクリックします
<img src="as_i_06.png" alt="alt" title="Android Studio install" width="450">
7. Android Studioの保存先を設定します、変更の必要がなければ```Next >```をクリックりします
<img src="as_i_07.png" alt="alt" title="Android Studio install" width="450">
8. ```Install```をクリックします
<img src="as_i_08.png" alt="alt" title="Android Studio install" width="450">
9. インストールの完了を待ちます
<img src="as_i_09.png" alt="alt" title="Android Studio install" width="450">
10. インストールが終わったら```Next >```をクリックします
<img src="as_i_10.png" alt="alt" title="Android Studio install" width="450">
11. ```Finish```をクリックすると、Android Studioが起動します。
引き続き、Android Studioの初回セットアップを行います
<img src="as_i_11.png" alt="alt" title="Android Studio install" width="450">
12. Android Studioが起動しAndroid SDK Componentのインストールが開始されます
<img src="as_s_01.png" alt="alt" title="Android Studio setup" width="450">
13. Welcom画面が表示されたら```Next```をクリック
<img src="as_s_02.png" alt="alt" title="Android Studio setup" width="450">
14. Install Type画面では**Standard**が選択されていることを確認し```Next```をクリックします
<img src="as_s_03.png" alt="alt" title="Android Studio setup" width="450">
15. Android Studioのテーマ、色合いを好みで選択し、```Next```をクリックします
<img src="as_s_04.png" alt="alt" title="Android Studio setup" width="450">
16. 確認画面が表示されたら```Finish```をクリックします
<img src="as_s_05.png" alt="alt" title="Android Studio setup" width="450">
17. Androidアプリの開発に必要なComponentのダウンロードを待ちます
<img src="as_s_06.png" alt="alt" title="Android Studio setup" width="450">
18. ダウンロードが終了したら、```Finish```をクリックします
<img src="as_s_07.png" alt="alt" title="Android Studio setup" width="450">
19. Android Studioの起動画面が表示されたら完了です
<img src="as_s_08.png" alt="alt" title="Android Studio setup complete" width="450">

以上でAndroid Studioのインストール、セットアップ作業は完了です。
Androidアプリの開発環境が整いましたので早速開発に入っていきましょう！

# 環境構築(Mac PC)

## java(JDK)のインストール

## Android Studioのインストール

