---
title: 検索結果画面への遷移実装
date: 2017-11-07 
tags:
---
検索結果一覧を表示する検索結果画面の作成とリスト表示レイアウトの実装方法を学習します。
また検索画面から検索結果画面を表示するための画面遷移処理の実装方法を学習します。

<!-- toc -->

[ボタンイベントの実装](/AndroidCourse/android/05-ButtonAction)からの引き続きの学習ページです。
# 当ページでの学習ポイント
* ListViewを利用したリストレイアウトの実装方法
* 新規画面の作り方
* 画面遷移処理の実装方法
* xmlファイルでのレイアウト作成のおさらい

蔵書検索を行なった結果複数の蔵書情報を取得できる場合があります、複数取得した蔵書情報を一覧表示します。
その前段として、一覧表示を行うためのウィジェットListViewの使い方を学習します。
また、検索画面で入力された文字の検索を行うために画面遷移処理及び、画面遷移時に次の画面へ情報を渡す処理の実装方法を学習します。

# 新しい画面を作成する
Androidアプリに置いて**画面 = Activity**であることは[Androidの概念](/AndroidCourse/android/02-AndroicConcept)で解説しました。
今回は蔵書検索アプリのプロジェクトに新しいActivityを追加していきます。

新しいActivityの追加はメニューかもしくは左のプロジェクトツリーが表示されているウィンドウで右クリック(macでは２本指でクリック)から新しいActivityの追加を行えます。

新しく作成するActivityは**Empty Activity**を選びます。
1. メニュー > File > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_1.png 500 CreateNewActivity %}
2. ウィンドウ右クリック > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_2.png 500 CreateNewActivity %}

新しく作成するActivityのファイル名と同時に作成するxmlファイルの名称の入力を求められます。
以下の名称で新しいActivityを作成します。

|項目|設定値|
|-------------|------------------|
|Activity Name|ResultListActivity|
|Generate Layout File|チェックを<font color="blue">つける</font>|
|Layout Name|activity_result_list|
|Luncher Activity|チェックを<font color="red">つけない</font>|
|Backwords Compatibility(AppCompat)|チェックを<font color="blue">つける</font>|
{% img /android/06-TransitionScreen/createactivity02.png 500 CreateNewActivity %}
項目の入力が終わったら`Finish`ボタンを押下します。

新しいActivityの作成が終わるとウィンドウは閉じAndroid Studioのエディタに戻ります。
プロジェクトツリーには<font color="green">ResultListActivity.java</font>と<font color="green">activity_result_list.xml</font>のファイルが追加されます。
{% img /android/06-TransitionScreen/createactivity03.png 350 CreateNewActivity %}

## AndroidManifestファイルの確認
Activityがプロジェクトに追加された場合でも、アプリで表示するためにはアプリの設計書となる`AndroidManifest.xml`に利用することを記述する必要があります。
`AndroidManifest.xml`の場所は以下にあります。
> app -> manifests -> AndroidManifest.xml

{% img /android/06-TransitionScreen/createactivity04.png 400 CreateNewActivity %}
作成した**ResultListActivity**が記載されているか確認すると
{% img /android/06-TransitionScreen/createactivity05.png 550 CreateNewActivity %}
無事に記述されていることが確認できました、他の行を確認すると検索画面として表示している**MainActivity**も記載されています。

`AndroidManifest.xml`ではアプリ名やアプリアイコン、アプリ起動時に表示するActivityなどの設定が記述された概要設計書のようなファイルになります、Push通知などの機能を追加する時にもファイルの修正が必要になるので覚えて置いてください。

# 画面遷移処理機能
新しいActivityが作成できたので、MainActivityの*蔵書検索ボタン*をクリックした時、ResultListActivityに表示が切り替わる様にMainActivity.javaを修正していきます。
```java MainActivity.java
...
    View.OnClickListener bookSearchEvent = new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            // コンソールログにボタンが押されたことを出力(表示)
            Log.d("BookSearchBtn", "onClick: BookSearch Button");
            // 入力された文字をToast(トースト)に表示
            Toast.makeText(getBaseContext()
                    , "入力された文字は [" + bookSearchEditor.getText().toString() + "]です。"
                    , Toast.LENGTH_LONG).show();
            //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            // 画面遷移するためのIntentをインスタンス化
            Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
            // 画面遷移アクションを実行
            startActivity(intent);
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        }
    };
...
```
コードの修正が終わったらエミュレータで動作確認してください。
以下のように画面遷移できましたか？
{% img /android/06-TransitionScreen/TransitionScreen.gif 300 TransitionScreen %}
またエミュレータ左下の三角ボタンをクリックすると前の画面に戻ります。
{% img /android/06-TransitionScreen/BackScreen.png 400 BackScreen %}

## Intent
Android開発における４大要素の一つとして紹介されている`Intent`が出てきました。
Intentは別Activityの呼び出し（画面遷移）や別のアプリを起動する時などに利用する要素になります、画面遷移時の利用方法としては
1. Intentクラスを宣言、インスタンス化
2. startActivityメソッドの引数に"1."で宣言したIntentをセット

この２つの手順でActivity間の画面遷移が実装できてしまいます。
今回画面遷移時に生成したIntentですが、
一つ目の引数にはContextと呼ばれるクラスをセット<font color="red">(Contextは少々ややこしいクラスなので今後のページで詳しく解説します）</font>Contextは現在開いているActivityが該当するので自身を表す”this”と言う属性を記述してセットする。
２つ目の引数に遷移したいActivityのクラス情報をセットする。

最初のうちは画面遷移を行う場合、上記のルールで画面遷移のコードを実装するよう癖をつけておきましょう。

# 検索結果画面レイアウト作成
検索画面同様にLayoutEditorとConstraintsLayoutを使って検索結果画面のレイアウト構成を作成していきます。

